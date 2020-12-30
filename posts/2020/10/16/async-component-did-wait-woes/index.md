---
type: Blog
categories: [Blog]
title: await and componentDidMount woes
summary:
  While converting React Native Bluetooth Classic to Typescript I ran into some issues with componentDidMount and await.
  A couple evenings down of trying to figure out componentDidMount being blocked and a resolution was found.
tags: [React Native, Node, Javascript]
---

First let me preface this by stating I'm not the biggest fan of Javascript. Don't get me wrong, I don't hate it, I'm just used to doing things in a ways that don't translate well to JS. This is probably a case of

> You can't teach an old dog new tricks

but I'm definitely doing my best to learn the new trick! It's just taking some time.

## The Problem

A year or so ago I wrote my first React Native Module [react-native-bluetooth-classic](https://kenjdavidson.github.io/react-native-bluetooth-class) and I'm currently in the process of adding some new functionality (multiple devices, etc.) and updating the example application with some much needed love. While doing so, I migrated the connection logic from the main `App` to the `ConnectionScreen`:

```java
// Ya, I also like working with classes instead of functions, I'm terrible
export default ConnectionScreen extends React.Component {
  constructor(props) {...}

  async componentDidMount() {
     try {
        let connection = await RNBluetoothClassic.connect(this.props.device.address, {});
        this.setState({connection});
     } catch(error) {
        addMessage(`Unable to connect to ${this.props.device.address}`);
     }
  }
}
```

Which I thought was exactly what should happen... The issue here is that when mounting the `ConnectionScreen` there was a lag (the time it took to connect/error) while attempting the connection. Very noticeable and very uncool!

## The Cause

Since I was following all the documentation and all the tutorials it didn't much make sense what was going on:

- I had `componentDidMount` set to `async` so that it would return a `Promise`
- I had `let connection = await RNBluetoothClassic.connect(this.props.device.address, {});` and my native method was configured with a `Promise` as it's last paraemeter.

But there was still that noticable blocking in what should be a non-blocking call. After looking around there was one post on Stack Overflow (a hero) [https://stackoverflow.com/questions/58506993/why-does-async-work-in-componentdidmount-lead-to-visual-lag-when-navigating-unle#comment113857184_58506993](https://stackoverflow.com/questions/58506993/why-does-async-work-in-componentdidmount-lead-to-visual-lag-when-navigating-unle#comment113857184_58506993) that brought up the idea of task queue and job queue, which after doing some research should have been **microtask** and **macrotask** queues.

Following the rabbit hole that is NodeJS documentation and tutorials, I found another amazingly informative posts:

- [https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/) which explains the event loop phases in detail.
- [https://blog.logrocket.com/a-complete-guide-to-the-node-js-event-loop/](https://blog.logrocket.com/a-complete-guide-to-the-node-js-event-loop/) another look at the event loop phases.

### My limited understanding of the issue

After reviewing all the information provided there were a few assumptions that were made (probably wrong, but seemed valid):

1. So essentially each phase of the event loop is the macrotask queue. The primary phase is the poll phase which accepts it's "jobs" (for lack of a better term) from a number of places:

- Promises
- Results of internal calls (fs, fetch, etc). This is important because of #2.

2. Each of the phases in and of itself is a microqueue - meaning that when a Promise is being run in the poll phase, if it creates any new Promise(s) they will be placed in the same poll microqueue (just at the end). This let's the initial Promise complete wholely, but then starts the subsequent Promise(s) in the same turn processing of the queue. From here:

- React Native magic happens to load the `ConnectionScreen`
- `ConnectionScreen.componentDidMount` Promise was created and added to the queue
- `RNBluetoothClassic.connect` Promise was added to the queue - because it came from the same macrotask it must run before any others can. This meant that event though the native connection was occurring in another thread (Java/Swift) React Native couldn't do anything about it due to the promise placement.

3. The single threadedness of Javascript is great - but in the grand scheme of things an illusion in my mind (which is part of the reason I dislike it so much). What is missed out on all this is how there is a wonderful world of native APIs (file system, network, etc.) that do work within differnet threads (literally the only way to make multiple things happen) that is just masked by this idea of a single threaded event loop.

Maybe I'm jaded or maybe I don't fully get it, but it just bothers me.

## The Resolution

With all that said, the point was to get my `ConnectionScreen.componentDidMount()` not blocking on the `RNBluetoothClassic.connect()` call. Now that I've got a (somewhat) better understanding of how things go together, the answer within the Stake Overflow question makes sense - force the connection call to be pushed to a new macrotask queue so that the current loading isn't blocked, the screen can show, then the connection can happen downstream:

```java
export default class ConnectionScreen extends React.Component {
  constructor(props) {...}

  async componentDidMount() {
     setTimeout(this.connect, 0);  // jump to next macrotask
  }

  async connect() {
    addMessage(`Attempting connection to ${this.props.device.address}`);

    try {
      let connection = RNBluetoothClassic.connect(this.props.device.address, {});
      this.setState({connection});
    } catch(error) {
      addMessage(`Unable to connect to ${this.props.device.address}`);
    }
  }
}
```

and voila! We have a non blocking screen with a successful connection happening.

Now some questions that I have with this, based on how other internal libraries are designed (and for that I probably need to dig through source) is whether all my React Native (JS) module functions wrap the Native implementations in a `setTimeout()` so that they all do this automatically? Or should it be left up to the implementation to determine? Is it the Javascript or Native implementations of FS and FETCH that perform this extra step of adding to a separate polling?

Thoughts for later.

#### Issues or Misunderstandings

Please, if you come across this and you feel like:

- I've made a huge mistake in how I've interpreted things
- The solution I've chosen is not the best/optimal and can be done better

please let me know! I'm definitely well behind the eight ball with Javascript but I'm trying to learn to love it!
