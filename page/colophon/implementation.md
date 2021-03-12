---
type: PageSection
title: Implementation
summary: Language, framework and deployment used for the website
order: 1
---

### Framework(s)

As mentioned before, this site is (and always will be) a work in progress. Things will be broken, missing, probably doing some funky things; for which I apologize.

The current implementation of the site is:

- Developed with [Gatsby](https://www.gatsbyjs.com/) and [React](https://reactjs.org/). I was a hard sell up front, but I'm 100% on board now.
- Hosted on [Gitub Pages](https://pages.github.com/) as all great seem to be.
- Styled using [Styled Components](https://styled-components.com/) - I have flip flopped from using Styled Components, to Grommet, to Antd and now back. I should never have changed to begin with.

### Design & planning

I've been attempting to categorize the types of `Mdx` created based on the highest level categories:

#### Articles

I decided to go with single serve `Mdx` files for publishing. I had originally started with `Md` but figured just having the ability to add components into the posts would be solid.

I'm slowly starting to customize the `components` to provide a little more functionality, I found that event though there are a solid number of plugsin available, they just didn't provide the control I wanted.

#### Timelines

I am in the process of converting my resume/experience timeline away from `Mdx` files (one per entry) into a `yaml` data file. The goal here is to add more than just work experience, while providing a better and more easily customized layout.

There are a number of wonderful timeline libs out there, why re-invent the wheel?

#### Pages/Sections

Pages are where things get hectic, I love the idea of continuing with `Mdx` for the pages, but this limits things somewhat. `JSX` in markdown is pretty solid when the Components are self serving - but when using container components to wrap content, the content is no longer in markdown.

The requirements to have:

- Each page customizable
- Each pages sections customizable

The `Mdx` plugin provides a nice way of wrapping each `Mdx` file:

- By using the configuration for templates
- By exporting a default component type

I feel like this just gives a little more option.

### Bitmoji

Yup, [Bitmojis](https://www.bitmoji.com/) are one of my online [guilty pleasures](https://en.wikipedia.org/wiki/Guilty_pleasure). I'm only somewhat ashamed to admit it!

Don't judge!

### Attribution

The following glorous people/group(s) need to be acknowledge:

#### Colours

I've been playing around with a few of the [top 50 schemes](https://visme.co/blog/website-color-schemes/) published here. Some of them don't work well without the higher end graphics/imagery for which they were designed; but they're fun to play around with.

- **Cool and Fresh** (4)
- **Striking and Simple** (12)
- **Minimal Yet Warm** (19)

> They don't provide links - so the numerical position in the list is the best I can provide.

#### Fonts

Fonts are provided by:

- **Merriweather** by [sorkintype.com](https://sorkintype.com)
- **Playfair Display** by [Claus Eggers Sørensen](github.com/clauseggers/Playfair-Display)
- **League Script** by [Haley Fiege](https://www.theleagueofmoveabletype.com/)

#### Icons

Icons are provided by:

- Hand Symbol Icons made by [Vitaly Gorbachev](https://www.flaticon.com/authors/vitaly-gorbachev) from [Flat Icon](https://www.flaticon.com/).
- Social Icons made by [Pixel Perfect](https://www.flaticon.com/authors/pixel-perfect) from [Flat Icon](https://www.flaticon.com/).
