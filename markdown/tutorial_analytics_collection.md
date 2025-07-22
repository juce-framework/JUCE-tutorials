---
title: App analytics collection
---
# Tutorial: App analytics collection

Collect app usage data from users in JUCE applications. Send analytics events to Google Analytics using the analytics module.

**LEVEL:** Intermediate<br/>
**PLATFORMS:** Windows, macOS, Linux, iOS, Android<br/>
**CLASSES:** [ThreadedAnalyticsDestination](https://docs.juce.com/master/classThreadedAnalyticsDestination.html "A base class for dispatching analytics events on a dedicated thread."), [ButtonTracker](https://docs.juce.com/master/classButtonTracker.html "A class that automatically sends analytics events to the Analytics singleton when a button is clicked..."), [WebInputStream](https://docs.juce.com/master/classWebInputStream.html "An InputStream which can be used to read from a given URL."), [CriticalSection](https://docs.juce.com/master/classCriticalSection.html "A re-entrant mutex."), [CriticalSection::ScopedLockType](https://docs.juce.com/master/classCriticalSection.html#a4d398019b977a4c2151c6f1e2a06210b "Provides the type of scoped lock to use with a CriticalSection.")

> [!WARNING]
> This project requires a Google Analytics account. If you need help with this, follow the instructions on the [Google Analytics website](https://support.google.com/analytics/) to open an account.

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/AnalyticsCollectionTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/AnalyticsCollectionTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

Please make sure you have your Google Analytics API key written down and ready for this tutorial to fully work.

# The demo project

The demo project shows a very simple UI with two buttons sending analytics events when pressed. Since the API key has not been set up yet, Google will not receive any events before implementation.

![](/_images/tutorial_analytics_collection_screenshot1.png "The demo project app window")

> [!TIP]
>This project uses Google Analytics to track app analytics but you can apply this to any other service you wish to use.The code presented here is broadly similar to the **AnalyticsCollection** from the JUCE Examples.

# Anatomy of events

Events describe how the user has interacted with the content in applications and are sent to the analytics tracking system. To better categorise and filter the interactions, events are structured using the following keywords:

- Category: Describes groups of events that are combined under the analytics reports.
- Action: Specifies the action that was performed to trigger the event.
- Label: Additional information explaining the specific object that interacted with the user.
- Value: [Optional](https://docs.juce.com/master/classOptional.html "A simple optional type.") integer to provide numerical data to the event in question.

All the events are sent with a unique user ID and a timestamp along with the keywords mentioned above. Additionally, users can be grouped into categories to better describe their capacity such as beta testers or developers.

# API key set up

The first step for the project to work properly is to set up the Google Analytics API key. You can find the Tracking ID in your Google Analytics dashboard here:

![](/_images/tutorial_analytics_collection_screenshot2.png "Google Analytics Tracking ID")

Copy this ID, and replace the `apiKey` placeholder variable in the `GoogleAnalyticsDestination` class:

```cpp
apiKey = "UA-XXXXXXXXX-1";
```

> [!WARNING]
> Ideally, this API key should not be visible in your binary distribution as there could be all sorts of malicious uses if discovered and may pollute your analytics data with spam. One way to prevent this would be to retrieve the API key dynamically at runtime (such as from your own server).

# Tracking app startup

Let's first start by tracking user-independent information such as app launches and define constant user information that will be used by the analytics system. In the constructor of the `MainContentComponent` class, we start by getting a reference to the [Analytics](https://docs.juce.com/master/classAnalytics.html "A singleton class to manage analytics data.") singleton by calling `Analytics::getInstance()` .

We can then set the user ID with `setUserID()` by choosing a unique identifier for this user [1]. Make sure not to include any sensitive personal information in this identifier. We can also set a user group on this user by calling `setUserProperties()` using a [StringPairArray](https://docs.juce.com/master/classStringPairArray.html "A container for holding a set of strings which are keyed by another string.") [2].

For the events to be received, we need to specify at least one destination to our [Analytics](https://docs.juce.com/master/classAnalytics.html "A singleton class to manage analytics data.") instance. We can optionally add multiple destinations if we wish. In this case we add an instance of the GoogleAnalyticsDestination class to the singleton [3].

Since the `MainContentComponent` constructor gets called when the MainWindow is instantiated, we can log this event using the function `logEvent()` right when the component gets owned by the MainWindow [4].

```cpp
MainContentComponent()
{
    // Add an analytics identifier for the user. Make sure you don't accidentally
    // collect identifiable information if you haven't asked for permission!
    juce::Analytics::getInstance()->setUserId ("AnonUser1234"); // [1]

    // Add any other constant user information.
    juce::StringPairArray userData;
    userData.set ("group", "beta");
    juce::Analytics::getInstance()->setUserProperties (userData); // [2]

    // Add any analytics destinations we want to use to the Analytics singleton.
    juce::Analytics::getInstance()->addDestination (new GoogleAnalyticsDestination()); // [3]

    // The event type here should probably be DemoAnalyticsEventTypes::sessionStart
    // in a more advanced app.
    juce::Analytics::getInstance()->logEvent ("startup", {}, DemoAnalyticsEventTypes::event); // [4]
```

Likewise, we can log the shutdown event in the `MainContentComponent` destructor right when the MainWindow gets deleted [5].

```cpp
~MainContentComponent() override
{
    // The event type here should probably be DemoAnalyticsEventTypes::sessionEnd
    // in a more advanced app.
    juce::Analytics::getInstance()->logEvent ("shutdown", {}, DemoAnalyticsEventTypes::event); // [5]
}
```

# Tracking Button behaviour

In order to add tracking to specific user actions, we need to define which user interactions we want recorded and sent. Fortunately to record button behaviour, we can use a handy class included in the JUCE analytics module called [ButtonTracker](https://docs.juce.com/master/classButtonTracker.html "A class that automatically sends analytics events to the Analytics singleton when a button is clicked...") that will automatically handle this for us.

Let's first declare a [ButtonTracker](https://docs.juce.com/master/classButtonTracker.html "A class that automatically sends analytics events to the Analytics singleton when a button is clicked...") as a member variable in the `MainContentComponent` class [1].

```cpp
juce::TextButton eventButton { "Press me!" }, crashButton { "Simulate crash!" };
std::unique_ptr<juce::ButtonTracker> logEventButtonPress; // [1]

JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR (MainContentComponent)
};
```

Now in the MainContentComponent constructor, we can link the specific [TextButton](https://docs.juce.com/master/classTextButton.html "A button that uses the standard lozenge-shaped background with a line of text on it.") object we want to track by passing it as an argument to the [ButtonTracker](https://docs.juce.com/master/classButtonTracker.html "A class that automatically sends analytics events to the Analytics singleton when a button is clicked...") constructor. We also set the event category and action properties to send when the event is fired [2].

```cpp
juce::StringPairArray logButtonPressParameters;
logButtonPressParameters.set ("id", "a");
logEventButtonPress.reset (new juce::ButtonTracker (eventButton, "button_press", logButtonPressParameters)); // [2]
}
```

> [!NOTE]
> Exercise: Create additional GUI components and implement tracking on them with different event parameters.

# Sending events

The JUCE analytics module handles the logging of events on a dedicated thread and sends the analytics data in batches periodically. Therefore, we need to temporarily store the events on local storage until the data is sent. In the rest of this tutorial, we will be working in the `GoogleAnalyticsDestination` class.

We first need to specify a location to store our analytics event data in the application data directory. For this we use the special location [`File::userApplicationDataDirectory`](https://docs.juce.com/master/classFile.html#a3e19cafabb03c5838160263a6e76313da0c9f89d8dc9f9f32c9eb42428385351d "The folder in which applications store their persistent user-specific settings.") to find the correct location and navigate to the corresponding application folder for our app [1]. If the location does not exist we create the folder [2] and save the file path as an XML file name extension [3].

We can now start the thread by using the `startAnalyticsThread()` function and specifying the waiting time between batches of events in milliseconds [4].

```cpp
GoogleAnalyticsDestination()
    : ThreadedAnalyticsDestination ("GoogleAnalyticsThread")
{
    {
        // Choose where to save any unsent events.

        auto appDataDir = juce::File::getSpecialLocation (juce::File::userApplicationDataDirectory)
                              .getChildFile (juce::JUCEApplication::getInstance()->getApplicationName()); // [1]

        if (!appDataDir.exists())
            appDataDir.createDirectory(); // [2]

        savedEventsFile = appDataDir.getChildFile ("analytics_events.xml"); // [3]
    }
```

```cpp
startAnalyticsThread (initialPeriodMs); // [4]
}
```

In the class destructor, we have to ensure that the last batch of events can be sent without the application being killed by the operating system. To allow this, we provide one last batch period while sleeping the thread before stopping it forcibly after 1 second. This provides enough time for one last sending attempt without elongating too much the application shutdown time.

```cpp
~GoogleAnalyticsDestination() override
{
    // Here we sleep so that our background thread has a chance to send the
    // last lot of batched events. Be careful - if your app takes too long to
    // shut down then some operating systems will kill it forcibly!
    juce::Thread::sleep (initialPeriodMs); // [5]

    stopAnalyticsThread (1000); // [6]
}
```

We can supply the maximum number of events to send in batches by overriding the `getMaximumBatchSize()` function like so:

```cpp
int getMaximumBatchSize() override { return 20; }
```

## Formatting the HTTP request

Now we need to format the correct HTTP request to log these events to the analytics server. The [URL](https://docs.juce.com/master/classURL.html "Represents a URL and has a bunch of useful functions to manipulate it.") we are trying to construct with its corresponding POST data in the case of a button press behaviour for example looks something like this:

```
POST /batch HTTP/1.1
Host: www.google-analytics.com

v=1 // Version Number
&aip=1 // Anonymise IP
&tid=UA-XXXXXXXXX-1 // Tracking ID
&t=event // Log Type
&ec=button_press // Event Category
&ea=a // Event Action
&cid=AnonUser1234 // User ID
```

- [v]: The batch logging API version.
- [aip]: The IP address of the sender is anonymised.
- [tid]: The Tracking ID for the corresponding app.
- [t]: The type of logging for the analytics system.
- [ec]: The category identifier for the logged event.
- [ea]: The action identifier for the logged event.
- [cid]: The user ID for the corresponding user.

In a typical app lifecycle, the batched logger will first process the appStarted event when the application is fired up. Then when the user clicks on the button we log the button_press event and finally log the appStopped event when the application quits.

In order to account for these 3 logging scenarios, we need to construct different requests in the `logBatchedEvents()` function:

```cpp
bool logBatchedEvents (const juce::Array<AnalyticsEvent>& events) override
{
    // Send events to Google Analytics.

    juce::String appData ("v=1&aip=1&tid=" + apiKey); // [1]

    juce::StringArray postData;

    for (auto& event : events) // [2]
    {
        juce::StringPairArray data;

        switch (event.eventType)
        {
            case (DemoAnalyticsEventTypes::event):
            {
                data.set ("t", "event");

                if (event.name == "startup")
                {
                    data.set ("ec", "info");
                    data.set ("ea", "appStarted");
                }
                else if (event.name == "shutdown")
                {
                    data.set ("ec", "info");
                    data.set ("ea", "appStopped");
                }
                else if (event.name == "button_press")
                {
                    data.set ("ec", "button_press");
                    data.set ("ea", event.parameters["id"]);
                }
                else if (event.name == "crash")
                {
                    data.set ("ec", "crash");
                    data.set ("ea", "crash");
                }
                else
                {
                    jassertfalse;
                    continue;
                }

                break;
            }

            default:
            {
                // Unknown event type! In this demo app we're just using a
                // single event type, but in a real app you probably want to
                // handle multiple ones.
                jassertfalse;
                break;
            }
        }

        data.set ("cid", event.userID); // [3]

        juce::StringArray eventData;

        for (auto& key : data.getAllKeys()) // [4]
            eventData.add (key + "=" + juce::URL::addEscapeChars (data[key], true));

        postData.add (appData + "&" + eventData.joinIntoString ("&")); // [5]
    }

    auto url = juce::URL ("https://www.google-analytics.com/batch")
                   .withPOSTData (postData.joinIntoString ("\n")); // [6]
```

- [1]: We start by adding the version number, anonymised IP and tracking ID to the appData string variable.
- [2]: Then for each event in the batch, we determine the type of event in question to set its category and action properties. If the event is a startup or a shutdown, we set the event category to \"info\" and set the action property to \"appStarted\" or \"appStopped\" respectively. If the event is a button pressing, we set the event category to \"button_press\" and retrieve its action property from the id parameter of the [ButtonTracker](https://docs.juce.com/master/classButtonTracker.html "A class that automatically sends analytics events to the Analytics singleton when a button is clicked...").
- [3]: We also set the user ID for the event to log.
- [4]: Now for all the individual [StringPairArray](https://docs.juce.com/master/classStringPairArray.html "A container for holding a set of strings which are keyed by another string.") entries, we concatenate keys with their corresponding values by inserting an equal sign in between and escaping any special characters from the [URL](https://docs.juce.com/master/classURL.html "Represents a URL and has a bunch of useful functions to manipulate it.").
- [5]: Finally, we can join all the event parameters together with ampersand signs in between and by prepending the initial appData content to the front.
- [6]: The [URL](https://docs.juce.com/master/classURL.html "Represents a URL and has a bunch of useful functions to manipulate it.") is eventually constructed with its POST data appended line by line. This way we can send multiple events in a single HTTP request.

> [!NOTE]
> Exercise: Modify the code above to handle all event properties including label and value attributes.

Now that we have our [URL](https://docs.juce.com/master/classURL.html "Represents a URL and has a bunch of useful functions to manipulate it.") ready we need to send the request to the server by creating a [WebInputStream](https://docs.juce.com/master/classWebInputStream.html "An InputStream which can be used to read from a given URL."). We first have to lock the [CriticalSection](https://docs.juce.com/master/classCriticalSection.html "A re-entrant mutex.") mutex declared as a member variable called webStreamCreation. Using a ScopedLock object allows us to automatically lock and unlock the mutex for the piece of code delimited by the curly brackets [1].

If the `stopLoggingEvents()` function was previously called due to the application terminating, we return immediately without attempting to initialise the [WebInputStream](https://docs.juce.com/master/classWebInputStream.html "An InputStream which can be used to read from a given URL.") [2]. Otherwise, we can create it in a std::unique_ptr by passing the previously constructed [URL](https://docs.juce.com/master/classURL.html "Represents a URL and has a bunch of useful functions to manipulate it.") as an argument and using POST as the method [3].

We can then connect to the specified [URL](https://docs.juce.com/master/classURL.html "Represents a URL and has a bunch of useful functions to manipulate it.") and perform the request using the `connect()` function on the [WebInputStream](https://docs.juce.com/master/classWebInputStream.html "An InputStream which can be used to read from a given URL.") [4]. If the response is successful, we just return positively from the function. Otherwise, we set an exponential decay on the batch period by multiplying the previous rate by 2 and return negatively from the function [5].

```cpp
{
    const juce::ScopedLock lock (webStreamCreation); // [1]

    if (shouldExit) // [2]
        return false;

    webStream.reset (new juce::WebInputStream (url, true)); // [3]
}

auto success = webStream->connect (nullptr); // [4]

// Do an exponential backoff if we failed to connect.
if (success)
    periodMs = initialPeriodMs;
else
    periodMs *= 2;

setBatchPeriod (periodMs); // [5]

return success;
}
```

When the application shuts down, we need to cancel connections to the [WebInputStream](https://docs.juce.com/master/classWebInputStream.html "An InputStream which can be used to read from a given URL.") if there are any that are concurrently running. By first acquiring the lock from the same [CriticalSection](https://docs.juce.com/master/classCriticalSection.html "A re-entrant mutex.") object using a ScopedLock, we ensure that the previously encountered critical section of the code in the `logBatchedEvents()` function will have terminated before [1]. Setting the shouldExit boolean to true prevents any new connections from being created subsequently [2]. Then we can finally cancel any [WebInputStream](https://docs.juce.com/master/classWebInputStream.html "An InputStream which can be used to read from a given URL.") connections using the `cancel()` function if there are any [3].

```cpp
void stopLoggingEvents() override
{
    const juce::ScopedLock lock (webStreamCreation); // [1]

    shouldExit = true; // [2]

    if (webStream.get() != nullptr) // [3]
        webStream->cancel();
}
```

This completes the part of the tutorial dealing with logging events. However, if the transmission of event data fails and the application terminates, we currently have no way of keeping track of unlogged events.

# Save and restore unlogged events

This section will cover the use of XML files to store any unlogged events to disk in the case of a lost connection.

The XML document storing unlogged event information will look something like this for a single button press:

```xml
<?xml version="1.0" ?>
<events> // Root XML element for the whole document.
<google_analytics_event
    name="button_press"
    type="event"
    timestamp="xxxx"
    user_id="AnonUser1234"
  > // Event node with name, type, timestamp and user ID.
<parameters id="a" /> // Parameters related to the parent event.
<user_properties group="beta" /> // Properties for the user in the parent event.
</google_analytics_event>
//...
</events>
```

We will look at the `saveUnloggedEvents()` and `restoreUnloggedEvents()` functions that deal with saving and restoring events respectively. The `saveUnloggedEvents()` function will build an XML structure based on the format shown above and save the content in an XML file:

```cpp
void saveUnloggedEvents (const std::deque<AnalyticsEvent>& eventsToSave) override
{
    // Save unsent events to disk. Here we use XML as a serialisation format, but
    // you can use anything else as long as the restoreUnloggedEvents method can
    // restore events from disk. If you're saving very large numbers of events then
    // a binary format may be more suitable if it is faster - remember that this
    // method is called on app shutdown so it needs to complete quickly!

    juce::XmlDocument previouslySavedEvents (savedEventsFile);
    std::unique_ptr<juce::XmlElement> xml (previouslySavedEvents.getDocumentElement()); // [1]

    if (xml.get() == nullptr || xml->getTagName() != "events") // [2]
        xml.reset (new juce::XmlElement ("events"));

    for (auto& event : eventsToSave)
    {
        auto* xmlEvent = new juce::XmlElement ("google_analytics_event"); // [3]
        xmlEvent->setAttribute ("name", event.name);
        xmlEvent->setAttribute ("type", event.eventType);
        xmlEvent->setAttribute ("timestamp", (int) event.timestamp);
        xmlEvent->setAttribute ("user_id", event.userID);

        auto* parameters = new juce::XmlElement ("parameters"); // [4]

        for (auto& key : event.parameters.getAllKeys())
            parameters->setAttribute (key, event.parameters[key]);

        xmlEvent->addChildElement (parameters);

        auto* userProperties = new juce::XmlElement ("user_properties"); // [5]

        for (auto& key : event.userProperties.getAllKeys())
            userProperties->setAttribute (key, event.userProperties[key]);

        xmlEvent->addChildElement (userProperties);

        xml->addChildElement (xmlEvent); // [6]
    }

    xml->writeTo (savedEventsFile); // [7]
}
```

- [1]: First we retrieve any previously saved events from the XML file stored at the previously defined file location and build an [XmlElement](https://docs.juce.com/master/classXmlElement.html "Used to build a tree of elements representing an XML document.") based on it.
- [2]: If the [XmlElement](https://docs.juce.com/master/classXmlElement.html "Used to build a tree of elements representing an XML document.") does not exist or does not have the root \"events\" node, we create it.
- [3]: For each unsaved event in the queue, we create a \"google_analytics_event\" node with the event name, type, timestamp and user ID as attributes.
- [4]: We also create a \"parameters\" node as a child node to the previously created one with event parameters as attributes to it.
- [5]: At the same hierarchy level, we create a \"user_properties\" node as a child node with user properties as attributes to it.
- [6]: We can then add the individual event nodes as children to the root \"events\" node.
- [7]: Finally, we write the XML structure to the XML file and store the events.

On the other hand, the `restoreUnloggedEvents()` function will in turn read an XML structure based on the same format shown previously and fill up the event queue:

```cpp
void restoreUnloggedEvents (std::deque<AnalyticsEvent>& restoredEventQueue) override
{
    juce::XmlDocument savedEvents (savedEventsFile);
    std::unique_ptr<juce::XmlElement> xml (savedEvents.getDocumentElement()); // [1]

    if (xml.get() == nullptr || xml->getTagName() != "events") // [2]
        return;

    auto numEvents = xml->getNumChildElements();

    for (auto iEvent = 0; iEvent < numEvents; ++iEvent)
    {
        auto* xmlEvent = xml->getChildElement (iEvent); // [3]

        juce::StringPairArray parameters;
        auto* xmlParameters = xmlEvent->getChildByName ("parameters"); // [4]
        auto numParameters = xmlParameters->getNumAttributes();

        for (auto iParam = 0; iParam < numParameters; ++iParam)
            parameters.set (xmlParameters->getAttributeName (iParam),
                xmlParameters->getAttributeValue (iParam));

        juce::StringPairArray userProperties;
        auto* xmlUserProperties = xmlEvent->getChildByName ("user_properties"); // [5]
        auto numUserProperties = xmlUserProperties->getNumAttributes();

        for (auto iProp = 0; iProp < numUserProperties; ++iProp)
            userProperties.set (xmlUserProperties->getAttributeName (iProp),
                xmlUserProperties->getAttributeValue (iProp));

        restoredEventQueue.push_back ({ xmlEvent->getStringAttribute ("name"), // [6]
            xmlEvent->getIntAttribute ("type"),
            static_cast<juce::uint32> (xmlEvent->getIntAttribute ("timestamp")),
            parameters,
            xmlEvent->getStringAttribute ("user_id"),
            userProperties });
    }

    savedEventsFile.deleteFile(); // [7]
}
```

- [1]: Same as before, we retrieve any previously saved events from the XML file stored at the previously defined file location and build an [XmlElement](https://docs.juce.com/master/classXmlElement.html "Used to build a tree of elements representing an XML document.") based on it.
- [2]: If the [XmlElement](https://docs.juce.com/master/classXmlElement.html "Used to build a tree of elements representing an XML document.") does not exist or does not have the root \"events\" node, we return from the function as there is nothing to do.
- [3]: We first retrieve a single event child node to parse from the root parent.
- [4]: For each attribute from the child \"parameters\" node, we set a key/value pair and add it to a [StringPairArray](https://docs.juce.com/master/classStringPairArray.html "A container for holding a set of strings which are keyed by another string.").
- [5]: For each attribute from the child \"user_properties\" node, we set a key/value pair and add it to a [StringPairArray](https://docs.juce.com/master/classStringPairArray.html "A container for holding a set of strings which are keyed by another string.").
- [6]: We can then push the individual events back into the event queue by setting the corresponding parameters from the [StringPairArray](https://docs.juce.com/master/classStringPairArray.html "A container for holding a set of strings which are keyed by another string.") objects.
- [7]: Finally, we delete the XML file from disk when done.

> [!TIP]
>We used XML as a serialisation format but if we need to save large amounts of unsaved events, a binary format would be more efficient.

<!-- -->

> [!NOTE]
> Exercise: Save and restore unlogged events in a different serialisation format such as [JSON](https://docs.juce.com/master/classJSON.html "Contains static methods for converting JSON-formatted text to and from var objects.") or in a binary format.

# Summary

In this tutorial, we have learnt how to track usage data with Google Analytics and the JUCE analytics module. In particular, we have:

- Sent analytics events to Google Analytics on a separate thread.
- Stored unsent events locally in an XML document.
- Restored saved events from the XML document to the event queue.

# See also

- [Tutorial: Getting started with Android](/tutorials/tutorial_android_studio/)
- [Tutorial: Managing Android screen sizes](/tutorials/tutorial_android_screen_sizes/)
- [Tutorial: In-App Purchases on desktop and mobile devices](/tutorials/tutorial_in_app_purchases/)
- [Tutorial: Push Notifications on desktop and mobile devices](/tutorials/tutorial_push_notifications/)
- [Tutorial: Package your app or plugin for distribution](/tutorials/tutorial_app_plugin_packaging/)
