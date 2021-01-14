# kenjdavidson.content

Article, Timeline and Page content for kenjdavidson.github.com.  Although Github and Gatsby are glorious, who knows what might come along; it makes sense to have things setup so that I can play around with different frameworks while using current content (without the risk or annoyance of changing the presentation project.

## Content

I read an article a while ago, that described how to separate your content from presentation when using **Gatsby**.

### Articles

writing/yyyy-mm-dd__article-slug/{index.mdx?}

Also including:

- images

### Timeline

timeline/yyyy-mm__institution-name_position-degree/{index.mdx?}

Also including:

- images
- downloads

### Page Content

page/{page-name}/{section-name.mdx?}

Also including:

- images

## Templating

Unsure at this point how crazy it will get, but effectively:

### Articles

Probably only one template

### Timeline

Most likely two types:

- Experience
- Education

Although could be the same

### Page Sections

Page sections will have the most options:

- Hero section with fluid image behind text
- Text section with images and text inline as normal Mdx
- Split section with image/text on right/left

## API(ish)

Content will be designed in `mdx` which taking advantage of `MDXProvider` makes it possible to use JSX.  It's important that the components used are as general as possible, so that they can be transferred and designed across different frameworks.   At the time of development `Grommet` was used, but had it's components wrapped and provided to `MDXProvider`, things like:

- Text content
- Icons
- Images

Need to all be managed correctly and available.  This may mean that we're more tied to Gatsby than any other framework, but hopefully not.
