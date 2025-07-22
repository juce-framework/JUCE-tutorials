---
title: Create a basic Audio/MIDI plugin, Part 2: Coding your plug-in
---
# Tutorial: Create a basic Audio/MIDI plugin, Part 2: Coding your plug-in

This tutorial follows on from [Tutorial: Create a basic Audio/MIDI plugin, Part 1: Setting up](/tutorials/tutorial_create_projucer_basic_plugin/), and will talk through starting from a clean project and ending up with a fully functioning, if somewhat simple, plug-in that can react to incoming MIDI notes.

**LEVEL:** Intermediate<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**Plugin Format:** VST, VST3, AU, Standalone<br/>
**CLASSES:** [AudioProcessorEditor](https://docs.juce.com/master/classAudioProcessorEditor.html "Base class for the component that acts as the GUI for an AudioProcessor."), [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins."), [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value."), [MidiMessage](https://docs.juce.com/master/classMidiMessage.html "Encapsulates a MIDI message."), [MidiBuffer](https://docs.juce.com/master/classMidiBuffer.html "Holds a sequence of time-stamped midi events.")

# Getting started

Launch the Projucer and create a new audio plug-in project with the name **TutorialPlugin** . If you don\'t remember how to do that, please refer to [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# Orientation

A newly-created audio plug-in project contains two main classes: `PluginProcessor` handles the audio and MIDI IO and processing logic, and `PluginEditor` handles any on screen GUI controls or visualisations.

When passing information between these two it is best to consider the processor as the parent of the editor. There is only one plug-in processor whereas you can create multiple editors. Each editor has a reference to the processor such that it can edit or access information and parameters from the audio thread. It is the editor's job to set and get information on this processor thread and not the other way around.

The main function we will be editing in the `PluginProcessor.cpp` file is the `processBlock()` method. This receives and produces both audio and MIDI data to the plug-in output. The main function we will change in the `PluginEditor.cpp` file is the constructor, where we initialise and set up our window and GUI objects, and also the `paint()` method where we can draw extra controls and custom GUI components.

The editor constructor currently has one method call --- `setSize (400, 300)` --- which sets the size of our plug-in window. Let's make a smaller window of `(200, 200)` for this simple application.

```cpp
TutorialPluginAudioProcessorEditor::TutorialPluginAudioProcessorEditor (TutorialPluginAudioProcessor& p)
    : AudioProcessorEditor (&p), audioProcessor (p)
{
    // This is where our plugin’s editor size is set.
    setSize (200, 200);
}
```

## Create a simple GUI control

We will create a slider object to change the volume of MIDI messages as they come in.

Create a new [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value.") object in the Editor header file called `midiVolume` [1]:

```cpp
class TutorialPluginAudioProcessorEditor : public juce::AudioProcessorEditor
{
public:
    TutorialPluginAudioProcessorEditor (TutorialPluginAudioProcessor&);
    ~TutorialPluginAudioProcessorEditor();

    //===================================================================
    void paint (juce::Graphics&) override;
    void resized() override;

private:
    // This reference is provided as a quick way for your editor to
    // access the processor object that created it.
    TutorialPluginAudioProcessor& audioProcessor;

    juce::Slider midiVolume; // [1]

    JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR (TutorialPluginAudioProcessorEditor)
};
```

> [!TIP]
>The [AudioProcessorEditor](https://docs.juce.com/master/classAudioProcessorEditor.html "Base class for the component that acts as the GUI for an AudioProcessor.") plays the same role in an Audio Plug-in that the *main content component* has in a standalone app. See [Tutorial: The main component](/tutorials/tutorial_main_component/).

We can set the properties of this slider with various functions in the editor constructor. We must also call `addAndMakeVisible (&midiVolume)` to attach our slider to the editor. There are many different slider styles and parameters to use and experiment with in your own project. For this tutorial adjust the slider parameters such that your editor constructor looks like this:

```cpp
TutorialPluginAudioProcessorEditor::TutorialPluginAudioProcessorEditor (TutorialPluginAudioProcessor& p)
    : AudioProcessorEditor (&p), audioProcessor (p)
{
    // This is where our plugin’s editor size is set.
    setSize (200, 200);

    // these define the parameters of our slider object
    midiVolume.setSliderStyle (juce::Slider::LinearBarVertical);
    midiVolume.setRange (0.0, 127.0, 1.0);
    midiVolume.setTextBoxStyle (juce::Slider::NoTextBox, false, 90, 0);
    midiVolume.setPopupDisplayEnabled (true, false, this);
    midiVolume.setTextValueSuffix (" Volume");
    midiVolume.setValue (1.0);

    // this function adds the slider to the editor
    addAndMakeVisible (&midiVolume);
}
```

JUCE windows have a method called `resized()` that is called once at the initialisation of the window and every time the window is resized by the user (if resizing is enabled). This is a good place to set the size and position of our sliders (and other GUI components) so they can be positioned relative to the window bounds.

```cpp
void TutorialPluginAudioProcessorEditor::resized()
{
    // sets the position and size of the slider with arguments (x, y, width, height)
    midiVolume.setBounds (40, 30, 20, getHeight() - 60);
}
```

Lets also change the `"Hello World"` text to `"Midi Volume"` in the `paint()` function and move it to the top. This function is where all custom shapes and GUI elements are drawn to the window.

```cpp
void TutorialPluginAudioProcessorEditor::paint (juce::Graphics& g)
{
    // fill the whole window white
    g.fillAll (juce::Colours::white);

    // set the current drawing colour to black
    g.setColour (juce::Colours::black);

    // set the font size and draw text to the screen
    g.setFont (15.0f);

    g.drawFittedText ("Midi Volume", 0, 0, getWidth(), 30, juce::Justification::centred, 1);
}
```

> [!TIP]
>You can learn more about Components and their methods `paint()` and `resized()` in [Tutorial: The Graphics class](/tutorials/tutorial_graphics_class/) and [Tutorial: Parent and child components](/tutorials/tutorial_component_parents_children/).

Running this program should create a plug-in that looks like this in the host editor:

![](/_images/tutorial_code_basic_plugin_1.png)

## Pass control information to the processor class

We now have an control that we can adjust, but that doesn't actually control anything. We need to intercept the incoming MIDI data and replace the note on volume with the volume of our slider, and this is done in the processor. In order to get the slider value to control the MIDI effect on the processor thread we need to create a new variable on the processor thread that we can use the slider to change.

Create a new public float variable called `noteOnVel` in the processor class header. This is the variable that we will set with the slider.

```cpp
public:
float noteOnVel;
```

We need to set this value whenever the slider is changed. To do this we use a slider listener callback function. Any class can inherit slider listener functionality but for the purposes of this tutorial we will add this functionality to the editor class.

> [!TIP]
>For a more in-depth description of listeners please see [Tutorial: Listeners and Broadcasters](/tutorials/tutorial_listeners_and_broadcasters/).

Add the inheritance [2] and the default callback function [3] so the editor class looks like this:

```cpp
class TutorialPluginAudioProcessorEditor : public juce::AudioProcessorEditor,
                                           private juce::Slider::Listener // [2]
{
public:
    TutorialPluginAudioProcessorEditor (TutorialPluginAudioProcessor&);
    ~TutorialPluginAudioProcessorEditor();

    //==================================================================
    // This is just a standard Juce paint method...
    void paint (juce::Graphics& g) override;

    void resized() override;

private:
    void sliderValueChanged (juce::Slider* slider) override; // [3]

    //==================================================================
    // This reference is provided as a quick way for your editor to
    // access the processor object that created it.
    TutorialPluginAudioProcessor& audioProcessor;

    juce::Slider midiVolume;

    JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR (TutorialPluginAudioProcessorEditor)
};
```

Now we add the slider listener to our volume slider in the editor constructor:

```cpp
TutorialPluginAudioProcessorEditor::TutorialPluginAudioProcessorEditor (TutorialPluginAudioProcessor& p)
    : AudioProcessorEditor (&p), audioProcessor (p)
{
    // ...

    // add the listener to the slider
    midiVolume.addListener (this);
}
```

...and insert the listener function that sets our public processor volume variable:

```cpp
void TutorialPluginAudioProcessorEditor::sliderValueChanged (juce::Slider* slider)
{
    audioProcessor.noteOnVel = midiVolume.getValue();
}
```

We now have a slider that controls our variable in the processor class. We now need to use this processor variable to alter our MIDI data.

## Modify MIDI notes

The `processBlock()` method in the processor class receives and produces both MIDI and audio buffers in real time. We are going to iterate through the midi buffer to intercept signals of `noteOn` type and set their velocity to the value of our slider.

The MIDI messages are all passed through this function. To alter the MIDI as it passes through we create a new [MidiBuffer](https://docs.juce.com/master/classMidiBuffer.html "Holds a sequence of time-stamped midi events.") object called `processedMidi` and append our modified MIDI signals to this new buffer before swapping it with the original at the end (this avoids direct modification problems). Remove the current code in the `processBlock()` method (this handles the audio buffer, which we do not need for this tutorial) and replace it with the code below.

```cpp
void TutorialPluginAudioProcessor::processBlock (juce::AudioBuffer<float>& buffer, juce::MidiBuffer& midiMessages)
{
    buffer.clear();

    juce::MidiBuffer processedMidi;

    for (const auto metadata : midiMessages)
    {
        auto message = metadata.getMessage();
        const auto time = metadata.samplePosition;

        if (message.isNoteOn())
        {
            message = juce::MidiMessage::noteOn (message.getChannel(),
                message.getNoteNumber(),
                (juce::uint8) noteOnVel);
        }

        processedMidi.addEvent (message, time);
    }

    midiMessages.swapWith (processedMidi);
}
```

Run the plug-in in the host environment and you will see that all MIDI note on signals are coming through our plug-in have the value set with our slider. The `if()` statement above can be also used to modify and apply various transformations and effects to other types of incoming MIDI signals. With these methods you can build more complex effects and GUIs.

> [!NOTE]
> Exercise: Experiment with other GUI components such as buttons and sliders, check the JUCE DemoRunner for a taste of JUCE's capabilities, and refer back to the API documentation for more information.

<!-- -->

> [!TIP]
>Generating audio using the incoming MIDI notes will be covered in a future tutorial (see [Tutorial: Build a MIDI synthesiser](/tutorials/tutorial_synth_using_midi_input/)). For now, please have a look at the `AudioPluginDemo` , which is located in `JUCE/examples/Plugins` .

# Summary

After reading this tutorial, you should be able to:

- Create an audio plug-in with a basic GUI,
- Let your plug-in receive MIDI data.

# See also

- [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/)
- [Tutorial: Create a basic Audio/MIDI plugin, Part 1: Setting up](/tutorials/tutorial_create_projucer_basic_plugin/)
- [Tutorial: Plugin examples](/tutorials/tutorial_plugin_examples/)
- [Tutorial: Configuring the right bus layouts for your plugins](/tutorials/tutorial_audio_bus_layouts/)
- [Tutorial: The main component](/tutorials/tutorial_main_component/)
- [Tutorial: The Graphics class](/tutorials/tutorial_graphics_class/)
- [Tutorial: Parent and child components](/tutorials/tutorial_component_parents_children/)
- [Tutorial: Listeners and Broadcasters](/tutorials/tutorial_listeners_and_broadcasters/)
