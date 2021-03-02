---
type: Post
category: Technology
title: "Simulating TOP N / RANK() with Jasper Reports"
summary: Ran into an issue where I needed to simulate TOP N / RANK() with Jasper Reports
tags: [Jasper Reports, Java, Scriptlets]
---

While attempting to get some reports working I ran into a problem where:

- Our database (version) doesn't support the `RANK()` function
- Jasper Server (a normally wonderful reporting library) wasn't correctly `summarizing` on `ReportGroup`

## The Issue

The report being worked on had a requirement that the **Top N** needed to be displayed, as with almost all cases the Top N needed to include ties.  So for example if we were looking up the **Top 3** from the following data:

| Wins | Seconds | Thirds |
| --- | --- | --- |
| **5** | 2 | 2 |
| **4** | 1 | 1 |
| **3** | 1 | 4 |
| **3** | 2 | 2 |
| **1** | 4 | 4 |

the top ranking values are `5, 4, 3, 3` so the report must show 4 people in this instance, not 3.  Generally you'd be able to:

- `RANK()` over `wins` in the query, although it may be that your database doesn't support `RANK()`
- Add a `TopNCountTotal` variable that only gets incremented when the group changes (which sadly doesn't work in Jasper properly)
- Filter over `topNumber` and `record.getWins()` in code before passing into the Jasper Report as a `JRDatasourceBean`, which in this case loses our ability to have the report available in multiple places.

my final solution was to create the following [Scriptlet](http://jasperreports.sourceforge.net/sample.reference/scriptlet/)

> There are situations when a calculation is required during the report filling stage that is not included in JasperReports provided calculations.
Examples of this may be complex String manipulations, building of Maps or Lists of objects in memory or manipulations of dates using 3rd party Java APIs.
Luckily JasperReports provides us with a simple and powerful means of doing this with Scriptlets.

...luckily

### Scriptlet

The scriptlet is very primitive, compared to other's I've needed, it really only needs to summarize the previous group count into the running total.  Using our example above, we'd like things to be incremented as such:

| Wins | Seconds | Thirds | Group Count | Total Count |
| --- | --- | --- | --- | --- |
| **5** | 2 | 2 | 1 | 0 |
| **4** | 1 | 1 | 1 | 1 |
| **3** | 1 | 4 | 1 | 2 |
| **3** | 2 | 2 | 2 | 2 |
| **1** | 4 | 4 | 1 | 3 |

we can see how the `Group Count` restarts at each group change and how the `Total Count` is summarized (is always a group behind the current row).  So in this case we know that we only want to display data with `$V{TopNCountTotal} < $P{topNumber}`.  Sadly we need to get all the data back in order to do this, but for the time being the data on which we're working isn't that big.

```java filename=GroupTopNCountScriptlet.java
public class GroupTopNCounterScriptlet extends JRDefaultScriptlet {
	public static final String GROUP_COUNT_VAR = "GroupTopN_COUNT";
	public static final String TOP_N_TOTAL = "TopNCountTotal";
	
	int topNTotalCounter = 0;
	
	@Override
	public void beforeGroupInit(String groupName) throws JRScriptletException {
		Integer groupCount = (Integer) getVariableValue(GROUP_COUNT_VAR);
		topNTotalCounter += (groupCount == null) ? 0 : groupCount;
		setVariableValue(TOP_N_TOTAL, topNTotalCounter);
	}
}
```

### Jasper Report 

Once the Scriptlet is made available and set on the Jasper Report, the only things remaining are to:

- Ensure that your query uses the appropriate sort field.  One thing I didn't mention here is that my sort field changes based on parameters, which is probably why you've asked 

> Why didn't he just count the distinct values as he went through the rows.

the main problem is that you can't dynamic a dynamic variable in Jasper.

- Configure your main `Detail` section to only display when appropriate:

```
$V{TopNCountTotal} < $P{topNumber}
```

- Keep a running total of your group counts only while the `TopNCountTotal` is lower than your `topNumber`.  This way you can display the total in the summary based on how many are actually displayed, and not in the full list.
