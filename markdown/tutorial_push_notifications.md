---
title: Push Notifications on desktop and mobile devices
---
# Tutorial: Push Notifications on desktop and mobile devices

Trigger local and remote notifications in your desktop and mobile applications. Learn how to send push notifications from a remote server to both macOS/iOS and Android devices.

**LEVEL:** Advanced<br/>
**PLATFORMS:** macOS, iOS, Android<br/>
**CLASSES:** [PushNotifications::Listener](structPushNotifications_1_1Listener.html "Register a listener (ideally on application startup) to receive information about notifications recei..."), [PushNotifications::Notification](structPushNotifications_1_1Notification.html "Represents a notification that can be sent or received."), [PushNotifications::Channel](structPushNotifications_1_1Channel.html "Android API level 26 or higher only: Represents notification channel through which notifications will..."), [PushNotifications::Settings](structPushNotifications_1_1Settings.html "Describes settings we want to use for current device.")

> [!WARNING]
> This project requires an Apple Developer account on macOS/iOS and a Google Firebase account on Android. If you need help with this, follow the instructions on the [Apple Developer](https://developer.apple.com/account/) and [Google Firebase](https://console.firebase.google.com/) websites to open up these accounts.

> [!WARNING]
> This project also requires a physical device to test push notifications as simulators do not support remote testing. Please make sure you have a device ready for this.

> [!NOTE]
> The features supported in Push Notifications will vary depending on platforms and OS versions. For macOS, a minimum of 10.7 is required for remote notifications and 10.8 for local notifications. For Android, a minimum API of 14 (Ice Cream Sandwich) is required for remote notifications and a minimum of 27 for notification channels. The tutorial has a minimum SDK version of 23, but you are welcome to change it to be 14 or above.

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/PushNotificationsTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/PushNotificationsTutorial.zip) . Unzip the project and open the first header file in the Projucer.

> [!WARNING]
> If using the PIP version of this project, please make sure to copy the `Resources` folder into the generated Projucer project.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The project provides a comprehensive user interface to send and receive both local and remote notifications. Separate tables are displayed to organise notification parameters and identifiers. If we run the mobile application in the iOS simulator, the window should look something like this:

![](/_images/tutorial_push_notifications_screenshot1.png "The demo project app window on iOS")

> [!TIP]
>The code presented here is broadly similar to the **PushNotificationsDemo** from the JUCE Examples.

# Initial Setup

In order for this project to function properly, we have to perform some initial setup procedures with appropriate developer consoles for specific deployment platform. Let's first allow appropriate permissions for push notifications in the Projucer. Under macOS and iOS, make sure to enable the **Push Notifications capability** field. Under Android, make sure to enable the **Remote Notifications** field.

![](/_images/tutorial_push_notifications_screenshot2_ios.png "Project settings window on iOS")

![](/_images/tutorial_push_notifications_screenshot2_android.png "Project settings window on Android")

We also need to specify custom Xcode resource folders for both macOS and iOS in order for the resources to be bundled in the executable as shown below:

![](/_images/tutorial_push_notifications_screenshot2_resources_macos.png "Custom resources on macOS")

![](/_images/tutorial_push_notifications_screenshot2_resources_ios.png "Custom resources on iOS")

Lastly, on Android we have to specify each resource we want to add in the executable individually as raw resources in both \"Debug\" and \"Release\" exporter settings as follows:

![](/_images/tutorial_push_notifications_screenshot2_resources_android_debug.png "Raw resources on Android (Debug)")

![](/_images/tutorial_push_notifications_screenshot2_resources_android_release.png "Raw resources on Android (Release)")

The full list of resources needed is the following:

```
../Resources/sounds/demonstrative.mp3
../Resources/sounds/isntit.mp3
../Resources/sounds/jinglebellssms.mp3
../Resources/sounds/served.mp3
../Resources/sounds/solemn.mp3
../Resources/images/ic_stat_name.png
../Resources/images/ic_stat_name2.png
../Resources/images/ic_stat_name3.png
../Resources/images/ic_stat_name4.png
../Resources/images/ic_stat_name5.png
```

The Projucer will automatically add the required entitlements to your deployment targets when saving the project and opening in your favourite IDE.

## Apple Developer

> [!TIP]
>If developing for Android, please skip to the next section [Google Firebase](#tutorial_push_notifications_initial_setup_google_firebase) for instructions.

On macOS and iOS, you will need to sign in with your Apple Developer account within Xcode and choose a development team in order to sign the application. Choose a unique bundle ID for your project. Xcode should automatically provide you with a Signing Certificate and Provisioning Profile as shown in the following screenshot:

![](/_images/tutorial_push_notifications_screenshot3.png "General settings window in Xcode")

Also make sure that the correct app capabilities have been ticked and approved in the **Capabilities** settings window. You should see the same information as follows (note there is no Background Modes section on macOS):

![](/_images/tutorial_push_notifications_screenshot4.png "Capabilities settings window in Xcode")

## Pusher

In order to test remote notifications on iOS, we require a server or an application that will send notifications using the Apple Push Notification Service (APNs). We could make use of Google Firebase to test these (like we do for the Android side of things), but thankfully there is an easier solution using an app called [Pusher](https://github.com/noodlewerk/NWPusher). We recommend this application for the purpose of this tutorial but you can use any other service of your choice that uses APNs.

![](/_images/tutorial_push_notifications_pusher_screenshot1.png "The Pusher app to test remote notifications")

## Generate an SSL Certificate

In order to send push notifications using the Pusher app, we need to generate an SSL certificate. To do this, log in to your Apple Developer account and navigate to your app ID on the **Certificates, IDs & Profiles** page. You should be able to configure the Push Notifications service by clicking on the **Edit** button at the bottom of the list. Follow the instructions on the website to create a certificate as shown on the following screenshot:

![](/_images/tutorial_push_notifications_screenshot5.png "Generate an SSL Certificate on the Apple Developer portal")

Once the certificate is generated and downloaded, double-click on it to add it to your Keychain. You will be able to select it in the Pusher app under the certificate dropdown as it will be listed automatically from your Keychain.

## Google Firebase

> [!TIP]
>If developing for macOS/iOS, please jump to the previous section [Apple Developer](#tutorial_push_notifications_initial_setup_apple_developer) for instructions.

For the purpose of this tutorial, we make use of Google Firebase to test remote notifications on Android. First, create a new project in the Firebase console and add an Android app to the project. Choose a unique package name for your project. If you navigate to the **Cloud Messaging** tab in the project settings, you can access the server sender ID on the following screen:

![](/_images/tutorial_push_notifications_screenshot6.png "Server sender ID in Google Firebase")

Insert this sender ID in the following code snippet of the `MainContentComponent` constructor:

```cpp
#if JUCE_ANDROID
remoteView.sendRemoteMessageButton.onClick = [this] {
    juce::StringPairArray data;
    data.set ("key1", "value1");
    data.set ("key2", "value2");

    static int id = 100;
    juce::PushNotifications::getInstance()->sendUpstreamMessage ("XXXXXXXXXXXX", // Insert sender ID here
        "com.juce.pushnotificationstutorial",
        juce::String (id++),
        "standardType",
        3600,
        data);
};
```

> [!TIP]
>Make sure that the bundle ID in Google Firebase matches the one in your project settings (all in lowercase).

## Generate google-services.json file

We also need to download the Firebase configuration file for remote notifications on Android. Navigate to the **General** tab in the project settings and click on download **google-services.json** as shown in the following screenshot:

![](/_images/tutorial_push_notifications_screenshot7.png "Download json file from Google Firebase")

Copy the file in your JUCE project directory and insert the relative path to the google-services.json file under the Android exporter settings in the **Remote Notifications Config File** field:

![](/_images/tutorial_push_notifications_screenshot8.png "Path for google-services file in exporter settings")

> [!TIP]
>You can also specify a path to google-services.json separately for each target under the \"Debug\" and \"Release\" sections. These will override the settings in the main Android exporter section.

Setup for push notifications should be complete by now and we can finally start implementing these features into the app.

# Notification Types

Notifications are useful to keep users informed with relevant content whether the app is running or not. They can simply display a message, show a dialog box or even play a sound. In general, there are two main types of notifications on all relevant platforms:

- Local: Triggered locally by the system, they do not require internet connection and can be scheduled by the app to fire even when the app is inactive.
- Remote: Pushed by a remote server, they require internet connection but can be sent anytime based on data received from customer devices.

Let's first look at simple local notifications that we can fire within the app.

# Registering as a Listener

The [PushNotifications](https://docs.juce.com/master/classPushNotifications.html "Singleton class responsible for push notifications functionality.") class in JUCE is implemented as a [singleton](https://en.wikipedia.org/wiki/Singleton_pattern) and its global instance can be accessed anytime using the function `PushNotifications::getInstance()` . The [PushNotifications](https://docs.juce.com/master/classPushNotifications.html "Singleton class responsible for push notifications functionality.") class offers an asynchronous interface where results of actions are propagated using the [PushNotifications::Listener](structPushNotifications_1_1Listener.html "Register a listener (ideally on application startup) to receive information about notifications recei...") interface. Simply register a listener with the [PushNotifications](https://docs.juce.com/master/classPushNotifications.html "Singleton class responsible for push notifications functionality.") instance to receive the callbacks. Consequently in the `MainContentComponent` class, we inherit from [PushNotifications::Listener](structPushNotifications_1_1Listener.html "Register a listener (ideally on application startup) to receive information about notifications recei...") [1]:

```cpp
class MainContentComponent : public juce::Component,
                             private juce::ChangeListener,
                             private juce::ComponentListener,
                             private juce::PushNotifications::Listener // [1]
{
```

In the constructor of the `MainContentComponent` class, we add this class as a listener to the [PushNotifications](https://docs.juce.com/master/classPushNotifications.html "Singleton class responsible for push notifications functionality.") instance [2]:

```cpp
juce::PushNotifications::getInstance()->addListener (this); // [2]
```

We also make sure to unsubscribe in the class destructor like so [3]:

```cpp
~MainContentComponent() override
{
    juce::PushNotifications::getInstance()->removeListener (this); // [3]
}
```

# Requesting Permissions

Before sending any type of notification, we first need to request permission to do so from the user. This is requested only once when the app is first launched and the settings remain saved until app deletion or until the user revokes the rights in the system settings. Therefore it is good practice to request permissions on each application startup and only when permission is not granted will the user be asked to grant it. On macOS/iOS, we call the `requestPermissionsWithSettings()` function which takes a [PushNotifications::Settings](structPushNotifications_1_1Settings.html "Describes settings we want to use for current device.") object as an argument [4]. On Android, we call the `setupChannels()` function which takes a [PushNotifications::ChannelGroup](structPushNotifications_1_1ChannelGroup.html "Android API level 26 or higher only: represents a channel group.") object and [PushNotifications::Channel](structPushNotifications_1_1Channel.html "Android API level 26 or higher only: Represents notification channel through which notifications will...") objects as arguments [5].

```cpp
#if JUCE_IOS || JUCE_MAC
paramControls.fireInComboBox.onChange = [this] { delayNotification(); };
juce::PushNotifications::getInstance()->requestPermissionsWithSettings (getNotificationSettings()); // [4]
#elif JUCE_ANDROID
juce::PushNotifications::ChannelGroup cg { "demoGroup", "demo group" };
juce::PushNotifications::getInstance()->setupChannels ({ { cg } }, getAndroidChannels()); // [5]
#endif
}
```

> [!TIP]
>Note that setupChannels() is only required from Android Oreo (API 26) onwards and it is ignored on earlier Android versions.

## Requesting Permissions on macOS/iOS

Let's first take a look at the macOS/iOS side.

There are three different types of permissions we need to request namely Alerts, Badges and Sounds. To facilitate this initialisation, we have created a `getNotificationSettings()` function where we insert this code. On the [PushNotifications::Settings](structPushNotifications_1_1Settings.html "Describes settings we want to use for current device.") object, set the corresponding variables to allow these features [6]. If you are deploying on macOS, this is all you need to request and you can return from the function. However, on iOS we have to define Action and Category objects defined as follows:

- Action: Represents an action that can be presented as a button or a text input to a notification.
- Category: Represents a set of actions that appear together in a notification.

In the same function, define actions and categories as follows:

```cpp
static juce::PushNotifications::Settings getNotificationSettings()
{
    juce::PushNotifications::Settings settings; // [6]
    settings.allowAlert = true;
    settings.allowBadge = true;
    settings.allowSound = true;

#if JUCE_IOS
    using Action = juce::PushNotifications::Settings::Action;
    using Category = juce::PushNotifications::Settings::Category;

    Action okAction;
    okAction.identifier = "okAction";
    okAction.title = "OK!";
    okAction.style = Action::button;
    okAction.triggerInBackground = true;

    Action cancelAction;
    cancelAction.identifier = "cancelAction";
    cancelAction.title = "Cancel";
    cancelAction.style = Action::button;
    cancelAction.triggerInBackground = true;
    cancelAction.destructive = true;

    Action textAction;
    textAction.identifier = "textAction";
    textAction.title = "Enter text";
    textAction.style = Action::text;
    textAction.triggerInBackground = true;
    textAction.destructive = false;
    textAction.textInputButtonText = "Ok";
    textAction.textInputPlaceholder = "Enter text...";

    Category okCategory;
    okCategory.identifier = "okCategory";
    okCategory.actions = { okAction };

    Category okCancelCategory;
    okCancelCategory.identifier = "okCancelCategory";
    okCancelCategory.actions = { okAction, cancelAction };

    Category textCategory;
    textCategory.identifier = "textCategory";
    textCategory.actions = { textAction };
    textCategory.sendDismissAction = true;

    settings.categories = { okCategory, okCancelCategory, textCategory }; // [7]
#endif

    return settings;
}
```

Here we have created three different actions and three different categories:

- OK Action: Specifies the action style as a button with an appropriate \"OK\" title that triggers in the background.
- Cancel Action: Specifies the action style as a button with an appropriate \"Cancel\" title that triggers in the background and appears as destructive.
- Text Action: Specifies the action style as a text input with appropriate placeholders and button text that triggers in the background.
- OK Category: Represents only the \"OK Action\".
- OK/Cancel Category: Represents the \"OK Action\" and the \"Cancel Action\".
- Text Category: Represents only the \"Text Action\" and specifies that a dismiss action will be sent.

All these categories are returned in the [PushNotifications::Settings](structPushNotifications_1_1Settings.html "Describes settings we want to use for current device.") object [7].

## Requesting Permissions on Android

Now let's take a look at the Android side.

There are different channels of importance that we can define with corresponding settings and these channels can in turn be part of channel groups to separate notifications visually. To facilitate this initialisation, we have created a `getAndroidChannels()` function where we insert this code. Create three different [PushNotifications::Channel](structPushNotifications_1_1Channel.html "Android API level 26 or higher only: Represents notification channel through which notifications will...") objects to specify different parameters [8]:

```cpp
static juce::Array<juce::PushNotifications::Channel> getAndroidChannels()
{
    using Channel = juce::PushNotifications::Channel;

    Channel ch1, ch2, ch3;

    ch1.identifier = "1";
    ch1.name = "HighImportance";
    ch1.importance = juce::PushNotifications::Channel::max;
    ch1.lockScreenAppearance = juce::PushNotifications::Notification::showCompletely;
    ch1.description = "High Priority Channel for important stuff";
    ch1.groupId = "demoGroup";
    ch1.ledColour = juce::Colours::red;
    ch1.bypassDoNotDisturb = true;
    ch1.canShowBadge = true;
    ch1.enableLights = true;
    ch1.enableVibration = true;
    ch1.soundToPlay = juce::URL ("demonstrative");
    ch1.vibrationPattern = { 200, 200, 200, 200, 200, 200, 200, 200, 200, 200, 200, 200 };

    ch2.identifier = "2";
    ch2.name = "MediumImportance";
    ch2.importance = juce::PushNotifications::Channel::normal;
    ch2.lockScreenAppearance = juce::PushNotifications::Notification::showPartially;
    ch2.description = "Medium Priority Channel for standard stuff";
    ch2.groupId = "demoGroup";
    ch2.ledColour = juce::Colours::yellow;
    ch2.canShowBadge = true;
    ch2.enableLights = true;
    ch2.enableVibration = true;
    ch2.soundToPlay = juce::URL ("default_os_sound");
    ch2.vibrationPattern = { 1000, 1000 };

    ch3.identifier = "3";
    ch3.name = "LowImportance";
    ch3.importance = juce::PushNotifications::Channel::min;
    ch3.lockScreenAppearance = juce::PushNotifications::Notification::dontShow;
    ch3.description = "Low Priority Channel for silly stuff";
    ch3.groupId = "demoGroup";

    return { ch1, ch2, ch3 };
}
```

- Channel 1: Represents the highest importance in notifications. We can specify parameters such as colours to be red, \"Do Not Disturb\" to be bypassed, a custom vibration pattern to be activated and even a custom sound file to be played.
- Channel 2: Represents a medium importance in notifications. This time we specify colours to be yellow, a different vibration pattern to be activated and a different custom sound file to be played.
- Channel 3: Represents the lowest importance in notifications. We decide to leave the default parameters on these.

All these channels are returned as an array of [PushNotifications::Channel](structPushNotifications_1_1Channel.html "Android API level 26 or higher only: Represents notification channel through which notifications will...") objects belonging to the same [PushNotifications::ChannelGroup](structPushNotifications_1_1ChannelGroup.html "Android API level 26 or higher only: represents a channel group.") [9].

> [!TIP]
>The getAndroidChannels() implementation must configure channels correctly by providing at least an identifier, a name and a group ID per channel.

<!-- -->

> [!NOTE]
> Exercise: Feel free to experiment and change these settings to get familiar with notification permissions on these platforms.

Setup for notifications is now complete on all platforms.

# Local notifications

When the user clicks on the send button in the local notifications tab, the following `sendLocalNotification()` function is called. This function first creates a [PushNotifications::Notification](structPushNotifications_1_1Notification.html "Represents a notification that can be sent or received.") object that we subsequently fill with corresponding parameters from the user interface input [1]. If the notification object is invalid due to incorrect parameters, we optionally show a message for the purpose of this tutorial [2]. Otherwise, if the object is valid we can call the `sendLocalNotification()` function on the [PushNotifications](https://docs.juce.com/master/classPushNotifications.html "Singleton class responsible for push notifications functionality.") instance [3].

```cpp
void sendLocalNotification()
{
    juce::PushNotifications::Notification n; // [1]

    fillRequiredParams (n); // [4]
    fillOptionalParamsOne (n);
#if JUCE_ANDROID
    fillOptionalParamsTwo (n);
    fillOptionalParamsThree (n);
#endif

    if (!n.isValid()) // [2]
    {
#if JUCE_IOS
        juce::String requiredFields = "identifier (from iOS 10), title, body and category";
#elif JUCE_ANDROID
        juce::String requiredFields = "channel ID (from Android O), title, body and icon";
#else
        juce::String requiredFields = "all required fields";
#endif

        juce::NativeMessageBox::showMessageBoxAsync (juce::AlertWindow::InfoIcon,
            "Incorrect notifications setup",
            "Please make sure that "
                + requiredFields + " are set.");

        return;
    }

    juce::PushNotifications::getInstance()->sendLocalNotification (n); // [3]
}
```

For now, we will focus on the required parameters that we fill in the `fillRequiredParams()` function where we pass the previously defined [PushNotifications::Notification](structPushNotifications_1_1Notification.html "Represents a notification that can be sent or received.") object as an argument [4].

> [!TIP]
>If interested by the optional parameters of push notifications, please refer to the last section of this tutorial [Customising notifications](#tutorial_push_notifications_customising_notifications) where we will cover this in depth.

```cpp
void fillRequiredParams (juce::PushNotifications::Notification& n)
{
    n.identifier = paramControls.identifierEditor.getText(); // [5.1]
    n.title = paramControls.titleEditor.getText(); // [5.2]
    n.body = paramControls.bodyEditor.getText(); // [5.3]
#if JUCE_IOS
    n.category = paramControls.categoryComboBox.getText(); // [6.1]
#elif JUCE_ANDROID || JUCE_MAC
    #if JUCE_MAC
    juce::String prefix = "images/";
    juce::String extension = ".png";
    #else
    juce::String prefix;
    juce::String extension;
    #endif
    if (paramControls.iconComboBox.getSelectedItemIndex() == 0) // [7]
        n.icon = prefix + "ic_stat_name" + extension;
    else if (paramControls.iconComboBox.getSelectedItemIndex() == 1)
        n.icon = prefix + "ic_stat_name2" + extension;
    else if (paramControls.iconComboBox.getSelectedItemIndex() == 2)
        n.icon = prefix + "ic_stat_name3" + extension;
    else if (paramControls.iconComboBox.getSelectedItemIndex() == 3)
        n.icon = prefix + "ic_stat_name4" + extension;
    else if (paramControls.iconComboBox.getSelectedItemIndex() == 4)
        n.icon = prefix + "ic_stat_name5" + extension;
#endif

#if JUCE_ANDROID
    // Note: this is not strictly speaking required param, just doing it here because it is the fastest way!
    n.publicVersion = new juce::PushNotifications::Notification(); // [8]
    n.publicVersion->identifier = "blahblahblah";
    n.publicVersion->title = "Public title!";
    n.publicVersion->body = "Public body!";
    n.publicVersion->icon = n.icon;

    n.channelId = String (paramControls.channelIdComboBox.getSelectedItemIndex() + 1); // [6.2]
#endif
}
```

On all platforms, a notification is defined with three main parameters that we set from user input of the required parameter tab:

- Identifier [5.1]: A unique ID to identify notifications internally. This is generally not shown to the user.
- Title [5.2]: The title of the notification shown at the top of the bubble.
- Body [5.3]: The content of the notification that can be expanded to show its totality if trimmed.

On macOS/iOS, we additionally provide the notification category to choose which actions to display [6.1]. On Android, we specify the channel number to specify the importance of the notification [6.2].

On macOS and Android, we need to provide an icon to show within the notification and we make sure that the path is platform-specific [7]. Finally on Android, we can optionally provide a different version of a notification that gets displayed on the device lock screen if we wish [8].

Local notifications are fairly trivial to implement so let's dive into remote notifications now.

# Remote notifications

> [!WARNING]
> This section will only work on a physical device. Do not attempt on a simulator as it will not function properly.

In order to receive remote notifications on macOS/iOS, we first need to get a device token that we will insert in the Pusher application to send test notifications. This token may change at anytime and it is therefore recommended to fetch a new token every time the application is launched. This is performed in the button lambda function by calling the `getDeviceToken()` function on the [PushNotifications](https://docs.juce.com/master/classPushNotifications.html "Singleton class responsible for push notifications functionality.") instance [1]:

```cpp
remoteView.getDeviceTokenButton.onClick = [] {
    juce::String token = juce::PushNotifications::getInstance()->getDeviceToken();

    DBG ("token = " + token);

    if (token.isEmpty())
        showRemoteInstructions();
    else
        juce::NativeMessageBox::showMessageBoxAsync (juce::AlertWindow::InfoIcon, "Device token", token);
};
```

As expected whenever the token is refreshed, a callback function called `deviceTokenRefreshed()` is triggered to notify us of this change:

```cpp
void deviceTokenRefreshed (const juce::String& token) override
{
    juce::NativeMessageBox::showMessageBoxAsync (juce::AlertWindow::InfoIcon,
        "Device token refreshed",
        token);
}
```

## Remote notifications on macOS/iOS

Let's jump to Pusher and see how we can send push notifications to our app on macOS/iOS. Copy the new device token from the debugger and launch the Pusher app. Paste the token in the appropriate **Device push token** field and make sure that the correct certificate is selected in the dropdown menu.

In the text field below, we can create our push notifications as a payload dictionary using the [JSON](https://docs.juce.com/master/classJSON.html "Contains static methods for converting JSON-formatted text to and from var objects.") format. A simple notification will look something like this:

```json
{
  "aps": {
    "alert": "Test Push Notification",
    "badge": 1,
    "sound": "default"
  },
  "juce": "tutorial",
  "foo": "bar"
}
```

The first element called \"aps\" will contain the essential information for our notification. In the above example, it describes the alert, the badge count and the sound to play when triggered. We can also send application specific data as shown in the example with the key/value pairs following the \"aps\" element.

In this second payload example, we customise the alert to contain more information such as a body and an action button in addition to the title field. We also decide to play a custom sound file when the notification is triggered:

```json
{
  "aps": {
    "alert": {
      "title": "Test Push Notification",
      "body": "Hello World!",
      "action-loc-key": "OK"
    },
    "badge": 2,
    "sound": "demonstrative.caf"
  }
}
```

In the third example, we make use of the notification categories defined in the [PushNotifications::Settings](structPushNotifications_1_1Settings.html "Describes settings we want to use for current device.") to easily define actions. We also instruct the OS to deliver the notification silently in the background by setting the `content-available` property:

```json
{
  "aps": {
    "category": "okCategory",
    "alert": "Test Push Notification",
    "badge": 3,
    "content-available": 1
  },
  "foobar": ["foo", "bar"]
}
```

> [!NOTE]
> Exercise: Experiment with different payload parameters and see how they are received within the app. Can you display an image in the notification body?

## Remote notifications on Android

Let's switch to the Android side now and make use of the Google Firebase console. In the console, navigate to the **Grow \> Notifications** tab and click on **Compose message** . In the following window, you will be able to fill in the information for the message and select which target we want to send the notification to.

On Android, we can also send messages upstream from the app to the server to communicate important user data. This is performed by defining a dictionary of key/value pairs as a [StringPairArray](https://docs.juce.com/master/classStringPairArray.html "A container for holding a set of strings which are keyed by another string.") object [1] and calling the `sendUpstreamMessage()` function on the [PushNotifications](https://docs.juce.com/master/classPushNotifications.html "Singleton class responsible for push notifications functionality.") instance [2]:

```cpp
remoteView.sendRemoteMessageButton.onClick = [this] {
    juce::StringPairArray data; // [1]
    data.set ("key1", "value1");
    data.set ("key2", "value2");

    static int id = 100;
    juce::PushNotifications::getInstance()->sendUpstreamMessage ("XXXXXXXXXXXX", // Insert sender ID here
        "com.juce.pushnotificationstutorial",
        String (id++),
        "standardType",
        3600,
        data); // [2]
};
```

Unfortunately, we need to explicitly pass the previously set server sender ID as an argument every time to do this.

When an upstream message is sent to the server we can expect a callback with the following `upstreamMessageSent()` function if the request was successful:

```cpp
void upstreamMessageSent (const juce::String& messageId) override
{
    juce::NativeMessageBox::showMessageBoxAsync (juce::AlertWindow::InfoIcon,
        "Upstream message sent",
        "Message id: " + messageId);
}
```

However if the request is unsuccessful, we receive the callback with the `upstreamMessageSendingError()` function instead:

```cpp
void upstreamMessageSendingError (const juce::String& messageId, const juce::String& error) override
{
    juce::NativeMessageBox::showMessageBoxAsync (juce::AlertWindow::InfoIcon,
        "Upstream message sending error",
        "Message id: " + messageId
            + "\nerror: " + error);
}
```

If the Google Firebase receives too many messages, it may start deleting them from the queue of pending messages. When this happens, we are notified via the `remoteNotificationsDeleted()` function:

```cpp
void remoteNotificationsDeleted() override
{
    juce::NativeMessageBox::showMessageBoxAsync (juce::AlertWindow::InfoIcon,
        "Remote notifications deleted",
        "Some of the pending messages were removed!");
}
```

On Android we can also subscribe and unsubscribe to specific topics by calling the `subscribeToTopic()` and `unsubscribeFromTopic()` functions respectively on the [PushNotifications](https://docs.juce.com/master/classPushNotifications.html "Singleton class responsible for push notifications functionality.") instance. In this example we choose to subscribe to and unsubscribe from the \"sports\" topic:

```cpp
remoteView.subscribeToSportsButton.onClick = [this] { juce::PushNotifications::getInstance()->subscribeToTopic ("sports"); };
remoteView.unsubscribeFromSportsButton.onClick = [this] { juce::PushNotifications::getInstance()->unsubscribeFromTopic ("sports"); };
```

# Handling received notifications

There is a number of different callbacks that can be invoked when a user acts on a notification. The exact behaviour may vary between platforms and OS versions so please refer to the [PushNotifications](https://docs.juce.com/master/classPushNotifications.html "Singleton class responsible for push notifications functionality.") class documentation for a detailed explanation.

The `handleNotification()` callback function will be invoked whenever a user presses on a notification. Normally you would use this function to process information from the notification based on the ID but for the purpose of this tutorial we simply show a message box with the three main parameters as follows:

```cpp
void handleNotification (bool isLocalNotification, const juce::PushNotifications::Notification& n) override
{
    juce::ignoreUnused (isLocalNotification);

    juce::NativeMessageBox::showMessageBoxAsync (juce::AlertWindow::InfoIcon,
        "Received notification",
        "ID: " + n.identifier
            + ", title: " + n.title
            + ", body: " + n.body);
}
```

The `handleNotificationAction()` callback function will be invoked whenever a user performs an action from a notification (such as pressing a button or entering a text input). This callback contains additional information pertaining to the type of action and an optional response in the form of text input for example. In this scenario, we also need to manually remove the notification from the history by providing the notification identifer to the `removeDeliveredNotification()` function as an argument and by calling it on the [PushNotifications](https://docs.juce.com/master/classPushNotifications.html "Singleton class responsible for push notifications functionality.") instance [9]:

```cpp
void handleNotificationAction (bool isLocalNotification,
    const juce::PushNotifications::Notification& n,
    const juce::String& actionIdentifier,
    const juce::String& optionalResponse) override
{
    juce::ignoreUnused (isLocalNotification);

    juce::NativeMessageBox::showMessageBoxAsync (juce::AlertWindow::InfoIcon,
        "Received notification action",
        "ID: " + n.identifier
            + ", title: " + n.title
            + ", body: " + n.body
            + ", action: " + actionIdentifier
            + ", optionalResponse: " + optionalResponse);

    juce::PushNotifications::getInstance()->removeDeliveredNotification (n.identifier);
}
```

As the name suggests, the following callback is triggered when the user dismisses a local notification before responding to it and we display the same message box as before:

```cpp
void localNotificationDismissedByUser (const juce::PushNotifications::Notification& n) override
{
    juce::NativeMessageBox::showMessageBoxAsync (juce::AlertWindow::InfoIcon,
        "Notification dismissed by a user",
        "ID: " + n.identifier
            + ", title: " + n.title
            + ", body: " + n.body);
}
```

If the user ignores a notification or does not act upon it, the notification stays in the list of delivered ones in the notification area of the device. To retrieve the list of delivered notifications, we can call the `getDeliveredNotifications()` function on the [PushNotifications](https://docs.juce.com/master/classPushNotifications.html "Singleton class responsible for push notifications functionality.") instance. A callback function named `deliveredNotificationsListReceived()` will subsequently be called and we can handle it by displaying the list in a message box as follows:

```cpp
void deliveredNotificationsListReceived (const juce::Array<juce::PushNotifications::Notification>& notifs) override
{
    juce::String text = "Received notifications: ";

    for (auto& n : notifs)
        text << "(" << n.identifier << ", " << n.title << ", " << n.body << "), ";

    juce::NativeMessageBox::showMessageBoxAsync (juce::AlertWindow::InfoIcon, "Received notification list", text);
}
```

On macOS/iOS, the application can schedule local notifications to be triggered at a given moment in the future. This convenient callback function named `pendingLocalNotificationsListReceived()` will receive an array of pending notifications when the `getPendingLocalNotifications()` function is called on the [PushNotifications](https://docs.juce.com/master/classPushNotifications.html "Singleton class responsible for push notifications functionality.") instance:

```cpp
void pendingLocalNotificationsListReceived (const juce::Array<juce::PushNotifications::Notification>& notifs) override
{
    juce::String text = "Pending notifications: ";

    for (auto& n : notifs)
        text << "(" << n.identifier << ", " << n.title << ", " << n.body << "), ";

    juce::NativeMessageBox::showMessageBoxAsync (juce::AlertWindow::InfoIcon, "Pending notification list", text);
}
```

> [!NOTE]
> Exercise: Experiment with different parameters and display these in the message boxes. Can you display the notification icon in the callback message box?

The core of this tutorial has been demonstrated and all callback functions from the [PushNotifications::Listener](structPushNotifications_1_1Listener.html "Register a listener (ideally on application startup) to receive information about notifications recei...") class were overriden but if you wish to learn more about optional notification parameters, please feel free to read on!

# Customising notifications

Notifications have numerous optional parameters that we can set in addition to the ones we have discovered so far in this tutorial. This optional section will cover these parameters in detail.

This first function called in the `sendLocalNotification()` function fills some optional parameters supported by all platforms:

```cpp
void fillOptionalParamsOne (juce::PushNotifications::Notification& n)
{
    n.subtitle = paramControls.subtitleEditor.getText(); // [1.1]
    n.badgeNumber = paramControls.badgeNumberComboBox.getSelectedItemIndex(); // [1.2]

    if (paramControls.soundToPlayComboBox.getSelectedItemIndex() > 0)
        n.soundToPlay = juce::URL (paramControls.soundToPlayComboBox.getItemText (paramControls.soundToPlayComboBox.getSelectedItemIndex())); // [1.3]

    n.properties = juce::JSON::parse (paramControls.propertiesEditor.getText()); // [1.4]

#if JUCE_IOS || JUCE_MAC
    n.triggerIntervalSec = double (paramControls.fireInComboBox.getSelectedItemIndex() * 10);
    n.repeat = paramControls.repeatButton.getToggleState();
#elif JUCE_ANDROID
    n.badgeIconType = (juce::PushNotifications::Notification::BadgeIconType) paramControls.badgeIconComboBox.getSelectedItemIndex(); // [2.1]
    n.tickerText = paramControls.tickerTextEditor.getText(); // [2.2]

    n.shouldAutoCancel = paramControls.autoCancelButton.getToggleState(); // [2.3]
    n.alertOnlyOnce = paramControls.alertOnlyOnceButton.getToggleState(); // [2.4]
#endif

#if JUCE_ANDROID || JUCE_MAC
    if (paramControls.actionsComboBox.getSelectedItemIndex() == 1)
    {
        juce::PushNotifications::Notification::Action a, a2;
        a.style = juce::PushNotifications::Notification::Action::button;
        a2.style = juce::PushNotifications::Notification::Action::button;
        a.title = a.identifier = "Ok";
        a2.title = a2.identifier = "Cancel";
        n.actions.add (a);
        n.actions.add (a2);
    }
    else if (paramControls.actionsComboBox.getSelectedItemIndex() == 2)
    {
        juce::PushNotifications::Notification::Action a, a2;
        a.title = a.identifier = "Input Text Here";
        a2.title = a2.identifier = "No";
        a.style = juce::PushNotifications::Notification::Action::text;
        a2.style = juce::PushNotifications::Notification::Action::button;
        a.icon = "ic_stat_name4";
        a2.icon = "ic_stat_name5";
        a.textInputPlaceholder = "placeholder text ...";
        n.actions.add (a);
        n.actions.add (a2);
    }
    else if (paramControls.actionsComboBox.getSelectedItemIndex() == 3)
    {
        juce::PushNotifications::Notification::Action a, a2;
        a.title = a.identifier = "Ok";
        a2.title = a2.identifier = "Cancel";
        a.style = juce::PushNotifications::Notification::Action::button;
        a2.style = juce::PushNotifications::Notification::Action::button;
        a.icon = "ic_stat_name4";
        a2.icon = "ic_stat_name5";
        n.actions.add (a);
        n.actions.add (a2);
    }
    else if (paramControls.actionsComboBox.getSelectedItemIndex() == 4)
    {
        juce::PushNotifications::Notification::Action a, a2;
        a.title = a.identifier = "Input Text Here";
        a2.title = a2.identifier = "No";
        a.style = juce::PushNotifications::Notification::Action::text;
        a2.style = juce::PushNotifications::Notification::Action::button;
        a.icon = "ic_stat_name4";
        a2.icon = "ic_stat_name5";
        a.textInputPlaceholder = "placeholder text ...";
        a.allowedResponses.add ("Response 1");
        a.allowedResponses.add ("Response 2");
        a.allowedResponses.add ("Response 3");
        n.actions.add (a);
        n.actions.add (a2);
    }
#endif
}
```

- Subtitle [1.1]: Additional text that can be displayed in the notification.
- Badge Number [1.2]: A number representing the count displayed in the badge icon before it is summed with other delivered notification badge numbers.
- Sound [1.3]: A custom sound to play when the notification is triggered. You can leave this field empty if you want the notification to be silent or use \"default_os_sound\" to trigger the default OS sound.
- Properties [1.4]: Additional properties that may be passed as a dictionary.

On macOS and iOS, you can also delay the trigger of a notification by a specified amount in seconds and decide to repeat the notification in question. On Android, you can provide a large image to display in the content of the notification and specify these additional parameters:

- Badge Icon Type [2.1]: The size of the badge icon or whether we want to hide it.
- Ticker Text [2.2]: Additional text used for accessibility.
- Auto Cancel [2.3]: Whether the notification should be canceled when clicked on it.
- Alert Once [2.4]: Whether the notification should only sound and vibrate when not already showing.

If you wish to have action buttons displayed in the notifications like on iOS, in Android and macOS we have to define them manually. This setup is essentially similar to the iOS one using the [PushNotifications::Settings](structPushNotifications_1_1Settings.html "Describes settings we want to use for current device.") object so we will not go into detail.

The next sets of parameters are Android only.

```cpp
void fillOptionalParamsTwo (juce::PushNotifications::Notification& n)
{
    using Notification = juce::PushNotifications::Notification;

    Notification::Progress progress;
    progress.max = paramControls.progressMaxComboBox.getSelectedItemIndex() * 10;
    progress.current = paramControls.progressCurrentComboBox.getSelectedItemIndex() * 10;
    progress.indeterminate = paramControls.progressIndeterminateButton.getToggleState();

    n.progress = progress; // [3.1]
    n.person = paramControls.personEditor.getText(); // [3.2]
    n.type = Notification::Type (paramControls.categoryComboBox.getSelectedItemIndex()); // [3.3]
    n.priority = Notification::Priority (paramControls.priorityComboBox.getSelectedItemIndex() - 2); // [3.4]
    n.lockScreenAppearance = Notification::LockScreenAppearance (paramControls.lockScreenVisibilityComboBox.getSelectedItemIndex() - 1); // [3.5]
    n.groupId = paramControls.groupIdEditor.getText(); // [3.6]
    n.groupSortKey = paramControls.sortKeyEditor.getText();
    n.groupSummary = paramControls.groupSummaryButton.getToggleState();
    n.groupAlertBehaviour = Notification::GroupAlertBehaviour (paramControls.groupAlertBehaviourComboBox.getSelectedItemIndex());
}
```

- Progress [3.1]: [Displays](https://docs.juce.com/master/classDisplays.html "Manages details about connected display devices.") specific types of notifications that show progress.
- Person [3.2]: Relates the notification to a specific person. This can be useful for instance in messaging apps.
- Type [3.3]: Specifies the category of notification for the OS to better handle the appearance.
- Priority [3.4]: Specifies the priority of notification to the OS to better handle the appearance.
- Lock Screen Appearance [3.5]: Whether to show or hide the notification on the lock screen.
- Group [3.6]: Specifies parameters relating to groups of notifications such as sorting order, group summary and group alert behaviour.

```cpp
void fillOptionalParamsThree (juce::PushNotifications::Notification& n)
{
    n.accentColour = paramControls.accentColourButton.findColour (juce::TextButton::buttonColourId, false); // [4.1]
    n.ledColour = paramControls.ledColourButton.findColour (juce::TextButton::buttonColourId, false); // [4.2]

    using Notification = juce::PushNotifications::Notification;
    Notification::LedBlinkPattern ledBlinkPattern;
    ledBlinkPattern.msToBeOn = paramControls.ledMsToBeOnComboBox.getSelectedItemIndex() * 200;
    ledBlinkPattern.msToBeOff = paramControls.ledMsToBeOffComboBox.getSelectedItemIndex() * 200;
    n.ledBlinkPattern = ledBlinkPattern; // [4.3]

    juce::Array<int> vibrationPattern;

    if (paramControls.vibratorMsToBeOnComboBox.getSelectedItemIndex() > 0 && paramControls.vibratorMsToBeOffComboBox.getSelectedItemIndex() > 0)
    {
        vibrationPattern.add (paramControls.vibratorMsToBeOffComboBox.getSelectedItemIndex() * 500);
        vibrationPattern.add (paramControls.vibratorMsToBeOnComboBox.getSelectedItemIndex() * 500);
        vibrationPattern.add (2 * paramControls.vibratorMsToBeOffComboBox.getSelectedItemIndex() * 500);
        vibrationPattern.add (2 * paramControls.vibratorMsToBeOnComboBox.getSelectedItemIndex() * 500);
    }

    n.vibrationPattern = vibrationPattern; // [4.4]

    n.localOnly = paramControls.localOnlyButton.getToggleState(); // [4.5]
    n.ongoing = paramControls.ongoingButton.getToggleState(); // [4.6]
    n.timestampVisibility = Notification::TimestampVisibility (paramControls.timestampVisibilityComboBox.getSelectedItemIndex()); // [4.7]

    if (paramControls.timeoutAfterComboBox.getSelectedItemIndex() > 0)
    {
        auto index = paramControls.timeoutAfterComboBox.getSelectedItemIndex();
        n.timeoutAfterMs = index * 1000 + 4000; // [4.8]
    }
}
```

- Accent Colour [4.1]: Changes the accent colour of the notification.
- LED Colour [4.2]: Changes the colour of the physical LED on the back of the device.
- LED Blink Pattern [4.3]: Allows customisation of the blinking pattern of the physical LED.
- Vibration Pattern [4.4]: Allows customisation of the vibration pattern of the physical device.
- Local [4.5]: Whether the notification should be broadcast to other connected user devices.
- Ongoing [4.6]: Whether the user is allowed to dismiss the notification or the system can only dismiss it manually.
- Timestamp [4.7]: Whether a timestamp or chronometer is displayed in the notification.
- Timeout [4.8]: Specifies the time after which the notification is automatically canceled if not already.

> [!TIP]
>The source code for this modified version of the code can be found in the `PushNotificationsTutorial_02.h` file of the demo project.

# Summary

In this tutorial, we have learnt how to handle Push Notifications on mobile and desktop. In particular, we have:

- Covered preliminary setup for different deployment platforms.
- Requested user permissions from the system to allow notifications.
- Displayed simple messages as local notifications from within the app.
- Handled push notifications that were sent from a remote server.
- Customised notifications by showing pictures and playing sounds.

# See also

- [Tutorial: Getting started with Android](/tutorials/tutorial_android_studio/)
- [Tutorial: Managing Android screen sizes](/tutorials/tutorial_android_screen_sizes/)
- [Tutorial: App analytics collection](/tutorials/tutorial_analytics_collection/)
- [Tutorial: In-App Purchases on desktop and mobile devices](/tutorials/tutorial_in_app_purchases/)
- [Tutorial: Package your app or plugin for distribution](/tutorials/tutorial_app_plugin_packaging/)
