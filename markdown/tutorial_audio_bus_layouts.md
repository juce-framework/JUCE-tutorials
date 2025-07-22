---
title: Configuring the right bus layouts for your plugins
---
# Tutorial: Configuring the right bus layouts for your plugins

Learn how to configure and restrict the bus layouts in your plugins for a multitude of configurations. Check the bus arrangements of your AU plugins using the auval tool provided in Apple's Logic Pro.

**LEVEL:** Beginner<br/>
**PLATFORMS:** Windows, macOS, Linux
**Plugin Format:** VST, VST3, AU, Standalone<br/>
**CLASSES:** [AudioProcessor::BusesLayout](structAudioProcessor_1_1BusesLayout.html "Represents the bus layout state of a plug-in."), [AudioProcessor::BusesProperties](structAudioProcessor_1_1BusesProperties.html "Structure used for AudioProcessor Callbacks."), [AudioChannelSet](https://docs.juce.com/master/classAudioChannelSet.html "Represents a set of audio channel types."), [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.")

# Getting started

To follow this tutorial, please make sure that you have a plugin project opened or alternatively create a new **Audio Plugin** project within the Projucer.

If you need help with this step, see [Tutorial: Create a basic Audio/MIDI plugin, Part 1: Setting up](/tutorials/tutorial_create_projucer_basic_plugin/).

> [!WARNING]
> To build an AudioUnit plugin and test it using the auval tool, you will need to build on macOS and have Logic Pro X installed.

# Introduction

The [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.") class in JUCE takes a number of input channels, processes the incoming data and then supplies the result to a number of output channels as represented in the following figure:

![](/_images/tutorial_audio_bus_layouts_screenshot1.png "Plugin with I/O channels")

For many types of audio processors, this is sufficient. However, you often want to represent these individual channels as a group in order to facilitate the communication between the host and the processor. For example, the first four input channels could be used as main channels and the last two input channels could be used as a sidechain, as shown below:

![](/_images/tutorial_audio_bus_layouts_screenshot2.png "Plugin with I/O buses")

A yellow arrow in the figure above is called a _Bus_. A bus is therefore a group of channels which typically share the same signal path inside your plugin, for example the main audio signal path or the sidechain signal path. In JUCE each bus has a name so that you can identify its specific purpose in the plugin.

> [!TIP]
>In almost every DAW, the first input bus or first output bus is referred to as the *main* bus. In JUCE you will sometimes find comments or method names referring to the main bus which is simply the first input or output bus.

However, grouping your audio processor's input and output channels and naming these groups is not sufficient. You also need to convey information about spatial positioning for each channel inside each bus. For example, you could configure your bus as a stereo bus (Left, Right), a quadraphonic bus (Left, Right, Left Surround, Right Surround) or a mono bus (Centre) etc. In JUCE, each bus is assigned an [AudioChannelSet](https://docs.juce.com/master/classAudioChannelSet.html "Represents a set of audio channel types.") that describes spatial positions for these sets of channels (for example: [AudioChannelSet::mono()](https://docs.juce.com/master/classAudioChannelSet.html#a17a0a8ab8f24eebf463baf8d99a39617 "Creates a one-channel mono set (centre)."), [AudioChannelSet::stereo()](https://docs.juce.com/master/classAudioChannelSet.html#a2f5463f3941cf8ca3ecf1eadf1537c19 "Creates a set containing a stereo set (left, right)."), [AudioChannelSet::quadraphonic()](https://docs.juce.com/master/classAudioChannelSet.html#ab216c3d458cbd3965e321a29d12d591d "Creates a set for quadraphonic surround setup (left, right, leftSurround, rightSurround)")).

It's important to note that the [AudioChannelSet](https://docs.juce.com/master/classAudioChannelSet.html "Represents a set of audio channel types.") of each bus is not fixed throughout the lifetime of a plugin. A DAW may request a different [AudioChannelSet](https://docs.juce.com/master/classAudioChannelSet.html "Represents a set of audio channel types.") for a particular bus at any time, for example when the DAW user switches the sidechain source from mono to stereo. To implement this, JUCE supports a callback to interrogate the plugin on which [AudioChannelSet](https://docs.juce.com/master/classAudioChannelSet.html "Represents a set of audio channel types.") objects each bus supports.

> [!TIP]
>There is one special [AudioChannelSet](https://docs.juce.com/master/classAudioChannelSet.html "Represents a set of audio channel types.") called [AudioChannelSet::disabled()](https://docs.juce.com/master/classAudioChannelSet.html#a18675dc100238ac22868c41717ef2ffe "Creates a zero-channel set which can be used to indicate that a bus is disabled.") which does not contain any channel. This [AudioChannelSet](https://docs.juce.com/master/classAudioChannelSet.html "Represents a set of audio channel types.") is used to indicate that a particular bus is not used at the moment, for instance when a sidechain is not connected in a DAW.

In summary, to communicate information about each channel of a plugin at any given time, the DAW and plugin must:

- Exchange information on the grouping of channels into buses.
- Exchange information on the name and current [AudioChannelSet](https://docs.juce.com/master/classAudioChannelSet.html "Represents a set of audio channel types.") of each bus.
- Negotiate which [AudioChannelSet](https://docs.juce.com/master/classAudioChannelSet.html "Represents a set of audio channel types.") objects are supported on each bus.

Let's start by exploring how to convey this information to the host as a plugin developer.

# Assigning BusesProperties

We will first focus on the first two points (grouping and naming of channels into buses and selecting an [AudioChannelSet](https://docs.juce.com/master/classAudioChannelSet.html "Represents a set of audio channel types.") for each bus) when writing a plugin. This information is communicated to the DAW by passing an [AudioProcessor::BusesProperties](structAudioProcessor_1_1BusesProperties.html "Structure used for AudioProcessor Callbacks.") instance to the constructor of the [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.") as follows:

```cpp
ExampleAudioProcessor()
    : AudioProcessor (BusesProperties().withInput ("Input", juce::AudioChannelSet::stereo(), true).withOutput ("Output", juce::AudioChannelSet::stereo(), true))
{
    //...
```

The code above informs the DAW that this plugin has two buses: one input bus named \"Input\" and one output bus named \"Output\". The second parameter in the `withInput()` and `withOutput()` functions describes the initial [AudioChannelSet](https://docs.juce.com/master/classAudioChannelSet.html "Represents a set of audio channel types.") of the respective bus. It's important to remember that the DAW can change this at any time. The third parameter indicates whether the bus is initially enabled or disabled. Since these two buses are main buses, it would not make much sense to disable them.

The previous example would typically be used in an effects plugin. However, a synth plugin would typically only have an output bus and therefore the constructor would look like this:

```cpp
ExampleAudioProcessor()
    : AudioProcessor (BusesProperties().withOutput ("Output", juce::AudioChannelSet::stereo(), true))
{
    //...
```

Another plugin type supported in JUCE is the midi effects plugin. As these only process midi, they don\'t have any audio bus at all:

```cpp
ExampleAudioProcessor()
    : AudioProcessor (BusesProperties())
{
    //...
```

> [!TIP]
>The above constructor code should not be confused with invoking the default [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.") constructor. For backward compatibility, the default constructor will create a plugin with one initially disabled input bus named \"Input\" and one initially disabled output bus named \"Output\" --- each as an [AudioChannelSet::stereo()](https://docs.juce.com/master/classAudioChannelSet.html#a2f5463f3941cf8ca3ecf1eadf1537c19 "Creates a set containing a stereo set (left, right).").

# Configuring the BusesLayout

The only step required now is to configure the [AudioChannelSet](https://docs.juce.com/master/classAudioChannelSet.html "Represents a set of audio channel types.") objects that a particular bus supports as the DAW may wish to change the initial [AudioChannelSet](https://docs.juce.com/master/classAudioChannelSet.html "Represents a set of audio channel types.") provided in the constructor at any time. It's important to note that there is no way for a plugin to actively change its own layout: the plugin is passive and will always be \"at the mercy\" of the host. It can only reject or accept requested [AudioChannelSet](https://docs.juce.com/master/classAudioChannelSet.html "Represents a set of audio channel types.") configurations.

To do this, the plugin needs to override the [AudioProcessor::isBusesLayoutSupported()](https://docs.juce.com/master/classAudioProcessor.html#a6e206c1d2987a250a2d6c2af4c3af01a "Callback to query if the AudioProcessor supports a specific layout.") callback. This callback takes a single BusesLayout parameter which is simply an array of [AudioChannelSet](https://docs.juce.com/master/classAudioChannelSet.html "Represents a set of audio channel types.") objects for each bus of the plugin.

> [!TIP]
>The number of elements in the arrays of supported input and output buses will always match the number of buses you specified in your [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.") constructor.

In your [AudioProcessor::isBusesLayoutSupported()](https://docs.juce.com/master/classAudioProcessor.html#a6e206c1d2987a250a2d6c2af4c3af01a "Callback to query if the AudioProcessor supports a specific layout.") callback, you need to return whether the given BusesLayout is supported or not. For example, the default callback is to accept any BusesLayout that the DAW requests as shown here:

```cpp
bool isBusesLayoutSupported (const BusesLayout& layouts) const
{
    return true;
}
```

Most effects plugins will only have a single main input and output bus. Typically, they require the [AudioChannelSet](https://docs.juce.com/master/classAudioChannelSet.html "Represents a set of audio channel types.") to be the same on the input and output side like so:

```cpp
bool isBusesLayoutSupported (const BusesLayout& layouts) const
{
    return layouts.getMainInputChannelSet() == layouts.getMainOutputChannelSet();
}
```

Remember that there is a special [AudioChannelSet](https://docs.juce.com/master/classAudioChannelSet.html "Represents a set of audio channel types.") called [AudioChannelSet::disabled()](https://docs.juce.com/master/classAudioChannelSet.html#a18675dc100238ac22868c41717ef2ffe "Creates a zero-channel set which can be used to indicate that a bus is disabled.") which indicates that a specific bus is disabled. Most effects plugins will not want their main buses to be disabled, so we can include a check for that in our callback as follows:

```cpp
bool isBusesLayoutSupported (const BusesLayout& layouts) const
{
    if (layouts.getMainInputChannelSet() == juce::AudioChannelSet::disabled()
        || layouts.getMainOutputChannelSet() == juce::AudioChannelSet::disabled())
        return false;

    return layouts.getMainInputChannelSet() == layouts.getMainOutputChannelSet();
}
```

Let's say that the plugin can only deal with mono-to-mono or stereo-to-stereo configurations. The last line of the above code already checks that the input and output layout must be the same, therefore we only need to do the preceding checks on either the input or output bus like this:

```cpp
bool isBusesLayoutSupported (const BusesLayout& layouts) const
{
    if (layouts.getMainInputChannelSet() == juce::AudioChannelSet::disabled()
        || layouts.getMainOutputChannelSet() == juce::AudioChannelSet::disabled())
        return false;

    if (layouts.getMainOutputChannelSet() != juce::AudioChannelSet::mono()
        && layouts.getMainOutputChannelSet() != juce::AudioChannelSet::stereo())
        return false;

    return layouts.getMainInputChannelSet() == layouts.getMainOutputChannelSet();
}
```

# Testing the BusesLayout

The bus layout configurations that we support in our plugins can be tested using different tools as it can be very easy to overlook edge cases in the layouts defined in the isBusesLayoutSupported() function.

In this tutorial we present two solutions, the **auval** tool included in Apple's Logic Pro to test and validate AudioUnits and the Audio Plugin Host included within JUCE that supports VST and AU plugins.

## Testing With the auval Tool

> [!TIP]
>This section covers tools that are only available on macOS for AudioUnits.

The **auval** tool validates AudioUnits before they can be loaded in Apple's plugin hosts such as Logic Pro X and Final Cut Pro X. It performs a variety of useful tests on your plugins but for the sake of this tutorial, we focus on the bus layout tests.

The **auval** tool can be accessed within Logic Pro X by navigating to **Logic Pro X \> Preferences \> Plug-in Manager...** and invoking the validation by clicking on **Reset & Rescan Selection** as shown here:

![](/_images/tutorial_audio_bus_layouts_screenshot3.png "The Plug-in Manager in Logic Pro X")

However we can also invoke it through the command line like so:

```cpp
auval - v aufx Test Manu
```

The first argument denotes the AU main type which defaults to \'aufx\' when implementing an effects plugin, the second and third arguments are the plugin code and the manufacturer code as specified in the Projucer settings of your project:

![](/_images/tutorial_audio_bus_layouts_screenshot4.png "The Projucer plugin settings")

As an example, the output of an AU validation looks something like the following:

```
AU Validation Tool
Version: 1.6.1a1
Copyright 2003-2013, Apple Inc. All Rights Reserved.
Specify -h (-help) for command options

--------------------------------------------------
VALIDATING AUDIO UNIT: 'aufx' - 'Test' - 'Manu'
--------------------------------------------------
Manufacturer String: Manu
AudioUnit Name: TestPlugin
Component Version: 1.0.0 (0x10000)

* PASS
--------------------------------------------------
FORMAT TESTS:

Reported Channel Capabilities (explicit):
[1, 1] [2, 2]

Input/Output Channel Handling:
1-1 1-2 1-4 1-5 1-6 1-7 1-8 2-2 2-4 2-5 2-6 2-7 2-8 4-4 4-5 5-5 6-6 7-7 8-8
X X

# # AudioChannelLayouts (5), Input Scope:
//...

Is Audio Channel Layout Available:
Mono Stereo Binau. AU_4 Ambi. AU_5 AU_5_0 AU_6 AU_6_0 AU_7_0 AU_7_0F AU_8 AU_5_1 AU_6_1 AU_7_1 AU_7_1F
X X X

//...

# # AudioChannelLayouts (5), Output Scope:
//...

Is Audio Channel Layout Available:
Mono Stereo Binau. AU_4 Ambi. AU_5 AU_5_0 AU_6 AU_6_0 AU_7_0 AU_7_0F AU_8 AU_5_1 AU_6_1 AU_7_1 AU_7_1F
X X X

//...

* PASS
--------------------------------------------------
AU VALIDATION SUCCEEDED.
--------------------------------------------------
```

As you can see in the above output, we can confirm that our test plugin only accepts mono-to-mono and stereo-to-stereo configurations which is what we want.

> [!WARNING]
> If using macOS versions between 10.12.0 and 10.13.1, auval contains a bug that may report the wrong bus layouts for your plugins.

## Testing With the Plugin Host

The other useful tool is the Audio Plugin Host included with JUCE that can be found in the `JUCE/examples` folder of the library. This tool allows you to test VST and AU plugins by changing the bus layouts at runtime.

> [!TIP]
>If you need help with building and using the Audio Plugin Host, please refer to [Tutorial: Create a basic Audio/MIDI plugin, Part 1: Setting up](/tutorials/tutorial_create_projucer_basic_plugin/) for detailed instructions.

Open the Audio Plugin Host and load an instance of the plugin you want to test. If you right click on the plugin and select **Configure Audio I/O** , you should see the following window appearing:

![](/_images/tutorial_audio_bus_layouts_screenshot5.png "Audio I/O settings of a plugin")

Here you can add or remove input or output buses, if your plugin allows it, change the channel set of the selected bus and enable/disable buses at your convenience.

Notice here that when you lead the plugin for the first time, you can see the input and output pins displaying the default BusesLayout configuration set in the [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.") constructor.

> [!NOTE]
> Exercise: Experiment with different bus layouts and check the behaviour of your plugin in the **auval** tool and the Audio Plugin Host.

# Summary

After reading this tutorial, you should be able to:

- Define input, output and sidechain buses with different channel sets.
- Configure your preferred bus layouts for your plugin type.
- Check the bus arrangement configurations of AudioUnits using the auval tool.
- Assign different bus properties in the JUCE audio plugin host.

# See also

- [Tutorial: Create a basic Audio/MIDI plugin, Part 1: Setting up](/tutorials/tutorial_create_projucer_basic_plugin/)
- [Tutorial: Create a basic Audio/MIDI plugin, Part 2: Coding your plug-in](/tutorials/tutorial_code_basic_plugin/)
- [Tutorial: Plugin examples](/tutorials/tutorial_plugin_examples/)
- [Tutorial: Cascading plug-in effects](/tutorials/tutorial_audio_processor_graph/)
