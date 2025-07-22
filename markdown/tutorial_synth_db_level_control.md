---
title: Control audio levels using decibels
---
# Tutorial: Control audio levels using decibels

This tutorial shows how to process audio to change its output level using the decibel scale. This is a more common way in which to present audio level values to the user within audio applications.

**LEVEL:** Intermediate<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**CLASSES:** [Decibels](https://docs.juce.com/master/classDecibels.html "This class contains some helpful static methods for dealing with decibel values."), [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value."), [String](https://docs.juce.com/master/classString.html "The JUCE String class!")

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/SynthDecibelLevelControlTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/SynthDecibelLevelControlTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

> [!TIP]
>This tutorial leads on from [Tutorial: Control audio levels](/tutorials/tutorial_synth_level_control/). You should have followed that tutorial before continuing.

# The demo project

In a similar way to the [Tutorial: Control audio levels](/tutorials/tutorial_synth_level_control/) demo project, the demo project for this tutorial presents a window containing a single slider. This time the slider value is represented in [decibels](https://en.wikipedia.org/wiki/Decibel) . This value in [decibels](https://en.wikipedia.org/wiki/Decibel) needs to be converted to a linear gain value before being used in the audio processing algorithm. Most audio applications express gain to users in [decibels](https://en.wikipedia.org/wiki/Decibel) as this often _feels_ more natural as the values are varied (or compared). The user interface for the demo project is shown in the following screenshot.

![](/_images/tutorial_synth_db_level_control_screenshot1.png "The demo project main window showing the control slider in decibels.")

# Creating a customised slider

Notice this time that the text displayed adjacent to the slider not only shows the value in [decibels](https://en.wikipedia.org/wiki/Decibel) but it also shows the suffix \"dB\". This is achieved by creating a custom slider class, `DecibelSlider` that inherits from the [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value.") class. In this custom slider class the text box interface is customised to display the value in [decibels](https://en.wikipedia.org/wiki/Decibel) . While a suffix to the text displayed within a [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value.") object's text box can be added using the [Slider::setTextValueSuffix()](https://docs.juce.com/master/classSlider.html#ac416a101b5d9a504f61e2f50dc593f61 "Sets a suffix to append to the end of the numeric value when it's displayed as a string.") function, we need one more customisation. This is to customise the way that values are converted such that we can display **-INF dB** when the level drops very low.

> [!TIP]
>*INF* refers to *infinity* where -INF dB is treated as silence for the purposes of our application.

## The Decibels class

The [Decibels](https://docs.juce.com/master/classDecibels.html "This class contains some helpful static methods for dealing with decibel values.") class contains a number of static functions necessary for the conversion between the values in [decibels](https://en.wikipedia.org/wiki/Decibel) and linear gain. It also provides a simple means of converting values in [decibels](https://en.wikipedia.org/wiki/Decibel) to a [String](https://docs.juce.com/master/classString.html "The JUCE String class!") object. For example, we override the virtual function [Slider::getTextFromValue()](https://docs.juce.com/master/classSlider.html#a5a1693166d0815f812202b1a3a6eb202 "Turns the slider's current value into a text string.") by using the [Decibels::toString()](https://docs.juce.com/master/classDecibels.html#a808a763495ccbe5d504fbd501bb85e0d "Converts a decibel reading to a string.") function (in the `DecibelSlider` class) like so:

```cpp
juce::String getTextFromValue (double value) override
{
    return juce::Decibels::toString (value);
}
```

This allows our `DecibelSlider` class to display the appropriate text in its text box for a given slider value.

The [Decibels](https://docs.juce.com/master/classDecibels.html "This class contains some helpful static methods for dealing with decibel values.") class doesn\'t have a function to convert a [String](https://docs.juce.com/master/classString.html "The JUCE String class!") object back to a value in [decibels](https://en.wikipedia.org/wiki/Decibel) , so we need to write our own. Here we override the [Slider::getValueFromText()](https://docs.juce.com/master/classSlider.html#a532774d3294a058784f7d4291b33b720 "Subclasses can override this to convert a text string to a value.") virtual function (again in the `DecibelSlider` class):

```cpp
double getValueFromText (const juce::String& text) override
{
    auto minusInfinitydB = -100.0;

    auto decibelText = text.upToFirstOccurrenceOf ("dB", false, false).trim(); // [1]

    return decibelText.equalsIgnoreCase ("-INF") ? minusInfinitydB
                                                 : decibelText.getDoubleValue(); // [2]
}
```

This enables the user to enter a value into the text box and have it checked and converted to a valid value for our slider. To do this we:

- [1]: Strip off the \"dB\" suffix from the text (if it is present), and trim the text to remove any leading or trailing whitespace (using the [String::trim()](https://docs.juce.com/master/classString.html#a5ed648f71d99bb8aef15e8ba35422a30 "Returns a copy of this string with any whitespace characters removed from the start and end.") function).
- [2]: Check if the remaining text is \"-INF\" and return -100 in this case (this is the default value in [decibels](https://en.wikipedia.org/wiki/Decibel) for -INF dB that the [Decibels](https://docs.juce.com/master/classDecibels.html "This class contains some helpful static methods for dealing with decibel values.") class uses, see the [Notes](#tutorial_synth_db_level_control_notes) for this tutorial for a discussion on this). Otherwise we convert this remaining text to a `double` value and return it.

# Using the slider value

In the [Tutorial: Control audio levels](/tutorials/tutorial_synth_level_control/) we accessed the slider's value directly in the `getNextAudioBlock()` function. Since the conversion from [decibels](https://en.wikipedia.org/wiki/Decibel) to linear gain involves some potentially CPU-intensive arithmetic, it would be wise to avoid performing the conversion too often, especially on the audio thread. In the demo project for this tutorial we store a `float` member `level` in the `MainContentComponent` class and update this value when the slider changes using a listener.

We initialise the `level` member to zero in the constructor and convert this to [decibels](https://en.wikipedia.org/wiki/Decibel) using the [Decibels::gainToDecibels()](https://docs.juce.com/master/classDecibels.html#a2a2c7152fe5560836cd81e6d167da537 "Converts a gain level into a dBFS value.") function to give the slider its initial position (using the [Slider::setValue()](https://docs.juce.com/master/classSlider.html#a430a5c4e56b39dd622f5800f787e0822 "Changes the slider's current value.") function) like so:

```cpp
MainContentComponent()
{
```

```cpp
decibelSlider.setValue (juce::Decibels::gainToDecibels (level));
decibelLabel.setText ("Noise Level in dB", juce::dontSendNotification);
```

## Converting decibels to linear gain

In our lambda function of our [Slider::onValueChange](https://docs.juce.com/master/classSlider.html#a680d007f6a824a28a60aa05b4045e794 "You can assign a lambda to this callback object to have it called when the slider value is changed.") helper object we perform the conversion from the [decibels](https://en.wikipedia.org/wiki/Decibel) scale used by the slider to the linear gain value we need for audio processing:

```cpp
decibelSlider.onValueChange = [this] { level = juce::Decibels::decibelsToGain ((float) decibelSlider.getValue()); };
```

This function will be called when we first set the slider's value using the [Slider::setValue()](https://docs.juce.com/master/classSlider.html#a430a5c4e56b39dd622f5800f787e0822 "Changes the slider's current value.") function in our constructor. This will call the lambda function assigned to the [Slider::onValueChange](https://docs.juce.com/master/classSlider.html#a680d007f6a824a28a60aa05b4045e794 "You can assign a lambda to this callback object to have it called when the slider value is changed.") helper object when the value changes and our `level` member will be set correctly.

> [!NOTE]
> Exercise: Add another [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") object to the `MainContentComponent` class that displays the linear gain value. - Add another [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value.") object to the `MainContentComponent` class that displays linear gain value allowing the user to view and specify the noise level using either slider. Both sliders should update correctly when either slider is moved. (See [Tutorial: The Slider class](/tutorials/tutorial_slider_values/) for a simple of example of converting between different units.)

## Processing the audio

In our `MainContentComponent::getNextAudioBlock()` we process the audio:

```cpp
void getNextAudioBlock (const juce::AudioSourceChannelInfo& bufferToFill) override
{
    auto currentLevel = level;
    auto levelScale = currentLevel * 2.0f;

    for (auto channel = 0; channel < bufferToFill.buffer->getNumChannels(); ++channel)
    {
        auto* buffer = bufferToFill.buffer->getWritePointer (channel, bufferToFill.startSample);

        for (auto sample = 0; sample < bufferToFill.numSamples; ++sample)
            buffer[sample] = random.nextFloat() * levelScale - currentLevel;
    }
}
```

Note that this is almost identical to the `getNextAudioBlock()` function from [Tutorial: Control audio levels](/tutorials/tutorial_synth_level_control/) _except_ that we just take a function-local copy of the `level` value then calculate our `levelScale` value as before.

## Issues with this approach

One issue with this approach is that the `level` variable might change its value abruptly during the execution of the audio thread (in this case, in between two calls to `getNextAudioBlock` ). Such changes will typically introduce audio artifacts such as an audible crackling. To avoid this, in a real-world synthesiser we would want a `level` value that changes smoothly. Techniques to accomplish this are explored in other tutorials (see [Tutorial: Build a sine wave synthesiser](/tutorials/tutorial_sine_synth/)).

Another issue is related to thread safety. Writing to a member variable like `level` in one thread (the GUI thread) and reading the same value from another thread (the audio thread) is technically undefined behaviour in C++ if no thread synchronisation takes place (either via critical sections or using atomics). These issues are beyond the scope of this tutorial and will be discussed in a future tutorial. In this particular case we don\'t have to worry about this too much, because on typical architectures (x86, x86_64, ARM) reading and writing a single `float` is an atomic operation: the reads and writes cannot be intermingled and are generally safe.

Thinking further, it might be tempting to optimise the code further by making `levelScale` a member variable too (and therefore not calculate it for every call to the `getNextAudioBlock()` function). But then the two member variables would not be updated as a single atomic operation anymore. There are, of course, ways around this as well, but this is beyond the scope of this tutorial.

# Notes

The code presented in the demo project for this tutorial assumes that we want to treat a value of -100 dB or lower as -INF dB (that is a linear gain value of zero). This value of -100 dB is the default value used by the [Decibels](https://docs.juce.com/master/classDecibels.html "This class contains some helpful static methods for dealing with decibel values.") class but you can override this in its calculations. This is achieved by providing an additional argument to each of the functions in the [Decibels](https://docs.juce.com/master/classDecibels.html "This class contains some helpful static methods for dealing with decibel values.") class specifying which value should be treated as -INF dB. For example, to use -96 dB (and below) as -INF dB when updating our slider value in the `MainContentComponent` constructor we could do this:

```cpp
decibelSlider.setValue (juce::Decibels::gainToDecibels (level, -96.0f));
```

But of course we need to ensure that all parts of our application use the same value for -INF dB. There is one potential problem with the code for the demo project since we hard-code `-100.0` in our `DecibelSlider::getValueFromText()` function. If the [Decibels](https://docs.juce.com/master/classDecibels.html "This class contains some helpful static methods for dealing with decibel values.") class changes its default value (for some reason) then our code would break. Unfortunately, this default value is a private member of the [Decibels](https://docs.juce.com/master/classDecibels.html "This class contains some helpful static methods for dealing with decibel values.") class, so we can\'t ask the [Decibels](https://docs.juce.com/master/classDecibels.html "This class contains some helpful static methods for dealing with decibel values.") class for its default value. Instead, we would need to specify our own default value and use this throughout.

> [!NOTE]
> Exercise: Modify the demo project to specify your own default value for -INF as -96 dB. You should need to update both the `DecibelSlider` class and the `MainContentComponent` class.

# Summary

In this tutorial we have introduced:

- The [Decibels](https://docs.juce.com/master/classDecibels.html "This class contains some helpful static methods for dealing with decibel values.") class.
- Converting back and forth between the [decibels](https://en.wikipedia.org/wiki/Decibel) scale and linear gain.
- Creating a custom slider for displaying values in the [decibels](https://en.wikipedia.org/wiki/Decibel) scale.

# See also

- [Tutorial: The Slider class](/tutorials/tutorial_slider_values/)
- [Tutorial: Build a white noise generator](/tutorials/tutorial_simple_synth_noise/)
- [Tutorial: Build a sine wave synthesiser](/tutorials/tutorial_sine_synth/)
- [Tutorial: Control audio levels](/tutorials/tutorial_synth_level_control/)
- [Tutorial: Build a MIDI synthesiser](/tutorials/tutorial_synth_using_midi_input/)
- [Tutorial: Wavetable synthesis](/tutorials/tutorial_wavetable_synth/)
