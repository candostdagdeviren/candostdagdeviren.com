---
layout: single
title: "Application Lifecycle in iOS"
date: 2018-11-11
categories: [Swift, English, Software Development]
tags: [Apple, Application Lifecycle, Best Practices, Development, iOS, NS for iOS Devs, Software, Software Architecture, Software Development, swift]
header:
  overlay_image: assets/images/ios-application-lifecycle/cover.jpx
  overlay_filter: rgba(0, 0, 0, 0.44)
  caption: Photo by [Bryan Minear](https://unsplash.com/photos/lzN6-hZyzfU) on [Unsplash](https://unsplash.com/)
excerpt: "Every iOS developer needs to understand the possible states and lifecycle of an iOS application. Knowing when the state enables us to work behind the scenes."
read_time: true
share: true
related: true
---

## Why do we need to know

Firstly, we should respond properly to the launch of the app.  Some third party libraries need setup in different states of the app. Also, we should prepare the app to work on the foreground. We might need to provide different user experience. For instance, our app can require password whenever it goes to foreground. On the other hand, our app can upload image while the app is in the background like Dropbox does. Even we might want to reopen from where the user left after the termination of the app. So, each feature requires different implementations in different app states. So, it can be critical to know the states and the transitions while providing different and better user experience.

## What do we need to know

Without further ado, here is the brief explanation of the states:

**_Not Running_**: App is not launched or terminated by the system.

**_Background_**: App is in the background and still able to run code. But the duration of this state is determined by the OS and code execution can be interrupted any time. If the developer requests extra time to execute some code, the period can be longer. There are some situations in which the app can be launched directly into the background mode. For example, app can start in background mode if the app starts downloading some content when the remote push notification is received.

**_Inactive_**: It’s generally a transition state between active and background state to launch of the app. For example, if we receive a phone call when the app is in the foreground, it goes into the inactive state.

**_Active_**: App is in the active state when it is in the foreground and it’s able to receive events.

**_Suspended_**: When the app is in the background state, the system suspends it and the app cannot execute any code. There isn’t any notification for the suspending event. The system automatically suspends the app.

## How do we know when a state transition is happening

`UIApplication` is an object that represents our application. It sets up everything and starts the app running. `UIApplication` object notifies us through `UIApplicationDelegate`. While`UIApplication` object is controlling the communication between the system and objects of the app and managing the event loop, we write our custom code in `UIApplicationDelegate` object to deal with the launch of the app and transitions between states and for many more things.

So, let’s take a look at which methods are called in `UIApplicationDelegate` and when do we need to add code into each of them.

* `application(_ application:willFinishLaunchingWithOptions:)`

When `willFinishLaunchingWithOptions` called, we know that our app’s launch process has been started, main storyboard or nib file has been loaded but the app is still in inactive state. If our app is launched by a specific reason like a remote push notification or home screen shortcut, this information will be inside the `options`. The important thing to know here is that the system might call other `UIApplicationDelegate` methods in these cases. For instance, if the app is called to open a URL, the system calls `application(_:open:options:)` method. `options` provide necessary information for us to react to these events properly. The other great example is Twitter feed. We can see that whenever we terminate the Twitter iOS app and open again, it resumes from the where we left. It doesn’t scroll to top all the time. If we want to implement the same [state restoring feature](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/StrategiesforImplementingYourApp/StrategiesforImplementingYourApp.html#//apple_ref/doc/uid/TP40007072-CH5-SW2), one of the required steps is calling the `makeKeyAndVisible()` method of `UIWindow` in `willFinishLaunchingWithOptions` instead calling it in `application(_ application:didFinishLaunchingWithOptions:)` method. `makeKeyAndVisible` method’s job is showing the window.  But still when we call it, it will not make the window visible instantly. UIKit waits for `application(_ application:didFinishLaunchingWithOptions:)` to finish first.

* `application(_ application:didFinishLaunchingWithOptions:)`

This method is called when the launch process is almost done and the app is almost ready. So, we have a space to do some final tweaks in the launch process. The difference with the previous one is, this time the state is changed, either foreground or background. But the app’s window or any UI is not presented yet.

When the app is launched via a URL, the system uses return values from these two methods and combines them to decide if the URL should be handled or not. If both of them returns `false`, the URL won’t be handled.

* `applicationDidBecomeActive(_ application:)`

This method is called when the app is transitioned from inactive state to active state. What is important here is the app also posts a `didBecomeActiveNotification` to notify listeners. For instance, if we have a timer running to measure how much the user spent in a screen, we might restart the timer when this notification is posted. Also, we can resume any other tasks which are paused when the app becomes inactive. (For example, when a phone call comes)

* `applicationWillResignActive(_ application:)`

This method is called when the app is about to transition from active to inactive state. So, similar to the previous method, we might need to pause any task or timers. For example, if we’re developing a game, we should pause the game when this method is called. In addition, the app will post `willResignActiveNotification` to inform listeners.

* `applicationDidEnterBackground(_ application:)`

This is one of the important methods if we want to execute some code on the background. [Apple says](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1622997-applicationdidenterbackground#discussion) that we have approximately five seconds to complete this method. After that, the app will be terminated and purged from the memory. If we need additional time to execute background code, we need to call `beginBackgroundTask(expirationHandler:)`. The crucial thing in here is, before this method exits, we need to finish up UI adjustments. For instance, the system takes a picture of the app when applicationDidEnterBackground returned. So, if we have a sensitive data in the screen, we need to hide or modify those views before the method returns. So, if our app is both trying to execute background code and update the UI, we should call `beginBackgroundTask(expirationHandler:)` first to request more time and then run these tasks on a dispatch queue or another thread. Thus, our app both do the UI adjustments and run background task. Last but not least, the app also posts a `didEnterBackgroundNotification` almost at the same time to inform listeners.

* `applicationWillEnterForeground(_ application:)`

This method is called when the app is about to enter the foreground. This method is always followed by the `applicationDidBecomeActive(_ application)` method.

* `applicationWillTerminate(_ application:)`

This method is called when the app is about to be terminated. We have approximately five seconds to perform any task and return. After five seconds, the system might kill the whole process. The essential thing to know in here is that when the user quits the app this method generally not called. If the app is running a background task and the system wants to terminate the app, this method may be called. As we see, there is no certainty in this method, sadly. Therefore, we shouldn’t perform any critical task in here like saving user data.

There is one more thing to mention about the app launch. When the app is launched, we need to prepare the UI as quickly as possible. We’ll talk about the threads and concurrency later. But, one thing to know is, every UI operation runs on the main thread. While implementing the above-mentioned methods, we need to run non-UI code asynchronously. So, we won’t block the main thread.

## Last Words

We took a look at the fundamentals and essentials of app lifecycle. I hope, it gives a brief understanding of the application lifecycle and state transitions. We might need to adopt our apps to state transitions to provide better user experience. Understanding the fundamentals of the states and transitions enables us to develop more robust apps.

Which strategies do you follow while running a background task? What do you think about the app lifecycle? Should Apple write more delegate methods or do you find some workarounds to make your wishes come true? Let me know your thoughts, comments or feedback on [Twitter @candostEN](https://twitter.com/candosten).

----
Further:

* [Graphical Representation of State Transitions](https://qph.fs.quoracdn.net/main-qimg-473264d5d9f0ec16d57b8dffcc9824d8)
* [Background Execution Programming Guide](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/BackgroundExecution/BackgroundExecution.html#//apple_ref/doc/uid/TP40007072-CH4-SW1)
* [Concurrency Programming Guide](https://developer.apple.com/library/archive/documentation/General/Conceptual/ConcurrencyProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008091)
