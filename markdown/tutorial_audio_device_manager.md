---
title: The AudioDeviceManager class
---
# Tutorial: The AudioDeviceManager class

This tutorial introduces the [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices.") class which is used for managing audio devices on all platforms. This allows you to configure things such as the device sample rate and the number of inputs and outputs.

**LEVEL:** Intermediate<br/>
**PLATFORMS:** Windows, macOS, Linux, iOS, Android<br/>
**CLASSES:** [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices."), [AudioDeviceSelectorComponent](https://docs.juce.com/master/classAudioDeviceSelectorComponent.html "A component containing controls to let the user change the audio settings of an AudioDeviceManager ob..."), [ChangeListener](https://docs.juce.com/master/classChangeListener.html "Receives change event callbacks that are sent out by a ChangeBroadcaster."), [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.")

# Getting started

> [!TIP]
>This tutorial assumes that you have a basic grasp of using the [AudioAppComponent](https://docs.juce.com/master/classAudioAppComponent.html "A base class for writing audio apps that stream from the audio i/o devices.") class for audio input and output (see [Tutorial: Processing audio input](/tutorials/tutorial_processing_audio_input/)). It also assumes that you are familiar with basic GUI layout techniques (see [Tutorial: Parent and child components](/tutorials/tutorial_component_parents_children/)).

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/AudioDeviceManagerTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/AudioDeviceManagerTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project is based on the Audio Application template from the Projucer. It presents an [AudioDeviceSelectorComponent](https://docs.juce.com/master/classAudioDeviceSelectorComponent.html "A component containing controls to let the user change the audio settings of an AudioDeviceManager ob...") object which allows you to configure your audio device settings. The demo project also presents a simple text console that reports the current audio device settings. The application also shows the current CPU usage of the audio processing element of the application.

> [!TIP]
>The code presented here is broadly similar to the **AudioSettingsDemo** from the JUCE Demo. The main difference is that the audio generated is the same as in the [Tutorial: Processing audio input](/tutorials/tutorial_processing_audio_input/) (this ring-modulates the audio input with white noise).

![](/_images/tutorial_audio_device_manager_screenshot1.png "An AudioDeviceSelectorComponent object showing settings for an AudioDeviceManager object")

# Audio Devices

JUCE provides a consistent means of accessing audio devices on all of the platforms it supports. While the demo application provided here may be deployed only to desktop platforms, this is only due to GUI layout constraints. Audio works seamlessly on mobile platforms, too.

In the Audio Application template, the [AudioAppComponent](https://docs.juce.com/master/classAudioAppComponent.html "A base class for writing audio apps that stream from the audio i/o devices.") class instantiates an [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices.") object `deviceManager` --- it is a public member, therefore it is accessible from your subclass. The [AudioAppComponent](https://docs.juce.com/master/classAudioAppComponent.html "A base class for writing audio apps that stream from the audio i/o devices.") class also performs some basic initialisation of this [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices.") object --- this happens when you call [AudioAppComponent::setAudioChannels()](https://docs.juce.com/master/classAudioAppComponent.html#a1cb6b457848fa80df7efc39cf095d465 "A subclass should call this from their constructor, to set up the audio.").

The [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices.") class will attempt to use the default audio device unless this is overridden. This may be configured in code or via the [AudioDeviceSelectorComponent](https://docs.juce.com/master/classAudioDeviceSelectorComponent.html "A component containing controls to let the user change the audio settings of an AudioDeviceManager ob...") as illustrated here. Device settings and preferences may be stored and recalled on subsequent applications launches. The [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices.") class can also fallback to the default device in such circumstances if the preferred device is no longer available (for example, if an external audio device has been unplugged since the last launch).

The [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices.") class is also a hub for incoming MIDI messages. This is explored in other tutorials (see [Tutorial: Handling MIDI events](/tutorials/tutorial_handling_midi_events/)).

The [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices.") class can broadcast changes to its settings as it inherits from the [ChangeBroadcaster](https://docs.juce.com/master/classChangeBroadcaster.html "Holds a list of ChangeListeners, and sends messages to them when instructed.") class. The right-hand side of our component posts some of the important audio device settings whenever a change in our [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices.") object is triggered.

## The AudioDeviceSelectorComponent class

The [AudioDeviceSelectorComponent](https://docs.juce.com/master/classAudioDeviceSelectorComponent.html "A component containing controls to let the user change the audio settings of an AudioDeviceManager ob...") class provides convenient way of configuring audio devices on all platforms. As mentioned above, this is displayed in the right-hand side of our user interface for the demo project. When the [AudioDeviceSelectorComponent](https://docs.juce.com/master/classAudioDeviceSelectorComponent.html "A component containing controls to let the user change the audio settings of an AudioDeviceManager ob...") object is constructed we pass it the [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices.") object that we want it to control along with a number of other options including the number of channels to support (see the [AudioDeviceSelectorComponent](https://docs.juce.com/master/classAudioDeviceSelectorComponent.html "A component containing controls to let the user change the audio settings of an AudioDeviceManager ob...") class [constructor](https://docs.juce.com/master/classAudioDeviceSelectorComponent.html#a859af6ef664a974dc7d2ce3d24d8f94c) for more information). Here we create the [AudioDeviceSelectorComponent](https://docs.juce.com/master/classAudioDeviceSelectorComponent.html "A component containing controls to let the user change the audio settings of an AudioDeviceManager ob...") object by passing it the [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices.") object that is the member of the [AudioAppComponent](https://docs.juce.com/master/classAudioAppComponent.html "A base class for writing audio apps that stream from the audio i/o devices."), allowing up to 256 input and output channels, hiding MIDI configuration, and showing our channels as single channels rather than stereo pairs:

```cpp
MainContentComponent()
    : audioSetupComp (deviceManager,
          0, // minimum input channels
          256, // maximum input channels
          0, // minimum output channels
          256, // maximum output channels
          false, // ability to select midi inputs
          false, // ability to select midi output device
          false, // treat channels as stereo pairs
          false) // hide advanced options
{
```

Our interface is configured to allow control over:

- Selecting the output device.
- Selecting the input device.
- Enabling and disabling the input and output channels.
- Choosing a sample rate (that is supported by the devices).
- Choosing an audio buffer size (block size).

You should notice that when we change any of these settings we get a new list of data posted into our little console window on the right-hand side of the interface. This is because the [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices.") class is a type of [ChangeBroadcaster](https://docs.juce.com/master/classChangeBroadcaster.html "Holds a list of ChangeListeners, and sends messages to them when instructed.") class.

In our `changeListenerCallback()` function we call our `dumpDeviceInfo()` function that accesses the [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices.") object to retrieve the current audio device. We then obtain various pieces of information about the device:

```cpp
void dumpDeviceInfo()
{
    logMessage ("--------------------------------------");
    logMessage ("Current audio device type: " + (deviceManager.getCurrentDeviceTypeObject() != nullptr ? deviceManager.getCurrentDeviceTypeObject()->getTypeName() : "<none>"));

    if (auto* device = deviceManager.getCurrentAudioDevice())
    {
        logMessage ("Current audio device: " + device->getName().quoted());
        logMessage ("Sample rate: " + juce::String (device->getCurrentSampleRate()) + " Hz");
        logMessage ("Block size: " + juce::String (device->getCurrentBufferSizeSamples()) + " samples");
        logMessage ("Bit depth: " + juce::String (device->getCurrentBitDepth()));
        logMessage ("Input channel names: " + device->getInputChannelNames().joinIntoString (", "));
        logMessage ("Active input channels: " + getListOfActiveBits (device->getActiveInputChannels()));
        logMessage ("Output channel names: " + device->getOutputChannelNames().joinIntoString (", "));
        logMessage ("Active output channels: " + getListOfActiveBits (device->getActiveOutputChannels()));
    }
    else
    {
        logMessage ("No audio device open");
    }
}
```

> [!TIP]
>We use our `getListOfActiveBits()` function to convert the [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") object that represents the list of active channels as a *bitmask* into a [String](https://docs.juce.com/master/classString.html "The JUCE String class!") object. The [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") object is used as a bitmask, similar to [std::bitset](http://www.cplusplus.com/reference/bitset/bitset/) or [std::vector\<bool\>](http://www.cplusplus.com/reference/vector/vector-bool/) . Here the channels are represented by a 0 (inactive) or 1 (active) in the bits comprising the [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") value. See [Tutorial: The BigInteger class](/tutorials/tutorial_big_integer/) for other operations that can be performed on [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") objects.

Responding to such changes is really useful in a real application as we will often need to know when the number of channels available to our application changes. Often, it is important to respond appropriately to changes in sample rate and the other audio parameters.

> [!NOTE]
> Exercise: Experiment with different settings for the [AudioDeviceSelectorComponent](https://docs.juce.com/master/classAudioDeviceSelectorComponent.html "A component containing controls to let the user change the audio settings of an AudioDeviceManager ob...") class constructor --- this may be especially useful if you have a multichannel soundcard where you can limit the number of channels that can be addressed by your app. You can also see how you can view the pairs of channels as stereo pairs.

The [AudioDeviceSelectorComponent](https://docs.juce.com/master/classAudioDeviceSelectorComponent.html "A component containing controls to let the user change the audio settings of an AudioDeviceManager ob...") class also contains a **Test** button. This plays a sine tone out of the device outputs, which is useful for users to test that audio output is working on their target device.

## CPU usage

We obtain the CPU usage for the audio processing element of our application by calling the [AudioDeviceManager::getCpuUsage()](https://docs.juce.com/master/classAudioDeviceManager.html#a79563c9d2cc87d059d8b101515e8dee1 "Returns the average proportion of available CPU being spent inside the audio callbacks.") function. In our `MainContentComponent` class we inherit from the [Timer](https://docs.juce.com/master/classTimer.html "Makes repeated callbacks to a virtual method at a specified time interval.") class, start the timer to trigger every 50ms. In our `timerCallback()` function we obtain the CPU usage from the [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices.") object. This value is displayed in a [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") object as a percentage (to six decimal places):

```cpp
void timerCallback() override
{
    auto cpu = deviceManager.getCpuUsage() * 100;
    cpuUsageText.setText (juce::String (cpu, 6) + " %", juce::dontSendNotification);
}
```

Since we are doing very little audio processing in this particular application, the CPU usage is likely to be very low --- probably less than 1% on many target devices. You can, however, see the effect that different sample rates and buffer size have on the CPU load by trying a combination of settings. In general, higher sample rates and smaller buffer sizes will use more CPU.

# Summary

This tutorial has introduced some of the features of the [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices.") class and audio devices in general. You should now know how to:

- Access the [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices.") object from within the [AudioAppComponent](https://docs.juce.com/master/classAudioAppComponent.html "A base class for writing audio apps that stream from the audio i/o devices.") object using the Audio Application template.
- Create a [AudioDeviceSelectorComponent](https://docs.juce.com/master/classAudioDeviceSelectorComponent.html "A component containing controls to let the user change the audio settings of an AudioDeviceManager ob...") object to control an [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices.") object.
- Respond to changes in audio devices.
- Access useful settings from with audio devices such as the sample rate and number of channels available.

# See also

- [Tutorial: Processing audio input](/tutorials/tutorial_processing_audio_input/)
