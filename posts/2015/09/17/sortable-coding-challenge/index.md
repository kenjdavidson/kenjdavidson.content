---
type: Blog
title: Sortable Coding Challenge
summary:
  A long time ago I was looking to get out of support and into a development role.  At this point I had education
  and some extra curricular development on my side - turned out to be quite the learning experience.
categories: [Blog]
tags: [Java, Employment]
---

When I was attempting to make the switch from application support/implementation to a completely development based role - which was shockingly harder than I had planned - I started coming across a number of coding challenge requirements on company sites. I was directed to the [Sortable Challenge](https://sortable.com/coding-challenge/) by my cousin who was trying to convince me that Cambridge/Kitchener was the place to be (regardless of the crazy commute). The only language I really had any experience with at the time was Java, which was good since that was one of the languages they were looking for on the original posting.

## Challenge Details

Looking at the current (and original, since it hasn't changed much):

> The goal is to match product listings from a 3rd party retailer, e.g. “Nikon D90 12.3MP Digital SLR Camera (Body Only)” against a set of known products, e.g. “Nikon D90”. <br /><br />We’ll provide you with a set of products and a set of price listings matching some of those products. The task is to match each listing to the correct product. Precision is critical. We much prefer missed matches (lower recall) over incorrect matches, so try hard to avoid false positives. A single price listing may match at most one product.

- Attempt to match the `manufacturer` to the `manufacturer`
- Attempt to match the `product_name` and `manufacturer` from the **Products** list and regex that with the `title` from the **Listing** list.

which resulted in the following:

> Your solution matched 15745 listings correctly, 1143 listings incorrectly and didn't make a match for 2739 other possible listings. Compared to other entrants, this puts your solution at #130, which is around the 30th percentile.

Not overly promising to say the least.

With regards to the `15745` and `1142` I could have been good, but the fact that `2739` were missed completely baffled me. After requesting a little idea of where I went wrong, and what could have been done to improve the algorithm, the response was pretty vague:

> We're not looking for a specific matching scheme. High level we're looking for aggressive matching with minimal false positives, and a reasonably efficient implementation.

In thinking about how it could have been made more aggressive, the only things I could think of were:

- Instead of a strict `manufacturer` to `manufacturer`, it could have used a soundex for some different spellings?
- Use the `announced-date` inconjunction with the `currency` to match timezones with descriptions?
- My regex skills needing to be seriously improved?

## Round Two

I may take another stab at this later, but for now I'll just suck up my failed attempt and move on.
