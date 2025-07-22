---
title: Build a multi-polyphonic synthesiser
---
# Tutorial: Build a multi-polyphonic synthesiser

Learn the basics of the MPE standard and how to implement a synthesiser that supports MPE. Hook your application up to a ROLI Seaboard Rise!\

**LEVEL:** Intermediate<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**CLASSES:** [MPESynthesiser](https://docs.juce.com/master/classMPESynthesiser.html "Base class for an MPE-compatible musical device that can play sounds."), [MPEInstrument](https://docs.juce.com/master/classMPEInstrument.html "This class represents an instrument handling MPE."), [MPENote](structMPENote.html "This struct represents a playing MPE note."), [MPEValue](https://docs.juce.com/master/classMPEValue.html "This class represents a single value for any of the MPE dimensions of control."), [SmoothedValue](https://docs.juce.com/master/classSmoothedValue.html "A utility class for values that need smoothing to avoid audio glitches.")

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/MPEIntroductionTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/MPEIntroductionTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

> [!TIP]
>It would be helpful to read [Tutorial: Build a MIDI synthesiser](/tutorials/tutorial_synth_using_midi_input/) first, as this is used as a reference point in a number of places.

# The demo project

The demo project is a simplified version of the `MPEDemo` project in the `JUCE/examples` directory. In order to get the most out of this tutorial you will need an [MPE](https://support.roli.com/article/what-is-mpe/) compatible controller. [MPE](https://support.roli.com/article/what-is-mpe/) stands for _MIDI Polyphonic Expression_, which is a new specification to allow multidimensional data to be communicated between audio products.

Some examples of such [MPE](https://support.roli.com/article/what-is-mpe/) compatible devices are ROLI's own Seaboard range (such as the [Seaboard RISE](https://roli.com/products/seaboard-rise) ).

> [!WARNING]
> The synthesiser may appear very quiet unless your controller transmits MIDI channel pressure and continuous controller 74 (_timbre_) in the way that the [Seaboard RISE](https://roli.com/products/seaboard-rise) does.

With a [Seaboard RISE](https://roli.com/products/seaboard-rise) connected to your computer the window of the demo application should look something like the following screenshot:

![](/_images/tutorial_mpe_introduction_screenshot1.png "The demo application")

You will need to enable one of the MIDI inputs (here you can see a [Seaboard RISE](https://roli.com/products/seaboard-rise) is shown as an option).

## The visualiser

Any notes played on your [MPE](https://support.roli.com/article/what-is-mpe/) compatible device will be visualised in the lower portion of the window. This is shown in the following screenshot:

![](/_images/tutorial_mpe_introduction_screenshot2.png "The visualiser")

One key feature of [MPE](https://support.roli.com/article/what-is-mpe/) is that each new MIDI note event is assigned its own MIDI channel, rather than all notes from a particular controller keyboard being assigned to the same MIDI channel. This allows each individual note to be controlled independently by control change messages, pitch bend message, and so on. In the JUCE implementation of [MPE](https://support.roli.com/article/what-is-mpe/) , a playing note is represented by an [MPENote](structMPENote.html "This struct represents a playing MPE note.") object. An [MPENote](structMPENote.html "This struct represents a playing MPE note.") object encapsulates the following data:

- The MIDI channel of the note.
- The initial MIDI note value of the note.
- The note-on velocity (or _strike_).
- The pitch-bend value for the note: derived from any MIDI pitch-bend messages received on this note's MIDI channel.
- The pressure for the note: derived from any MIDI channel pressure messages received on this note's MIDI channel.
- The _timbre_ for the note: typically derived from any controller messages on this note's MIDI channel for controller 74.
- The note-off velocity (or _lift_): this is only valid after the note-off event has been received and until the playing sound has stopped.

With no notes playing you can see that the visualiser represents a conventional MIDI keyboard layout. Each note is represented in the visualiser in the demo application as follows:

- A grey filled circle represents the note-on velocity (a larger circle for higher velocity).
- The MIDI channel for the note is displayed above the \"+\" symbol within this circle;
- The initial MIDI note name is displayed below the \"+\" symbol.
- An overlaid white circle represents the current pressure for this note (again, a larger circle for higher pressure).
- The horizontal position of the note is derived from the original note and any pitch bend that has been applied to this note.
- The vertical position of the note is derived from the _timbre_ parameter for the note (from MIDI controller 74 on this note's MIDI channel).

## Other setting up

Before delving further into other aspects of the [MPE](https://support.roli.com/article/what-is-mpe/) specification, which are demonstrated by this application, let's look at some of the other things our application uses.

First of all, our `MainComponent` class inherits from the [AudioIODeviceCallback](https://docs.juce.com/master/classAudioIODeviceCallback.html "One of these is passed to an AudioIODevice object to stream the audio data in and out.") [1] and [MidiInputCallback](https://docs.juce.com/master/classMidiInputCallback.html "Receives incoming messages from a physical MIDI input device.") [2] classes:

```cpp
class MainComponent : public juce::Component,
                      private juce::AudioIODeviceCallback, // [1]
                      private juce::MidiInputCallback // [2]
{
public:
```

We also have some important class members in our `MainComponent` class:

```cpp
juce::AudioDeviceManager audioDeviceManager; // [3]
juce::AudioDeviceSelectorComponent audioSetupComp; // [4]

Visualiser visualiserComp;
juce::Viewport visualiserViewport;

juce::MPEInstrument visualiserInstrument;
juce::MPESynthesiser synth;
juce::MidiMessageCollector midiCollector; // [5]

JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR (MainComponent)
};
```

The [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices.") [3] class handles the audio and MIDI configuration on our computer, while the [AudioDeviceSelectorComponent](https://docs.juce.com/master/classAudioDeviceSelectorComponent.html "A component containing controls to let the user change the audio settings of an AudioDeviceManager ob...") [4] class gives us a means of configuring this from the graphical user interface (see [Tutorial: The AudioDeviceManager class](/tutorials/tutorial_audio_device_manager/)). The [MidiMessageCollector](https://docs.juce.com/master/classMidiMessageCollector.html "Collects incoming realtime MIDI messages and turns them into blocks suitable for processing by a bloc...") [5] class allow us to easily collect messages into blocks of timestamped MIDI messages in our audio callback (see [Tutorial: Build a MIDI synthesiser](/tutorials/tutorial_synth_using_midi_input/)).

It is important that the [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices.") object is listed first since we pass this to the constructor of the [AudioDeviceSelectorComponent](https://docs.juce.com/master/classAudioDeviceSelectorComponent.html "A component containing controls to let the user change the audio settings of an AudioDeviceManager ob...") object:

```cpp
MainComponent()
    : audioSetupComp (audioDeviceManager, 0, 0, 0, 256,
          true, // showMidiInputOptions must be true
          true,
          true,
          false)
```

Notice another important argument that is passed to the [AudioDeviceSelectorComponent](https://docs.juce.com/master/classAudioDeviceSelectorComponent.html "A component containing controls to let the user change the audio settings of an AudioDeviceManager ob...") constructor: the `showMidiInputOptions` must be `true` to show our available MIDI inputs.

We set up our [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices.") object in a similar way to [Tutorial: The AudioDeviceManager class](/tutorials/tutorial_audio_device_manager/), but we need also to add a MIDI input callback [6]:

```cpp
audioDeviceManager.initialise (0, 2, nullptr, true, {}, nullptr);
audioDeviceManager.addMidiInputDeviceCallback ({}, this); // [6]
audioDeviceManager.addAudioCallback (this);
```

### The MIDI input callback

The `handleIncomingMidiMessage()` is called when each MIDI message is received from any of the active MIDI inputs in the user interface:

```cpp
void handleIncomingMidiMessage (juce::MidiInput* /*source*/,
    const juce::MidiMessage& message) override
{
    visualiserInstrument.processNextMidiEvent (message);
    midiCollector.addMessageToQueue (message);
}
```

Here we pass each MIDI message to both:

- our `visualiserInstrument` member --- which is used to drive the visualiser display; and
- the `midiCollector` member --- which in turn passes the messages to the synthesiser in the audio callback.

### The audio callback

Before any audio callbacks are made, we need to inform the `synth` and `midiCollector` members of the device sample rate, in the `audioDeviceAboutToStart()` function:

```cpp
void audioDeviceAboutToStart (juce::AudioIODevice* device) override
{
    auto sampleRate = device->getCurrentSampleRate();
    midiCollector.reset (sampleRate);
    synth.setCurrentPlaybackSampleRate (sampleRate);
}
```

The `audioDeviceIOCallbackWithContext()` function appears to do nothing MPE-specific:

```cpp
void audioDeviceIOCallbackWithContext (const float* const* /*inputChannelData*/,
    int /*numInputChannels*/,
    float* const* outputChannelData,
    int numOutputChannels,
    int numSamples,
    const juce::AudioIODeviceCallbackContext& /*context*/) override
{
    // make buffer
    juce::AudioBuffer<float> buffer (outputChannelData, numOutputChannels, numSamples);

    // clear it to silence
    buffer.clear();

    juce::MidiBuffer incomingMidi;

    // get the MIDI messages for this audio block
    midiCollector.removeNextBlockOfMessages (incomingMidi, numSamples);

    // synthesise the block
    synth.renderNextBlock (buffer, incomingMidi, 0, numSamples);
}
```

> [!TIP]
>In fact, this is rather similar to the `SynthAudioSource::getNextAudioBlock()` function in [Tutorial: Build a MIDI synthesiser](/tutorials/tutorial_synth_using_midi_input/).

# Core MPE classes

All of the [MPE](https://support.roli.com/article/what-is-mpe/) specific processing is handled by the [MPE](https://support.roli.com/article/what-is-mpe/) classes: [MPEInstrument](https://docs.juce.com/master/classMPEInstrument.html "This class represents an instrument handling MPE."), [MPESynthesiser](https://docs.juce.com/master/classMPESynthesiser.html "Base class for an MPE-compatible musical device that can play sounds."), [MPESynthesiserVoice](https://docs.juce.com/master/classMPESynthesiserVoice.html "Represents an MPE voice that an MPESynthesiser can use to play a sound."), [MPEValue](https://docs.juce.com/master/classMPEValue.html "This class represents a single value for any of the MPE dimensions of control."), and [MPENote](structMPENote.html "This struct represents a playing MPE note.") (which we mentioned earlier).

## The MPEInstrument class

The [MPEInstrument](https://docs.juce.com/master/classMPEInstrument.html "This class represents an instrument handling MPE.") class maintains the state of the currently playing notes according to the MPE specification. An [MPEInstrument](https://docs.juce.com/master/classMPEInstrument.html "This class represents an instrument handling MPE.") object can have one or more listeners attached and it can broadcast changes to notes as they occur. All you need to do is feed the [MPEInstrument](https://docs.juce.com/master/classMPEInstrument.html "This class represents an instrument handling MPE.") object the MIDI data and it handles the rest.

In the `MainComponent` constructor we configure the [MPEInstrument](https://docs.juce.com/master/classMPEInstrument.html "This class represents an instrument handling MPE.") in _legacy mode_ and set the default pitch bend range to 24 semitones:

```cpp
visualiserInstrument.enableLegacyMode (24);
```

This special mode is for backwards compatibility with non-MPE MIDI devices and the instrument will ignore the current MPE zone layout.

> [!TIP]
>See [Tutorial: Understanding MPE zones](/tutorials/tutorial_mpe_zones/) for an introduction to more flexible approaches using *zones* and *zone layouts*.

In the `MainComponent::handleIncomingMidiMessage()` function we pass the MIDI messages on to our `visualiserInstrument` object:

```cpp
visualiserInstrument.processNextMidiEvent (message);
```

In this example we are using an [MPEInstrument](https://docs.juce.com/master/classMPEInstrument.html "This class represents an instrument handling MPE.") object directly as we need it to update our visualiser display. For the purposes of audio synthesis we don\'t need to create a separate [MPEInstrument](https://docs.juce.com/master/classMPEInstrument.html "This class represents an instrument handling MPE.") object. The [MPESynthesiser](https://docs.juce.com/master/classMPESynthesiser.html "Base class for an MPE-compatible musical device that can play sounds.") object contains an [MPEInstrument](https://docs.juce.com/master/classMPEInstrument.html "This class represents an instrument handling MPE.") object that it uses to drive the synthesiser.

## The MPESynthesiser class

We set our [MPESynthesiser](https://docs.juce.com/master/classMPESynthesiser.html "Base class for an MPE-compatible musical device that can play sounds.") with the same configuration as our `visualiserInstrument` object (in legacy mode with a pitch bend range of 24 semitones):

```cpp
synth.enableLegacyMode (24);
synth.setVoiceStealingEnabled (false);
```

The [MPESynthesiser](https://docs.juce.com/master/classMPESynthesiser.html "Base class for an MPE-compatible musical device that can play sounds.") class can also handle voice stealing for us, but as you can see here, we turn this off. When voice stealing is enabled, the synth will try to take over an existing voice if it runs out of voices and needs to play another note.

As we have already seen in the `MainComponent::audioDeviceAboutToStart()` function we need to set the [MPESynthesiser](https://docs.juce.com/master/classMPESynthesiser.html "Base class for an MPE-compatible musical device that can play sounds.") object's sample rate to work correctly:

```cpp
synth.setCurrentPlaybackSampleRate (sampleRate);
```

And as we have also already seen in the `MainComponent::audioDeviceIOCallback()` function, we simply pass it a [MidiBuffer](https://docs.juce.com/master/classMidiBuffer.html "Holds a sequence of time-stamped midi events.") object containing messages that we want it to use to perform its synthesis operation:

```cpp
synth.renderNextBlock (buffer, incomingMidi, 0, numSamples);
```

## The MPESynthesiserVoice class

You can generally use the [MPESynthesiser](https://docs.juce.com/master/classMPESynthesiser.html "Base class for an MPE-compatible musical device that can play sounds.") and [MPEInstrument](https://docs.juce.com/master/classMPEInstrument.html "This class represents an instrument handling MPE.") classes as they are (although both classes can be used as base classes if you need to override some behaviours). The most important class you _must_ override in order to use the [MPESynthesiser](https://docs.juce.com/master/classMPESynthesiser.html "Base class for an MPE-compatible musical device that can play sounds.") class is the [MPESynthesiserVoice](https://docs.juce.com/master/classMPESynthesiserVoice.html "Represents an MPE voice that an MPESynthesiser can use to play a sound.") class. This actually generates the audio signals from your synthesiser's voices.

> [!TIP]
>This is similar to the [SynthesiserVoice](https://docs.juce.com/master/classSynthesiserVoice.html "Represents a voice that a Synthesiser can use to play a SynthesiserSound.") class that is used with the [Synthesiser](https://docs.juce.com/master/classSynthesiser.html "Base class for a musical device that can play sounds.") class, but it is customised to implement the [MPE](https://support.roli.com/article/what-is-mpe/) specification. See [Tutorial: Build a MIDI synthesiser](/tutorials/tutorial_synth_using_midi_input/).

The code for our voice class is in the `MPEDemoSynthVoice` class of the demo project. Here we implement the `MPEDemoSynthVoice` class to inherit from the [MPESynthesiserVoice](https://docs.juce.com/master/classMPESynthesiserVoice.html "Represents an MPE voice that an MPESynthesiser can use to play a sound.") class:

```cpp
class MPEDemoSynthVoice : public juce::MPESynthesiserVoice
{
```

We have some member variables to keep track of values to control the level, timbre, and frequency of the tone that we generate. In particular, we use the [SmoothedValue](https://docs.juce.com/master/classSmoothedValue.html "A utility class for values that need smoothing to avoid audio glitches.") class, which is really useful for smoothing out discontinuities in the signal that would be otherwise caused by value changes (see [Tutorial: Build a sine wave synthesiser](/tutorials/tutorial_sine_synth/)).

```cpp
juce::SmoothedValue<double> level, timbre, frequency;

double phase = 0.0;
double phaseDelta = 0.0;
double tailOff = 0.0;

// some useful constants
static constexpr auto maxLevel = 0.05;
static constexpr auto maxLevelDb = 31.0;
static constexpr auto smoothingLengthInSeconds = 0.01;
};
```

### Starting and stopping voices

The key to using the [MPESynthesiserVoice](https://docs.juce.com/master/classMPESynthesiserVoice.html "Represents an MPE voice that an MPESynthesiser can use to play a sound.") class is to access its [MPESynthesiserVoice::currentlyPlayingNote](https://docs.juce.com/master/classMPESynthesiserVoice.html#aea34fcbdb3fa0961f73d5f9a1f34e255) (protected) [MPENote](structMPENote.html "This struct represents a playing MPE note.") member to access the control information about the note during the various callbacks. For example, we override the [MPESynthesiserVoice::noteStarted()](https://docs.juce.com/master/classMPESynthesiserVoice.html#ae10c1cdfb37208b96b56863d47a3a193 "Called by the MPESynthesiser to let the voice know that a new note has started on it.") function like this:

```cpp
void noteStarted() override
{
    jassert (currentlyPlayingNote.isValid());
    jassert (currentlyPlayingNote.keyState == juce::MPENote::keyDown
             || currentlyPlayingNote.keyState == juce::MPENote::keyDownAndSustained);

    // get data from the current MPENote
    level.setTargetValue (currentlyPlayingNote.pressure.asUnsignedFloat());
    frequency.setTargetValue (currentlyPlayingNote.getFrequencyInHertz());
    timbre.setTargetValue (currentlyPlayingNote.timbre.asUnsignedFloat());

    phase = 0.0;
    auto cyclesPerSample = frequency.getNextValue() / currentSampleRate;
    phaseDelta = 2.0 * juce::MathConstants<double>::pi * cyclesPerSample;

    tailOff = 0.0;
}
```

The following \"five dimensions\" are stored in the [MPENote](structMPENote.html "This struct represents a playing MPE note.") object as [MPEValue](https://docs.juce.com/master/classMPEValue.html "This class represents a single value for any of the MPE dimensions of control.") objects:

- _note-on velocity_: in the [MPENote::noteOnVelocity](structMPENote.html#a9322650db7f2e76cec724746d1a75c1a "The velocity ("strike") of the note-on.") member
- _pitch bend_: in the [MPENote::pitchbend](structMPENote.html#aebfdbfc173bc19cc97b97659c412078f "Current per-note pitchbend of the note (in units of MIDI pitchwheel position).") member
- _pressure_: in the [MPENote::pressure](structMPENote.html#af04939884198c48da9927e76e4f3ff90 "Current pressure with which the note is held down.") member
- _timbre_: in the [MPENote::timbre](structMPENote.html#aea15b7b7d5b9c55cbf7d479aa8e46a90 "Current value of the note's third expressive dimension, typically encoding some kind of timbre parame...") member
- _note-off velocity_: in the [MPENote::noteOffVelocity](structMPENote.html#a9e46888c40a2d3eaf4b8c5129b21de6e "The release velocity ("lift") of the note after a note-off has been received.") member

[MPEValue](https://docs.juce.com/master/classMPEValue.html "This class represents a single value for any of the MPE dimensions of control.") objects make it easy to create values from 7-bit or 14-bit MIDI value sources, and to obtain these values as floating-point values in the range 0..1 or -1..+1.

> [!TIP]
>The [MPEValue](https://docs.juce.com/master/classMPEValue.html "This class represents a single value for any of the MPE dimensions of control.") class stores the value internally using the 14-bit range.

The `MainComponent::noteStopped()` function triggers the \"release\" of the note envelope (or stops it immediately, if requested):

```cpp
void noteStopped (bool allowTailOff) override
{
    jassert (currentlyPlayingNote.keyState == juce::MPENote::off);

    if (allowTailOff)
    {
        // start a tail-off by setting this flag. The render callback will pick up on
        // this and do a fade out, calling clearCurrentNote() when it's finished.

        if (tailOff == 0.0) // we only need to begin a tail-off if it's not already doing so - the
            // stopNote method could be called more than once.
            tailOff = 1.0;
    }
    else
    {
        // we're being told to stop playing immediately, so reset everything..
        clearCurrentNote();
        phaseDelta = 0.0;
    }
}
```

> [!TIP]
>This is very similar to `SineWaveVoice::stopNote()` function in [Tutorial: Build a MIDI synthesiser](/tutorials/tutorial_synth_using_midi_input/). There isn\'t anything MPE-specific here.

<!-- -->

> [!NOTE]
> Exercise: Modify the `MainComponent::noteStopped()` function to allow the note-off velocity (_lift_) to modify the rate of release of the note. Faster lifts should result in a shorter release time.

### Parameter changes

There are callbacks that tell us when either the pressure, pitch bend, or timbre have changed for this note:

```cpp
void notePressureChanged() override
{
    level.setTargetValue (currentlyPlayingNote.pressure.asUnsignedFloat());
}

void notePitchbendChanged() override
{
    frequency.setTargetValue (currentlyPlayingNote.getFrequencyInHertz());
}

void noteTimbreChanged() override
{
    timbre.setTargetValue (currentlyPlayingNote.timbre.asUnsignedFloat());
}
```

Again, we access the [MPESynthesiserVoice::currentlyPlayingNote](https://docs.juce.com/master/classMPESynthesiserVoice.html#aea34fcbdb3fa0961f73d5f9a1f34e255) member to obtain the current value for each of these parameters.

### Generating the audio

The `MainComponent::renderNextBlock()` actually generates the audio signal, mixing this voice's signal into the buffer that is passed in:

```cpp
void renderNextBlock (juce::AudioBuffer<float>& outputBuffer,
    int startSample,
    int numSamples) override
{
    if (phaseDelta != 0.0)
    {
        if (tailOff > 0.0)
        {
            while (--numSamples >= 0)
            {
                auto currentSample = getNextSample() * (float) tailOff;

                for (auto i = outputBuffer.getNumChannels(); --i >= 0;)
                    outputBuffer.addSample (i, startSample, currentSample);

                ++startSample;

                tailOff *= 0.99;

                if (tailOff <= 0.005)
                {
                    clearCurrentNote();

                    phaseDelta = 0.0;
                    break;
                }
            }
        }
        else
        {
            while (--numSamples >= 0)
            {
                auto currentSample = getNextSample();

                for (auto i = outputBuffer.getNumChannels(); --i >= 0;)
                    outputBuffer.addSample (i, startSample, currentSample);

                ++startSample;
            }
        }
    }
}
```

It calls `MainComponent::getNextSample()` to generate the waveform:

```cpp
float getNextSample() noexcept
{
    auto levelDb = (level.getNextValue() - 1.0) * maxLevelDb;
    auto amplitude = std::pow (10.0f, 0.05f * levelDb) * maxLevel;

    // timbre is used to blend between a sine and a square.
    auto f1 = std::sin (phase);
    auto f2 = std::copysign (1.0, f1);
    auto a2 = timbre.getNextValue();
    auto a1 = 1.0 - a2;

    auto nextSample = float (amplitude * ((a1 * f1) + (a2 * f2)));

    auto cyclesPerSample = frequency.getNextValue() / currentSampleRate;
    phaseDelta = 2.0 * juce::MathConstants<double>::pi * cyclesPerSample;
    phase = std::fmod (phase + phaseDelta, 2.0 * juce::MathConstants<double>::pi);

    return nextSample;
}
```

endcode

This simply cross fades between a sine wave and a (non-bandlimited) square wave, based on the value of the _timbre_ parameter.

> [!NOTE]
> Exercise: Modify the `MPEDemoSynthVoice` class to crossfade between two sine waves, one octave appart, in response to the _timbre_ parameter.

# Summary

In this tutorial we have introduced some of the [MPE](https://support.roli.com/article/what-is-mpe/) based classes in JUCE. You should now know:

- What [MPE](https://support.roli.com/article/what-is-mpe/) is.
- That [MPE](https://support.roli.com/article/what-is-mpe/) compatible devices will allocate each note to their own MIDI channels.
- How the [MPENote](structMPENote.html "This struct represents a playing MPE note.") class stores information about a note including its MIDI channel, the original note number, velocity, pitch bend, and so on.
- That the [MPEInstrument](https://docs.juce.com/master/classMPEInstrument.html "This class represents an instrument handling MPE.") class maintains the state of the currently playing notes.
- That the [MPESynthesiser](https://docs.juce.com/master/classMPESynthesiser.html "Base class for an MPE-compatible musical device that can play sounds.") class contains an [MPEInstrument](https://docs.juce.com/master/classMPEInstrument.html "This class represents an instrument handling MPE.") object that it uses to drive the synthesiser.
- That you must implement a class that inherits from the [MPESynthesiserVoice](https://docs.juce.com/master/classMPESynthesiserVoice.html "Represents an MPE voice that an MPESynthesiser can use to play a sound.") class to implement your synthesiser's audio code.

# See also

- [Tutorial: Understanding MPE zones](/tutorials/tutorial_mpe_zones/)
- [Tutorial: Handling MIDI events](/tutorials/tutorial_handling_midi_events/)
- [Tutorial: Create MIDI data](/tutorials/tutorial_midi_message/)
- [Tutorial: Build a MIDI synthesiser](/tutorials/tutorial_synth_using_midi_input/)
