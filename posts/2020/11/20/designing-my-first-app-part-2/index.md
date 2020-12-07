---
type: Blog
categories: [Project]
title: Standardbred Mobile - Framework Selection Process - Part 2
summary: Continuing the journey of how I created my first React Native App
tags: [React Native, Android, IOS]
series: Designing My First React Native App
order: 2
---

If you made it through [part 1](/writing/2020/01/03/designing-my-first-app-part-1) (I'm aware it's been forever since then; but this has been drafted, just never released) you'll remember that after a bunch of decision making the final choices were:

- React Native
- Redux
- Axios

With the requirement of creating a new Bluetooth native module that supported MFi (External Accessory) on IOS.   I'll jump right into the application creation process - as the Bluetooth module requires it's own article (which can be found [here](https://kenjdavidson.com/2019-09-01-getting-started-with-sc-mobile).

## Creating the App

Well it seems like a pretty logical step to follow the available documentation on creating a React Native app - straight from the horses mouth, the React Native CLI did the trick (as I would expect).  From the original [Getting Started](https://facebook.github.io/react-native/docs/getting-started) guide, it was pretty straight forward once the correct path was chosen:

React Native CLI Quickstart due to the fact that I knew Expo wasn't going to be an option with Bluetooth.  There was no point in starting there and having to eject at a later time anyhow.
Development OS: MacOS
Target OS: Android - since I figured it would be better starting off with something I was comfortable with, rather than jumping straight into IOS.
Note - an important step here is making sure you have `watchman` installed.  I've had to reset my laptop a few times and forgot about it after setting up the project again.  It might just be me and my lack of memory, but always double check things are installed when you are re-configuring, assumptions will lead to some annoying errors down the road.

```bash
$ npx react-native init standardbred-mobile-app
```

The current version of React Native at the time of creating the app was `0.59.9`.  I get that there have been substantial releases since then, but I haven't had a chance to update.

The application was created as expected and it was time to figure out where to go from there.   I had recently started using Visual Studio Code, which also seemed to have a decent review of the [vscode-react-native extension](https://github.com/microsoft/vscode-react-native).   Once everything was setup, my environment looked like:

#### Visual Studio Code

`vscode-react-native extension`

Open project `~/git/standardbred-mobile-app`

For the most part your development will be done only in VS Code, specifically now that React Native 0.60+ has auto linking.  Even before auto-linking the `react-native link` command did a good job - but I always like digging a little deeper into frameworks like this to ensure I've got at least a 6-7/10 understanding of how things work.

#### Android Environment

To get started I made sure that I had:
A couple AVDs downloaded and setup
A connection to my phone configured for Development
Another key point here is that when you're working with an active device you'll need to remember the following:

```bash
$ adb devices
$ adb reverse tcp:8081 tcp:8081
```
to ensure that your physical phone can react the Metro packaging service.  This is documented pretty well in the [Running on Device](https://facebook.github.io/react-native/docs/running-on-device#method-1-using-adb-reverse-recommended) section, but just something to always remember.  I don't generally use the `-s <device name>` argument as I'm usually only connected to one device.

#### Firing up the Starter App

Once everything seems good it's time to see if the starter app will run correctly.  There are a few different ways to do this:

1) Directly from **VS Code** using the `Command Palette` - quick way to test the app proper
React Native: Run Android
will start both the Metro packager service and the build / install process.

2) Indirectly from **VS Code** and **Android Studio** - used for developing/debugging native modules
VS Code `Command Palette` React Native: Start Packager
Android Studio Debug
Again, since most of my experience is in Android Studio, not VS Code, there are a lot of things I could probably streamline in the build / debug process.  But for now this seemed to get me started.

Either way, the app loaded onto my phone just as expected.  Things to play with:
The [remote debugger](https://facebook.github.io/react-native/docs/debugging) - I primarily use the Chrome debugger (not the standalone) which seems to serve my purpose well.
## (Re)Structuring the App

The starter app is pretty straight forward, but going back to some of the best practices discussed earlier, I wanted to lay out a good foundation so that I didn't have to refactor a bunch of stuff later.  One of the first articles on structuring I read was [https://medium.com/react-native-training/best-practices-for-creating-react-native-apps-part-1-66311c746df3](https://medium.com/react-native-training/best-practices-for-creating-react-native-apps-part-1-66311c746df3) which hit on a number of items that I knew would be required.  A second article I read, which I actually followed a little more during this process was [https://www.freecodecamp.org/news/how-to-structure-your-project-and-manage-static-resources-in-react-native-6f4cfc947d92/](https://www.freecodecamp.org/news/how-to-structure-your-project-and-manage-static-resources-in-react-native-6f4cfc947d92/).


#### Assets

Which includes icons, fonts and images.   Following the **freecodecamp** article above with regards to the **R namespace** section made sense to me, as I was (at this point) more comfortable working with the Android resources.   Examples and details can be found [here](https://www.freecodecamp.org/news/how-to-structure-your-project-and-manage-static-resources-in-react-native-6f4cfc947d92/#the-r-namespace).  

#### Components 

The components were broken up into two types:

**Components** which were named this way are designed as reusable items that can be used anywhere, these (for the most part) are presentation components.  Although due to this being my first app, I planned for some initial confusion and bad planning and would allow the first few versions to also have container components.

**Screens** were going to be the high level single purpose container component.

at a very high level, a screen containing multiple components would look like:

```jsx
<HorseLookupScreen>
  <Toolbar>
     <SearchText/>
     <BluetoothStatusIcon/>
   <Toolbar>
   <FlatList/>
</HorseLookupScreen>
```

> Note - Another item that I wish I had started with while working - Styled Components.  The current implementation is using the built in `StyleSheets` due to me >not wanting to add in too many things at one time.  This was a bad move, the more I've been using React[Native], styled components makes things so much  simpler. 

#### Navigation

Navigation wasn't a huge deal at this point, the app only had two screens, Login and HorseLookup. 

#### Persistence

Persistence also wasn't a huge deal at this point, the decision was to get the application out:
Users would have to log in each time
We are only married to one scanner model, which is configured separately, there was no need to manage the state.
although after reading it looked like React Native persistence was substantially easier than I had imagined it would be - look for a future article following that process.

## Libraries and Packages

Another great item from the [freecodecamp](https://www.freecodecamp.org/news/how-to-structure-your-project-and-manage-static-resources-in-react-native-6f4cfc947d92/#library) was the usage of `index.js` and `package.json` to reduce and clean up imports.  The author **Khoa Pham** even admits that this point segments React Native developers, which I can fully get, but it was a decision that I liked and decided to follow.

In my mind changing:

```javascript
import SearchText from '../../../components/SearchText';
```

to this:

```javascript
import SearchText from '@scmobile/components/SearchText';
```

just looks cleaner.  I've added the `@scmobile` prefix to everything, as I wasn't sure whether I would want to break some of these components out into their own library to use across other applications.  I figured it could possibly be less issue then.

Taking it a step further resulted in:

```javascript
import { SearchText } from '@scmobile/components';
```

which after getting more of the components built out made things a bit better(er?).

## Final Structure

Folder stucture seems to always be a point of **discussion** when working on projects, this is how the final product ended up:

```bash
├── App.js
├── README.md
├── __tests__
│  └── App-test.js
├── android
├── app.json
├── babel.config.js
├── index.js
├── ios
├── metro.config.js
├── node_modules
├── package-lock.json
├── package.json
├── resources
│   └── images
└── src
    ├── common
    │   ├── api
    │   ├── btscanner
    │   ├── components
    │   ├── i18n
    │   └── package.json
    ├── redux
    │   ├── auth
    │   ├── i18n
    │   ├── meta
    │   ├── package.json
    │   └── redux.js
    ├── resources
    │   ├── R.js
    │   ├── colors.js
    │   ├── images
    │   ├── images.js
    │   ├── languages
    │   ├── package.json
    │   └── styles.js
    └── screens
        ├── MainScreen.js
        ├── home
        ├── horse
        ├── login
        └── package.json
```
