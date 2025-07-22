---
title: Understanding MPE zones
---
# Tutorial: Understanding MPE zones

Learn the concept of zones and the conventions for zone layouts as defined in the MPE standard. Hook up your MPE synthesiser to an MPE-compatible device.

**LEVEL:** Intermediate<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**CLASSES:** [MPEZoneLayout](https://docs.juce.com/master/classMPEZoneLayout.html "This class represents the current MPE zone layout of a device capable of handling MPE."), MPEZoneLayout::Zone, [MPEMessages](https://docs.juce.com/master/classMPEMessages.html "This helper class contains the necessary helper functions to generate MIDI messages that are exclusiv..."), [MPESynthesiser](https://docs.juce.com/master/classMPESynthesiser.html "Base class for an MPE-compatible musical device that can play sounds.")

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/MPEZonesTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/MPEZonesTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

> [!TIP]
>It would be helpful to read [Tutorial: Build a multi-polyphonic synthesiser](/tutorials/tutorial_mpe_introduction/) first, as this is used as a reference point in a number of places.

# The demo project

The demo project is similar to the `MPEDemo` project in the `JUCE/examples` directory and builds upon the simplified version in [Tutorial: Build a multi-polyphonic synthesiser](/tutorials/tutorial_mpe_introduction/). In order to get the most out of this tutorial you will need an [MPE](https://support.roli.com/article/what-is-mpe/) compatible controller. [MPE](https://support.roli.com/article/what-is-mpe/) stands for _MIDI Polyphonic Expression_, a specification that allows multidimensional data to be communicated between audio products.

Some examples of such [MPE](https://support.roli.com/article/what-is-mpe/) compatible devices are ROLI's own Seaboard range (such as the [Seaboard RISE](https://roli.com/products/seaboard-rise) ) and the BLOCKS range (such as the [Lightpad Block](https://roli.com/products/lightpad-block) ).

With a [Lightpad Block](https://roli.com/products/lightpad-block) connected to your computer the window of the demo application should look something like the following screenshot:

![](/_images/tutorial_mpe_zones_screenshot1.png "The demo application")

You will need to enable one of the MIDI inputs (here you can see a [Lightpad Block](https://roli.com/products/lightpad-block) is shown as an option).

Any notes played on your [MPE](https://support.roli.com/article/what-is-mpe/) compatible device will be visualised in the lower portion of the window as explained in the [Tutorial: Build a multi-polyphonic synthesiser](/tutorials/tutorial_mpe_introduction/) tutorial.

# MPE Specifications

In the introductory tutorial [Tutorial: Build a multi-polyphonic synthesiser](/tutorials/tutorial_mpe_introduction/), we were able to implement an MPE-compatible synthesiser easily by bypassing the standard MPE configuration process using legacy mode. In this tutorial, let's configure our synthesiser by following the procedure described in the latest MPE standard.

For the synthesiser to be able to determine whether we are working in MPE mode or not pertains to the notion of MPE zones. If at least one zone is defined using an MPE Configuration Message (MCM), then we are in MPE mode. Otherwise, if no zones are defined, then MPE mode is off. So what are zones?

## MPE Zones

The concept of zones is an MPE-specific term describing a group of contiguous MIDI channels comprising of one Master Channel and one or more Member Channels.

A Master Channel receives messages that apply to the entire zone whereas Member Channels receive messages that only apply individually.

In MPE, we can have at most two zones and they are defined as Lower and Upper zones.

- _Lower Zone_: Channel 1 as the Master Channel and one or more Member Channels allocated increasing from channel 2.
- _Upper Zone_: Channel 16 as the Master Channel and one or more Member Channels allocated descending from channel 15.

A Member Channel can only belong to one zone at a time and the most recent MCM takes precedent over previous ones.

As an example, we can set the Lower Zone to contain channels 2 to 10 and the Upper Zone to contain channels 11 to 15.

Alternatively, we can also restrict ourselves to a single zone, using either the Lower or Upper Zone. However, it is recommended to use the Lower Zone by default. In a single zone scenario, the remaining unused Master Channel can be used as a Member Channel for the other zone, resulting in a maximum of 15 Member Channels.

An MPE zone can be turned off by sending an MCM without any member channels to the zone's master channel and MPE mode is therefore turned off when all zones are empty.

In JUCE, the implementation of a zone is encapsulated in the MPEZoneLayout::Zone struct and different zone configurations can be defined using the [MPEZoneLayout](https://docs.juce.com/master/classMPEZoneLayout.html "This class represents the current MPE zone layout of a device capable of handling MPE.") class.

Zones are a useful means to provide different timbral characteristics using only one MPE controller and facilitates the propagation of MIDI messages across groups of channels using Master Channels.

## MIDI Modes

There are mainly two MIDI modes that are supported with MPE: MIDI Mode 3 and 4.

- **MIDI Mode 3** (_Poly Mode_): In Poly Mode, a single MIDI channel can hold multiple notes simultaneously however, channel messages will affect all active notes on that channel. When a new MIDI note is created, the controller tries to assign it to an empty channel when possible, otherwise assigns it to a channel with existing active notes.
- **MIDI Mode 4** (_Mono Mode_): In Mono Mode, a single MIDI channel can only hold a single note and when a new MIDI note is created, if the other channels are full, the controller will overwrite existing active notes.

MPE is designed to work properly using MIDI Mode 3 (Poly Mode) but can still be used using MIDI Mode 4 (Mono Mode).

## Note Level vs Zone Level Messages

Depending on whether a MIDI message is sent on a Master Channel or a Member Channel we will refer to them as Zone Level messages or Note Level messages, respectively. Some messages can be sent to both the Master Channel and Member Channels within a zone. When this happens, the receiving synthesiser has to combine both information in an appropriate manner.

Messages that must be sent as Zone Level are the following:

- **CC #1, #33**: Control Change for _Modulation_. This CC can be sent at Note Level but will be ignored.
- **CC #7, #39**: Control Change for _Volume_. This CC can be sent at Note Level but will be ignored.
- **CC #64**: Control Change for a _Damper Pedal_. This CC can be sent at Note Level but will be ignored.
- **CC #120**: Control Change to turn all sounds off. This CC can be sent at Note Level but will be ignored.
- **CC #127**: Control Change to reset all controllers. This CC can only be sent at Zone Level.
- _Polyphonic Key Pressure_: PKP is a future extension of the MPE specifications that are sent on the Master Channel.
- _MPE Configuration_ (MCM): MCMs are sent on the Master Channels to configure as described earlier.

Messages that can be sent as Zone Level and Note Level are the following:

- _Pitch Bend_: Control Change for the first dimension also called _Glide_. _Pitch Bend_ information must be combined if received at both message levels.
- _Channel Pressure_: Control Change for the second dimension also called _Press_. _Channel Pressure_ information must be combined if received at both message levels.
- _Timbre_ (**CC #74**): Control Change for the third dimension also called _Slide_. _Timbre_ information must be combined if received at both message levels.
- _Pitch Bend Sensitivity_: Control Change for changing pitch bend sentivity that should be sent to all Member Channels when used as Note Level.
- _Program Change/Bank Select_: Program Changes are sent on the Master Channel unless in MIDI Mode 4 where they are sent on Member Channels.

Messages that are usually sent as Note Level are the following:

- _Note On/Off_: _Note On_ and _Note Off_ messages should be sent on the appropriate Member Channel but are permitted on the Master Channel for backwards compatibility.
- _MIDI Mode_ (**CC #126, #127**): Control Changes for switching between MIDI Mode 3 and 4 are sent to the lowest Member Channel.

These message levels are important to remember as they influence the design decisions when defining zones within your synthesiser.

# Configuring Zones

Without legacy mode as implemented in [Tutorial: Build a multi-polyphonic synthesiser](/tutorials/tutorial_mpe_introduction/), our synth will not output any sound until we configure at least one zone.

In the `MPESetupComponent` class, we add three button callbacks using lambda functions that allow creation and deletion of zones. We can either create lower and upper zones or clear all the zones in our zone layout.

```cpp
MPESetupComponent()
{
    addAndMakeVisible (isLowerZoneButton);
    isLowerZoneButton.setToggleState (true, juce::NotificationType::dontSendNotification);
```

```cpp
addAndMakeVisible (setZoneButton);
setZoneButton.onClick = [this] { setZoneButtonClicked(); };
addAndMakeVisible (clearAllZonesButton);
clearAllZonesButton.onClick = [this] { clearAllZonesButtonClicked(); };
```

When the user decides to set a zone, the `setZoneButtonClicked()` function is called as defined here:

```cpp
void setZoneButtonClicked()
{
    auto isLowerZone = isLowerZoneButton.getToggleState();
    auto numMemberChannels = memberChannels.getText().getIntValue();
    auto perNotePb = notePitchbendRange.getText().getIntValue();
    auto masterPb = masterPitchbendRange.getText().getIntValue();

    if (isLowerZone)
        zoneLayout.setLowerZone (numMemberChannels, perNotePb, masterPb);
    else
        zoneLayout.setUpperZone (numMemberChannels, perNotePb, masterPb);

    listeners.call ([&] (Listener& l) { l.zoneChanged (isLowerZone, numMemberChannels, perNotePb, masterPb); });
}
```

We first create new local variables to store the lower/upper zone choice, the number of member channels, the zone level pitch bend and the note level pitch bend. We then set the Zone in our [MPEZoneLayout](https://docs.juce.com/master/classMPEZoneLayout.html "This class represents the current MPE zone layout of a device capable of handling MPE.") object by calling the corresponding `setLowerZone()` or `setUpperZone()` function.

To handle the callback when clearing the zones, we simply call the `clearAllZones()` function on the [MPEZoneLayout](https://docs.juce.com/master/classMPEZoneLayout.html "This class represents the current MPE zone layout of a device capable of handling MPE.") object which removes all the zones as follows:

```cpp
void clearAllZonesButtonClicked()
{
    zoneLayout.clearAllZones();
    listeners.call ([] (Listener& l) { l.allZonesCleared(); });
}
```

# Configuring the Synth

Since the `MPESetupComponent` class acts as a broadcaster, we can register as a listener in the `MainComponent` class in order to receive callbacks when the zone layout has changed.

```cpp
class MainComponent : public juce::Component,
                      private juce::AudioIODeviceCallback,
                      private juce::MidiInputCallback,
                      private MPESetupComponent::Listener
{
public:
```

We can then override the corresponding functions to configure the synthesiser accordingly.

In the `zoneChanged()` callback, we set the newly-created Zone to the [MPEZoneLayout](https://docs.juce.com/master/classMPEZoneLayout.html "This class represents the current MPE zone layout of a device capable of handling MPE.") member variable [1]. We can then pass the [MPEZoneLayout](https://docs.juce.com/master/classMPEZoneLayout.html "This class represents the current MPE zone layout of a device capable of handling MPE.") object to the [MPESynthesiser](https://docs.juce.com/master/classMPESynthesiser.html "Base class for an MPE-compatible musical device that can play sounds.") by calling `setZoneLayout()` on it [2]:

```cpp
void zoneChanged (bool isLowerZone, int numMemberChannels, int perNotePitchbendRange, int masterPitchbendRange) override
{
    auto* midiOutput = audioDeviceManager.getDefaultMidiOutput();
    if (midiOutput != nullptr)
    {
        if (isLowerZone)
            midiOutput->sendBlockOfMessagesNow (juce::MPEMessages::setLowerZone (numMemberChannels, perNotePitchbendRange, masterPitchbendRange));
        else
            midiOutput->sendBlockOfMessagesNow (juce::MPEMessages::setUpperZone (numMemberChannels, perNotePitchbendRange, masterPitchbendRange));
    }

    if (isLowerZone)
        zoneLayout.setLowerZone (numMemberChannels, perNotePitchbendRange, masterPitchbendRange);
    else
        zoneLayout.setUpperZone (numMemberChannels, perNotePitchbendRange, masterPitchbendRange);

    visualiserInstrument.setZoneLayout (zoneLayout);
    synth.setZoneLayout (zoneLayout);
    colourPicker.setZoneLayout (zoneLayout);
}
```

In the `allZonesCleared()` callback, we empty all zones in the [MPEZoneLayout](https://docs.juce.com/master/classMPEZoneLayout.html "This class represents the current MPE zone layout of a device capable of handling MPE.") member variable [3]. We can then similarly pass the [MPEZoneLayout](https://docs.juce.com/master/classMPEZoneLayout.html "This class represents the current MPE zone layout of a device capable of handling MPE.") object to the [MPESynthesiser](https://docs.juce.com/master/classMPESynthesiser.html "Base class for an MPE-compatible musical device that can play sounds.") by calling `setZoneLayout()` on it [4]:

```cpp
void allZonesCleared() override
{
    auto* midiOutput = audioDeviceManager.getDefaultMidiOutput();
    if (midiOutput != nullptr)
        midiOutput->sendBlockOfMessagesNow (juce::MPEMessages::clearAllZones());

    zoneLayout.clearAllZones();
    visualiserInstrument.setZoneLayout (zoneLayout);
    synth.setZoneLayout (zoneLayout);
    colourPicker.setZoneLayout (zoneLayout);
}
```

When the number of [MPESynthesiserVoice](https://docs.juce.com/master/classMPESynthesiserVoice.html "Represents an MPE voice that an MPESynthesiser can use to play a sound.") objects is modified, we receive the `numberOfVoicesChanged()` callback which allows us to remove or add voices by respectively using the `reduceNumVoices()` and `addVoice()` functions on the [MPESynthesiser](https://docs.juce.com/master/classMPESynthesiser.html "Base class for an MPE-compatible musical device that can play sounds.") object:

```cpp
void numberOfVoicesChanged (int numberOfVoices) override
{
    if (numberOfVoices < synth.getNumVoices())
        synth.reduceNumVoices (numberOfVoices);
    else
        while (synth.getNumVoices() < numberOfVoices)
            synth.addVoice (new MPEDemoSynthVoice());
}
```

# Assigning Pitch Bend

If we run the synthesiser, we are able to add lower and upper zones with a customisable number of member channels per zone.

![](/_images/tutorial_mpe_zones_screenshot2.png "Different zones in the demo application")

Try assigning different pitch bend sensitivities to the lower and upper zones and notice the impact on the pitch bend of notes assigned to a midi channel of a different zone.

> [!NOTE]
> Excerise: Assign different sounds to lower and upper zones in the `MPEDemoSynthVoice` class and crossfade between a triangle and a sawtooth waveform in response to the _timbre_ parameter.

<!-- -->

> [!TIP]
>The source code for this exercise can be found in the `MPEZonesTutorial_02.h` file of the demo project.

# Summary

In this tutorial, we have learnt how to manage MPE zones and notes. In particular, we have:

- Learnt the conventions when setting up a Zone and an [MPEZoneLayout](https://docs.juce.com/master/classMPEZoneLayout.html "This class represents the current MPE zone layout of a device capable of handling MPE.").
- Understood the two MIDI modes compatible with MPE.
- Explored the different Note Level and Zone Level messages.
- Assigned different pitch bend sensitivities to lower and upper zones.

# See also

- [Tutorial: Build a multi-polyphonic synthesiser](/tutorials/tutorial_mpe_introduction/)
- [Tutorial: Handling MIDI events](/tutorials/tutorial_handling_midi_events/)
- [Tutorial: Create MIDI data](/tutorials/tutorial_midi_message/)
- [Tutorial: Build a MIDI synthesiser](/tutorials/tutorial_synth_using_midi_input/)
