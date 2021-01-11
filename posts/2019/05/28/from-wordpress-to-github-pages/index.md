---
type: Post
title: Wordpress to Github Pages
summary:
  To save a little bit of cash on GoDaddy hosting, as well as get some experience with Github Pages, I took the time move
  (and document the process) of that conversion.
category: Technology
tags: [Github Pages, Jekyll]
---

I started working with Wordpress after losing my job a few years ago, I wanted to get into a development role and I wasn't
sure which direction I wanted to go in: UX, server side, full stack, nor which languages I wanted to start working with
on a full time basis. At this point I'd done a number of things with Java and C++ but found that PHP was on a number
of job descriptions and figured I'd give it a go. Wordpress felt like a good fit as it would allow me to get my feet wet
with the language and an establish framework (although in hindsight I could have gone with something a little more robust).

I'll be honest, PHP isn't quite my favorite language, and I found a couple other roles using Java on the back end, which
allowed me to start paying a little more attention to other frameworks like AngularJS and React/React Native. I also wanted
to cut out pay GoDaddy for hosting, that I wasn't really using, nothing like shelling out money for a site that hasn't
been updated in half a decade.

## Startup

Whenever I work with a new technology I generally try to read as much as possible before actually starting, things like
framework/language specifics, project best practices, gotchas, etc. are where I tend to start (as I think everyone
should). With this project specifically, I figured the most important things were:

- Getting a handle on Ruby, since I haven't done any real development with Ruby it was going to be the biggest issue. Lucky
  for me the Ruby part of Jekyll wasn't really that important, as getting off the ground with a basic implementation seemed
  to be straight forward and require little to no actual Ruby development (check)
- Walking through the Jekyll documentation. This was pretty straight forward, the docs were pretty decent and there were no
  real missing pieces of information. (check)
- Next was figuring out the theming/layout of a project. I have to say I didn't like any of the Github standard themes, so
  figuring out the `remote-theme` was the next task. With just a basic project with an `index.md` I set out at finding a simple,
  yet pleasing, theme. It took a while to find one that actually worked, most of them required that I fork the theme project
  and implement my site within it (not cool). I finally found [Lagom](https://github.com/swanson/lagom) by
  [Swanson](https://github.com/swanson) which I found appealing, simple and gave me the opportunity to customize to my liking. (check)

## Create Project

First thing first, using the [Github Pages](https://pages.github.com/) documentation a new project was required, the name has to match `username.github.io`, easy enough - create a new repository `kenjdavidson.github.com`.

## Remote Theme Setup

Next I had to get the theme setup, again there's a ton of documentation on this around the net, the most difficult part was picking a
theme that I could stand. I'm not too flashy, and love simple things (personally) when I came across Lagom it was the right choice.
First thing was to link the Lagom theme to my new project, following the docs I was able to update my `_config.yml` to look like this:

```markdown
author: Ken Davidson
email: ken.j.davidson@live.ca
title: Welcome to kenjdavidson.github.io - relocating from Wordpress
description:

url: http://kenjdavidson.com

remote_theme: swanson/lagom
```

After committing and waiting a bit for the build, my single (well now double due to \_config.yml) file site is looking good and
Lagom'ed. As mentioned before, a number of the available themes didn't actually work, in most cases they were missing some
configuration for CSS and therefore the styles weren't being picked up correctly. I skipped those ones as none of them were so
high on my list that it was worth working it out.

(So I can't help ya there)

## Overriding Lagom

Even though I loved Lagom, there were some changes that needed to be made. It made sense to fork the project and start working on
it on my own, not too soon after Mr. Swanson archived the project. After forking just a couple updates to `_config.yml`:

```
author: Ken Davidson
email: ken.j.davidson@live.ca
title: Welcome to kenjdavidson.github.io - relocating from Wordpress
description:

url: http://kenjdavidson.com

remote_theme: kenjdavidson/lagom
```

Throw down a little `bundle exec jekyll serve` (there were a number of other issues I ran into with Jekyll and Ruby) which took a while
and I should probably document later.

As mentioned, the theme is pretty straight forward, but there were a number of changes that needed to be made (for me) in order
to make it more dynamic (just incase I decided to modify or change it in the future, I wanted to take advantage of Jekyll's
configuration and variables rather than hard code/overwrite full files.

The theme itself provides a `_data/theme.yml` which provides the ability to configure theme specific details: name, logo,
social links, etc. I didn't mind these so much, but wanted to improve/expand in order to populate more of the hardcoded
values:

- The logo pulled from /logo.png (default) or gravatar if provided. Since I don't have either, and wanted to customize I added
  another option (default) to look at `site.data.theme.logo` which can be provided a url (https://github.com/kenjdavidson.png in my case)
  to pull in any image. Note - `assets/css/all.css` also needed updating to set the size and border of the image, but I'll get into
  the CSS in the SCSS conversion section.
- Other screen elements that needed to be changed off the bat were site description and sidebar information. I'm still debating
  leaving these in the `theme.yml`, or maybe having a fall back to the `_config.yml`. For example, for the user name, it can be done
  with `site.data.theme.name` but should probably have a fallback to `site.author` since it seems likely that other themes would use the
  same.

## SCSS Conversion

Lagom uses [getskeleton.com](http://getskeleton.com) as it's CSS framework, which isn't based on SASS on it's own, but there were a number of SASS ports - after looking around I found that [https://github.com/atomicpages/skeleton-sass](https://github.com/atomicpages/skeleton-sass) was the most recently updated and seemed to be straight forward. In keeping with the whole theme, it was important that the styles were always overridable. For that reason the file `assets/css/all.css` should be copied and updated to the extending site.

I also converted Font Awesome to it's SASS implementation, instead of the direct include, just to provide for a little more overriding when required.

## Continued

I'm going to look to continue this topic after getting a little deeper in the process.
