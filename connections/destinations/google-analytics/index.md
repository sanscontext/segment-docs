
## Getting Started


When you toggle on Google Analytics in Segment, this is what happens:

  - Our CDN is updated within 5-10 minutes. Then our snippet will start asynchronously loading Google Analytics javascript library onto your web page. **This means you should remove Google's snippet from your page.**

  - Your Google Analytics real-time dashboard will start showing live concurrent visitors.

  - Any iOS and Android apps running our mobile libraries will start sending data to Google Analytics. New settings will take up to an hour to propagate to all of your existing users.Or if you just added the iOS or Android library to your app code, it'll be instantaneous!

  - Google Analytics will start automatically collecting data on your site or mobile app. It takes several hours for Google to process this data and add it to your reports, but you should still see events showing up in their real-time events dashboard.

Segment supports Google Analytics client-side and the server-side tracking, in addition to mobile app analytics for iOS and Android.

These docs will only cover GA Universal features, since the [Classic tracking method has been depreciated](http://analytics.blogspot.com/2014/04/universal-analytics-out-of-beta-into.html).

- - -


## Page & Screen

When you call [`page`](/docs/spec/page), we send a pageview to Google Analytics. Pageviews can be sent from the browser or through any of our server-side libraries.

The resulting `page` event name in Google Analytics will correspond to the `fullName` of your page event. `fullName` consists of a combination of the `category` and `name` parameters. For example, `analytics.page('Home');` would produce a `Home` page event in GA's dashboard, whereas `analytics.page('Retail Page', 'Home');` would produce an event called `Retail Page Home`.

Note that when sending `page` views from one of Segment's server-side libraries, a `url` property is required. Otherwise, Google Analytics will silently reject your `page` event.

When you call [`screen`](/docs/spec/screen) in your mobile app, we send a screen view to Google Analytics for mobile apps.

If you are sending a [`screen`](/docs/spec/screen) call server-side, you must pass in an [application name](https://developers.google.com/analytics/devguides/collection/analyticsjs/field-reference#appName) through Segment's `context.app.name` object or Google will reject your event.

If you've set an application name in your Android or iOS project, Segment will grab the name and pass `context.app.name` automatically. For iOS, Segment collects your project's `infoDictionary` and uses whatever name you've set there. You can see [Segment's iOS Library code in action](https://github.com/segmentio/analytics-ios/blob/760be85a5119c2e8bd31a745ce2ec30385a0ad69/Pod/Classes/Internal/SEGSegmentIntegration.m#L110), and you can read more about how to [set the display name for your iOS app](https://developer.apple.com/library/content/qa/qa1823/_index.html).


### Virtual Pageviews

Virtual pageviews are when you send a pageview to Google Analytics when the page URL didn't actually change. You can do this through Segment by simply calling [`page`](/docs/spec/page) with optional properties, like this:

```javascript
analytics.page({
  title: 'Signup Modal',
  url: 'https://segment.com/#signup',
  path: '/#signup',
  referrer: 'https://segment.com/'
});
```


### URL Query Strings

By default we will only send the domain and path to Google Analytics. For example, if someone lands on:

```
http://domain.com/page/?xyz=123&r=5
```

Segment will send this as the URL to Google Analytics:

```
http://domain.com/page/
```

In some cases, like using Google Analytics to track search queries, you may want to pass the whole URL with query string to Google Analytics. To make that happen check the **Include the Query String in Pageviews** box under Advanced Options in the Google Analytics sheet on the Segment destinations catalog.


## Identify

It is against Google's terms of service to pass Personally Identifiable Information (PII) to your Google Analytics reporting interface. For that reason Segment will never pass anything from an [`identify`](/docs/spec/identify) call to Google unless you specifically tell us to. You can read about Google's best practices for avoiding this [here](https://support.google.com/analytics/answer/6366371?hl=en).


### User ID

Google Analytics Universal tracking method allows you to set a user ID for your identified visitors. [Read more here](https://support.google.com/analytics/answer/3123663).

To use this feature you must enable User-ID in your Google Analytics property and create a User-ID view, [read more here](https://support.google.com/analytics/answer/3123666).

If you want to pass the `id` from your [`identify`](/docs/spec/identify) calls to Google Analytics - enable **Send User-ID to GA** in your Advanced Google Analytics settings on the Segment destinations catalog.

Here's an example:

```javascript
analytics.identify('12345', {
  email: 'example@example.com',
  name: 'Jake Peterson'
});
```

In this example we will set the `User-ID` to `12345` for Google Analytics, but we won't share the `email` or `name` traits with Google.

If you are passing an **email**, **phone number**, **full name** or other PII as the `id` in [`identify`](/docs/spec/identify) do not use this feature. That is against the Google Analytics terms of service and your account could be suspended.


### Custom Dimensions

Google Analytics has multiple scopes for each custom dimensions: hit (synonymous with events), session, user, product (required enhanced ecommerce to be enabled).

Our client-side analytics.js library supports both hit and product scoped custom dimensions.
Our mobile libraries (Android and iOS) support only hit scoped custom dimensions.

We do not currently support session or user scoped custom dimenstions.

Configuring Custom Dimensions:
First, configure the Custom Dimensions in your Google Analytics admin page. [Read how to set those up here](https://support.google.com/analytics/answer/2709829?hl=en).

Once you are set up in Google Analytics, you are ready to map traits and properties to your custom dimensions.
From your Segment Dashboard, open the destinations catalog and select the Google Analytics destination, then Advanced Options. Locate Custom Dimensions and declare the mapping.

Here's an example of mapping "Gender" to "dimension9" and "User Type" to "dimension17":

![custom dimension mapping screenshot](images/dimension-mapping.png)

**Note:** A particular trait or property may only be mapped to a single Custom Dimension at a time.

Once all your dimensions have been mapped, we will check user traits and properties in [`identify`](/docs/spec/identify), [`track`](/docs/spec/track) and [`page`](/docs/spec/page) calls to see if they are defined as a dimension. If they are defined in your mapping, we will send that dimension to Google Analytics.

**Note:** Traits in [`Identify`](/docs/spec/identify) calls that map to Custom Dimensions will only be recorded to Google Analytics when the next [`track`](/docs/spec/track) or [`page`](/docs/spec/page) call is fired from the browser.

Continuing the example above, we can set the **Gender** trait with the value of **Male**, which maps to `dimension9`, and it will be passed to Google Analytics **when we make the 'Viewed History' [`track`](/docs/spec/track) call**.

```javascript
analytics.identify({
  Gender: 'Male'
});
```
```javascript
analytics.track('Viewed History');
```
### Server side Identify

If you are sending `.identify()` calls from your server side libraries or have Segment Cloud Apps that send back `.identify()` calls with enriched user traits, you can send that data to your GA account via custom dimensions and metrics. Unlike the client side destination which has the luxury of browsers and the global window `ga` tracker, for server side we will check your `traits` and your settings for custom dimension/metric mappings and send it with an explicit event.

You can specify in the setting what you want this event action to be named. We will fallback to a default of **'User Enriched'**. Since event category is also required, you can specify which `trait` you want us to set this value as. For example, if you send a trait such as `type`, we will set the value of `traits.type` as the event category if defined and otherwise, we will fallback to **'All'**.

**NOTE**: We will always mark this event as a **Non Interaction** hit. This is also **only** available if you are using Universal GA.

### A/B Test Versions to Dimensions

Segment makes it simple to save your A/B testing versions to custom dimensions in Google Analytics. All you have to do is map an experiment to a custom dimension in the Google Analytics Advanced Options inside Segment.

If you are using Cloud Mode or server-side Google Analytics destinations, you can also send this data automatically using our `experiment_id`, `experiment_name`, `variation_id`, and `variation_name` properties. If both an experiment and variation are defined, then this will send automatically. Note that we will use the ids over the names, if both exist (for example, sending experiment_id, experiment_name, and variation_name in a call will ultimately send experiment_id and variation_name).

When you have an active A/B test on a page, Segment will either set that experiment as a property or a user trait, depending on how you opt to send experiment data to other tools on your A/B testing tool's Segment settings page. The property or trait for A/B test experiments are labeled like this:

```javascript
'Experiment: EXPERIMENT_NAME': 'EXPERIMENT_VARIATION'
```

For example, if you have an experiment called **Home CTA** and a visitor sees a variation called **Create free account now**, Segment sets the following property or trait:

```javascript
'Experiment: Home CTA': 'Create free account now'
```

If you want to record that property or trait as a custom dimension you'd map **Experiment: Home CTA** to a custom dimension, like this:

![a b test custom dimension mapping screenshot](images/ab-mapping.png)

*Remember: You'll need to setup dimension13 inside of your Google Analytics Admin first as described at the top of this docs section.*

## Track

We'll record a Google Analytics event whenever you make a [`track`](/docs/spec/track) call. You can see your events inside Google Analytics under **Behavior** -> **Events** -> **Overview**. Keep reading for more details about the Google Analytics event category, action, label, value and how to populate them.

Events can be sent from the browser, your server, or our mobile SDKs. Here's a basic [`track`](/docs/spec/track) example:

```javascript
analytics.track('Logged In');
```

For this example these event attributes are sent to Google Analytics:

<table>
  <tr>
    <td>**Event Category**</td>
    <td>All</td>
  </tr>
  <tr>
    <td>**Event Action**</td>
    <td>Logged In</td>
  </tr>
</table>

And another [`track`](/docs/spec/track/) example, this time with all Google Analytics event parameters:

{% comment %}\{\{\{api-example '{
  "userId": "12345",
  "action": "track",
  "event": "Logged In",
  "properties": {
    "category": "Account",
    "label": "Premium",
    "value": 50
  }
}'{% endcomment %}

That call will create a Google Analytics event with these attributes:

<table>
  <tr>
    <td>**Event Category**</td>
    <td>Account</td>
  </tr>
  <tr>
    <td>**Event Action**</td>
    <td>Logged In</td>
  </tr>
  <tr>
    <td>**Event Label**</td>
    <td>Premium</td>
  </tr>
  <tr>
    <td>**Event Value**</td>
    <td>50</td>
  </tr>
</table>

For **Event Value** you can name the event property `value` or `revenue`. We recommend using `value` for client-side tracking and `revenue` for more accurate server-side revenue tracking. Calling it `revenue` is best if the event made you money directly. That way we can also pass the revenue data to other destinations you have enabled.


### Non-interaction Events

To create an event with the `nonInteraction` flag just pass us an event property labeled `nonInteraction` with the value of 1. You can also set all events to be non-interactive by default in the Advanced Options.

Here's an example:

{% comment %}\{\{\{api-example '{
  "action": "track",
  "event": "Viewed Legal Info",
  "properties": {
    "nonInteraction": 1
  }
}'{% endcomment %}

- - -


## E-Commerce

Segment supports Google Analytics basic e-commerce tracking across all our libraries. All you have to do is adhere to our [e-commerce tracking API](/docs/spec/ecommerce/v2/) and we'll record the appropriate data to Google Analytics.


### Required Steps

All of our [e-commerce events](/docs/spec/ecommerce/v2/) are recommended, but not required. The only required event is `Order Completed`. For each order completed you must include an `orderId`, and for each product inside that order, you must include an `id` and `name` for each product. **All other properties are optional**.

The most important thing to remember in Google's Universal Analytics is to enable e-commerce tracking for the view you want to track transactions to. This can be done inside of Google Analytics by clicking:

**Admin > View Settings > Ecommerce Settings switch to ON**

Without this step transactions will not show up in your reports.


## Enhanced E-Commerce

Segment supports Google Analytics Enhanced E-Commerce tracking across both our client-side (analytics.js, android-analytics, ios-analytics) and server-side destinations. Enhanced Ecommerce allows you to derive insights by combining impression data, product data, promotion data, and action data. This is required for product-scoped custom dimensions.

To get started, you need only enable enhanced ecommerce and adhere to our standard [e-commerce tracking API](/docs/spec/ecommerce/v2/), and we'll record the data to Google Analytics with their enhanced ecommerce API.

### Required Steps (enhanced)

Similar to regular e-commerce, the only required event is `Order Completed`. This call also must include an `orderId` and an array of products, each containing an `id` or `name`.

For all events that include product details you must pass either `name` or `product_id`. For `product_id` we default to `properties.product_id` and fallback to `properties.sku`.

*Segment's Android SDK v2.0.0 does not support `properties.sku` since no mapping to this property is available in Google's latest SDK, so you must pass a product_id.*

**All other properties are optional**. The Refunded Order event also requires an `orderId`.

In order to see Enhanced E-Commerce data in your reports, you must be using Google Analytics Universal and enable Enhanced E-Commerce in your Google Analytics:

**Admin > View Settings > Enhanced Ecommerce Settings switch to ON**

Lastly, you have to enable Enhanced Ecommerce in the Google Analytics destination settings.

### Measuring Checkout Steps

To take full advantage of all the features of Enhanced E-commerce, you'll want to take advantage of some specific events. The biggest differentiator between e-commerce and enhanced e-commerce is support for checkout steps. To take advantage of tracking your checkout funnel and measuring metrics like cart abandonment, etc, you'll first need to configure your checkout funnel in the Google Analytics admin interface, giving easily readable labels to the numeric checkout steps:

![enhanced ecommerce checkout funnel](images/checkout-funnel.png)

Then you'll instrument your checkout flow with `Viewed Checkout Step` and `Completed Checkout Step` for each step of the funnel you configured in the Google Analytics admin interface, passing the step number and step-specific options through as a property of those events:

```js
//upon arrival at first checkout step ('Review Cart' per the screenshot example above)
analytics.track('Viewed Checkout Step', {
  step: 1
});

//upon completion of first checkout step ('Review Cart')
analytics.track('Completed Checkout Step', {
  step: 1
});

//upon arrival at second checkout step ('Collect Payment Info' per the screenshot example above)
analytics.track('Viewed Checkout Step', {
  step: 2
});

//upon completion of this checkout step ('Collect Payment Info')
analytics.track('Completed Checkout Step', {
  step: 2,
//if this is the shipping step
  shippingMethod: 'FedEx',
//if this is the payment step
  paymentMethod: 'Visa'
});

//upon arrival at third checkout step ('Confirm Purchase Details' per the screenshot example above)
analytics.track('Viewed Checkout Step', {
  step: 3
});

//upon completion of third checkout step ('Confirm Purchase Details')
analytics.track('Completed Checkout Step', {
  step: 3,
//you will need to provide either an empty shippingMethod or paymentMethod for the event to send.
  shippingMethod: '' // or paymentMethod: ''
});

//upon arrival at fourth checkout step ('Receipt' per the screenshot example above)
analytics.track('Viewed Checkout Step', {
  step: 4
});

//upon completion of fourth checkout step ('Receipt')
analytics.track('Completed Checkout Step', {
  step: 4,
//you will need to provide either an empty shippingMethod or paymentMethod for the event to send.
  shippingMethod: '' // or paymentMethod: ''
});
```

*Note*: `shippingMethod` and `paymentMethod` are semantic properties so if you want to send that information, please do so in this exact spelling!

You can have as many or as few steps in the checkout funnel as you'd like. The 4 steps above merely serve as an example. Note that you'll still need to track the `Order Completed` event per our standard [e-commerce tracking API](/docs/spec/ecommerce/v2/) after you've tracked the checkout steps.

For client-side integrations, to leverage the ability to track Checkout Steps and Options, we use Google Analytics' ProductAction class. You can read their developer docs for information on specific methods:
- [Android](https://developers.google.com/android/reference/com/google/android/gms/analytics/ecommerce/ProductAction)
- [iOS](https://developers.google.com/analytics/devguides/collection/ios/v3/reference/interface_g_a_i_ecommerce_product_action)
- [Analytics.js - Enhanced E-Commerce](https://developers.google.com/analytics/devguides/collection/analyticsjs/enhanced-ecommerce)
- [Analytics.js - E-Commerce](https://developers.google.com/analytics/devguides/collection/analyticsjs/ecommerce)

### Measuring Promotions

Enhanced Ecommerce allows you to go beyond measuring product performance to measure the internal and external marketing efforts that support those products. To take advantage of enhance e-commerce's promotion reports, you can easily collect data about promotion impressions and promotion clicks with Analytics.js, like so:

```js
analytics.track('Viewed Promotion', {
  id: <id>,
  name: <name>,
  creative: <creative>, // optional
  position: <position> // optional
});
```

```js
analytics.track('Clicked Promotion', {
  id: <id>,
  name: <name>,
  creative: <creative>, // optional
  position: <position> // optional
});
```

For client-side integrations, to leverage the ability to measure promotions, we use Google Analytics' Promotions class. You can read their developer docs for information on specific methods:
- [Android](https://developers.google.com/android/reference/com/google/android/gms/analytics/ecommerce/Promotion)
- [iOS](https://developers.google.com/analytics/devguides/collection/ios/v3/reference/interface_g_a_i_ecommerce_promotion)
- [Analytics.js - Enhanced E-Commerce](https://developers.google.com/analytics/devguides/collection/analyticsjs/enhanced-ecommerce)
- [Analytics.js - E-Commerce](https://developers.google.com/analytics/devguides/collection/analyticsjs/ecommerce)

### Coupons

If you want to send coupon data to your `Order Completed` event when using Enhanced E-commerce, you can simply add the `coupon` property on the order level or the product level or both. In the below example, note that our Google Analytics Ecommerce destination accepts `total` *or* `revenue`, but not both. We recommend using `revenue` because several other destinations use `revenue` too. For better flexibility and total control over tracking, we let you decide how to calculate how coupons and discounts are applied. For example:

```js
analytics.track({
  userId: '019mr8mf4r',
  event: 'Order Completed',
  properties: {
    orderId: '50314b8e9bcf000000000000',
    total: 27.5,
    shipping: 3,
    tax: 2,
    discount: 2.5,
    coupon: 'hasbros',
    currency: 'USD',
    repeat: true,
    products: [
      {
        id: '507f1f77bcf86cd799439011',
        sku: '45790-32',
        name: 'Monopoly: 3rd Edition',
        price: 19,
        quantity: 1,
        category: 'Games',
        coupon: '15%OFF'
      },
      {
        id: '505bd76785ebb509fc183733',
        sku: '46493-32',
        name: 'Uno Card Game',
        price: 3,
        quantity: 2,
        category: 'Games',
        coupon: '20%OFF'
      }
    ]
  }
});
```

### Measuring Product Impressions

Enhanced Ecommerce also allows you to collect impression information from users who have viewed or filtered through lists containing products. This allows you to collect information about which products have been viewed in a list, which filters/sorts have been applied to a list of search results, and the positions that each product had within that list.

Product impressions are mapped to our 'Product List Viewed' and 'Product List Filtered' analytics.js events. You can find more information about the parameters and requirements here in our [e-commerce tracking API](/docs/spec/ecommerce/v2/).

Analytics.js allows you to easily collect this data and send it forward, like such:

```js
analytics.track('Product List Viewed', {
  category: 'cat 1',
  list_id: '1234',
  products: [
    {
      product_id: '507f1f77bcf86cd799439011',
      sku: '45790-32',
      name: 'Monopoly: 3rd Edition',
      price: 19,
      category: 'Games'
    }
  ]
});
```

```js
analytics.track('Product List Filtered', {
  category: 'cat 1',
  list_id: '1234',
  filters: [
    {
      type: 'department',
      value: 'beauty'
    },
    {
      type: 'price',
      value: 'under'
    }],
  sorts:[ {
    type: 'price',
    value: 'desc'
  }],
  products: [
    {
      product_id: '507f1f77bcf86cd799439011',
      sku: '45790-32',
      name: 'Monopoly: 3rd Edition',
      price: 19,
      category: 'Games'
    }
  ]
});
```

### Refunds

For refunds to work, you need to have enhanced e-commerce turned on.

For full refunds, fire this event whenever an order/transaction gets refunded:

```js
analytics.track('Order Refunded', {
    order_id: '50314b8e9bcf000000000000',
  });
```

For partial refunds, you must include the productId and quantity for the items you want refunded:

```js
analytics.track('Order Refunded', {
    order_id: '50314b8e9bcf000000000000',
    products: [
      {
      product_id: '123abc',
      quantity: 200
      }
    ]
  });
```

- - -


## Mobile Apps

Segment supports Google Analytics mobile app analytics via our iOS and Android sources. For getting started with our mobile sources, check out the [iOS](/docs/sources/mobile/ios/) and [Android](/docs/sources/mobile/android/) technical docs.

When including Segment-GoogleAnalytics in your project, we bundle IDFA support by default. You can choose to exclude IDFA Support by specifying `pod "Segment-GoogleAnalytics/Core"`. Doing this, we will only bundle the Segment and Core GA libraries, excluding GoogleIDFASupport.

You'll need to create a new Google Analytics property for your mobile app. You can't mix website and mobile apps within the same Google Analytics property. You can however mix Android and iOS implementations of the same app, or many different builds of the same app inside the same property.

Here are [Google's Best Practices for Mobile App Analytics](https://support.google.com/analytics/answer/2587087):

  - Track different apps in separate properties
  - Track different platforms of an app in separate properties
  - Track app editions based on feature similarities
  - Track different app versions in the same property


### Add the Mobile Tracking Id Field

The first thing you'll want to do if you're bundling the Segment-GoogleAnalytics SDK is to add your **Mobile Tracking Id** to your Google Analytics settings inside Segment. This ensures that data can flow from each user's mobile device to Google Analytics. Otherwise, Segment won't know where to send your data, and the events will be lost.


### When Will I See Data?

If you already have an app deployed with the Segment library, and you just turned on Google Analytics mobile, it will take up to an hour for all your mobile users to refresh their Segment settings cache, and learn about the new service that you want to send to.

After the settings cache refreshes, our library will automatically start sending data to Google Analytics.


### Android Permissions

You'll need to make sure you added these permissions to your `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```


### Calling Google Analytics Directly

Since our SDKs bundle the Google Analytics SDK, you can access the Google Analytics `Tracker` object directly. Here's an Android example:
```java
GoogleAnalytics ga = GoogleAnalytics.getInstance(this);
Tracker tracker = ga.newTracker('<your tracking id>');
```
```java
// perform custom actions, such as user timings
tracker.send(new HitBuilders.TimingBuilder()
    .setCategory(getTimingCategory())
    .setValue(getTimingInterval())
    .setVariable(getTimingName())
    .setLabel(getTimingLabel())
    .build());
```

This allows you to perform custom actions with Google Analytics, such as [user timings](https://developers.google.com/analytics/devguides/collection/android/v4/usertimings).

- - -


## Server Side

When you track an event or pageview with one of our server-side libraries or [HTTP API](/docs/sources/server/http/) we will send it along to the Google Analytics REST API.

**You must include a server-side tracking ID in your Google Analytics destination settings or we won't pass server-side events to Google Analytics.** The tracking ID can be the same UA code as your regular property ID, or you can choose to send the server-side events to a separate Google Analytics property.

### Combining Server-side and Client-side Events

Google Analytics uses cookies to keep track of visitors and their sessions while visiting your website. The cookie data is stored in the visitor's browser, and is sent along to Google Analytics every time a new pageview or event occurs. This allows Google Analytics to show  a single unique visitor between multiple page reloads.

Your servers also have access to this cookie, so they can re-use it when you send server-side events to Segment. If you don't use the existing cookie Segment has to create a new one to make the server-side request to Google Analytics. When we create a new cookie the client-side and server-side events from the same user will look like two distinct visitors in Google Analytics.

If you want to use server-side Google Analytics, there are three options with Segment:

1. **Pass your Google Analytics cookies to Segment (preferred).**
2. Use two Google Analytics profiles: one for client-side data and one for server-side data.
3. Ignore the additional visitors generated by not passing the cookie.


### Passing Cookies from Universal Analytics

Universal Analytics (analytics.js) uses the [`clientId`](https://developers.google.com/analytics/devguides/collection/analyticsjs/cookie-usage#analyticsjs) to keep track of unique visitors.

*A Google Analytics Universal cookie will look like this:*
```
_ga=GA1.2.1033501218.1368477899;
```

The `clientId` is this part: `1033501218.1368477899`

You can double check that it's your `clientId` by running this script in your javascript console:

```javascript
ga(function (tracker) {
    var clientId = tracker.get('clientId');
    console.log('My GA universal client ID is: ' + clientId);
});
```

If you want our server-side destination to use your user's `clientId`, pass it to us in the `context['Google Analytics'].clientId` object. You must pass this value manually on every call as we do not store this value for you. If you do not pass this through, we look for the `userId` or `anonymousId` value and set it as the `cid`.

*Here's a Ruby example:*
```ruby
Analytics.track(
  user_id: '019mr8mf4r',
  event: 'Clicked a Link',
  properties: {
    linkText     : 'Next'
  },
  context: {
    'Google Analytics' => {
        clientId: '1033501218.1368477899'
    }
  }
)
```


### User Agent

By default, we won't set the `user-agent` header. If you have your user's `user-agent` server-side, you can send it to us using the `context` object. The `context` object is an optional argument supported by all of our server-side sources.

Here's a Ruby example:

```ruby
Analytics.track(
  user_id: '019mr8mf4r',
  event: 'Loaded a Page',
  properties: {
    url: 'http://initech.com/pricing'
  },
  context: {
    user_agent: 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_8_2) AppleWebKit/537.17 (KHTML, like Gecko) Chrome/24.0.1312.57 Safari/537.17'
  }
)
```


### Visitor Geo-Location

Google Analytics uses the IP address of the HTTP request to determine the location of the visitor. This happens automatically for client-side and mobile tracking, but takes a little more work for server-side calls.

For geo-location to work from a server-side call you'll need to include the visitor's `ip` in your `.track()` call.

*Here's a Ruby example:*
```ruby
Analytics.track(
    user_id: '019mr8mf4r',
    event: 'Purchased Item',
    properties: { revenue: 39.95 }
    context: { ip: '11.1.11.11' })
```


### UTM Parameters

If you want to send UTM parameters to Google Analytics via one of the Segment server-side sources they need to be passed manually. The client-side Javascript library ([Analytics.js](/docs/sources/website/analytics.js)) is highly recommended for collecting this data since it all happens automatically.

Your UTM params need to be passed in the `context` object in `context.campaign`. For Google Analytics `campaign.name`, `campaign.source` and `campaign.medium` all need to be sent together for things to show up in reports. The other two params (`campaign.term` and `campaign.content`) are both optional, but will be forwarded to GA if you send them to Segment.


---


## Features

We support all of the following Google Analytics features:

- [Client-side (Analytics.js) library methods](#client-side-library-methods)
- [Anonymize IP Address](#anonymize-ip-address)
- [Cookie Domain Name](#cookie-domain-name)
- [Custom Dimensions](#custom-dimensions)
- [Cross-domain Tracking](#cross-domain-tracking)
- [Ecommerce Transactions](#e-commerce)
- [Events](#track)
- [Ignored Referrers](#ignored-referrers)
- [Multiple Trackers](#multiple-trackers)
- [Query strings in Pageview](#url-query-strings)
- [Remarketing](#remarketing) (Demographics & Interest Reports)
- [Server-Side Tracking](#server-side)
- [Site Search](#site-search)
- [User-ID](#user-id)
- [Virtual Pageviews](#virtual-pageviews)
- [Optimize](#optimize)

### Client-Side Library Methods

Because Segment's client-side snippet wraps Google Analytics's Javascript, all GA library methods that don't map to Segment methods are available client side. Although invoking a native library method won't send data to Segment or other Segment-enabled destinations, the method *will* send data to Google Analytics.

To access Google Analytics methods while using Segment, write these methods inside an `analytics.ready()` function, for example:

```javascript
analytics.ready(function(){
  // GA library methods here
})
```


### Anonymize IP Address

Check the box in the Advanced Options for Google Analytics inside of Segment.


### Remarketing

Google's remarketing (The remarketing tag formerly known as Doubleclick) is used to tag visitors for remarketing campaigns. It is also used to identify demographic and interest data on visitors that is displayed in Demographic & Interest reports inside of Google Analytics.

Turn this feature on by checking the box in your Google Analytics destination settings.

Since remarketing is loaded through Segment Google Analytics will not be able to validate that the code is present on the page. Just click **Skip validation** and your data will start showing up within a few hours.


### Across Sub-domains

This works automatically if you're using the Universal tracking method. To track across sub-domains we recommend upgrading to universal if you haven't already.

If you need to set a specific domain name keep reading :)


### Multiple Trackers

Although Segment does not support loading multiple trackers through the destinations settings page (you will probably run into Google Analytics's [rate limits](https://developers.google.com/analytics/devguides/collection/ios/v3/limits-quotas?hl=en)), you can load a 2nd tracker on the page manually.

Here's how you'd initialize the second tracker and send a pageview to the second tracker Google Analytics property:

```javascript
analytics.ready(function(){
  ga('create', 'UA-XXXXX-Y', 'auto', {'name': 'secondTracker'});
  ga('secondTracker.send', 'pageview');
})
```

*Note*: Make sure this script is placed after your Segment snippet, ideally at the end of the head tag.

After you create the second tracker, you probably want to use our `.on()` emitter to automatically send data to this separate Google Analytics instance based on when you make other Segment calls.

The below code would trigger an event to Google Analytics when you make a Segment track call.

```javascript
analytics.on('track', function(event, properties, options){
   // custom logic based on event properties
  ga('secondTracker.send', {
    hitType: 'event',
    eventCategory: properties.category || 'All',
    eventAction: event,
    eventLabel: properties.label || 'All'
  })
});
```

**Important**: Keep in mind you will need to do all the data translation/properties mapping inside this `.on()` function before you send the event to Google Analytics like you see in our [destination code](https://github.com/segment-integrations/analytics.js-integration-google-analytics/blob/master/lib/index.js#L161-L207).

To do this server side, you can create a separate [source](https://help.segment.com/hc/en-us/articles/204892239-What-are-sources-) in Segment, and within this source enter your GA credentials for the second tracker.

This source can be your server-side source. From there, its easy to send data to multiple projects server-side, as you can see in this [Node example](/docs/sources/server/node/#multiple-clients) you can initialize multiple instances of our library.

### Cookie Domain Name

The Google Analytics **Cookie Domain Name** setting allows you to specify the domain that the `_ga` cookie will be set on. By default the cookie is placed on the top level domain: `domain.com`.

We default the **Cookie Domain Name** to `auto`, which automatically sets the cookie at the root domain level, which allows you to track across multiple sub-domains, but does not work on `localhost`. You can find this setting in your Google Analytics destination settings.

If you need to test on `localhost`, but don't need to track between multiple sub-domains, then you can set the domain to `none`.

If you only want the cookie to persist on a single sub-domain, enter that sub-domain in the **Cookie Domain Name** field, like this: `swingline.initech.com`. In this case visitors to `conclusions.initech.com` or `initech.com` will not be tracked.

For more information on Google Analytics cookies and domains name see [Google's docs on the subject](https://developers.google.com/analytics/devguides/collection/analyticsjs/domains).


### Cross-Domain Tracking

Segment supports Google Analytics tracking across multiple top level domains, but it requires a bit of work from you. There are two ways to track visitors across domains.


#### Tracking Visitors with User-ID

If you're identifying your users with a [User-ID](#user-id) cross-domain tracking becomes simple. All you have to do is make sure you identify your users on each domain and Google will merge those users together as one.

The only problem with this approach is that it only works for identified users, anonymous visitor sessions will not be maintained across domains.


#### Tracking Anonymous Visitors

When a visitor comes to your website, `domain1.com`, Google Analytics sets a first-party cookie that represents that user. That cookie looks like `182119591.1441315536`, and is tied to `domain1.com` (making it a first party cookie).

When your visitor clicks a link to go another domain, let's say `domain2.com`, you'll need to tell the new site about the `domain1.com` cookie. This is done by rewriting your `domain2.com` links to include this `domain1.com` cookie, like so:

```html
http://company2.com?_ga=1.182119591.1441315536.1362115890410
```

Luckily, Google Analytics providers an auto-linking plugin to make this easier. To access the `ga` methods while using Segment they must be inside an `analytics.ready()` function, which should appear after your basic Segment snippet, like this:

```javascript
analytics.ready(function () {
    ga('require', 'linker');
    ga('linker:autoLink', ['company2.com']);
});
```

To make things easy Segment enables `allowLinker` by default so all you need to do is run these two functions with any domains you want to track across to in the second call above.

You'll have to send the `clientId` as described in the [Google Analytics Domain Guide](https://developers.google.com/analytics/devguides/collection/analyticsjs/cross-domain) to get this setup.


### Site Search

In order to populate the Site Search report in Google Analytics there are a few you need to do...

1. When someone searches on your site, the search term they used must be added to the URL query, like this: `domain.com?s=coconuts`. The key ("s" in this case) can be any letter or string of letters.

2. In your Segment source destinations catalog open the Google Analytics settings, click to the Advanced Options tab, scroll down and make sure the box is checked for **Include the Querystring in Page Views**.

3. Inside Google Analytics, go to the **Admin** section, then click **View Settings** for the view you want to add Site Search to. Turn on **Site search Tracking** and enter the string from #1 into the Query parameter field. In this example it'd look like this:

![google analytics site search form](images/site-search.png)


### Webmaster Tools

When you use Segment to load Google Analytics, our script loads the Google Analytics script. If you use [Google Analytics as the verification option](https://support.google.com/webmasters/answer/1120006?hl=en) in Google Webmaster Tools, you'll need to switch to the [Meta tags verification option](https://support.google.com/webmasters/answer/79812?hl=en) instead. This will require you to find the `<meta name=google-site-verification" ..>`  tag in Webmaster Tools and place it in your master HTML template.


### Cannonical Urls

We take care of tracking the canonical URL to Google Analytics for you automatically. As long as there is a `<meta rel="canonical">` tag on your page, we'll make sure Google Analytics gets the right canonical URL from it.

### Optimize

If you'd like to integrate with Google Analytics' [Optimize plugin](https://support.google.com/360suite/optimize/answer/6262084#optimize-ga-plugin), all you have to do is insert your Optimize **Container ID** in your destination settings and we will require the plugin when we initialize GA!

*Note*: Please make sure your Container ID is spelled correctly and that your Optimize container is ENABLED w/in Google. Otherwise, your GA destination will silently error out every time you try to make any tracking calls.

You may, however, want to deploy [page hiding](https://support.google.com/360suite/optimize/answer/6262084#page-hiding) to prevent the page from flashing / flickering when the A/B test is loaded. This is recommended by Google. This code must be added manually by customers since it needs to load synchronously. Note that the Optimize container ID must be included in this snippet too.

- - -


## Troubleshooting

### Metrics vs. Dimensions

They both allow you to track custom data properties in Google Analytics. However, Metrics are for event properties with a numeric data type and Dimensions are for event properties with a string data type.


### Real-Time Reports

Google Analytics doesn't process their data in real-time in most of their reports. The easiest way to see if the data is streaming in is to check the Real-Time reports inside Google Analytics.

If you see events in your real-time reports, but they never show up in other reports that is usually due to a filter you have applied. You can see your active filters inside Google Analytics by clicking on **Admin** then under your View on the right click on **Filters**.


### Self Referrals

This article does a great job of explaining GA self referrals and how to fix them: https://threeventures.com/how-to-fix-self-referrals-in-google-analytics/


### Time Frame

Google Analytics's default reporting time frame is a month ago to yesterday. You'll need to adjust it from a month ago to today's date to see today's already processed events.


### HTTPS

If your site uses `https://`, please go to your Google Analytics property settings page and change your **Site URL** to use the `https://` protocol.


### Bounce Rates

Using Segment won't affect your bounce rates in Google Analytics.

If you see your bounce rates drop after installing Segment make sure you don't have multiple copies of our snippet on your page. Also be sure you're not calling `page` more than once when the page loads.

If you call `track` on page load make sure to set `nonInteraction` to `1`. You can also set all events to be non-interactive by default in `Advanced Options`.  Read more in our [non-interaction events](#non-interaction-events) docs.


### Traffic from Boardman or Segmentio Browser

If you are seeing traffic from Boardman or see Segment as the browser, this is most likely because you are sending calls to Google Analytics from the **server side** (our AWS servers reside in Boardman, Oregon). In order to prevent the Boardman issue, you would have to manually pass the `IP` information in the `context` object from the server.

Here is an example:

```ruby
Analytics.track(
    user_id: '507f191e810c19729de860ea',
    event: 'Visited Agency Profile',
    properties: { name: 'Ram Estate Agent', favorite_color: 'blue' },
    context: { ip: '127.0.0.1' }
)
```

To prevent the Segment as the browser issue, you want to manually pass in the `user_agent`:

```ruby
Analytics.track(
    user_id: '507f191e810c19729de860ea',
    event: 'Visited Agency Profile',
    properties: { name: 'Ram Estate Agent', favorite_color: 'blue' },
    context: { user_agent: 'some user-agent' }
)
```

### Providing Required Field Display Name

Google Analytics requires the `context.app.name` passed in each call. Since the `analytics-ios` SDK pulls it in [locally](https://developer.apple.com/library/ios/documentation/General/Reference/InfoPlistKeyReference/Articles/CoreFoundationKeys.html#//apple_ref/doc/uid/20001431-110725), you may see the error "`context.app.name` required" if you are not providing a `CFBundleDisplayName` within your **Info.plist** file.

To resolve this error, ensure you [provide a localized info dictionary](https://github.com/segmentio/analytics-ios/blob/760be85a5119c2e8bd31a745ce2ec30385a0ad69/Pod/Classes/Internal/SEGSegmentIntegration.m#L110) as outlined [here](https://developer.apple.com/library/ios/qa/qa1823/_index.html).