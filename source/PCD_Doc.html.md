---
title: PCD Documentation



search: true
---

# Android

Predictive Content Delivery (PCD) SDK prepositions content, based on user preferences and policies set up between the client and server. There is absolutely no need for a developer to initiate downloads as prepositioning operates on a notification basis. Downloads also continue whether the app is active in the foreground, running in the background, or entirely closed. The SDK acts as a data cache, providing access to content along with its current download state. 

Configuring your project to support these features is covered below.

## Prerequisites for using the SDK (Obtaining key for GCM)

To generate a key from Google's firebase console using GCM in android app, follow these instructions:

* Go to [firebase console] (https://console.firebase.google.com/project/_/settings/cloudmessaging).
* Create a new project.
* It will give you a server-key that will be used by backend developers when creating push requests
* You also need to save the Sender ID given here that will be used on Android for receiving push notifications.

![Prerequisites for using the SDK](https://github.com/Rach20/slate/blob/master/source/1.png)


## File Location to Add Dependencies

Android projects quite often rely on 3rd party libraries and SDKs, and have thier own way to add the dependencies.

You need to add these 3rd party dependencies to a file named build.gradle(Module:app) which is present by default in Android (Refer to the image, the file is highlighted in blue). This file is located in a folder named 'app'.

![File Location to Add Dependencies](2.png)


## Dependencies to be Added

To start using the SDK, you will be required to add the following dependencies:

<code>
compile 'com.google.android.gms:playservicesbase:7.0.0'<br>
compile 'com.google.android.gms:playservicesgcm:7.0.0'<br>
compile 'com.google.android.gms:playservicesads:7.0.0'<br>
compile(name:'voc­sdk­release­{version}', ext:'aar') <br>
</code>

The requirement is that, the project should be implementing API 15 or greater. These dependencies include Google Play services, Google Cloud Messaging and all the dependencies required for PCD.

![Dependencies to be Added](3.png)

## Component Details File

All the details of the components being used in the project are present in a file known as AndroidManifest.xml, which is more like an index. It contains information about all the Activity, Broadcast Receiver, User Permissions and more.. To locate this file, you need to navigate to app>src>main.

![Component Details File](4.png)

## Add Permissions

For proper functioning of the SDK, you need some permissions from the user. Permissions like Internet, Access Network State, WiFi State Access and the permission to write to external storage. All these permissions are required to be added to the _AndroidManifest.xml_ file.

Following are the permissions:

<code>
uses-permission android:name="android.permission.INTERNET"
uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"
uses-permission android:name="android.permission.ACCESS_WIFI_STATE"
uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
</code>

![Add Permissions](5.png)

## Add a Receiver and a Provider

For the SDK to work correctly, we need to add a receiver to listen to Voc Status messages and a provider to access the cached content.

It can be added in _AndroidManifest.xml_ inside the application tag.

![Add a Receiver and a Provider](66.png)

#iOS

Predictive Content Delivery (PCD) SDK prepositions content, based on user preferences and policies set up between the client and server. There is absolutely no need for a developer to initiate downloads as prepositioning operates on a notification basis. Downloads also continue whether the app is active in the foreground, running in the background, or entirely closed. The SDK acts as a data cache, providing access to content along with its current download state. Configuring your project to support these features is covered below.

## Prerequisites

The PCD SDK requires **iOS 8** or higher.

For registration, a **PCD SDK license key** is a mandate.

The application's product name **(Project → Choose target → Build Settings → Packaging → Product name)** must match the name provided on the PCD Web portal SDK license page. The portal field for this is **"iOS Application ID."** The app must enable Background Execution and Remote Notifications in order to preposition the content.

![Prerequisites](7.png)

## Add the framework

- Download the PCD SDK and unzip it in a folder under your project, e.g. _~/myproject/pcd_sdk._
- Navigate to **VocSdk.framework** under _~/myproject/pcd_sdk/iphoneos/_ and copy it to your project folder, e.g. ~/myproject/VocSdk.framework. This is done so that the framework is available before the first build and there-on you can add it to your project. On each build afterward it will be copied here by the copy_vocsdk build phase script.
- Add the framework to your **Xcode** project.
  - Open your project in Xcode.
  - Open the **File** menu.
  - Click **Add Files** to
  - Choose **~/myproject/VocSdk.framework**

 ![Add the framework](8.png)

## Link the SDK to the project

Now, you will need to link the SDK to your project, follow these steps:

By clicking the project name in the Project navigator, open **project settings.**
Click the **General tab.**
Under **Embedded Binaries,** click '+' and choose **VocSdk.framework**
Click Add.

![Link the SDK to the project](9.png)

## Add a build phase to Project settings

Before you go ahead building your app, the correct framework for your platform simulator or device must be copied into the right place and this requires a build phase to be added to your project settings.

Follow these set of instructions:

* Choose your **build target* in Project Settings, then click on **Build Phases.**
* Next, to add a new build phase, click the '+' and choose **'New Run Script Phase.'**
* Drag this build phase to position it after Target Dependencies.
* Optionally, single click its name and rename it to **copy_vocsdk.**
* To expand the copy_vocsdk row, click the arrow.
* Leave the default shell setting of _/bin/sh._
* The following script needs to be added to the script area.

The "copy_vocsdk.sh" path is relative to your .xcodeproj file and will need modification depending on where you unzipped it (in this example, it is pcd_sdk).

<code>. ./pcd_sdk/copy_vocsdk.sh "${PROJECT_DIR}/pcd_sdk"</code>

The second parameter, "${PROJECT_DIR}/pcd_sdk", is the location of the /iphoneos and /iphonesimulator folders from the distribution archive.

In this example, the folder structure is as follows: <br>
/myproject/myproject.xcodeproj /myproject/pcd_sdk/copy_vocsdk.sh /myproject/pcd_sdk/iphoneos/ /myproject/pcd_sdk/iphonesimulator/ 

The script will copy the correct VocSdk.framework to /myproject/ folder at the start of every build.

This is how the build phase will finally look.

![Add a build phase to Project settings](10.png)

## Integrate your iOS

Integrating with your iOS Application - Including the SDK.

Follow these instructions to integrate your iOS.

Wherever the SDK is required, import the VocSdk header in any of the class files. For your app, you will need to define a single VocService object.

Create your VocService object in the app delegate to make the SDK available early in the app lifecycle and to simplify access to the VocService from other classes.

**#import @property (strong, nonatomic) id vocService;**

### Initialization
The SDK is initialized by the VocServiceFactory call

createServiceWithDelegate:delegateQueue:options:error:.

To begin prepositioning files as early as possible, initialization and registration should take place early in the app lifecycle. The recommended place for this is in AppDelegate's application:didFinishLaunchingWithOptions:

The create call inputs a reference to the SDK delegate as well as a configuration options dictionary. To respond to SDK activity, the SDK delegate is the class you need to designate.

The example below is inserted into **AppDelegate.m.** It initializes the SDK and tells it that AppDelegate will be the delegate to handle SDK messages.

<code>
(BOOL)application:(UIApplication *)application <br>didFinishLaunchingWithOptions:(NSDictionary *)launchOptions <br>
{ <br>
NSError *error = nil; <br>
self.vocService = [VocServiceFactory createServiceWithDelegate:self <br>delegateQueue:[NSOperationQueue mainQueue] options:nil error:&error]; <br>
if (!self.vocService)<br>
{ <br>
// error handling ­ could not start service 7 <br>
return NO; <br>
} <br>
// app initialized return YES; <br>
} 
</code>

![Integrate your iOS](11.png)


Need Help? 
If questions come up along the way, check out the frequently asked questions. Still puzzled? Visit our community discussion forums, probably there are others who have already bumped into the same issue as yours.

