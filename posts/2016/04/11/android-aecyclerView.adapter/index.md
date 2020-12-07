---
type: Blog
title: Android RecyclerView. Adapter implements Filterable
summary: Walk through of how I added filter functionality to my Wingapedia app using RecyclerView.
categories: [Blog]
tags: [Java, Android]
---

While working with the `RecyclerView.Adapter` while building my Wingapedia Android app I needed to write a custom filter. Essentially I had two groupings of flavours and temperatures (I have to admit, now that I look back I definitely did some things wrong when designing this, but that's for another post) which I wanted to use to filter the items in the list.

```java
public class Flavour {
  protected String name; 
  protected String description; 
  protected boolean isBbq; 
  protected boolean isGarlic; 

  // Setters and getters oh my
}
```

On the RecyclerView fragment we want to be able to filter on the name or description (entered in an EditText) as well as any of the isBbq or isGarlic flag within the class. If we were just to create a defualt custom filter it would look like this:

```java
public class FlavourFilter extends Filter {
  @Override
  protected FilterResults performFiltering(CharSequence constraint) {
    for (Flavour f : mItems) {
      if (f.getName().contains(constraint) || f.getDescription().contains(constraint) {
        mFilteredList.add(f);
      }
    }
  }
 
  @Override
  protected void publishResults(CharSequence constraint, FilterResults results) {
    // Use mFilteredList to update mItems
  }
}
```

This is all well and good for Filters which only search by name and description, but what if we need to pass in and filter on more options within the item?  We need to either pass in:

- The full object to perform testing based on all (or some) fields.
- A byte array used to determine whether flags are on or off.
- An object consisting of only the items we need to filter on.

In this particular case I (update: sadly) decided that it would be a good idea to pass a JSON object from my `FilterFlavourView` (responsible for selecting different temperatures, toppings, spices, etc). The JSON object contains the items that I'm interested in:

- `searchString` which is a text based lookup.
- `temperatureFlag` which is the byte flags of temperatures selected.
- `tagList` which is a list of flavour tags.

All of which are combined to filter the appropriate list.  The `performFiltering` method gets replaced with:

```java
@Override
protected FilterResults performFiltering(CharSequence constraint) {
  FilterResults results = new FilterResults();

  // Set the constraint to a valid value
  filterString = (constraint != null) ? constraint.toString() : "";

  // There was no Cache set to this adapter so we will return nothing
  // or there was no constraint set so we will return nothing.
  if (constraint == null || constraint.length() == 0) {
    List filtered = new ArrayList(mFullList);
    results.values = filtered;
    results.count = filtered.size();
    return results;
  }

  try {
    JSONObject filter = new JSONObject(constraint.toString());
    mSearchString = filter.getString("Search");
    mTemperatureFlag = filter.getInt("Temperatures");
    mTagList = filter.getString("Tags");
  } catch (Exception e) {
    // Attempting to map the JSONObject failed, most likely
    // constraint is just a String
    mSearchString = constraint.toString();
    mTemperatureFlag = 0;
    mTagList = "";
  }

  mTags = parseTags();

  List<Flavour> filtered = new ArrayList<>();
  boolean foundTag;
  for (Object obj : mFullList) {
    Flavour flavour = (Flavour) obj;
    //Log.v("Filtering", "Flavour " + flavour.toString());

    // If Tags were provided in the Search request, check to see if the
    // Tag is associated to the wing Flavour
    foundTag = false;
    if (mTags != null) {
      for (int i = 0; i < mTags.length; i++) {
        if (flavour.getTags().contains(mTags[i])) {
          foundTag = true;
          break;
        }
      }
    } else {
      foundTag = true;
    }

    if ( (mSearchString.length() == 0
              || flavour.getName().toLowerCase().contains(mSearchString.toLowerCase()))
          && (mTemperatureFlag == Temperature.ALL_TEMPS
              || (flavour.getTemperature().getFilterFlag() & mTemperatureFlag) != 0)
          && foundTag ){

      filtered.add(flavour);
    }
  }

  results.values = filtered;
  results.count = filtered.size();

  return results;
}
```

The filtered list is now combined with the three lookups that were required: search text, temperature and flavour.

#### P.S ~ May 2017

With a combination of All Star Wings going out of business (at least near me) and Google Play requiring a privacy policy (which I didn't have time nor interest in writing) **Wingapedia** has sadly been removed from the store.  

#### P.P.S ~ Jan 2018 

I've had some interest come up in bringing this app back online, although making it customizable for the food industry.  I'll probably look into bringing this back up using React Native or Flutter.