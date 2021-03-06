This repository contains the wrapper code and binaries for the unity plugins derived from the Appboy Android and iOS SDKs.

## Plugin Setup

Before you can start using Appboy in your Unity scripts, you'll need to import the plugin files to your Unity project. First, clone this repo. If you're not using any other plugins, all you have to do is copy the `Plugins` directory from this repo into the `Assets` folder of your Unity project. 

If you already have a `/<your-project>/Assets/Plugins` directory (probably because you're using another plugin already), copy `Plugins/Appboy/AppboyBinding.cs`, `Plugins/WindowsPhone8UnityAdapter.dll` and `Plugins/WindowsUniversalUnityAdapter.dll` into `/<your-project>/Assets/Plugins`. Then copy the contents of `Plugins/iOS`, `Plugins/Android`, `Plugins/WP8`, and `Plugins/Metro`  from this repo into `/<your-project>/Assets/Plugins/iOS`, `/<your-project>/Assets/Plugins/Android`, `/<your-project>/Assets/Plugins/WP8`, and `/<your-project>/Assets/Plugins/Metro` respectively.

## iOS Setup Instructions
<ol>
<li>
First, generate your Xcode project in Unity by clicking on "File" -> "Build Settings...", then selecting iOS as the platform and clicking "Build". You'll be prompted for a name/location to build the app in. Since you'll be modifying the built output, we recommend including the built app in your project's root Unity folder (the same place that you have your <code>Assets</code> directory).
</li>
<li>
Confirm that Unity has copied the files <code>AppboyBinding.m</code>, <code>AppboyUnityManager.h</code>, and <code>AppboyUnityManager.mm</code> to the "Libraries" directory of your generated project. Note that they will not be included in the XCode project, so you'll need to check for their presence manually. If Unity fails to copy the files automatically, manually copy them from this repo.
</li>
<li>
Include AppboyUnityManager.h in your Xcode project (even though the file itself is already in the Libraries directory) by right clicking on Classes and selecting "Add Files to ..."
</li>
<li>
Now, you will need to perform the first three normal integration steps for adding the Appboy SDK to an iOS project. These steps are documented under "Basic SDK Integration," "Add the iOS Libraries," and "Configure the Appboy Library and Framework" in our iOS Integration Instructions at: http://documentation.appboy.com/ios-sdk-integration.html#cloning-the-appboy-sdk
<br><br>
<b>Note:</b> As of version 2.3.1, the Appboy iOS SDK has split into two libraries, AppboyKit and AppboyKitWithoutFacebookSupport. More details of the split can found here (https://github.com/Appboy/appboy-ios-sdk and https://github.com/Appboy/appboy-ios-sdk/blob/master/CHANGELOG.md#231). Since the Unity platform provides Facebook support and therefore the AppboyKitWithoutFacebookSupport is the correct SDK to use in your Unity application. 
</li>
<li>
Next, we'll need to make some modifications to your generated <code>Classes\AppController.mm</code>. A version of these modifications is included at the bottom of the integration directions linked to in the previous step, the followin are meant to replace those).
<ul>
<li>
At the top of the file add the following import statements:
<pre><code>#import "AppboyKit.h"
#import "AppboyUnityManager.h"
</code></pre>
</li>
<li>
In the method <code>applicationDidFinishLaunchingWithOptions</code>, add the following code block above the return statement, replacing "YOUR-API-KEY" with your Appboy API key:
<pre><code>[Appboy startWithApiKey:@"YOUR-API-KEY"
        inApplication:application
        withLaunchOptions:launchOptions];
[Appboy sharedInstance].slideupDelegate = [AppboyUnityManager sharedInstance];
</code></pre>
</li>
<li>
If you want to fetch the slide up message from Appboy, add this line to <code>applicationDidFinishLaunchingWithOptions</code>:
<pre><code>
[[AppboyUnityManager sharedInstance] addSlideupListenerWithObjectName:@"Your Unity Game Object Name" callbackMethodName:@"Your Unity Call Back Method Name"];
</code></pre>
Replace @"Your Unity Game Object Name" with the unity object name you want to listen to the slide up message, and also change @"Your Unity Call Back Method Name" to the call back method name that handles the slide up message. Please make sure the call back method is in the unity object you just passed in as the first parameter. 
</li>
<li>
If you want to use push notifications, also add this code to <code>applicationDidFinishLaunchingWithOptions</code>:
<pre><code>[[UIApplication sharedApplication] registerForRemoteNotificationTypes: 
          (UIRemoteNotificationTypeAlert | 
           UIRemoteNotificationTypeBadge |     
           UIRemoteNotificationTypeSound)];
[[AppboyUnityManager sharedInstance] addPushReceivedListenerWithObjectName:@"Your Unity Game Object Name" callbackMethodName:@"Your Unity Call Back Method Name"];
[[AppboyUnityManager sharedInstance] addPushOpenedListenerWithObjectName:@"Your Unity Game Object Name" callbackMethodName:@"Your Unity Call Back Method Name"];
</code></pre>
This line to <code>didRegisterForRemoteNotificationsWithDeviceToken</code>:
<pre><code>[[Appboy sharedInstance] registerPushToken:
                [NSString stringWithFormat:@"%@", deviceToken]];
</code></pre>
And this line to <code>didReceiveRemoteNotification</code>:
<pre><code>[[AppboyUnityManager sharedInstance] registerApplication:application
                didReceiveRemoteNotification:userInfo];
</code></pre>
</li>
<li>
To help verify the code modifications you made, there is an example modified AppController.mm generated by Unity 4.1.2 in Docs/iOS in this repo.
</li>
</ul>
</li>
<li>
If you're using push notifications you'll need to follow the standard setup for Apple push which you can find at https://appboy.zendesk.com/entries/23690991-iOS-Push-Notifications
</li>
<li>
As you make updates to your app from Unity, you should choose the same location to generate the Xcode project each time. Unity will prompt you to replace or append the existing folder. If you choose "Append," you shouldn't have to redo any of your Appboy setup in the future.
</li>
</ol>

## Android Setup Instructions
<ol>
<li>
First, we'll confirm that you've copied the plugin folder correctly and identify all of the locations where you'll need to insert configuration specific to your app. From the root of your Unity project, you should be able to run the following and find all of the locations that must be modified to fully setup Appboy for Android:
<br />
<code>
        grep -r REPLACE Assets/Plugins/
</code>
<br />
The output should look something like this:
<pre><code>Android/AndroidManifest.xml:  package="REPLACE_WITH_YOUR_BUNDLE_IDENTIFIER" android:versionCode="1" android:versionName="0.0"&gt;
Android/AndroidManifest.xml:  &lt;permission android:name="REPLACE_WITH_YOUR_BUNDLE_IDENTIFIER.permission.C2D_MESSAGE" android:protectionLevel="signature" /&gt;
Android/AndroidManifest.xml:  &lt;uses-permission android:name="REPLACE_WITH_YOUR_BUNDLE_IDENTIFIER.permission.C2D_MESSAGE" /&gt;
Android/AndroidManifest.xml:        &lt;category android:name="REPLACE_WITH_YOUR_BUNDLE_IDENTIFIER" /&gt;
Android/AndroidManifest.xml:        &lt;action android:name="REPLACE_WITH_YOUR_BUNDLE_IDENTIFIER.intent.APPBOY_NOTIFICATION_OPENED" /&gt;
Android/res/values/appboy.xml:  &lt;string name="com_appboy_api_key">REPLACE_WITH_YOUR_APPBOY_API_KEY&lt;/string&gt;
Android/res/values/appboy.xml:  &lt;string name="com_appboy_push_gcm_sender_id">REPLACE_WITH_YOUR_GOOGLE_API_PROJECT_NUMBER&lt;/string&gt;  &lt;!-- Replace with your Google API project number --&gt;
</code></pre>
</li>
<li>Next, you need to find your "Bundle Identifier" from Unity. This is available from the Android tab of the "Player Settings" pane (accessible by clicking the Player Settings button in File -> Build Settings). The Player Settings pane looks like this in Unity 4: 
<br />
<img height=500 src="Docs/Images/UnityBundleIdentifier.png" />
</li>
<li>
In AndroidManifest.xml, replace all instances of <code>REPLACE_WITH_YOUR_BUNDLE_IDENTIFIER</code> with the Bundle Identifier you found in Step 2. Your Bundle Identifier is usually of the form "com.unity.appname".
</li>
<li>
Now you must configure the plugin by replacing a few values in the Appboy configuration file (Plugins/Android/res/values/appboy.xml). Replace the text <code>REPLACE_WITH_YOUR_APPBOY_API_KEY</code> with your API key. If you don't have your API key, simply login to the Appboy dashboard. If you don't have an app or account yet, you'll need to create them by following the wizard. Once you've created an app, navigate to the "Settings" page by clicking the gear icon to the right of your app name in the top left of the Apps tab. You can use the default Debug API Key that will have been created for you for debugging. When you're ready to deploy your application, you should update this to use your Production key. 
</li>
<li>
For push notifications to work, you will now need to insert your GCM Sender ID from Google into the same appboy.xml configuration file. If you don't have a GCM Sender ID yet, you'll need to follow the GCM setup instructions from Google. Once you have the ID, change <code>REPLACE_WITH_YOUR_GOOGLE_API_PROJECT_NUMBER</code> to your GCM ID. Since the GCM ID is a number, you shouldn't surround the value with quotes. Your ID should look something like <code>134664038331</code>.
<li>
At this point, you should be able to run the grep command from Step 1 and have no results. If there are any additional instances of <code>REPLACE</code> remaining, repeat these steps.
</li>
<li>Next, you must configure push notifications on the Appboy <a href="http://dashboard.appboy.com">dashboard</a>. Navigate back to the "Settings" page by clicking the gear icon to the right of your app name. Under the Push Notifications section of the App Settings page, insert your GCM API key. This is different from your GCM Sender ID and can be found on the API Access page of your Google API project. Your Google Project API key should look something like <code>ABcdEfGHij1_klm2NOPQrwTuVWxyZ3A4bCDeF5g</code>. Please refer back to <a href="http://developer.android.com/google/gcm/gs.html">GCM setup instructions</a> for more help.
</li>
<li>
Finally, you should confirm that your Android app is only targeting version 2.2 (Froyo) and above. This setting is available from the same "Player Settings" pane that you found the Bundle Identifier in during Step 2. Android versions previous to 2.2 represent a vanishingly small portion of active Android devices and are not supported by Appboy.
</li>
</ol>

### Advanced Android Integration Options
The default AndroidManifest.xml file provided has three Activity classes registered. 
<ul>
<li>com.appboy.unity.AppboyUnityPlayerProxyActivity</li>
<li>com.appboy.unity.AppboyUnityPlayerActivity</li>
<li>com.appboy.unity.AppboyUnityPlayerNativeActivity</li>
</ul>
These Activity classes are integrated with the Appboy SDK, calling the necessary methods to collect analytics, 
provide support for programmatic sending of feedback, slideup message callbacks, push notification callbacks. 
All three activities are provided in order to provide a drop-in replacement for the existing Unity activities.
Most Unity integrations will not require you to change the Activity list, however, if you're only targeting 
devices running Android OS Gingerbread or newer, you can remove the 
<code>com.appboy.unity.AppboyUnityPlayerProxyActivity</code> and 
<code>com.appboy.unity.AppboyUnityPlayerActivity</code> and set the 
<code>com.appboy.unity.AppboyUnityPlayerNativeActivity</code> as the main Activity class.

<b>Note:</b> All Activity classes registered in your AndroidManifest.xml file must be fully integrated with the Appboy 
ndroid SDK. If you add your own Activity class, you must follow these <a href="http://documentation.appboy.com/sdk-integration-android.html#android">integration instructions</a> to ensure that analytics are being collected. 

### Prime 31 compatability ###
In order to use the Appboy Unity plugin with Prime31 plugins, edit the AndroidManifest.xml to use the Prime31 compatible Activity classes and GCM receiver. Change all four references of 
<ul>
<li>com.appboy.unity.AppboyUnityPlayerProxyActivity</li>
<li>com.appboy.unity.AppboyUnityPlayerActivity</li>
<li>com.appboy.unity.AppboyUnityPlayerNativeActivity</li>
<li>com.appboy.unity.AppboyUnityGcmReceiver</li>
</ul>
to 
<ul>
<li>com.appboy.unity.prime31compatible.AppboyUnityPlayerProxyActivity</li>
<li>com.appboy.unity.prime31compatible.AppboyUnityPlayerActivity</li>
<li>com.appboy.unity.prime31compatible.AppboyUnityPlayerNativeActivity</li>
<li>com.appboy.unity.prime31compatible.AppboyUnityGcmReceiver</li>
</ul>

## Windows Phone 8  Setup Instructions ##

Once you have placed the plugin DLLs in the proper location in your Unity project, build your Windows Phone 8 application from your Unity project.  You will then have a working Windows Phone 8 XAML application with the required Appboy plugin DLLs referenced from your project.  

You can then follow the normal integration instructions found <a href="https://documentation.appboy.com/SDK_Integration/Windows/Phone8">here</a>, with the following exception:  

<b>Note:</b> Unity does processing and modification of Windows Phone 8 plugin DLLs.  As a result, you must use the DLLs that are output from the Unity build; do not replace a DLL with a different version after you've built from Unity.  In particular, if you are using our UI library, you must modify the references to point at the DLLs unity outputs (not the ones found using Nuget).  If you need to update a DLL version, you can place it in the ```Plugins/WP8``` directory and then use the ones output from Unity.

## Windows Universal Setup Instructions ##

Once you have placed the plugin DLLs in the proper location in your Unity project, build your Windows Store or Phone 8.1 application from your Unity project.  You will then have a working XAML application with the required Appboy plugin DLLs referenced from your project.  

You can then follow the normal integration instructions found <a href="https://documentation.appboy.com/SDK_Integration/Windows/Universal">here</a>.  
