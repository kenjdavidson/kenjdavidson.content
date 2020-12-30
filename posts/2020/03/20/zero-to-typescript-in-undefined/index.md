---
type: Blog
categories: [Blog]
title: Zero to TypeScript in ... I'll let you know when it's done.
summary:
  So I've been disliking Javascript for a while now, I won't lie - but I understand necessity.  Looks like it's time to give
  Typescript a go and try to make the best of it.  Enjoy my suffering.
tags: [TypeScript, JavaScript, VSCode]
---

I know what you're thinking...

<div style="margin: 2em;"><blockquote class="cite"><p>
TypeScript has been out for ages, how's this guy just getting into it now?
</p></blockquote></div>

Well let me tell you, I've never been the biggest fan of JavaScript! If you're still reading, let me explain; my educational background and pretty much all of my development experience has been in languages like C/C++, Java and to a lesser extent C#. For whatever reason, I feel substantially more comfortable working with compiled, strongly typed lanuages. Life is simple, Java provides you with two main build systems [Maven](https://maven.apache.org/) and [Gradle](https://gradle.org/); with the C family it's pretty much just [Visual Studio](https://visualstudio.microsoft.com/).

I've finally gotten a number of projects I've been working on to a place where I can take some time off and concentrate on something else. It seemed like TypeScript was something that would help me cross over fully into the JavaScript world, with it providing a little strongly typed love.

## Project Overview

Everyone is used to `hello-world` or `counting-ts` when reading tutorials. As much as I love reading about implementing something that really doesn't touch on real world functionality, I decided that this would be a good spot to work on a custom library that I've been thinking about.

### Canada Golf Api

My goal is to design and develop a client library (TypeScript) that will be usable within a Node environment; I don't see a real need for client/web environment as it would require passwords/tokens being made available (although this could change). This isn't exactly a public API, so who knows if it'll get shut down, but it's where I'm going to start.

## Reading Preparation

Like you (maybe) I've read a large number of tutorials and information on how to setup and configure a new TypeScript library. They document the appropriate build and deployment processes, with conflicting information at times:

### TypeScript Node Starter

[TypeScript Node Starter](https://github.com/microsoft/TypeScript-Node-Starter)

Straight from Microsoft themselves, following this template seems like a good start. This project includes:

- TypeScript
- Linting
- Build and deployment scripts

All the good things that I feel I would benefit from. What is missing though, is any kind of transpiler or packager? So do I need one?

### TypeScript &amp; Babel

[TypeScript &amp; Babel](https://iamturns.com/TypeScript-babel/)

So a couple other Googles and I found this article, while reading it there were two main points that jumped out at me as concerning:

> Most TypeScript developers experience slow compilation times during development / watch mode. You’re coding away, you save a file, and… then… here it comes… annnnd… finally, you see your change.

And this bad boy:

> Yeah, you know it’s broken. You’ve probably broken a few unit tests too. But you’re just experimenting at this point. It’s infuriating to continuously ensure all your code is type safe all the time.

It's just the old timer in me I guess, but this **is exactly what I want while developing**. For right now, I'm going to continue without Babel, and hope that the speed issue above is only apparent with extremely large projects.

### Webpack or Rollup or None?

There are a number of recent articles I found documenting each of the popular choices:

- [https://marcobotto.com/blog/compiling-and-bundling-TypeScript-libraries-with-webpack/](https://marcobotto.com/blog/compiling-and-bundling-TypeScript-libraries-with-webpack/)
- [https://hackernoon.com/building-and-publishing-a-module-with-TypeScript-and-rollup-js-faa778c85396](https://hackernoon.com/building-and-publishing-a-module-with-TypeScript-and-rollup-js-faa778c85396)
- [https://medium.com/cameron-nokes/the-30-second-guide-to-publishing-a-TypeScript-package-to-npm-89d93ff7bccd](https://medium.com/cameron-nokes/the-30-second-guide-to-publishing-a-TypeScript-package-to-npm-89d93ff7bccd)

Again, more and more ways to do things, this obviously becomes better over time and with practice, but but since I have neither it's just a confusing start. For the time being I'm going to simplify (in the hopes I can still build out later) and go with TypeScript, just TypeScript.

## Initialize the Project

No time like the present to get started - first I'll create the project structure, initialize npm (yup, I know yarn is out there but baby steps) then git:

```shell
$ mkdir golf-canada-js && cd golf-canada-js
$ npm init

Press ^C at any time to quit.
package name: (golf-canada-blog)
version: (1.0.0)
description: Golf canada scores/member client API
entry point: (index.js) dist/index.js
test command:
git repository: https://github.com/kenjdavidson/golf-canada-js
keywords: golf canada javascript
author: Ken Davidson
license: (ISC) MIT

$ mkdir src
$ mkdir test
$ tree
.
├── package.json
├── src
└── test
```

Pretty much straight forward project creation.

## TypeScript &amp; the Goodies

Next to install TypeScript and the extras that I've seen floating around, again I'm trying to keep things straight forward for this project. I've decided to start with:

```shell
$ npm install --save-dev typescript ts-lint jest ts-jest @types/jest @types/node prettier
$ ./node_modules/typescript/bin/tsc --init
message TS6071: Successfully created a tsconfig.json file.
```

- **TypeScript** installed locally - from what I've read there are pros and cons to this, but since I'm just starting out and not really sure how well this will go; I figure local will have less impact.
- **ts-lint** and **prettier** to make sure things stay clean
- **jest** and **ts-jest** for testing - which should be a whole other article on it's own
- and finally the appropriate **@types** for **jest** and **node**

### Configure tsconfig

With TypeScript installed, it needs to be configured - although there are a ton of options, there are no more than Maven or Gradle when you get down to it - it just takes a lot of reading to figure out what. Thankfully the default `tsconfig.json` file is pretty well documented, along with the [TypeScript handbook](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html). With the parsed out lines, the final(ish) file:

```json
{
  "compilerOptions": {
    "incremental": true /* Enable incremental compilation */,
    "target": "es5" /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', 'ES2018', 'ES2019', 'ES2020', or 'ESNEXT'. */,
    "module": "commonjs" /* Specify module code generation: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', 'es2020', or 'ESNext'. */,
    "lib": [
      "es2016"
    ] /* Specify library files to be included in the compilation. */,
    "declaration": true /* Generates corresponding '.d.ts' file. */,
    "declarationMap": true /* Generates a sourcemap for each corresponding '.d.ts' file. */,
    "outDir": "./dist" /* Redirect output structure to the directory. */,
    "strict": true /* Enable all strict type-checking options. */,
    "baseUrl": "./src" /* Base directory to resolve non-absolute module names. */,
    "esModuleInterop": true /* Enables emit interoperability between CommonJS and ES Modules via creation of namespace objects for all imports. Implies 'allowSyntheticDefaultImports'. */,
    "forceConsistentCasingInFileNames": true /* Disallow inconsistently-cased references to the same file. */
  },
  "include": ["src/**/*.ts"],
  "exclude": ["dist", "node_modules", "**/*.spec.ts"]
}
```

## Attempting the First Compile

At this point, from what I've read, I should be able to start throwing down some TypeScript and getting it to compile. To start small, I decided to work on the the interfaces related to one of the open API endpoints available - user profile and handicap summaries.

```typescript
// src/scores/Club.ts
export default interface Club {
  name: string;
  line1: string;
  line2?: string;
  city: string;
  region: string;
  phone: string;
  url: string;
}
```

And then attempt a compile:

```shell
$ npm run watch-ts
[10:19:33 PM] Starting compilation in watch mode...
[10:19:38 PM] Found 0 errors. Watching for file changes.
```

Seems good so far, let's try and add and build a new file:

```typescript
// src/scores/ScoreSummary.ts
export default interface ScoreSummary {
  course: string;
  datePlayed: Date;
  holesPlayed: number;
  score: number;
  isUsedInCalc: boolean;
}
```

and... we're good!! This is a slow process, since it's summer and I love being outside my off hours development takes a hit. I'll look to continue this in the fall/winter when I get more time to fill with programming and writing.

Check back, I have plans at continuing this when summer, golf and my son aren't getting in the way!
