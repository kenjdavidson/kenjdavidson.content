---
type: Post
category: Technology
title: A year of Bluetooth Classic Maintenence
summary: It's been a cool (albeit annoying at times) experience to maintain my first real project.
tags: [React Native, Bluetooth, Open Source]
---

So it's actually probably been longer than a year, but if we just go back to when I started/released `v1.60.x-rcY` it's close enough.  When I was first re-implementing [react-native-bluetooth-serial](https://www.npmjs.com/package/react-native-bluetooth-serial) I thought it would be for my own copmanies app; I was not planning nor ready to start maintaining something that would be used all over the world.

> `v1.60.x-rcY` is still in release candidate mode, since I've been continuing to work with users on adding the final touches to get it where it needs to be in order to drop the `rcY` portion.

## Background

I was tasked with designing and developing my companies microchip scanning app - which in and of itself is it's own article series - which used an MFI compliant microchip scanner.  I'd never done any IOS development before and I'd only played with Android a little.  While looking over the options there were only a couple:

- I'd have to write an Android and IOS version of the App and communication peices
- I'd use React Native or Flutter and I'd have to just write the communication peice

React Native posed the biggest up and down sides.  For one, there would be one code base, I wouldn't have to maintain two applications (which really isn't actually the truth once you get started, but it's a great way to suck you in) but I would have to at least write the [native module]() for the Bluetooth communication.   Since the latter was happening anyhow, the former made the choice pretty straight forward.

I ended up keeping the project public, since the original [react-native-bluetooth-serial](https://www.npmjs.com/package/react-native-bluetooth-serial) was public and seemed to not be maintained anymore but had questions and requests for the functionality that I was going to be adding.  I wasn't sure how or if this project would be found by anyone else, but hey, why not give it a chance.

## v0.10.x was Released

The actual first version was `v0.9.x` which started with React Native v0.59.4, but since shortly after that React Native v0.60.0 was released with breaking changes I released `v0.10.0` to match.  The main changes here were the auto registration of the module; which turns out didn't really matter much at the time.  When I released the library the top priorites for me were:

### Converting IOS Bluetooth to MFi

This was the sole reason for me (re)writing the library becuase there was absolutely nothing out there already.  After getting to know how Apple and the MFi program worked, it made sense - why the hell would anyone design hardware with BLE available that required them to signup and maintain their MFi status.  I guess it's an Apple thing (don't get me started on my Apple opinions) and it's required in some ways, but man it makes life annoying.

I've written a couple articles about this processs:

- [https://kenjdavidson.com/writing/2019/06/02/my-foray-into-swift-streams](/writing/2019/06/02/my-foray-into-swift-streams)
- [https://kenjdavidson.com/writing/2020/10/16/async-component-did-wait-woes](/writing/2020/10/16/async-component-did-wait-woes)

### Standardizing Method Calls

One of the other things I wanted was the consistency across the native calls - I get this is a choice some won't agree with - I chose to make all of the calls `Promise` based.  Even the calls that didn't really require a promise (getBondedDevices, isEnabled, etc.) but maybe it's the OCD in me or the fact that I'm not JavaScripts biggest fan (although Typescript helps out a lot) and it just seemed like a good idea at the time.

Moving to promises across the board, also allowed being able to consistently throw `Error` on IOS when a feature wasn't available.  I absolutely hate the idea of having two files `feature.android.js` and `feature.ios.js` when it's just as easy to catch and check an `Error` for not implemented.   This gave users the opportunity to:

- create multiple files
- use the `Platform` functionality within a single file
- show a resulting error that the feature isn't available on IOS

I didn't want to force users into any of these.  Plus if they chose the latter and one of the features was implemented on IOS, there would be no changes needed (which was my goal, unrealized, but a goal never the less).

### Issues and Features

This is where the *fun?* began, there was a steady influx of issues and feature requests that started coming in, it was an exciting and (equally) aggravating time.  Some solid feature and pull requests came in:

- [Jose Gallegos](https://github.com/kenjdavidson/react-native-bluetooth-classic/pulls?q=is%3Apr+author%3AJose-Gallegos) provided some great documentation changes
- [iamandiradu](https://github.com/kenjdavidson/react-native-bluetooth-classic/pulls?q=is%3Apr+author%3Aiamandiradu) saved my life with some Podspec help
- [tpettrov](https://github.com/kenjdavidson/react-native-bluetooth-classic/pulls?q=is%3Apr+author%3Atpettrov) added one of the most used features

#### Accept Mode

As mentioned above, this was one of the features that took the library to the next level, provided by [Anton Pettrov](https://github.com/tpettrov).  It allowed the connection between two Android devices and effectively resolved like 50% of the feature requests from the original project.

SOLID!

## v1.60.x made an Appearance

One of the other major issues (or requests) was that it wasn't easy to send different data, the original (and this) library was only required to send String based data in either ASCII or UTF-8.  There were growing requests (and resolutions which required changes to the library) in order to send binary/byte data.  For the purpose of (a) helping these people out and (b) attempting to get more experience myself, I set out on upgrading the library.  The top requirements were:

- To connect to multiple devices at one time
- To provide each of these with their own communication settings

### Connect to Multiple Devices

This required the removal (or reworking) of the `BluetoothClassicService` into a manageable `DeviceConnection` that could be maintained.

### Provide Device Based Configuration

This one was a little more annoying, in that there needed to be a way to extend the functionality without bloating the current library.  This requires heavy use of the `autoloading` functionality on both Android and IOS as well as designing custom `DeviceConnection` logic within the host application.  There are still issues with this, and I'm not overly happy with it - but it works and can be updated down the road.

The logic was broken into three configurable peices:

- `Acceptors` which are used to listen for incomming connections - and pass off the resulting `BluetoothSocket`
- `Connectors` which do the connecting - and again pass off the `BluetoothSocket`
- `DeviceConnection` which receives the `BluetoothSocket` and manage the communication

## The Positives

One of the cool things about maintaining this, is that I've been able to help and see the library in action.  The following are some instances of this:

### High School Final Project

This was actually pretty cool!  These three kids emailed me out of the blue asking for help using the library for one of their school projects.  This was the start to an email I received from Ben asking for assistance.

```txt
Hi Mr. Davidson,

I'm Ben <redacted>, and I'm a high school student in New Jersey. My team and I are taking an engineering design course this semester, and for our major project we are building a device to help visually-impaired people cross the street. We're doing this with a device rigged with sensors that will send a vibratory alert to a user's phone with a certain app. The device is an Arduino Uno with an HC-05 Bluetooth classic module.
```

After a couple weeks of communication we were able to:

- Clean up the application code
- Get some more features working

They eventually got their application working on a **Nintendo Switch** which is crazy.  I hope they got an A!

**Pending confirmation on attaching their images/screenshots**

### Rock Wall Climbing

I haven't reached out to **Zlatanius** yet, but I came across his use of the project, and it looks awesome! [https://github.com/Zlatanius/LED-Rock-Climbing-Wall-RN-project](https://github.com/Zlatanius/LED-Rock-Climbing-Wall-RN-project).  

When I get time I'd like to take a deeper dive into how he's using the library.

### Any Others?

If you've got a project using this library and would like to chat (or let me write about it) please reach out.  Getting to work with the positives every once in a while (the people that is) definitely makes the negatives a little less annoying.

## The Pain Points!!

... although being in the industry for a while I should have expected.

I'm the first to admit that I'm no where near the **top** level of developers in the world.  But I do my best to read and learn and figure things out on my own, I think that's the most important thing.  It continues to amaze me how many people just expect things to work without:

- Reading documentation - which I've done my best to maintain as well as the library itself.
- Attempting to debug things on their own
- Open issues with absolutely no information what so-ever
- Time expectations

TLDR; just me whining about things, [feel free to skip](#v2.60.x-roadmap)

### Not Reading Documentation

I have to admit, I've been guilty of this as well, but when someone opens an issue stating that something doesn't work or isn't working as it should when the documentation clearly says WHY it's not working that way is painful.

I'd have to say the top subject for this is the MFi protocols from Apple, people seem to just completely ignore this documentation.  I appreciate that it's a lot to read, I do, but it's a requirement of the job!

### Not Debugging First

Bluetooth is a fickle beast.  That combined with React Native makes this library very susceptible to bugs when not used exactly how it was intended.   The expecation that I have the ability to debug the library with every single Android device and version is mind boggling!  What's even more crazy is that after an issue is opened and there's a response asking for more details - there is no response.

### Poorly Openned Issues

I do my best to provide as much information as possible; with regards to bugs, reasoning, etc. and it's painful to see issues getting opened with a line of text saying "<this> just doesn't work".   What I would like to do (and this is super petty of me I know) is to be able to reverse open source some of these things.  
  
Take the anonimity right out of the process!!

I've opened myself up by sharing my name and face and work with the world.  I get that there are negatives that come along with the positives.  What I'd love to do is be able to go back to someones work and post about something they've done.

> Hey John Doe's employer!  This is how your employee is using (and requesting use) of an open source library.  I hope he does a better job filling out internal tickets or providing information to clients.

Yes, I know I'm jaded at times.

### Time Expectations

I work 9-5 and have a 4 year old son and a wife doing her MBA.  As much as I would love to have all the time in the world it's just not possible.  I do my best to answer questions but I am starting to rely on the *kindness of strangers* a little more with regards to this library.  I've had some requests from people that just don't get that this is a side project - and expect the world from it (and me).

## v2.60.x Roadmap

At a super high level, the next round of development will include:

- Cleaning up the abstract logic
- Moving the Android specific connection logic to `Services` to allow for background communication.  Sadly this isn't doable on IOS, but at this point IOS seems to be falling behind anyhow.  
- Look into what is available in `CoreBluetooth` on IOS with regards to classic.  I don't think this provides `RFCOMM` support, but I think it does provide `L2CAP` support.
