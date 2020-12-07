---
type: Blog
categories: [Project]
title: Standardbred Mobile - Framework Selection Process - Part 1
description: Walking through my development of Standardbred Mobile App from design to release.
tags: [React Native, Android, IOS]
series: Designing My First React Native App
order: 1
---



My company recently released an Android and IOS app for our members to scan microchip data from their horses.  Our US counterparts released the same type of app a few months before us, through a development shop which specializes in applications specific to this particular microchip - our choice at the time was (a) use the same shop to create essentially the same app for us, for a cost; or (b) design and develop the app in house.   I want to walk through the full decision making process from start to end - for  my own personal reference - and to possibly help anyone facing the same questions.  Just a few notes to being:

My company and department are pretty small, so in most cases **in house** means me.
Specific details have been removed - cost, salary, etc. from the content - but they should always be included (as exact as possible).

## Review of Development Options

When starting the process out, as mentioned, there were two options (vendor or in house).   Each of these would obviously have pros and cons that needed to be weighed:

#### Cost
As a non profit this was probably one of the most important items.  The cost of the vendor application was a little more than what I would have expected.  With regard to doing it in house, it came down to be being willing to do it *for time in lieu* as there was no time during business hours.  Since it seemed like a cool thing to get under my belt, this was a pretty easy choice.

#### Timeline
We needed the app to be released within the month, this was to align with our hardware release (scanners) being sold to our membership.  Obviously the increased timeline caused the vendor price to go up, while my lieu time remained constant.

#### Maintenance
My manager loves running things in house, which made the maintenance of the app more important (than I had originally thought).  Future proofing, fixing bugs, etc in hours (or on my weekends) would also cut the costs that would have accumulated using the vendor.

#### Look & Feel
I'm not going to say I'm the greatest designer in the world (far from it) but I do think that I have an idea of what looks good, and more importantly what does not.  The base vendor app was (in my opinion) plain, it looked old and not really matching what the marketing team had in mind.  Again making changes to the look would up vendor costs - where as it would just eat into our designers and marketing time.

#### Results...

After all was said and done, the only major concerns for the in house development was:
Me getting lieu for a few days of doing something new (and interesting anyhow)
Me leaving sometime down the road and needing a new developer to take over (kind of moot since it would happen anyhow)
so the decision was made to go in house.

## Technology Options

This was my first (non experimental) app release - in the past I'd played around with Android development, but had never really touched IOS development (Objective C never appealed to me and I wasn't the biggest IOS fan anyhow, so I never had an opportunity).  There were other technologies I knew about, like Xamarin, Phone Gap and Titanium (which I had done some development in) which after re-visiting the subject, React Native and Flutter popped up.  It was time for another pros and cons!!

#### Native (Java / ObjectiveC / Swift)
Since the majority of my development is in Java, this wasn't a concern - the fact that I was never the biggest fan of ObjectiveC/Swift to begin with was just a negative I'd have to power through.   Swift always had the edge (with regards to learning new languages) as I just don't find Objective C an attractive language.

#### Xamarin
The first cross platform option was Xamarin.  From what I'd read it had a decent following was backed by Microsoft.  I have some experience working with C#, so the language wouldn't be an issue - just the framework.  The issue here, none of our other codebase is C#, so it would be a hard sell.

#### React Native
I'd read a bunch about React Native and it seemed interesting - again it had a huge base and backed by Facebook (even though I'm not the biggest Facebook fan).  It was Javascript, which we use in house and all the developers have experience with, so it would just come down to learning the framework.

#### Flutter
This was just something I found out while Googling around.  I wasn't a huge fan of learning a new language as well as a new framework all together, and I knew the rest of the team wasn't either.

*Note* - Other cross platform frameworks like Phone Gap and Titanium were options, but I'd used them in the past and just wasn't a fan of them.

#### Result
Since most of these included learning something new, as well as still being in the time crunch, it made sense to limit the learning curve and pick something that had longer term gains.  **React Native** seemed like the best option:

With a wide user base and resources it would take the sting out of issues with the framework
Facebook probably isn't going anywhere, at least for a while
I already work with JavaScript and I might run into Objective C or Swift down the road (this also made it easier to sell if I left, as everyone works with JavaScript in some fashion)
with those two decisions down, it was time to bring in other teams and get started.

## Designing the Application (UI)

I'll be honest, this is where I hand a lot of the work off to the marketing team - I think rightly so - as they had interest in making clients happy, making something that looked good and functioned well.  The first iteration of the app was only going to be two screens:
A login screen 
A horse lookup screen that allowed scanning
The design was pretty simple - although the iterations of images took a while (mainly due to IOS requires being what they are).

## Designing the Application (Core)

This was where things got a little tricky, re: the first time I've ever used React Native (let alone React) and I'm responsible for producing an app that both functions and looks good.   I'll be honest it was a little nerve wracking, coupled with the fact that I'm one of the most anxious people known to man kind (just ask my finger nails - ya gross I get it!).   Once I got passed the "what did I get myself into" it was time to start googling:

First thing first - best practices!!

This should probably be the first thing everyone looks at before starting a project they are going into blind (hell - sometimes you should just check back even if you've been working with something for a while to make sure there are no changes and you're still on track).  I knew that the app was eventually going to be more than just the two screens for this iteration, so I needed to ensure that:
The application could manage user login and sessions. 
The application could make network requests to specified services, with said sessions available.
The application would need to make **Bluetooth Connections** view **CLASSIC** with our vendor hardware.
After checking out a bunch of great posts:

- [https://medium.com/react-native-training/best-practices-for-creating-react-native-apps-part-1-66311c746df3](https://medium.com/react-native-training/best-practices-for-creating-react-native-apps-part-1-66311c746df3)
- [https://www.innofied.com/top-10-react-native-best-practices-to-follow/](https://www.innofied.com/top-10-react-native-best-practices-to-follow/)

Pretty much anything on Robin Wieruch's blog [https://www.robinwieruch.de/blog](https://www.robinwieruch.de/blog)
a common theme came up - **React Redux** (just as an FYI, although I'd worked with JavaScript it was not my primary language and I had just started using AngularJS a couple years ago on some very basic projects - so yes, a lot of this is new to me at the time).   Most of the posts said that Redux was overkill for such a small project, and it did seem like overkill at the start, but if the goal was to eventually build the app out it was important to have a base that was capable of growth, rather than having to refactor the current code just to start adding more features.  In my mind spending more time upfront is always a better option.

Next was how to get access to the services [typing clicks here] it seemed that the most popular method for making network request, keeping in mind the choice in Redux, was to use **Axios** as it provided the networking, the access to **Redux** stored sessions and a little more configuration that might be usable down the road. 

Finally we needed to communicate with the hardware through Bluetooth classic.  They key here is the the **Classic** - which Android supports majestically, but Apple (un-shockingly) has their on proprietary nonsense called **MFi** that requires vendors to jump through a bunch of hoops in designing hardware.  Which is pretty mind boggling since BLE is available without much issue - what's so specially about classic? Except Apple being Apple.   Back to the library hunt - there were a number of libraries accessing Bluetooth classic on Android but literally zero on IOS.  Was my decision to choose React Native a terrible one?  Initially it looked like it, but after continuing to look around it seemed like there were no real libraries for use on IOS to communicate with Bluetooth classic anyhow - thankfully this meant that no matter what choice I made, I was going to need to write my own communication library.

Thankfully our hardware vendors were super great with their knowledge and information which made the process easy.  Read more about my [react-native-bluetooth-classic](https://kenjdavidson.github.io/react-native-bluetooth-classic) project in the next few parts of this series.

Now that a few key decisions were made it was time to [get started](/2020-11-20-designing-my-first-app-part-2).  Check out the second part of this article to walk through the process of:

- getting my environment setup and running
- building the initial application
- getting to a place where I could start actually working on the app