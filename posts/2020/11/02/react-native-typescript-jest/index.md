---
type: Blog
category: Technology
title: Typescript React Native with Jest
summary:
  Still going down the Typescript conversion of React Native Bluetooth Classic it was time to get the library setup with
  actual unit testing.  Follow along with another few days of failures and successes.
tags: [React Native, Typescript, Jest]
---

So I've been slowly converting [react-native-bluetooth-classic](https://kenjdavidson.com/react-native-bluetooth-classic) to Typescript, which went off fairly well, and decided it would be a good idea to start adding tests. Figured it would get me some experience working with [Jest](https://jestjs.io/) and getting a better product available for everyone.

> I think I've made a huge mistake!!

Although I don't actually regret the decision, it's defintely been more of a process than I had originally anticipated. This post isn't going to be anything original, it's just going to be the process that was required in order to get things working.

## Getting Started

Following the treasure trail of internet posts documenting how to get Typescript, Jest and React Native working together, it was apparent that it would be a little bit of work, but generally fairly straight forward. The consensus is that the following libraries are required:

- [Jest](https://jestjs.io/) obviously as the testing framework. Since it's developed by Facebook and baked into React Native it makes sense.
- [ts-jest](https://kulshekhar.github.io/ts-jest/) which from Medium and other sites is the best Typescript/Jest library. It also provides a pretty decide guide on setting things up.

## Installing the Things

First thing first, lets follow [ts-jest/install](https://kulshekhar.github.io/ts-jest/user/install) and get all our ducks in a row. First we'll install all the

```shell
npm install --save-dev jest ts-jest @types/jest
```

- Install all the libraries required (Typescript was already installed)

## Configuring the Things

Once we've got things installed, we can get to configuring, here's another instance of following the internet treasure trail. We take a look at the available documentation on **ts-jest** there is a page specific to [ts-config/react-native](https://kulshekhar.github.io/ts-jest/user/react-native/). The first thing we come to when we get there, is that we should check out another post on [react-native Typescript](http://reactnative.dev/blog/2018/05/07/using-typescript-with-react-native).

### Using TypeScript with React Native

[Using TypeScript with React Native](http://reactnative.dev/blog/2018/05/07/using-typescript-with-react-native)

Jumping down the page the first section of importance is [adding typescript](http://reactnative.dev/blog/2018/05/07/using-typescript-with-react-native#adding-typescript), the key parts being the added typescript transformer and the **react native config** file:

```shell
npm install --save-dev react-native-typescript-transformer
npm install --save-dev @types/jest @types/react @types/react-native @types/react-test-renderer
touch rn-cli.config.js
```

uncommenting the appropriate lines in `tsconfig.json`:

```json
{
  /* Search the config file for the following line and uncomment it. */
  "allowSyntheticDefaultImports": true /* Allow default imports from modules with no default export. This does not affect code emit, just typechecking. */
}
```

and finally adding the `rn-cli.config.js` configuration:

```javascript
// rn-cli.config.js
module.exports = {
  getTransformModulePath() {
    return require.resolve("react-native-typescript-transformer");
  },
  getSourceExts() {
    return ["ts", "tsx"];
  }
};
```

The next key spot is [adding typescript testing infrastrucure](http://reactnative.dev/blog/2018/05/07/using-typescript-with-react-native#adding-typescript-testing-infrastructure) where it says to add the configuration to `package.json`, which clearly went against most of the other configuration places. After trying both ways, I can confirm that having the Jest configuration inside `package.json` does not work.

So we'll just stick with the same content in `jest.config.js`:

```javascript
{
  "jest": {
    "preset": "react-native",
    "moduleFileExtensions": [
      "ts",
      "tsx",
      "js"
    ],
    "transform": {
      "^.+\\.(js)$": "<rootDir>/node_modules/babel-jest",
      "\\.(ts|tsx)$": "<rootDir>/node_modules/ts-jest/preprocessor.js"
    },
    "testRegex": "(/__tests__/.*|\\.(test|spec))\\.(ts|tsx|js)$",
    "testPathIgnorePatterns": [
      "\\.snap$",
      "<rootDir>/node_modules/"
    ],
    "cacheDirectory": ".jest/cache"
  }
}
```

So this is where the first issue happens, because this is a library it has `react` and `react-native` set as peer dependencies.

```shell
> react-native-bluetooth-classic@1.0.0-rc.2 test /Users/kendavidson/git/react-native-bluetooth-classic
> jest

● Validation Error:

  Preset react-native is invalid:

  The "id" argument must be of type string. Received type object
  TypeError [ERR_INVALID_ARG_TYPE]: The "id" argument must be of type string. Received type object
```

Installing `react` and `react-native` (following the lowest version requirements in the peer dependencies) as dev dependencies cleared this issue up.

### ts-config React Native

Now that we've followed that doc, we can go back to the [ts-config/react-native](https://kulshekhar.github.io/ts-jest/user/react-native/) page and continue to follow along. First we need to create the Babel configuration:

```javascript
// babel.config.js
module.exports = {
  presets: ["module:metro-react-native-babel-preset"]
};
```

and then we need to modify the Jest configuration:

```javascript
// jest.config.js
const { defaults: tsjPreset } = require("ts-jest/presets");

module.exports = {
  ...tsjPreset,
  preset: "react-native",
  transform: {
    ...tsjPreset.transform,
    "\\.js$": "<rootDir>/node_modules/react-native/jest/preprocessor.js"
  },
  globals: {
    "ts-jest": {
      babelConfig: true
    }
  },
  // This is the only part which you can keep
  // from the above linked tutorial's config:
  cacheDirectory: ".jest/cache"
};
```

Which essentially rewrites the original file, pretty much we're just adding the `react-native/jest/preprocessor.js` so that the imported React Native modules are _compiled_ correctly during testing.

## Writing a Test

The first thing that I wanted to do, was move the validation of IOS and Android logic to Javascript, instead of bothing to call IOS Native for things that don't exist. To do this it made sense to test things based on `Platform` (mocking Platform):

```javascript
/// __tests__/BluetoothClassicModule.ios.test.js
import { Platform } from 'react-native';

// No dice
jest.mock('Platform', () => {
  ...
});

// No dice
jest.mock('/react-native/Libraries/Utilities/Platform', () => {
  ...
});
```

This one took a bunch of Googling. While attempting to mock the `Platform` module directly, would continually throw errors that `Platform could not be found in /react-native/Libraries/Utlities/react-native-implementation.js`. The key here is [https://github.com/facebook/react-native/issues/26579#issuecomment-535244001](https://github.com/facebook/react-native/issues/26579#issuecomment-535244001) which explains:

> This is intentional and you need to mock modules the same way as any other JS module now. You could in theory specify the path to the TextInput module, but the path is a private implementation detail that could change between releases. The logically correct approach is to code to the interface and mock out react-native.

which makes complete sense. The resulting mock becomes:

```javascript
/// __tests__/BluetoothClassicModule.ios.test.js
import { Platform } from "react-native";

jest.mock("react-native", () => ({
  Platform: { OS: "ios" }
}));

describe("React Native Platform", () => {
  test("Platform.OS should be 'ios'", () => {
    expect(Platform.OS).toBe("ios");
  });
});
```

which finally results in a successful test:

```shell
> react-native-bluetooth-classic@1.0.0-rc.2 test /Users/kendavidson/git/react-native-bluetooth-classic
> jest

 PASS  __tests__/BluetoothModule.ios.test.ts
  React Native Platform
    ✓ Platform.OS should be 'ios' (2 ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        2.03 s, estimated 7 s
Ran all test suites.
```

## Sadly a Waste of Time

And just as I thought, you take one step forward and two steps back. Since I'm working on a library I want to be able to test said library without having to push changes to Git/Npm in order to test. For this reason I have the following structure:

```shell
|- react-native-bluetooth-classic
  |- BluetoothClassciExample
  | |- node_modules
  |- node_modules
```

and I run `react-native start` out of `BluetoothClassciExample`. To get this working I have a customized `metro.config.js` which pulls in my `react-native-bluetooth-classic` module from `../`. The sad thing here is that I'm unaware of a way to make it NOT use the now required local node_modules dependency of `react-native`.

Now because `react-native` is being loaded twice:

```shell
┌──────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│  Running Metro Bundler on port 8081.                                         │
│                                                                              │
│  Keep Metro running while developing on any JS projects. Feel free to        │
│  close this tab and run your own Metro instance if you prefer.               │
│                                                                              │
│  https://github.com/facebook/react-native                                    │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘

Looking for JS files in
   /Users/kendavidson/git/react-native-bluetooth-classic/BluetoothClassicExample
   /Users/kendavidson/git/react-native-bluetooth-classic
```

There are a large number of conflicts and issues which which cause a bunch of crazy errors:

- [https://github.com/expo/expo/issues/916](https://github.com/expo/expo/issues/916)
- [https://stackoverflow.com/questions/46312103/react-native-module-hmrclient-is-not-a-registered-callable-module-calling-enabl](https://stackoverflow.com/questions/46312103/react-native-module-hmrclient-is-not-a-registered-callable-module-calling-enabl)

which all seem to point to things like **hot reloading**, **dev mode**, resetting cache, adb reverse, etc. None of which work, so sadly at this point it looks like I either get:

1. to run tests
2. to run dev apps easier

and at this point I'm going with option 2. Sorry testing!

<s>If you've got a way for me to load `../` but ignore `../node_modules` during development, shoot me an email. But until then I'll have to put this on the back burner.</s>

### Edit Nov 02 2020

After some late night and early morning Googling, I came across a customization of Metro that seems to fit better with my example [https://medium.com/@charpeni/setting-up-an-example-app-for-your-react-native-library-d940c5cf31e4](https://medium.com/@charpeni/setting-up-an-example-app-for-your-react-native-library-d940c5cf31e4) which makes the following changes:

#### Replaces `resolver.extraNodeModules` with `watchFolders`

Which at this point I'm unsure of the differences (will review) but it's worth a shot.

#### Adds in `resolver.blacklist`

Which doesn't even exist on the [Metro config](https://facebook.github.io/metro/docs/configuration/#blocklist) where it's called `resolver.blockList`. The issue here is that it specifically says:

> A RegEx defining which paths to ignore, however if a blocklisted file is required it will be brought into the dependency graph.

But it's worth a shot!!

### Edit Nov 03 2020

After following the previous posts information, there are still errors that revolve around `lib/node_modules/react-native` being installed. When attempting to run, it's still attempting to load the `react-native` from the `../node_modules` lib folder instead of from the `/example/node_modules/` folder causing all those wonderful duplicate React Native issues.

So at this point, I can write my tests with `react` and `react-native` installed, then uninstall them when I want to start doing more live testing within the **BluetoothClassicExample** app.

Since this seems to work for this post (and others) I'm starting to think that the issue is the introduction of Typescript during the build process. But at this point I'd rather keep Typescript (and deal with the [un]installing) rather than go back to plain JavaScript or do the half way kludge of `babel-typescript` at this point.

### Edit Nov 20 2020

This edit was a little late, but things are finally working. Using the same `metro-config` above, but changing the strcuture of the projects, I'm able to finally:

- Run the development application
- Have `react` and `react-native` devDependencies for testing

using the following structure:

```shell
|- git
  |- react-native-bluetooth-classic
  |- react-native-bluetooth-classic-apps
    |- BluetoothClassicExample
```

I know it might be a little much, but I chose the extra layer in case I needed to provide sample apps for things like: different versions, bug fixes, etc. This also cleans up the `react-native-bluetooth-classic` project in that only tests are required.

Now I just have to write them :(
