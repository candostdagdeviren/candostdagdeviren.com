---
layout: single
title: "Using Firebase Cloud Messaging for Remote Notifications in iOS"
date: 2016-11-05
categories: [Tools, English]
tags: [APNS, Apple, automate, Firebase, how to, iOS, Software, Software Development, swift, swift-3, swift3]
header:
  overlay_image: assets/images/using-firebase-cloud-messaging-for-remote-notifications-in-ios/cover.png
  overlay_filter: rgba(0, 0, 0, 0.44)
excerpt: "Firebase gives you the tools and infrastructure you need to build better apps and grow successful businesses. It provides several features divided into three groups develop, grow and earn."
read_time: true
share: true
related: true
---

Let’s talk about Firebase a little bit first.

In Firebase website, it says “Firebase gives you the tools and infrastructure you need to build better apps and grow successful businesses.”. It provides several features divided into three groups **develop**, **grow** and **earn** and all of them connected to Analytics. It doesn’t matter which feature you are using, you’ll get Analytics free. Features inside Grow and Earn groups are completely free.Actually, only the 4 of 15 features in total are paid. You can check them out [here](https://firebase.google.com/features/).

{:refdef: style="text-align: center;"}
![using-firebase-cloud-messaging-for-remote-notifications-in-ios-1]({{ site.baseurl }}/assets/images/using-firebase-cloud-messaging-for-remote-notifications-in-ios/1.png){:height="555px" width="888px"}
{: refdef}
<div class="image-caption">Firebase features</div>

## Firebase Cloud Messaging (FCM)

Cloud Messaging is one of the free cool features of Firebase. It’s easy to fire push notifications from the server to FCM and it handles the rest for iOS, Android, and the Web. I will skip the setup in this post. You can follow thewell-writtenn [setup guide](https://firebase.google.com/docs/cloud-messaging/ios/client) and add Firebase to your project.

{:refdef: style="text-align: center;"}
![using-firebase-cloud-messaging-for-remote-notifications-in-ios-2]({{ site.baseurl }}/assets/images/using-firebase-cloud-messaging-for-remote-notifications-in-ios/2.png)
{: refdef}

You can connect to Firebase Cloud Messaging via Firebase Console or app server. Firebase Console is pretty easy but it’s manual job. I’ll cover the app server side in this post. There are **three** ways to send notification via Firebase. You can send notifications to **one device** or **device groups** or **topics**. I would like to send notifications to all devices of specific user and I don't want to store tokens. I'll skip to send notification to only one device way in here. Also device groups are used for grouping one user's all devices. By this way, you can send notifications to all devices of one user. So, both of them requires storing tokens. Firebase suggests to use device groups, if you need to send messages to multiple devices per user. But as I said I don't want to store tokens. So I'm going to focus on only topics in this post. If you prefer storing tokens and dealing with them, you can use other two ways.

What is **topic**? Topic is based on publish/subscribe model. Each device can subscribe different topics. Server sends a request to topics. FCM handles rest and all subscribed devices get notification. Such an easy and powerful way. But as every service, it has pros and cons. Topics are not optimized for fast delivery. So there can be latency between sending notifications and receiving them. Topics are optimized for throughput.

## Real Life Scenarios

We have a microservice that creates push payloads when it received an event. Our lovely pusher has no idea about client devices and doesn’t have any storage. When new event occurred on underlying systems, it gets notified. Then it sends notifications to mobile platforms.

Our pusher sends an HTPP request to FCM. You can learn more about server implementation of FCM in [here](https://firebase.google.com/docs/cloud-messaging/server). Our basic request structure is like this:

```bash
https://fcm.googleapis.com/fcm/send
Content-Type:application/json
Authorization:key=SERVER_API_KEY

{
  "condition": "'condition1' in topics &amp;&amp; 'condition2' in topics",
  "notification": {
      "category": "notification_category",
      "title_loc_key": "notification_title",
      "body_loc_key": "notification_body",
      "badge": 1
  },
  "data": {
    "data_type": "notification_data_type",
    "data_id": "111111",
    "data_detail": "FOO",
    "data_detail_body": "BAR"
  }
}
```

This request sends proper structure to APNs via FCM. You can learn more about notification payload keys and their value types more from [here](https://firebase.google.com/docs/cloud-messaging/http-server-ref#notification-payload-support). Also you can put your extra information to process in your app to “data” section. In the request, you’ll see **condition** key. It’s used to send push notification to **topics**. Every condition is topic in here. This request sends notifications to devices which subscribed to *condition1* and *condition2*. We can use different conditions in there. For instance, we can use **`||`** instead of **&&**. In that case, FCM sends notification to devices which subscribed to *condition1* or *condition2*. It's possible to combine topics and create complex conditions like *condition1 && (condition2 `||` condition3)*. Only limitation is only one operator is allowed in high level. It's not possible to create a conditions like *condition1 && condition1 && condition3*, it has to be grouped.

We have enough information about server and FCM. Let’s move on the iOS side. (I won't get into the notification handling. It's completely another topic.)

FCM makes everything so easier in iOS side. There are couple things to do. After creating your APNs certificates and completing necessary steps in setup guide, you should configure your app for FCM. To do that, add this line `FIRApp.configure()` in your `AppDelegate`'s `application:didFinishLaunchingWithOptions:` method. This method configures your app using `GoogleService-Info.plist` file.

Firebase uses method swizzling to map APNs token to FCM registration token. Also for capturing analytics about delivery information of messages. I prefer disabling swizzling and handling these steps by myself. To disable swizzling, you need to set `FirebaseAppDelegateProxyEnabled` key to `NO` in your application `Info.plist` file. From now on, we are responsible for APNs key mapping and analytics data. Let's map APNs token to FCM token first. Here's how to do it:

```swift
func application(_ application: UIApplication,
                 didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
  #if PROD_BUILD
    FIRInstanceID.instanceID().setAPNSToken(deviceToken, type: .prod)
  #else
    FIRInstanceID.instanceID().setAPNSToken(deviceToken, type: .sandbox)
  #endif
}
```

Important thing in here is `FIRInstanceIDAPNSTokenType`. There are three types available `.prod`, `.sandbox` and `.none`. To get notification, one of the Production or Sanbox token types must be used.

Last thing we have to do without method swizzling is sending message analytics to FCM. So, we'll be able to use Analytics from Firebase Console. Here's how to do it:

```swift
func application(_ application: UIApplication,
        didReceiveRemoteNotification userInfo: [AnyHashable : Any],
                 fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Swift.Void) {
    FIRMessaging.messaging().appDidReceiveMessage(userInfo)
}
```

We setup FCM, mapped our token to FCM token and sent message analytics. Last thing remaining is subscribing to topics. To subscribe a new topic is really easy. It's just one line:

```swift
FIRMessaging.messaging().subscribe(toTopic: "/topics/condition1")
// Don't forget to add /topics/ prefix while subscribing
```

If there is no topic in FCM, it creates a new topic automatically, when you've subscribed. As you remember we talked about combining topics in our HTTP request's **condition** key. Also in the app side, we can subscribe **multiple topics** to get more notifications.

Let's get back to our case. We would like to send new notifications to specific user. To do that, we created topics using unique user ids. It sounds like misusing of topics but it was the only way in our case. Each user subscribes to topic named something like "userID_11111". And in our lovely pusher we used user ids in condition.

Most of the apps has user login and logout cases. When user logged out, you don't want to send them notifications. To cover these cases, you can unsubscribe from topics. FCM removes that device token from topic list (Topic always will be there. You cannot delete topics). Unsubscribing is similar to subscribing:

```swift
FIRMessaging.messaging().unsubscribe(fromTopic: "/topics/condition1")
// Don't forget to add /topics/ prefix while unsubscribing
```

We covered sending push notification and FCM topics until now. FCM is really powerful and easy. But like any other services, we faced with some problems.

## Problems

First problem is subscribing to new topic can take some time on FCM. If you don't get notification directly after subscribing, you should wait couple of minutes.

The other problem is handling push notification was different on Android side. If you want to create custom nice notifications, you should send different payload. In above HTPP request, we had **notification** and **data** keys. If Android app gets notification key, it has no control on it. It cannot customize notification and device displays default notification with given title and body. To create custom style notification, they need to use **data** key and create notification. When we faced with this solution, we came up with an idea of **iOS** and **android** topics. We created two more topics. iOS devices subscribe to **iOS** topic, Android devices subscribe to **Android** topic. Our lovely pusher sends two requests for each event. For Android, our **condition** key's value is *"'userID_1111' in topics && 'android' in topics"* and it has no **notification** key. For iOS, it is **`"'userID_1111' in topics && 'iOS' in topics"`**. Maybe it's not the perfect way to handle these cases. But it's working very well and we don't care device token handling. If you have better ideas or ways, please let me know.

The last problem is configuring sandbox and production types for Firebase. Most of the iOS apps doesn't have separate target for different release modes. For different configurations like Debug, Test and Release, you have to setup different apps in Firebase Console. Then, you'll have separate GoogleService-Info.plist for different configurations. Renaming them and configuring Firebase with these files won't work. Only solution I can find is running script. Put different .plist files for different Firebase apps under separate folders. You can name them like prod, beta, etc.. Add run following script before Compile Sources in Build Phases in your app target. (Don't forget to configure below script with your folder naming and path)

```bash
isRelease=`expr "$GCC_PREPROCESSOR_DEFINITIONS" : ".*release=\([0-9]*\)"`
RESOURCE_PATH=${SRCROOT}/firebase/beta
if [ $isRelease = 1 ]; then
  RESOURCE_PATH=${SRCROOT}/firebase/release
fi
BUILD_APP_DIR=${BUILT_PRODUCTS_DIR}/${PRODUCT_NAME}.app
echo "Copying all files under ${RESOURCE_PATH} to ${BUILD_APP_DIR}"
cp -v "${RESOURCE_PATH}/"* "${BUILD_APP_DIR}/"
```

This script takes the located config files (GoogleService-Info.plist) according to app build configuration. And duplicates it to the build app directory before compiling sources. Then, Firebase will identify .plist file as identical like there is only one file. You can configure script and copy only .plist files. Logic is the same.

This is it. Easy setup, easy configuration. Possibilities of using FCM with other features of Firebase gives a lot more advantages. Maybe this will be the subject of other posts.

Thanks for reading! Help spread the word ❤️🚀.

Do you have questions, suggestions, comments or ideas for upcoming blog posts contact me on [Twitter @candosten](http://bit.ly/2oWdga9) or write a comment!😍 You can also follow me on [GitHub](http://bit.ly/1S1gP9z).