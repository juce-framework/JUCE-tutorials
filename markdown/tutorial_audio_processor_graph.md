---
title: Cascading plug-in effects
---
# Tutorial: Cascading plug-in effects

Create your own channel strip by learning how to daisy chain audio processors or plugins using an [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors."). Learn how to use the [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors.") in both a plugin and standalone application context.

**LEVEL:** Intermediate<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**CLASSES:** [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins."), [AudioProcessorPlayer](https://docs.juce.com/master/classAudioProcessorPlayer.html "An AudioIODeviceCallback object which streams audio through an AudioProcessor."), [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors."), [AudioProcessorGraph::AudioGraphIOProcessor](https://docs.juce.com/master/classAudioProcessorGraph_1_1AudioGraphIOProcessor.html "A special type of AudioProcessor that can live inside an AudioProcessorGraph in order to use the audi..."), [AudioProcessorGraph::Node](https://docs.juce.com/master/classAudioProcessorGraph_1_1Node.html "Represents one of the nodes, or processors, in an AudioProcessorGraph."), [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices.")

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/AudioProcessorGraphTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/AudioProcessorGraphTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project simulates a channel strip where different audio processors can be applied in series. There are three available slots that can be individually bypassed and three different processors including an oscillator, a gain control and a filter that can be chosen from. The plugin applies processing on incoming audio and propagates the modified signal to the output.

![](/_images/tutorial_audio_processor_graph_screenshot1.png "The channel strip plugin window")

# Setting up the AudioProcessorGraph

The [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors.") is a special type of [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.") that allows us to connect several [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.") objects together as nodes in a graph and play back the result of the combined processing. In order to wire-up graph nodes together, we have to add connections between channels of nodes in the order we wish to process the audio signal.

The [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors.") class also offers special node types for input and output handling of audio and midi signals within the graph. An example graph of the channel strip would look something like this when connected properly:

![](/_images/tutorial_audio_processor_graph.svg "")

Let's start by setting up the main [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors.") to receive incoming signals and send them back to the corresponding output unprocessed.

In order to reduce the character count for nested classes used frequently in this tutorial, we first declare a `using` for the AudioGraphIOProcessor class and the Node class in the `TutorialProcessor` class as follows:

```cpp
using AudioGraphIOProcessor = juce::AudioProcessorGraph::AudioGraphIOProcessor;
using Node = juce::AudioProcessorGraph::Node;
```

Then we declare the following private member variables using the shortened version of class names like so:

```cpp
std::unique_ptr<juce::AudioProcessorGraph> mainProcessor;
```

```cpp
Node::Ptr audioInputNode;
Node::Ptr audioOutputNode;
Node::Ptr midiInputNode;
Node::Ptr midiOutputNode;
```

Here we create pointers to the main [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors.") as well as the input and output processor nodes which will be instantiated later on within the graph.

Next, in the `TutorialProcessor` contructor we set the default bus properties for the plugin and instantiate the main [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors.") as shown here:

```cpp
TutorialProcessor()
    : AudioProcessor (BusesProperties().withInput ("Input", juce::AudioChannelSet::stereo(), true).withOutput ("Output", juce::AudioChannelSet::stereo(), true)),
      mainProcessor (new juce::AudioProcessorGraph()),
```

Since we are dealing with a plugin, we need to implement the isBusesLayoutSupported() callback to inform the plugin host or DAW about which channel sets we support. In this example we decide to only support mono-to-mono and stereo-to-stereo configurations like this:

```cpp
bool isBusesLayoutSupported (const BusesLayout& layouts) const override
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

> [!TIP]
>If you want to learn more about bus layouts of plugins, please refer to [Tutorial: Configuring the right bus layouts for your plugins](/tutorials/tutorial_audio_bus_layouts/).

For the `TutorialProcessor` to be able to process audio through the graph provided, we have to override the three main functions of the [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.") class that perform signal processing namely the prepareToPlay(), releaseResources() and processBlock() functions and call the same respective functions on the [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors.").

Let's start with the prepareToPlay() function. First we inform the [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors.") on the number of I/O channels, the sample rate and the number of samples per block by calling the setPlayConfigDetails() function like follows:

```cpp
void prepareToPlay (double sampleRate, int samplesPerBlock) override
{
    mainProcessor->setPlayConfigDetails (getMainBusNumInputChannels(),
        getMainBusNumOutputChannels(),
        sampleRate,
        samplesPerBlock);

    mainProcessor->prepareToPlay (sampleRate, samplesPerBlock);

    initialiseGraph();
}
```

We then call the prepareToPlay() function on the [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors.") with the same information and call the `initialiseGraph()` helper function which we define later on to create and connect the nodes in the graph.

The releaseResources() function is self-explanatory and simply calls the same function on the [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors.") instance:

```cpp
void releaseResources() override
{
    mainProcessor->releaseResources();
}
```

Finally in the processBlock() function, we clear the samples contained in any additional channels that may contain garbage data just in case and call the `updateGraph()` helper function later defined that will rebuild the graph if the channel strip configuration was changed. The processBlock() function is eventually called on the [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors.") at the end of the function:

```cpp
void processBlock (juce::AudioSampleBuffer& buffer, juce::MidiBuffer& midiMessages) override
{
    for (int i = getTotalNumInputChannels(); i < getTotalNumOutputChannels(); ++i)
        buffer.clear (i, 0, buffer.getNumSamples());

    updateGraph();

    mainProcessor->processBlock (buffer, midiMessages);
}
```

The `initialiseGraph()` function called earlier in the prepareToPlay() callback starts by clearing the [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors.") of any nodes and connections that were previously present. This also takes care of deleting the corresponding [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.") instances associated to the deleted nodes in the graph. We then proceed to instantiate the AudioGraphIOProcessor objects for the graph I/O and add the [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.") objects as nodes in the graph.

```cpp
void initialiseGraph()
{
    mainProcessor->clear();

    audioInputNode = mainProcessor->addNode (std::make_unique<AudioGraphIOProcessor> (AudioGraphIOProcessor::audioInputNode));
    audioOutputNode = mainProcessor->addNode (std::make_unique<AudioGraphIOProcessor> (AudioGraphIOProcessor::audioOutputNode));
    midiInputNode = mainProcessor->addNode (std::make_unique<AudioGraphIOProcessor> (AudioGraphIOProcessor::midiInputNode));
    midiOutputNode = mainProcessor->addNode (std::make_unique<AudioGraphIOProcessor> (AudioGraphIOProcessor::midiOutputNode));

    connectAudioNodes();
    connectMidiNodes();
}
```

We still have to add connections between the newly-created nodes in the graph to propagate the audio/midi data and this is performed with the following helper functions:

```cpp
void connectAudioNodes()
{
    for (int channel = 0; channel < 2; ++channel)
        mainProcessor->addConnection ({ { audioInputNode->nodeID, channel },
            { audioOutputNode->nodeID, channel } });
}
```

Here we call the addConnection() function on the [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors.") instance by passing the source and destination nodes we wish to connect in the form of a Connection object. These require a nodeID and a channel index for building the appropriate connections and the whole process is repeated for all the required channels.

```cpp
void connectMidiNodes()
{
    mainProcessor->addConnection ({ { midiInputNode->nodeID, juce::AudioProcessorGraph::midiChannelIndex },
        { midiOutputNode->nodeID, juce::AudioProcessorGraph::midiChannelIndex } });
}
```

The same is performed on the midi I/O nodes with the exception of the channel index argument. Since the midi signals are not sent through regular audio channels, we have to supply a special channel index specified as an enum in the [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors.") class.

At this stage of the tutorial, we should be able to hear the signal pass through the graph without being altered.

> [!WARNING]
> Beware of screaming feedback when testing the plugin with built-in inputs and outputs. You can avoid this problem by using headphones.

# Implementing different processors

In this part of the tutorial, we create different processors that we can use within our channel strip plugin to alter the incoming audio signal. Feel free to create additional processors or customise the following ones to your taste.

In order to avoid repeated code for the different processors we want to create, let's start by declaring an [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.") base class that will be inherited by the individual processors and override the necessary functions only once for simplicity's sake.

```cpp
class ProcessorBase : public juce::AudioProcessor
{
public:
    //==============================================================================
    ProcessorBase()
        : AudioProcessor (BusesProperties().withInput ("Input", juce::AudioChannelSet::stereo()).withOutput ("Output", juce::AudioChannelSet::stereo()))
    {
    }

    //==============================================================================
    void prepareToPlay (double, int) override {}
    void releaseResources() override {}
    void processBlock (juce::AudioSampleBuffer&, juce::MidiBuffer&) override {}

    //==============================================================================
    juce::AudioProcessorEditor* createEditor() override { return nullptr; }
    bool hasEditor() const override { return false; }

    //==============================================================================
    const juce::String getName() const override { return {}; }
    bool acceptsMidi() const override { return false; }
    bool producesMidi() const override { return false; }
    double getTailLengthSeconds() const override { return 0; }

    //==============================================================================
    int getNumPrograms() override { return 0; }
    int getCurrentProgram() override { return 0; }
    void setCurrentProgram (int) override {}
    const juce::String getProgramName (int) override { return {}; }
    void changeProgramName (int, const juce::String&) override {}

    //==============================================================================
    void getStateInformation (juce::MemoryBlock&) override {}
    void setStateInformation (const void*, int) override {}

private:
    //==============================================================================
    JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR (ProcessorBase)
};
```

> [!TIP]
>The following three processors will make use of the DSP module to facilitate implementation and if you want to learn more about DSP you can refer to [Tutorial: Introduction to DSP](/tutorials/tutorial_dsp_introduction/) for a more in-depth explanation.

## Implementing an oscillator

The first processor is a simple oscillator that generates a constant sine wave tone at 440Hz.

We derive the `OscillatorProcessor` class from the previously-defined `ProcessorBase` , override the getName() function to provide a meaningful name and declare a [dsp::Oscillator](https://docs.juce.com/master/classdsp_1_1Oscillator.html "Generates a signal based on a user-supplied function.") object from the DSP module:

```cpp
class OscillatorProcessor : public ProcessorBase
{
public:
    //...
    const juce::String getName() const override { return "Oscillator"; }

private:
    juce::dsp::Oscillator<float> oscillator;
};
```

In the constructor, we set the frequency and the waveform of the oscillator by calling respectively the setFrequency() and initialise() functions on the [dsp::Oscillator](https://docs.juce.com/master/classdsp_1_1Oscillator.html "Generates a signal based on a user-supplied function.") object as follows:

```cpp
OscillatorProcessor()
{
    oscillator.setFrequency (440.0f);
    oscillator.initialise ([] (float x) { return std::sin (x); });
}
```

In the prepareToPlay() function, we create a [dsp::ProcessSpec](structdsp_1_1ProcessSpec.html "This structure is passed into a DSP algorithm's prepare() method, and contains information about vari...") object to describe the sample rate and number of samples per block to the [dsp::Oscillator](https://docs.juce.com/master/classdsp_1_1Oscillator.html "Generates a signal based on a user-supplied function.") object and pass the specifications by calling the prepare() function on it like so:

```cpp
void prepareToPlay (double sampleRate, int samplesPerBlock) override
{
    juce::dsp::ProcessSpec spec { sampleRate, static_cast<juce::uint32> (samplesPerBlock), 2 };
    oscillator.prepare (spec);
}
```

Next, in the processBlock() function we create a [dsp::AudioBlock](https://docs.juce.com/master/classdsp_1_1AudioBlock.html "Minimal and lightweight data-structure which contains a list of pointers to channels containing some ...") object from the AudioSampleBuffer passed as an argument and declare the processing context from it as a [dsp::ProcessContextReplacing](structdsp_1_1ProcessContextReplacing.html "Contains context information that is passed into an algorithm's process method.") object that is subsequently passed to the process() function of the [dsp::Oscillator](https://docs.juce.com/master/classdsp_1_1Oscillator.html "Generates a signal based on a user-supplied function.") object as shown here:

```cpp
void processBlock (juce::AudioSampleBuffer& buffer, juce::MidiBuffer&) override
{
    juce::dsp::AudioBlock<float> block (buffer);
    juce::dsp::ProcessContextReplacing<float> context (block);
    oscillator.process (context);
}
```

Finally, we can reset the state of the [dsp::Oscillator](https://docs.juce.com/master/classdsp_1_1Oscillator.html "Generates a signal based on a user-supplied function.") object by overriding the reset() function of the [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.") and calling the same function onto it:

```cpp
void reset() override
{
    oscillator.reset();
}
```

We now have an oscillator that we can use in our channel strip plugin.

> [!NOTE]
> Exercise: Modify the initialise() function of the oscillator to produce a different waveform and change the target frequency.

## Implementing a gain control

The second processor is a simple gain control that attenuates the incoming signal by -6dB.

We derive the `GainProcessor` class from the previously-defined `ProcessorBase` , override the getName() function to provide a meaningful name and declare a [dsp::Gain](https://docs.juce.com/master/classdsp_1_1Gain.html "Applies a gain to audio samples as single samples or AudioBlocks.") object from the DSP module:

```cpp
class GainProcessor : public ProcessorBase
{
public:
    //...
    const juce::String getName() const override { return "Gain"; }

private:
    juce::dsp::Gain<float> gain;
};
```

In the constructor, we set the gain in decibels of the gain control by calling the setGainDecibels() function on the [dsp::Gain](https://docs.juce.com/master/classdsp_1_1Gain.html "Applies a gain to audio samples as single samples or AudioBlocks.") object as follows:

```cpp
GainProcessor()
{
    gain.setGainDecibels (-6.0f);
}
```

In the prepareToPlay() function, we create a [dsp::ProcessSpec](structdsp_1_1ProcessSpec.html "This structure is passed into a DSP algorithm's prepare() method, and contains information about vari...") object to describe the sample rate, number of samples per block and number of channels to the [dsp::Gain](https://docs.juce.com/master/classdsp_1_1Gain.html "Applies a gain to audio samples as single samples or AudioBlocks.") object and pass the specifications by calling the prepare() function on it like so:

```cpp
void prepareToPlay (double sampleRate, int samplesPerBlock) override
{
    juce::dsp::ProcessSpec spec { sampleRate, static_cast<juce::uint32> (samplesPerBlock), 2 };
    gain.prepare (spec);
}
```

Next, in the processBlock() function we create a [dsp::AudioBlock](https://docs.juce.com/master/classdsp_1_1AudioBlock.html "Minimal and lightweight data-structure which contains a list of pointers to channels containing some ...") object from the AudioSampleBuffer passed as an argument and declare the processing context from it as a [dsp::ProcessContextReplacing](structdsp_1_1ProcessContextReplacing.html "Contains context information that is passed into an algorithm's process method.") object that is subsequently passed to the process() function of the [dsp::Gain](https://docs.juce.com/master/classdsp_1_1Gain.html "Applies a gain to audio samples as single samples or AudioBlocks.") object as shown here:

```cpp
void processBlock (juce::AudioSampleBuffer& buffer, juce::MidiBuffer&) override
{
    juce::dsp::AudioBlock<float> block (buffer);
    juce::dsp::ProcessContextReplacing<float> context (block);
    gain.process (context);
}
```

Finally, we can reset the state of the [dsp::Gain](https://docs.juce.com/master/classdsp_1_1Gain.html "Applies a gain to audio samples as single samples or AudioBlocks.") object by overriding the reset() function of the [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.") and calling the same function onto it:

```cpp
void reset() override
{
    gain.reset();
}
```

We now have a gain control that we can use in our channel strip plugin.

> [!NOTE]
> Exercise: Modify the setGainDecibels() function of the gain control to reduce the gain furthermore or boost the signal. (Careful with levels when boosting!)

## Implementing a filter

The third processor is a simple high pass filter that reduces the frequencies below 1kHz.

We derive the `FilterProcessor` class from the previously-defined `ProcessorBase` , override the getName() function to provide a meaningful name and declare a [dsp::ProcessorDuplicator](structdsp_1_1ProcessorDuplicator.html "Converts a mono processor class into a multi-channel version by duplicating it and applying multichan...") object from the DSP module. This allows us to use a mono processor of the [dsp::IIR::Filter](https://docs.juce.com/master/classdsp_1_1IIR_1_1Filter.html "A processing class that can perform IIR filtering on an audio signal, using the Transposed Direct For...") class and convert it into a multi-channel version by providing its shared state as a [dsp::IIR::Coefficients](structdsp_1_1IIR_1_1Coefficients.html "A set of coefficients for use in an Filter object.") class:

```cpp
class FilterProcessor : public ProcessorBase
{
public:
    FilterProcessor() {}
    //...
    const juce::String getName() const override { return "Filter"; }

private:
    juce::dsp::ProcessorDuplicator<juce::dsp::IIR::Filter<float>, juce::dsp::IIR::Coefficients<float>> filter;
};
```

In the prepareToPlay() function, we first generate the coefficients used for the filter by using the makeHighPass() function and assign it as the shared processing state to the duplicator. We then create a [dsp::ProcessSpec](structdsp_1_1ProcessSpec.html "This structure is passed into a DSP algorithm's prepare() method, and contains information about vari...") object to describe the sample rate, number of samples per block and number of channels to the [dsp::ProcessorDuplicator](structdsp_1_1ProcessorDuplicator.html "Converts a mono processor class into a multi-channel version by duplicating it and applying multichan...") object and pass the specifications by calling the prepare() function on it like so:

```cpp
void prepareToPlay (double sampleRate, int samplesPerBlock) override
{
    *filter.state = *juce::dsp::IIR::Coefficients<float>::makeHighPass (sampleRate, 1000.0f);

    juce::dsp::ProcessSpec spec { sampleRate, static_cast<juce::uint32> (samplesPerBlock), 2 };
    filter.prepare (spec);
}
```

Next, in the processBlock() function we create a [dsp::AudioBlock](https://docs.juce.com/master/classdsp_1_1AudioBlock.html "Minimal and lightweight data-structure which contains a list of pointers to channels containing some ...") object from the AudioSampleBuffer passed as an argument and declare the processing context from it as a [dsp::ProcessContextReplacing](structdsp_1_1ProcessContextReplacing.html "Contains context information that is passed into an algorithm's process method.") object that is subsequently passed to the process() function of the [dsp::ProcessorDuplicator](structdsp_1_1ProcessorDuplicator.html "Converts a mono processor class into a multi-channel version by duplicating it and applying multichan...") object as shown here:

```cpp
void processBlock (juce::AudioSampleBuffer& buffer, juce::MidiBuffer&) override
{
    juce::dsp::AudioBlock<float> block (buffer);
    juce::dsp::ProcessContextReplacing<float> context (block);
    filter.process (context);
}
```

Finally, we can reset the state of the [dsp::ProcessorDuplicator](structdsp_1_1ProcessorDuplicator.html "Converts a mono processor class into a multi-channel version by duplicating it and applying multichan...") object by overriding the reset() function of the [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.") and calling the same function onto it:

```cpp
void reset() override
{
    filter.reset();
}
```

We now have a filter that we can use in our channel strip plugin.

> [!NOTE]
> Exercise: Modify the coefficients of the filter to produce a low pass or band pass filter with different cut-off frequencies and resonance.

# Connecting graph nodes together

Now that we have implemented multiple processors that can be used within the [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors."), let's start connecting them together depending on the user selection.

In the `TutorialProcessor` class, we add three [AudioParameterChoice](https://docs.juce.com/master/classAudioParameterChoice.html "Provides a class of AudioProcessorParameter that can be used to select an indexed,...") and four [AudioParameterBool](https://docs.juce.com/master/classAudioParameterBool.html "Provides a class of AudioProcessorParameter that can be used as a boolean value.") pointers as private member variables to store the parameters chosen in the channel strip and their corresponding bypass states. We also declare node pointers to the three processor slots when later instantiated within the graph and provide the selectable choices as a [StringArray](https://docs.juce.com/master/classStringArray.html "A special array for holding a list of strings.") for convenience.

```cpp
juce::StringArray processorChoices { "Empty", "Oscillator", "Gain", "Filter" };

std::unique_ptr<juce::AudioProcessorGraph> mainProcessor;

juce::AudioParameterBool* muteInput;

juce::AudioParameterChoice* processorSlot1;
juce::AudioParameterChoice* processorSlot2;
juce::AudioParameterChoice* processorSlot3;

juce::AudioParameterBool* bypassSlot1;
juce::AudioParameterBool* bypassSlot2;
juce::AudioParameterBool* bypassSlot3;

Node::Ptr audioInputNode;
Node::Ptr audioOutputNode;
Node::Ptr midiInputNode;
Node::Ptr midiOutputNode;

Node::Ptr slot1Node;
Node::Ptr slot2Node;
Node::Ptr slot3Node;

//==============================================================================
JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR (TutorialProcessor)
};
```

Then in the constructor we can instantiate the audio parameters and call the addParameter() function to tell the [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.") about which parameters should be available in the plugin.

```cpp
TutorialProcessor()
    : AudioProcessor (BusesProperties().withInput ("Input", juce::AudioChannelSet::stereo(), true).withOutput ("Output", juce::AudioChannelSet::stereo(), true)),
      mainProcessor (new juce::AudioProcessorGraph()),
      muteInput (new juce::AudioParameterBool ("mute", "Mute Input", true)),
      processorSlot1 (new juce::AudioParameterChoice ("slot1", "Slot 1", processorChoices, 0)),
      processorSlot2 (new juce::AudioParameterChoice ("slot2", "Slot 2", processorChoices, 0)),
      processorSlot3 (new juce::AudioParameterChoice ("slot3", "Slot 3", processorChoices, 0)),
      bypassSlot1 (new juce::AudioParameterBool ("bypass1", "Bypass 1", false)),
      bypassSlot2 (new juce::AudioParameterBool ("bypass2", "Bypass 2", false)),
      bypassSlot3 (new juce::AudioParameterBool ("bypass3", "Bypass 3", false))
{
    addParameter (muteInput);

    addParameter (processorSlot1);
    addParameter (processorSlot2);
    addParameter (processorSlot3);

    addParameter (bypassSlot1);
    addParameter (bypassSlot2);
    addParameter (bypassSlot3);
}
```

This tutorial makes use of the [GenericAudioProcessorEditor](https://docs.juce.com/master/classGenericAudioProcessorEditor.html "A type of UI component that displays the parameters of an AudioProcessor as a simple list of sliders,...") class, which automatically creates a [ComboBox](https://docs.juce.com/master/classComboBox.html "A component that lets the user choose from a drop-down list of choices.") for each of the parameters in the plug-in's processor that is an [AudioParameterChoice](https://docs.juce.com/master/classAudioParameterChoice.html "Provides a class of AudioProcessorParameter that can be used to select an indexed,...") type and a [ToggleButton](https://docs.juce.com/master/classToggleButton.html "A button that can be toggled on/off.") for each [AudioParameterBool](https://docs.juce.com/master/classAudioParameterBool.html "Provides a class of AudioProcessorParameter that can be used as a boolean value.") type.

> [!TIP]
>To learn more about audio parameters and how to customise them, please refer to [Tutorial: Adding plug-in parameters](/tutorials/tutorial_audio_parameter/). For a more seamless and elegant method for saving and loading parameters, you can take a look at [Tutorial: Saving and loading your plug-in state](/tutorials/tutorial_audio_processor_value_tree_state/).

In the first part of the tutorial when setting up the [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors."), we noticed that we call the `updateGraph()` helper function in the processBlock() callback of the `TutorialProcessor` class. The purpose of this function is to update the graph by reinstantiating the proper [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.") objects and nodes as well as reconnecting the graph depending on the current choices selected by the user so let's implement that helper function like this:

```cpp
void updateGraph()
{
    bool hasChanged = false;

    juce::Array<juce::AudioParameterChoice*> choices { processorSlot1,
        processorSlot2,
        processorSlot3 };

    juce::Array<juce::AudioParameterBool*> bypasses { bypassSlot1,
        bypassSlot2,
        bypassSlot3 };

    juce::ReferenceCountedArray<Node> slots;
    slots.add (slot1Node);
    slots.add (slot2Node);
    slots.add (slot3Node);
```

The function starts by declaring a local variable representing the state of the graph and whether it has changed since the last iteration of the audio block processing. It also creates arrays to facilitate iteration over the processor choices, bypass states and their corresponding nodes in the graph.

In the next part, we iterate over the three available processor slots and check the options that were selected for each of the [AudioParameterChoice](https://docs.juce.com/master/classAudioParameterChoice.html "Provides a class of AudioProcessorParameter that can be used to select an indexed,...") objects as follows:

```cpp
for (int i = 0; i < 3; ++i)
{
    auto& choice = choices.getReference (i);
    auto slot = slots.getUnchecked (i);

    if (choice->getIndex() == 0) // [1]
    {
        if (slot != nullptr)
        {
            mainProcessor->removeNode (slot.get());
            slots.set (i, nullptr);
            hasChanged = true;
        }
    }
    else if (choice->getIndex() == 1) // [2]
    {
        if (slot != nullptr)
        {
            if (slot->getProcessor()->getName() == "Oscillator")
                continue;

            mainProcessor->removeNode (slot.get());
        }

        slots.set (i, mainProcessor->addNode (std::make_unique<OscillatorProcessor>()));
        hasChanged = true;
    }
    else if (choice->getIndex() == 2) // [3]
    {
        if (slot != nullptr)
        {
            if (slot->getProcessor()->getName() == "Gain")
                continue;

            mainProcessor->removeNode (slot.get());
        }

        slots.set (i, mainProcessor->addNode (std::make_unique<GainProcessor>()));
        hasChanged = true;
    }
    else if (choice->getIndex() == 3) // [4]
    {
        if (slot != nullptr)
        {
            if (slot->getProcessor()->getName() == "Filter")
                continue;

            mainProcessor->removeNode (slot.get());
        }

        slots.set (i, mainProcessor->addNode (std::make_unique<FilterProcessor>()));
        hasChanged = true;
    }
}
```

- [1]: If the choice remains in the \"Empty\" state, we first check whether the node was previously instantiated to a different processor and if so, we remove the node from the graph, clear the reference to the node and set the `hasChanged` flag to true. Otherwise, the state has not changed and the graph does not need rebuilding.
- [2]: If the user chooses the \"Oscillator\" state, we first check whether the currently instantiated node is already an oscillator processor and if so, the state has not changed and we continue onto the next slot. Otherwise, if the slot was already occupied we remove the node from the graph, set the reference to a new node by instantiating the oscillator and set the `hasChanged` flag to true.
- [3]: We proceed to do the same for the \"Gain\" state and instantiate a gain processor if necessary.
- [4]: Again, we repeat the same process for the \"Filter\" state and instantiate a filter processor if needed.

The next section is only performed if the state of the graph has changed and we start connecting the nodes as follows:

```cpp
if (hasChanged)
{
    for (auto connection : mainProcessor->getConnections()) // [5]
        mainProcessor->removeConnection (connection);

    juce::ReferenceCountedArray<Node> activeSlots;

    for (auto slot : slots)
    {
        if (slot != nullptr)
        {
            activeSlots.add (slot); // [6]

            slot->getProcessor()->setPlayConfigDetails (getMainBusNumInputChannels(),
                getMainBusNumOutputChannels(),
                getSampleRate(),
                getBlockSize());
        }
    }

    if (activeSlots.isEmpty()) // [7]
    {
        connectAudioNodes();
    }
    else
    {
        for (int i = 0; i < activeSlots.size() - 1; ++i) // [8]
        {
            for (int channel = 0; channel < 2; ++channel)
                mainProcessor->addConnection ({ { activeSlots.getUnchecked (i)->nodeID, channel },
                    { activeSlots.getUnchecked (i + 1)->nodeID, channel } });
        }

        for (int channel = 0; channel < 2; ++channel) // [9]
        {
            mainProcessor->addConnection ({ { audioInputNode->nodeID, channel },
                { activeSlots.getFirst()->nodeID, channel } });
            mainProcessor->addConnection ({ { activeSlots.getLast()->nodeID, channel },
                { audioOutputNode->nodeID, channel } });
        }
    }

    connectMidiNodes();

    for (auto node : mainProcessor->getNodes()) // [10]
        node->getProcessor()->enableAllBuses();
}
```

- [5]: First we remove all the connections in the graph to start from a blank state.
- [6]: Then, we iterate over the slots and check whether they have an [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.") node within the graph. If so we add the node to our temporary array of active nodes and make sure to call the setPlayConfigDetails() function on the corresponding processor instance with channel, sample rate and block size information to prepare the node for future processing.
- [7]: Next, if there are no active slots found this means that all the choices are in an \"Empty\" state and the audio I/O processor nodes can be simply connected together.
- [8]: Otherwise, it means that there is at least one node that should lie between the audio I/O processor nodes. Therefore we can start connecting the active slots together in an ascending order of slot number. Notice here that the number of pairs of connections we need is only the number of active slots minus one.
- [9]: We can then finish connecting the graph by linking the audio input processor node to the first active slot in the chain and the last active slot to the audio output processor node.
- [10]: Finally, we connect the midi I/O nodes together and make sure that all the buses in the audio processors are enabled.

```cpp
for (int i = 0; i < 3; ++i)
{
    auto slot = slots.getUnchecked (i);
    auto& bypass = bypasses.getReference (i);

    if (slot != nullptr)
        slot->setBypassed (bypass->get());
}

audioInputNode->setBypassed (muteInput->get());

slot1Node = slots.getUnchecked (0);
slot2Node = slots.getUnchecked (1);
slot3Node = slots.getUnchecked (2);
}
```

In the last section of the `updateGraph()` helper function, we deal with the bypass state of the processors by checking whether the slot is active and bypass the [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.") if the check box is toggled. We also check whether to mute the input to avoid feedback loops when testing. Then, we assign back the newly-created nodes to their corresponding slots for the next iteration.

The plugin should now run by processing incoming audio through the loaded processors within the graph.

> [!NOTE]
> Exercise: Create an additional processor node of your choice and add it to the [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors."). (For example a processor handling midi messages.)

# Convert the plugin into an application

If you are interested in using the [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors.") within a standalone app, this optional section will delve into this in detail.

First of all, we have to convert our main `TutorialProcessor` class into a subclass of [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") instead of [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins."). To match the naming convention of other JUCE GUI applications we also rename the class name to `MainComponent` as follows:

```cpp
class MainComponent : public juce::Component,
                      private juce::Timer
{
public:
```

> [!TIP]
>If using a PIP file to follow this tutorial, make sure to change the \"mainClass\" and \"type\" fields to reflect the change and amend the \"dependencies\" field appropriately. If using the ZIP version of the project, make sure that the `Main.cpp` file follows the \"GUI Application\" template format.

When creating a plugin, all the IO device management and playback functionalities are controlled by the host and therefore we don\'t need to worry about setting these up. However, in a standalone application we have to manage this ourselves. This is why we declare an [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices.") and an [AudioProcessorPlayer](https://docs.juce.com/master/classAudioProcessorPlayer.html "An AudioIODeviceCallback object which streams audio through an AudioProcessor.") as private member variables in the `MainComponent` class to allow communication between our [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors.") and the audio IO devices available on the system.

```cpp
juce::AudioDeviceManager deviceManager;
juce::AudioProcessorPlayer player;
```

The [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices.") is a convenient class that manages audio and midi devices on all platforms and the [AudioProcessorPlayer](https://docs.juce.com/master/classAudioProcessorPlayer.html "An AudioIODeviceCallback object which streams audio through an AudioProcessor.") allows for easy playback through an [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors.").

In the constructor, instead of initialising plugin parameters we create regular GUI components and initialise the [AudioDeviceManager](https://docs.juce.com/master/classAudioDeviceManager.html "Manages the state of some audio and midi i/o devices.") and the [AudioProcessorPlayer](https://docs.juce.com/master/classAudioProcessorPlayer.html "An AudioIODeviceCallback object which streams audio through an AudioProcessor.") like so:

```cpp
MainComponent()
    : mainProcessor (new juce::AudioProcessorGraph())
{
```

```cpp
auto inputDevice = juce::MidiInput::getDefaultDevice();
auto outputDevice = juce::MidiOutput::getDefaultDevice();

mainProcessor->enableAllBuses();

deviceManager.initialiseWithDefaultDevices (2, 2); // [1]
deviceManager.addAudioCallback (&player); // [2]
deviceManager.setMidiInputDeviceEnabled (inputDevice.identifier, true);
deviceManager.addMidiInputDeviceCallback (inputDevice.identifier, &player); // [3]
deviceManager.setDefaultMidiOutputDevice (outputDevice.identifier);

initialiseGraph();

player.setProcessor (mainProcessor.get()); // [4]

setSize (600, 400);
startTimer (100);
}
```

Here we first initialise the device manager with the default audio device and two inputs and outputs each [1]. We then add the [AudioProcessorPlayer](https://docs.juce.com/master/classAudioProcessorPlayer.html "An AudioIODeviceCallback object which streams audio through an AudioProcessor.") as an audio callback to the device manager [2] and as a midi callback by using the default midi device [3]. After graph initialisation, we can set the [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors.") as the processor to play by calling the setProcessor() function on the [AudioProcessorPlayer](https://docs.juce.com/master/classAudioProcessorPlayer.html "An AudioIODeviceCallback object which streams audio through an AudioProcessor.") [4].

```cpp
~MainComponent() override
{
    auto device = juce::MidiInput::getDefaultDevice();

    deviceManager.removeAudioCallback (&player);
    deviceManager.setMidiInputDeviceEnabled (device.identifier, false);
    deviceManager.removeMidiInputDeviceCallback (device.identifier, &player);
}
```

Then in the destructor, we make sure to remove the [AudioProcessorPlayer](https://docs.juce.com/master/classAudioProcessorPlayer.html "An AudioIODeviceCallback object which streams audio through an AudioProcessor.") as an audio and midi callback on application shutdown.

Notice that unlike the plugin implementation, the [AudioProcessorPlayer](https://docs.juce.com/master/classAudioProcessorPlayer.html "An AudioIODeviceCallback object which streams audio through an AudioProcessor.") deals with processing the audio automatically and therefore it will take care of calling the prepareToPlay() and processBlock() functions on the [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors.") for us.

However we still need to find a way to update the graph when the user changes parameters and we do so by deriving the `MainComponent` from the [Timer](https://docs.juce.com/master/classTimer.html "Makes repeated callbacks to a virtual method at a specified time interval.") class and overriding the timerCallback() function like so:

```cpp
void timerCallback() override { updateGraph(); }
```

> [!TIP]
>Using a timer callback is not the most efficient solution and it is generally best practice to register as a listener to the appropriate components.

Finally, we modify the `updateGraph()` function to set the playback configuration details from the [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors.") instead of the main [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.") since the latter was replaced by the [AudioProcessorPlayer](https://docs.juce.com/master/classAudioProcessorPlayer.html "An AudioIODeviceCallback object which streams audio through an AudioProcessor.") in our standalone app scenario:

```cpp
for (auto slot : slots)
{
    if (slot != nullptr)
    {
        activeSlots.add (slot);

        slot->getProcessor()->setPlayConfigDetails (mainProcessor->getMainBusNumInputChannels(),
            mainProcessor->getMainBusNumOutputChannels(),
            mainProcessor->getSampleRate(),
            mainProcessor->getBlockSize());
    }
}
```

Your plugin should now run as an application after these changes.

> [!WARNING]
> Again, beware of screaming feedback when testing the app with built-in inputs and outputs. You can avoid this problem by using headphones.

<!-- -->

> [!TIP]
>The source code for this modified version of the plug-in can be found in the `AudioProcessorGraphTutorial_02.h` file of the demo project.

# Summary

In this tutorial, we have learnt how to manipulate an [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors.") to cascade the effects of plugins. In particular, we have:

- Built a channel strip with a series of audio processors.
- Learnt how to connect nodes in an [AudioProcessorGraph](https://docs.juce.com/master/classAudioProcessorGraph.html "A type of AudioProcessor which plays back a graph of other AudioProcessors.").
- Created a standalone app from the graph using an [AudioProcessorPlayer](https://docs.juce.com/master/classAudioProcessorPlayer.html "An AudioIODeviceCallback object which streams audio through an AudioProcessor.").

# See also

- [Tutorial: Adding plug-in parameters](/tutorials/tutorial_audio_parameter/)
- [Tutorial: Saving and loading your plug-in state](/tutorials/tutorial_audio_processor_value_tree_state/)
- [Tutorial: Configuring the right bus layouts for your plugins](/tutorials/tutorial_audio_bus_layouts/)
