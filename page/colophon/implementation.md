---
type: Section
title: Implementation
summary: Language, framework and deployment used for the website
page: Technical Details
order: 1
---

As mentioned before, this site is (and always will be) a work in progress.  Things will be broken, missing, probably doing some funky things; for which I apologize.

The current implementation of the site is based upon:

- [Gatsby](https://www.gatsbyjs.com/) and [React](https://reactjs.org/) cause they are awesome and as new to them as I am, I'm hooked.
- Hosted on [Gitub Pages](https://pages.github.com/) as all great seem to be.
- Styled using [Grommet](https://v2.grommet.io/), a React based library that (doesn't mesh 100% with Gatsby) provides some unique and useful functionality.  

### Grommet

I'll be completely honest - it looked cool and I do really like the library.  But after getting a little more of it under my belt, I think that it was a bit much for my personal site.  There are a few things like:

- Theming
- Responsiveness (a bigger one)
- Anchors - which is a massive annoyance.  They don't work well with Gatsby, and require some magic wrapping to convert to `<Link/>` or injected with `navigate(href)` based on whether it's an internal link or not.  Without this, Gatsby becomes un-Gatsby and super annoying.
- and some other things

That I've needed to work around (most likely in not the best way) in order to get things how I prefer them.

When I get a chance, I see myself moving away form Grommet, back to `theme-ui` and regular `styled-components` (or maybe `emotion`).  We'll see, but at this point the positives don't outweight the negatives.

### Antd

With the pain points I've mentioned, I have been researching other frameworks and I will probably try convert [Grommet](https://v2.grommet.io/) to [Antd](https://ant.design/).  It looks like it's got a much richer set of components - which more importantly contain the functionalty that I'm missing and don't particularly want to write myself into Grommet.

The page template definitely isn't that difficult to replicate and I've gotten a working system of:

#### `Posts`

For writing/blog entries.

#### `Timelines`

For resume and education.

#### `Pages` / `PageSections`

Providing a method of describing and building pages externally from the `/src/` which I'm hoping to eventually just use as gatsby plugins.  


