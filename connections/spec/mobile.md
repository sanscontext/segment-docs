---
title: 'Native Mobile Spec'
sidebar: Mobile
---

One of the core components of the Segment [Spec](/docs/spec/) is the [`track`](/docs/spec/track) method. It records any arbitrary event that the user has triggered. For Mobile tracking, in addition to `screen` calls, you'll want to send **specific event names** that we recognize semantically. That way, we can transform them correctly before sending them off to downstream destinations.

By standardizing the events that comprise the core **mobile application lifecycle** and associated **mobile campaign and referral events**, Segment and our partners can, wherever possible, automatically collect and forward these events on your behalf and build downstream destinations which take full advantage of the semantic meaning associated with these events and their properties.

**Note:** If you're already collecting similar events, we recommend migrating to these event names so that you can take advantage of available features in our destinations which depend on the spec as they become available.

These events pair nicely with our [ecommerce spec](/docs/spec/ecommerce/v2/) for mobile marketplaces to take full advantage of features like dynamic ads in Facebook and the ability to take full advantage of server-side destinations with Mobile Attribution Platforms like Tune and Kochava.

**Note** Per our [Privacy Policy](https://segment.com/docs/legal/privacy/#sensitive-personal-information) and applicable terms, please don't send us sensitive personal information about your users. Certain features from Segment and our partners allow you to opt-in to automatically track data (for example: Application Installed or Deep Link Clicked). When working with these features and Segment in general, be cognizant of the data that is being tracked to ensure its matching both your obligations under your agreement with Segment and the privacy expectations of your users.

## Overview of Events

The Segment Native Mobile Spec includes the following semantic events:

**Application Lifecycle Events**
- [Application Installed](#application-installed)
- [Application Opened](#application-opened)
- [Application Updated](#application-updated)
- [Application Backgrounded](#application-backgrounded)
- [Application Crashed](#application-crashed)
- [Application Uninstalled](#application-uninstalled)

**Campaign Events**
- [Push Notification Received](#push-notification-received)
- [Push Notification Tapped](#push-notification-tapped)
- [Push Notification Bounced](#push-notification-bounced)
- [Install Attributed](#install-attributed)
- [Deep Link Clicked](#deep-link-clicked)
- [Deep Link Opened](#deep-link-opened)


 We recommend using the above event names if you're going to be integrating the events yourself. This will ensure that they can be mapped effectively in downstream tools.

Additionally, though they're not formally part of the Native Mobile Spec, we also collect `Order Completed` from our ecommerce spec automatically upon in-app purchases on iOS and can collect screen views automatically in iOS and Android.

## Lifecycle Events

Mobile applications live within a fairly bounded lifecycle. In order to understand and communicate effectively with your users, it’s crucial to instrument the core flows associated with installing and opening your app. The following events, many of which we can capture automatically in the latest versions of our SDKs, allow you to get a picture of top-line metrics like DAUs, MAUs, Screen Views per session, etc. Automatic tracking of lifecycle events is completely optional - you can learn how to enable and disable them in our [iOS](https://segment.com/docs/sources/mobile/ios/#step-1-install-the-sdk) and [Android](https://segment.com/docs/sources/mobile/android/quickstart/#step-2-initialize-the-client) library docs.

The following events will be tracked automatically when lifecycle events are enabled:

- [Application Installed](#application-installed)
- [Application Opened](#application-opened)
- [Application Updated](#application-updated)

### Application Installed

This event fires when a user **first** opens your mobile application. Note, if the user never opens your app after installing, we will not be able to collect this event. This event does not wait for attribution or campaign information to be received, and is collected automatically by our SDKs. Advertising providers like Facebook and Google require discrete install events to correctly attribute installs to ads served through their platform.

{% comment %}\{\{\{api-example '{ "userId": "019mr8mf4r", "action": "track", "event": "Application Installed", "properties": { "version": "1.2.3", "build": 1234 }}' }}}{% endcomment %}

**Property** | **Type** | **Description**
---          | ---      | ---
`version`    | String   | The version installed.
`build`      | Number   | The build number of the installed app.


### Application Opened

This event fires when a user launches or foregrounds your mobile application after the first open. It will fire after the `Application Installed` event and again after the app is re-opened after being closed. This event does not wait for attribution information to be received but may include information about referring applications or deep link URLs if available to the application upon open.

{% comment %}\{\{\{api-example '{"userId": "019mr8mf4r", "action": "track", "event": "Application Opened", "properties": { "from_background": false, "referring_application": "GMail", "url": "url://location" }}' }}}{% endcomment %}

 **Property** | **Type** | **Description**
 ---          | ---      | ---
`from_background` | Boolean | If application [transitioned](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIApplicationDelegate_Protocol/#//apple_ref/doc/uid/TP40006786-CH3-SW52) from "Background" to "Inactive" state prior to foregrounding (as opposed to from "Not Running" state). **Collected on iOS only**.
`url` | String | The value of `UIApplicationLaunchOptionsURLKey` from `launchOptions`.**Collected on iOS only**.
`referring_application` | String | The value of `UIApplicationLaunchOptionsSourceApplicationKey` from `launchOptions`. **Automatically collected on iOS only**.
`version`    | String   | The version installed.
`build`      | Number   | The build number of the installed app.

**NOTE: When an App returns from the background this event will fire on iOS only. Apps returning from the background are not currently supported on Android as Application Opened events.**.

### Application Backgrounded

This event should be sent when a user backgrounds the application upon [`applicationDidEnterBackground`](https://developer.apple.com/reference/uikit/uiapplicationdelegate/1622997-applicationdidenterbackground)

{% comment %}\{\{\{api-example '{ "userId": "019mr8mf4r", "action": "track", "event": "Application Backgrounded", "properties": {}}' }}}{% endcomment %}

### Application Updated

This event fires when a user updates the application. Our SDK will automatically collect this event in lieu of an "Application Opened" event when we determine that the Open is first since an update.

{% comment %}\{\{\{api-example '{ "userId": "019mr8mf4r", "action": "track", "event": "Application Updated", "properties": { "previous_version": "1.1.2", "previous_build": 1234,  "version": "1.2.0", "build": 1456 }}' }}}{% endcomment %}

**Property** | **Type** | **Description**
---          | ---      | ---
`previous_version` | String | The previously recorded version.
`previous_build` | Number | The previously recorded build.
`version` | String | The new version.
`build` | Number | The new build.

### Application Uninstalled

Fire this event when a user uninstalls the application. Several destination partners will detect this for you using Silent Push Notifications and send this event to Segment on your behalf.

{% comment %}\{\{\{api-example '{ "userId": "019mr8mf4r", "action": "track", "event": "Application Uninstalled", "properties": {}}' }}}{% endcomment %}


### Application Crashed

You can send this event when you receive a crash notification from your app, but is not meant to supplant traditional crash reporting tools. By tracking crashes as an analytics event with device and user information, you can analyze the which types of users are impacted by crashes and how those crashes, in turn, affect their engagement. You may also want to target those customers with tailored communications in other channels if they've encountered several crashes.

{% comment %}\{\{\{api-example '{ "userId": "019mr8mf4r", "action": "track", "event": "Application Crashed", "properties": {}}' }}}{% endcomment %}

## Campaign Events

As the walls between apps become increasingly lowered, capturing information about the content and campaigns that drive users to engage with your app is critical to building more targeted, relevant, personalized experiences for your users.

### Install Attributed

When Segment or an integrated partner can discern the source of an install, we'll collect an `Install Attributed` event. This event may be sent to Segment via server-to-server connection from your attribution provider, or directly on the device via packaged destinations. In either case, this will happen **after** install, and does not apply to all installs, which is why it is a discrete event.

{% comment %}\{\{\{api-example '{ "userId": "019mr8mf4r", "action": "track", "event": "Install Attributed", "properties": { "provider": "Tune/Kochava/Branch", "campaign": { "source": "Network/FB/AdWords/MoPub/Source", "name": "Campaign Name", "content": "Organic Content Title",  "ad_creative": "Red Hello World Ad", "ad_group": "Red Ones" }}}' }}}{% endcomment %}

**Property**            | **Type** | **Description**
---                     | ---      | ---
`provider`              | String   | The attribution provider.
`campaign[source]`      | String   | Campaign source — attributed ad network
`campaign[name]`        | String   | The name of the attributed campaign.
`campaign[medium]`      | String   | Identifies what type of link was used.
`campaign[content]`     | String   | The content of the campaign.
`campaign[ad_creative]` | String   | The ad creative name.
`campaign[ad_group]`    | String   | The ad group name.

### Push Notification Received

This event can be sent when a push notification is received in the app. It can be automatically enabled on [iOS](/sources/mobile/ios/#automatic-push-notification-tracking).

{% comment %}\{\{\{api-example '{ "userId": "019mr8mf4r", "action": "track", "event": "Push Notification Received", "properties": { "campaign": { "medium": "Push", "source": "Vendor Name", "name": "Referral Flow", "content": "Your friend invited you to play a match."}}}' }}}{% endcomment %}

**Property**        | **Type** | **Description**
-------             | -----    | -----
`campaign[name]`    | String   | Campaign Name.
`campaign[medium]`  | String   | Identifies what type of link was used (Push Notification).
`campaign[content]` | String   | Push notification content content
`campaign[source]`  | String   | Designates the push provider. (Optional)

### Push Notification Tapped

This event can be sent when a user taps on a push notification associated with your app. It can be automatically enabled on [iOS](/sources/mobile/ios/#automatic-push-notification-tracking).

{% comment %}\{\{\{api-example '{ "userId": "019mr8mf4r", "action": "track", "event": "Push Notification Tapped", "properties": {"action": "Accept", "campaign": { "medium": "Push", "source": "Vendor Name", "name": "Referral Flow", "content": "Your friend invited you to play a match." }}}' }}}{% endcomment %}

**Property**        | **Type** | **Description**
-------             | -----    | -----
`action`            | String   | If this notification is "[actionable](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/IPhoneOSClientImp.html#//apple_ref/doc/uid/TP40008194-CH103-SW26)", the custom action tapped. **Default:** "Open"
`campaign[name]`    | String   | Campaign Name.
`campaign[medium]`  | String   | Identifies what type of link was used (Push Notification).
`campaign[content]` | String   | Push notification content content
`campaign[source]`  | String   | Designates the push provider. (Optional)

### Push Notification Bounced

This event fires when a push notification from a provider bounces. If your push notification provider forwards push lifecycle events to Segment, they should include this event in their suite.

{% comment %}\{\{\{api-example '{ "userId": "019mr8mf4r", "action": "track", "event":"Push Notification Bounced", "properties": { "action": "Accept", "campaign": { "medium": "Push", "source": "Vendor Name", "name": "Referral Flow", "content": "Your friend invited you to play a match." }}}' }}}{% endcomment %}

**Property**        | **Type** | **Description**
-------             | -----    | -----
`action`            | String   | If this notification is "[actionable](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/IPhoneOSClientImp.html#//apple_ref/doc/uid/TP40008194-CH103-SW26)", the custom action tapped. **Default:** "Open"
`campaign[name]`    | String   | Campaign Name.
`campaign[medium]`  | String   | Identifies what type of link was used (Push Notification).
`campaign[content]` | String   | Push notification content content
`campaign[source]`  | String   | Designates the push provider. (Optional)

### Deep Link Clicked

This event may be provided by deep link providers or an internal redirect service if you use one in order to provide a waypoint funnel step between your content or advertisement and the resulting app open. If you're using an integration that supports calling these events, they can be enabled on [iOS](http://localhost:8000/docs/sources/mobile/ios/#automatic-deep-link-tracking).

{% comment %}\{\{\{api-example '{"userId": "019mr8mf4r", "action": "track", "event": "Deep Link Clicked", "properties": {"provider": "Branch Metrics", "url": "brnch.io/1234"}}' }}}{% endcomment %}

**Property** | **Type** | **Description**
---          | ---      | ---
`provider`   | String   | The deep link provider.
`url`        | String   | The deep link URL clicked.

### Deep Link Opened

When your application is opened via a referring link, Segment or your packaged deep link partner will fire this event on your behalf. If the deep link has additional data associated with it, either passed through the third party service or as `annotations` in `launchOption`, you may want to include those values as properties here as well. This event is fired *in addition* to the associated `App Opened` event. If you're using an integration that supports calling these events, they can be enabled on [iOS](http://localhost:8000/docs/sources/mobile/ios/#automatic-deep-link-tracking).

{% comment %}\{\{\{api-example '{"userId": "019mr8mf4r", "action": "track", "event": "Deep Link Opened", "properties": {"provider": "Branch Metrics", "url": "app://landing" }}'}}} {% endcomment %}

**Property** | **Type** | **Description**
---          | ---      | ---
`provider`   | String   | The deep link provider.
`url`        | String   | The App URL opened.