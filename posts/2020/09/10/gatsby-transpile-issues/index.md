---
type: Blog
categories: [Blog]
title: No exports main resolved ... @babel/helper-compilation-targets/package.json
summary: Node broke Gatsby and messed up my Github Actions 
tags: [Gatsby, Node, Github Actions]
---

After the recent version of Node was released my Gatsby (as well as others) stopped building on Github.  To keep my site up to date (or at least up to date as it can be - I don't post a lot) I use a wonderful Github Action to publish - [enriikke/gatsby-gh-pages-action@v2](https://github.com/enriikke/gatsby-gh-pages-action).  Recently though the error started:

```bash
failed Building production JavaScript and CSS bundles - 1.468s
error Generating JavaScript bundles failed

[BABEL] /home/runner/work/kenjdavidson.github.io/kenjdavidson.github.io/.cache/production-app.js: No "exports" main resolved in /home/runner/work/kenjdavidson.github.io/kenjdavidson.github.io/node_modules/@babel/helper-compilation-targets/package.json
not finished Generating image thumbnails - 11.907s
not finished run queries - 1.713s
npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! kenjdavidson.github.io@0.9.0 build: `gatsby build`
npm ERR! Exit status 1
npm ERR! 
npm ERR! Failed at the kenjdavidson.github.io@0.9.0 build script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

npm ERR! A complete log of this run can be found in:
npm ERR!     /home/runner/.npm/_logs/2020-08-19T14_12_53_132Z-debug.log
##[error]The process '/usr/local/bin/npm' failed with exit code 1
```

This was due to the latest version of Node and the `@babel/helper-compilation-targets` dependency.  The primary way to resolve this was to update the `package-lock.json` so that it used the correct dependency versions.  But in my mind updating the lock file goes against the whole point of having the lock file.  The solution I chose was to update the Github Action to use the appropriate version of Node - until I had time to work through the entire set of dependencies and keeping the project whole.

I updated my Github action to contain the following:

```yml
# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner on which this job will run
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: ['12.16.3']

    # Steps represent a sequence of tasks that will be executed as part of the job
    # - Checkout gatsby branch
    # - Update authentication for Github Package Registry @kenjdavidson/base16-scss
    # - Build gh-pages using action
    steps:
      - uses: actions/checkout@v1
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node-version }}
      - name: Authenticate with GitHub package registry
        run: echo "//npm.pkg.github.com/:_authToken=${{ secrets.ACCESS_TOKEN }}" > ~/.npmrc
      - uses: enriikke/gatsby-gh-pages-action@v2
        with:
          access-token: ${{ secrets.ACCESS_TOKEN }}
```

where the key lines are `node-version: ['12.16.3]`.  Pushing the build version back to the known good version required for these dependencies resolved the issue and allowed my site to once again be published.

#### Upgrading Node

Once I get a chance to upgrade Node on my personal machine, I'll be able to get the `package.json` dependencies to match the updated Node version - but from now on I'll probably try to keep this Action using the same build that matches the Gatsby requirement.
