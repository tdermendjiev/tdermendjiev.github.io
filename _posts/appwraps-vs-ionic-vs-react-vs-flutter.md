# Appwraps vs. Ionic vs. React Native vs. Flutter

Created: September 19, 2022 9:51 AM
Last Edited Time: September 27, 2022 11:11 AM

Mobile app developers have to contend with a wide variety of operating systems, screen sizes, and other hardware differences when creating apps that need to work on both Android and iOS devices. In the past, developers would have to create separate apps for each platform. But now there are a number of cross-platform mobile development frameworks that allow developers to create apps that work on both Android and iOS with a single codebase.

This can help to reduce the cost of development as developing two separate apps can be expensive. There are some downsides though, for example, that the app might not be as optimized for each platform or it can be more difficult to debug and test.

![Screenshot 2022-09-19 at 12.54.09.png](Appwraps%20vs%20Ionic%20vs%20React%20Native%20vs%20Flutter%20d6e9f5ebd93149f5847162697d25bf51/Screenshot_2022-09-19_at_12.54.09.png)

### Hybrid vs. Cross-platform

First, we need to clarify these two categories of apps. Hybrid apps are basically websites that are wrapped in a webview and they can use device features via frameworks as Cordova.

React-Native and Flutter are cross-platform frameworks. They can access native APIs directly without restrictions and UI elements are native.

Appwraps has some of the both worlds - most of the time the UI is a webview (as with Ionic), but thanks to the NativeScript runtime you can access iOS and Android features directly without having to rely on plugins. With that said, you can also create native UI elements from the website’s javascript code.a

### Reusability

Code reuse is usually the major reason why companies develop cross-platform apps. 

Being a cross-platform framework, **React Native** and **Flutter** allow a decent code reuse between iOS and Android. About 90-95% of the code can be reused. In some cases you will have to write Swift/Objective-C code with React Native. Flutter provides many ready-to-use widgets. 

Javascript code reuse is easy with **Ionic** because the UI is rendered in a webview. For native features it may become more complex because it relies on plugins.

**Appwraps** provides close to 100% of code reuse as you can basically deploy your existing website as a mobile app or even use the URL of an already deployed one. In the same codebase developers can add logic that will be executed only if the app is launched in Appwraps and use NativeScript’s cross-platform modules or 3rd party plugins for higher level of reusability. If you don’t want to use plugins Appwraps also gives a direct access to native APIs thanks to the NativeScript runtime.

### Native access

**React Native** gives access to platform APIs but not for *all* of them - for some you need to write a native module or use a third-party one.

**Ionic** steps on Cordova or Capacitor for native API access but all of it is done through third party plugins which may cause issues with security or reliability.

Direct native access is one of the biggest upsides of **Appwraps** - thanks to the **NativeScript** runtime you can call any iOS or Android API directly from your javascript code.  Let’s have a look at this example from the NativeScript [docs](https://docs.nativescript.org/native-api-access.html):

```jsx
if (global.isIOS) {
  const batteryLevel = UIDevice.currentDevice.batteryLevel
}
```

And for Android:

```jsx
if (global.isAndroid && Device.sdkVersion >= '21') {
  const bm = Utils.android
    .getApplicationContext()
    .getSystemService(android.content.Context.BATTERY_SERVICE)
  const batLevel = bm.getIntProperty(android.os.BatteryManager.BATTERY_PROPERTY_CAPACITY)
}
```

Of course you don’t necessarily have to do this because **Appwraps** provides also access to **NativeScript’s [cross platform modules](https://v6.docs.nativescript.org/core-concepts/modules.html).**

If you want to do the same with **Flutter** you either have to use a [package](https://pub.dev/packages/battery_plus) or create one yourself using [channels](https://docs.flutter.dev/development/platform-integration/platform-channels).

### Performance

Speaking of performance, **Appwraps** and **Ionic** share similar characteristics - most of the time it will depend on the webview performance, but with latest iOS and Android updates webviews are really fast. Of course the UI won’t be as smooth as it’s possible with **React Native** and **Flutter** but in the most cases it won’t make difference.

### Languages and frameworks

Both **Ionic** and **React Native** are Javascript-based frameworks. Ionic can be used with any Javascript framework, whilst for React Native you will need to know **React.** Both have huge communities so you are in good hands whatever you prefer.

**Flutter** on the other side uses **Dart**, which is compiled to native code which runs on the target device. Dart is modern, powerful and it’s popularity is growing.

If you want to get the most of **Appwraps** you will need Javascript because it’s based on NativeScript’s runtime. The good thing is it doesn’t matter what framework you used or where (or whether) or app is deployed as soon as you can execute any javascript code.

### UI Elements

As I already mentioned **Ionic** and **Appwraps** render your app’s UI in a webview which for certain scenarios may be less smooth than native components (as with **React Native** and **Flutter**) - e.g. if you have a lot of interaction and animations. With the introducing of WKWebView in iOS, though, web UI loading and performance improved a lot and most of the time you won’t be able to distinguish between a webview app and a native one. (*You can check our sample application built with OnsenUI or any productive Ionic application, really.*) In cases where your user will mostly read or browse and especially if you will open webviews anyway (e.g. news websites, online store etc.) **Ionic** or **Appwraps** is your best bet.

### Development speed

 ****A metric that is really important for business. How long would it take to get to the Appstore and Play store? With React Native and Flutter you will obviously have to develop your app from scratch and therefore deal with developer teams, managers and other staff depending on your base. Of course, if you have a website built with React or Flutter (it’s possible) you can reuse some of the components, but still in the most cases you expect to wait for months.

Ionic allows much better reusability and therefore faster development but you will still need a developer to convert your web app or start from scratch.

Here is where **Appwraps** shines the most - you can have your mobile app ready to be published in hours and you don’t need a developer or coding skills if there is already a website - just upload the app assets (logo, launchscreen, icons) and fill the info and you will get the app project ready to be built and published (or you can use Appwraps publishing service). You (or your dev team) can start adding mobile features right away.

**Even if you decide to use other cross-platform technology or even go native you can still go with Appwraps until your app is ready.** 

### Cost of development

I wont go into details here because cost may depend on so many factors but it should be safe to say that an app developed from scratch may cost between $30 000 and $100 000 (if it’s a large one a lot more). Should you cut some parts and your app is really simple it’s completely possible to go between $5000 and $20000 but I don’t think it’s realistic to expect less.

**Going mobile with Appwraps would cost you less than $1000 (Appstore fees included)  if your web app is already responsive. This makes it a no-brainer even if you want to develop a new app.**
