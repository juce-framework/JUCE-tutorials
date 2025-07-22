---
title: Handling MIDI events
---
# Tutorial: Handling MIDI events

This tutorial illustrates how to handle MIDI input events. In addition to handing MIDI data from an external source, an on-screen keyboard component is introduced.

**LEVEL:** Intermediate<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**CLASSES:** [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices."), [MidiMessage](https://docs.juce.com/master/classMidiMessage.html "Encapsulates a MIDI message."), [MidiInputCallback](https://docs.juce.com/master/classMidiInputCallback.html "Receives incoming messages from a physical MIDI input device."), [ComboBox](https://docs.juce.com/master/classComboBox.html "A component that lets the user choose from a drop-down list of choices."), [MidiKeyboardComponent](https://docs.juce.com/master/classMidiKeyboardComponent.html "A component that displays a piano keyboard, whose notes can be clicked on."), [MidiKeyboardState](https://docs.juce.com/master/classMidiKeyboardState.html "Represents a piano keyboard, keeping track of which keys are currently pressed."), [CallbackMessage](https://docs.juce.com/master/classCallbackMessage.html "A message that invokes a callback method when it gets delivered."), [ScopedValueSetter](https://docs.juce.com/master/classScopedValueSetter.html "Helper class providing an RAII-based mechanism for temporarily setting and then re-setting a value.")

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/HandlingMidiEventsTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/HandlingMidiEventsTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

> [!TIP]
>Ideally, you should have an external MIDI source attached to your computer. Failing this, some kind of virtual MIDI source (that create virtual MIDI ports on the computer) would be helpful.

# The demo project

The demo project presents an on-screen MIDI keyboard and allows the user to select one of the hardware device MIDI inputs using a combo-box. MIDI events received from either of these sources are displayed in the lower part of the window. This is shown in the following screenshot:

![](/_images/tutorial_handling_midi_events_screenshot1.png "The application window")

# MIDI input

This tutorial demonstrates how to handle MIDI input in a basic application. JUCE makes it easy to discover the list of connected hardware MIDI interfaces. It also provides the [MidiKeyboardComponent](https://docs.juce.com/master/classMidiKeyboardComponent.html "A component that displays a piano keyboard, whose notes can be clicked on.") class that allows you to display an on-screen keyboard. First, let's look at the member variables in our `MainContentComponent` class:

```cpp
juce::AudioDeviceManager deviceManager; // [1]
juce::ComboBox midiInputList; // [2]
juce::Label midiInputListLabel;
int lastInputIndex = 0; // [3]
bool isAddingFromMidiInput = false; // [4]

juce::MidiKeyboardState keyboardState; // [5]
juce::MidiKeyboardComponent keyboardComponent; // [6]

juce::TextEditor midiMessagesBox;
double startTime;
```

- [1]: We use the [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices.") class to find which MIDI input devices are enabled.
- [2]: We display the names of the MIDI input devices in this combo-box for the user to select.
- [3]: This is used to de-register a previously selected MIDI input when the user selects a different input.
- [4]: This flag is used to indicate that MIDI data is arriving from an external source, rather than mouse-clicks on the on-screen keyboard.
- [5]: The [MidiKeyboardState](https://docs.juce.com/master/classMidiKeyboardState.html "Represents a piano keyboard, keeping track of which keys are currently pressed.") class keeps track of which MIDI keys are currently held down.
- [6]: This is the on-screen keyboard component.

In the `MainContentComponent` constructor we initialise [3], [4], and [6]. We also take a note of the application start time so we can display the MIDI data timestamps relative to this.

```cpp
MainContentComponent()
    : keyboardComponent (keyboardState, juce::MidiKeyboardComponent::horizontalKeyboard),
      startTime (juce::Time::getMillisecondCounterHiRes() * 0.001)
{
```

We must pass a [MidiKeyboardState](https://docs.juce.com/master/classMidiKeyboardState.html "Represents a piano keyboard, keeping track of which keys are currently pressed.") object to initialise the [MidiKeyboardComponent](https://docs.juce.com/master/classMidiKeyboardComponent.html "A component that displays a piano keyboard, whose notes can be clicked on.") object. And, since these are statically allocated objects the [MidiKeyboardState](https://docs.juce.com/master/classMidiKeyboardState.html "Represents a piano keyboard, keeping track of which keys are currently pressed.") must be listed first in our member variables.

## MIDI input list

The combo-box containing the list of MIDI inputs is populated by getting the list of MIDI inputs connected to the computer from the [MidiInput](https://docs.juce.com/master/classMidiInput.html "Represents a midi input device.") class using the MidiInput::getDevices() function:

```cpp
addAndMakeVisible (midiInputList);
midiInputList.setTextWhenNoChoicesAvailable ("No MIDI Inputs Enabled");
auto midiInputs = juce::MidiInput::getAvailableDevices();

juce::StringArray midiInputNames;

for (auto input : midiInputs)
    midiInputNames.add (input.name);

midiInputList.addItemList (midiInputNames, 1);
midiInputList.onChange = [this] { setMidiInput (midiInputList.getSelectedItemIndex()); };

// find the first enabled device and use that by default
for (auto input : midiInputs)
{
    if (deviceManager.isMidiInputDeviceEnabled (input.identifier))
    {
        setMidiInput (midiInputs.indexOf (input));
        break;
    }
}

// if no enabled devices were found just use the first one in the list
if (midiInputList.getSelectedId() == 0)
    setMidiInput (0);
```

If the user changes the selected MIDI input then the lambda function assigned to the [ComboBox::onChange](https://docs.juce.com/master/classComboBox.html#a9cf2e20990541b9fbb539cd4a8e0ac4e "You can assign a lambda to this callback object to have it called when the selected ID is changed.") helper object will be called:

```cpp
midiInputList.onChange = [this] { setMidiInput (midiInputList.getSelectedItemIndex()); };
```

The `setMidiInput()` function makes our application start listening to the selected device. It also enables the device if it is currently disabled:

```cpp
void setMidiInput (int index)
{
    auto list = juce::MidiInput::getAvailableDevices();

    deviceManager.removeMidiInputDeviceCallback (list[lastInputIndex].identifier, this);

    auto newInput = list[index];

    if (!deviceManager.isMidiInputDeviceEnabled (newInput.identifier))
        deviceManager.setMidiInputDeviceEnabled (newInput.identifier, true);

    deviceManager.addMidiInputDeviceCallback (newInput.identifier, this);
    midiInputList.setSelectedId (index + 1, juce::dontSendNotification);

    lastInputIndex = index;
}
```

## Handling external MIDI input

We implement the [MidiInputCallback::handleIncomingMidiMessage()](https://docs.juce.com/master/classMidiInputCallback.html#acdf6d7e27b96a50fe06922737f0d082e "Receives an incoming message.") [pure virtual](http://www.learncpp.com/cpp-tutorial/126-pure-virtual-functions-abstract-base-classes-and-interface-classes/) function. This updates the keyboard state (which in turn will update the [MidiKeyboardComponent](https://docs.juce.com/master/classMidiKeyboardComponent.html "A component that displays a piano keyboard, whose notes can be clicked on.") object):

```cpp
void handleIncomingMidiMessage (juce::MidiInput* source, const juce::MidiMessage& message) override
{
    const juce::ScopedValueSetter<bool> scopedInputFlag (isAddingFromMidiInput, true);
    keyboardState.processNextMidiEvent (message);
    postMessageToList (message, source->getName());
}
```

Notice the `scopedInputFlag` variable makes use of the [ScopedValueSetter](https://docs.juce.com/master/classScopedValueSetter.html "Helper class providing an RAII-based mechanism for temporarily setting and then re-setting a value.") class. This does the following:

- It stores the current state of the `isAddingFromMidiInput` member.
- It sets the `isAddingFromMidiInput` member to true.
- When the function exits it reset the value of `isAddingFromMidiInput` member to the state it was in at the start of the function.

## The MIDI keyboard state and component

In the `MainContentComponent` constructor the [MidiKeyboardComponent](https://docs.juce.com/master/classMidiKeyboardComponent.html "A component that displays a piano keyboard, whose notes can be clicked on.") object is added to our `MainContentComponent` parent component and made visible. We also listen to the [MidiKeyboardState](https://docs.juce.com/master/classMidiKeyboardState.html "Represents a piano keyboard, keeping track of which keys are currently pressed.") object (_not_ the component):

```cpp
addAndMakeVisible (keyboardComponent);
keyboardState.addListener (this);
```

The MidiKeyboardStateListener class has two [pure virtual](http://www.learncpp.com/cpp-tutorial/126-pure-virtual-functions-abstract-base-classes-and-interface-classes/) functions that we must implement. These are the [MidiKeyboardStateListener::handleNoteOn()](https://docs.juce.com/master/classMidiKeyboardState_1_1Listener.html#a854381a64354389e0cb2c15ec1fb4b01 "Called when one of the MidiKeyboardState's keys is pressed.") and [MidiKeyboardStateListener::handleNoteOff()](https://docs.juce.com/master/classMidiKeyboardState_1_1Listener.html#aedc74bd79578cb19edba66d38e9a93cf "Called when one of the MidiKeyboardState's keys is released.") functions.

```cpp
void handleNoteOn (juce::MidiKeyboardState*, int midiChannel, int midiNoteNumber, float velocity) override
{
    if (!isAddingFromMidiInput)
    {
        auto m = juce::MidiMessage::noteOn (midiChannel, midiNoteNumber, velocity);
        m.setTimeStamp (juce::Time::getMillisecondCounterHiRes() * 0.001);
        postMessageToList (m, "On-Screen Keyboard");
    }
}

void handleNoteOff (juce::MidiKeyboardState*, int midiChannel, int midiNoteNumber, float /*velocity*/) override
{
    if (!isAddingFromMidiInput)
    {
        auto m = juce::MidiMessage::noteOff (midiChannel, midiNoteNumber);
        m.setTimeStamp (juce::Time::getMillisecondCounterHiRes() * 0.001);
        postMessageToList (m, "On-Screen Keyboard");
    }
}
```

Here you can see how the `isAddingFromMidiInput` member is used. This prevents events that arrived from the hardware input from being posted to our list more than once.

## Posting messages to the list

The `postMessageToList()` function may look a little unusual at first:

```cpp
void postMessageToList (const juce::MidiMessage& message, const juce::String& source)
{
    (new IncomingMessageCallback (this, message, source))->post();
}
```

The `IncomingMessageCallback` class is a subclass of the [CallbackMessage](https://docs.juce.com/master/classCallbackMessage.html "A message that invokes a callback method when it gets delivered.") class. We need to use this since we can\'t be sure from which thread the `postMessageToList()` function will be called. It will be called from the message thread if the user clicks on the [MidiKeyboardComponent](https://docs.juce.com/master/classMidiKeyboardComponent.html "A component that displays a piano keyboard, whose notes can be clicked on.") object. But, if the data arrives from an external MIDI source then it will be called from the background MIDI thread (possibly an operating system thread).

The [CallbackMessage](https://docs.juce.com/master/classCallbackMessage.html "A message that invokes a callback method when it gets delivered.") class provides a means of calling a function on the message thread. The [CallbackMessage](https://docs.juce.com/master/classCallbackMessage.html "A message that invokes a callback method when it gets delivered.") class is a kind of [ReferenceCountedObject](https://docs.juce.com/master/classReferenceCountedObject.html "A base class which provides methods for reference-counting.") class. This is why we don\'t (apparently) need to store the `IncomingMessageCallback` object anywhere. In fact, the `IncomingMessageCallback::post()` function (which is the [MessageManager::MessageBase::post()](https://docs.juce.com/master/classMessageManager_1_1MessageBase.html#a2bcdf57123967a59650bd46543906796) function) adds the object to a queue that is handled by the [MessageManager](https://docs.juce.com/master/classMessageManager.html "This class is in charge of the application's event-dispatch loop.") class. The [MessageManager](https://docs.juce.com/master/classMessageManager.html "This class is in charge of the application's event-dispatch loop.") class will eventually find this object in the queue and call the `IncomingMessageCallback::messageCallback()` function on the message thread. Once this function has been called, the `IncomingMessageCallback` object will be deleted. Thus the lifetime of this object is handled (almost) automatically.

> [!TIP]
>This is only really necessary since we need to send the data to the message thread. It is likely that some kind of inter-thread communication is necessary in a MIDI application but the exact implementation depends on the circumstances.

## Displaying the messages

The `addMessageToList()` and `getMidiMessageDescription()` functions are very similar to these functions from [Tutorial: Create MIDI data](/tutorials/tutorial_midi_message/). The main difference is that we make a note of the source [7] of the MIDI message (which hardware input, or the on-screen keyboard):

```cpp
void addMessageToList (const juce::MidiMessage& message, const juce::String& source)
{
    auto time = message.getTimeStamp() - startTime;

    auto hours = ((int) (time / 3600.0)) % 24;
    auto minutes = ((int) (time / 60.0)) % 60;
    auto seconds = ((int) time) % 60;
    auto millis = ((int) (time * 1000.0)) % 1000;

    auto timecode = juce::String::formatted ("%02d:%02d:%02d.%03d",
        hours,
        minutes,
        seconds,
        millis);

    auto description = getMidiMessageDescription (message);

    juce::String midiMessageString (timecode + " - " + description + " (" + source + ")"); // [7]
    logMessage (midiMessageString);
}
```

> [!NOTE]
> Exercise: Add some sliders to the user interface that transmit and respond to messages such as modulation wheel (CC1) and pitch wheel.

# Summary

This tutorial has introduced some classes for handing and displaying MIDI input events. In particular you should be able to:

- List the available MIDI input devices.
- Create a menu of MIDI input devices.
- Listen to MIDI arriving at a hardware input.
- Display MIDI note data using the [MidiKeyboardComponent](https://docs.juce.com/master/classMidiKeyboardComponent.html "A component that displays a piano keyboard, whose notes can be clicked on.") class.
- Post messages from other threads to be be dealt with on the message thread using the [CallbackMessage](https://docs.juce.com/master/classCallbackMessage.html "A message that invokes a callback method when it gets delivered.") class.

# See also

- [Tutorial: The AudioDeviceManager class](/tutorials/tutorial_audio_device_manager/)
- [Tutorial: Create MIDI data](/tutorials/tutorial_midi_message/)
- [Tutorial: Build a MIDI synthesiser](/tutorials/tutorial_synth_using_midi_input/)
