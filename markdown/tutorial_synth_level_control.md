# Tutorial: Control audio levels

This tutorial shows how to process audio to change its output level. This is achieved by processing the low-level audio sample data.

**LEVEL:** Intermediate<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**CLASSES:** [AudioAppComponent](https://docs.juce.com/master/classAudioAppComponent.html "A base class for writing audio apps that stream from the audio i/o devices."), [Random](https://docs.juce.com/master/classRandom.html "A random number generator."), [AudioSourceChannelInfo](structAudioSourceChannelInfo.html "Used by AudioSource::getNextAudioBlock()."), [AudioBuffer](https://docs.juce.com/master/classAudioBuffer.html "A multi-channel buffer containing floating point audio samples."), [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value.")

# Getting started

<p class="callout-tip">
This tutorial leads on from [Tutorial: Build a white noise generator](/tutorials/tutorial_simple_synth_noise/), which you should have read and understood first. It also assumes that you are familiar with manipulating [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value.") objects (see [Tutorial: The Slider class](/tutorials/tutorial_slider_values/)).
</p>

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/SynthLevelControlTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/SynthLevelControlTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project presents a window containing a single slider that can be used to control the output level of a white noise generator. This is shown in the following screenshot.

![](/_images/tutorial_synth_level_control_screenshot1.png "The demo project main window showing the level control slider.")

Run the project from within your IDE to confirm that you can indeed control the level of the white noise generated.

# Level control as multiplication

Examining the code, you may notice that our `MainContentComponent` class does _not_ inherit the from [Slider::Listener](https://docs.juce.com/master/classSlider.html#a1977aeac9b4363e8ed0cac0ac103055a) class. In fact, we grab the value of the slider as soon as our `getNextAudioBlock()` function is called:

```cpp
void getNextAudioBlock (const juce::AudioSourceChannelInfo& bufferToFill) override
{
    auto level = (float) levelSlider.getValue();
```

This technique is acceptable in a simple application such this, but in more complex applications you almost certainly want to use a different technique. It is better practice to store the values that controls your audio processing system in the \"data model\" of your application rather than relying solely on your UI controls to store these values.

> [!WARNING]
> While it is acceptable to call [Slider::getValue()](https://docs.juce.com/master/classSlider.html#a288c6f5c7a76100a1e7526e002e10eb5 "Returns the slider's current value.") in the `getNextAudioBlock()` function you _must not_ call the [Slider::setValue()](https://docs.juce.com/master/classSlider.html#a430a5c4e56b39dd622f5800f787e0822 "Changes the slider's current value.") function within this the `getNextAudioBlock()` function. This is because this code is running on the audio thread. You must not do anything that changes the state of a UI object from the audio thread, although it is acceptable to query the state of UI objects, as long as you\'re sure these don\'t have side effects.

In order to generate the white noise at the level dictated by the slider, we need to perform some basic arithmetic. The [Random::nextFloat()](https://docs.juce.com/master/classRandom.html#aec88d4e5cf44faaa038f6cfb41e96406 "Returns the next random floating-point number.") function always generates values between 0.0 and 1.0. If we plot this as an audio waveform, then it would look something like this:

![](/_images/tutorial_synth_level_control_graph1.png "Random numbers generated between 0.0 and 1.0")

A simple way to approach this is to first scale the noise, such that it is always scaled between -1.0 and 1.0. To do this, we can multiply by 2.0 and subtract 1.0. This would result in a signal as shown below:

![](/_images/tutorial_synth_level_control_graph2.png "Random numbers generated between -1.0 and 1.0")

This is implemented in code within our `getNextAudioBlock()` function as follows:

```cpp
void getNextAudioBlock (const juce::AudioSourceChannelInfo& bufferToFill) override
{
    auto level = (float) levelSlider.getValue();

    for (auto channel = 0; channel < bufferToFill.buffer->getNumChannels(); ++channel)
    {
        auto* buffer = bufferToFill.buffer->getWritePointer (channel, bufferToFill.startSample);

        for (auto sample = 0; sample < bufferToFill.numSamples; ++sample)
        {
            auto noise = random.nextFloat() * 2.0f - 1.0f;
            buffer[sample] = noise * level;
        }
    }
}
```

## A minor optimisation

Reducing the number of arithmetic operations is often a goal for DSP. This is one case where we can avoid performing one multiplication operation per sample. To achieve this, first we could multiply the values between 0.0 and 1.0 by a value equal to _double_ the required output level. Let's say we want the level to be 0.25. If we multiply our random values by 0.5 we would get something like this:

![](/_images/tutorial_synth_level_control_graph3.png "Random numbers scaled between 0.0 and 0.5")

Finally, we need to offset the random values such that they are centred around zero. To do this, we subtract a value equal to half of the value we multiplied by. This is, of course, is the original level value. The final result will be something like this:

![](/_images/tutorial_synth_level_control_graph4.png "Random numbers scaled and offset to between -0.25 and 0.25")

This could be implemented in code via our `getNextAudioBlock()` function where you should see that there is now one fewer multiplication operation per sample:

```cpp
void getNextAudioBlock (const juce::AudioSourceChannelInfo& bufferToFill) override
{
    auto level = (float) levelSlider.getValue();
    auto levelScale = level * 2.0f;

    for (auto channel = 0; channel < bufferToFill.buffer->getNumChannels(); ++channel)
    {
        auto* buffer = bufferToFill.buffer->getWritePointer (channel, bufferToFill.startSample);

        for (auto sample = 0; sample < bufferToFill.numSamples; ++sample)
            buffer[sample] = random.nextFloat() * levelScale - level;
    }
}
```

> [!TIP]
>The code for this revised version can be found in the `SynthLevelControlTutorial_02.h` file of the demo project.

One remaining problem with this implementation is that the level is only updated to a new constant value for each of the audio blocks generated. In this case, it isn\'t audible (as the source sound in white noise). If this was applied to most audio content, then this technique would introduce audio artifacts. This is because the level value may be jumping quite abruptly between each audio block, rather than changing smoothly during the audio blocks. Techniques for avoiding these artifacts (such as audible crackling when the level changes) are explored in other tutorials (see [Tutorial: Build a sine wave synthesiser](/tutorials/tutorial_sine_synth/)).

<p class="callout-note"> Exercise:
Add a second slider to the user interface. Use the first slider to control the level of the left channel and this second slider to control the level of the right channel.
</p>

# Summary

This tutorial has illustrated how to control the level of an audio signal. We have covered the following topics:

- Modifying the level of a signal by multiplying the signal values by the required output level.
- Using a slider to control an audio level.

# See also

- [Tutorial: The Slider class](/tutorials/tutorial_slider_values/)
- [Tutorial: Build a white noise generator](/tutorials/tutorial_simple_synth_noise/)
- [Tutorial: Build a sine wave synthesiser](/tutorials/tutorial_sine_synth/)
- [Tutorial: Control audio levels using decibels](/tutorials/tutorial_synth_db_level_control/)
- [Tutorial: Build a MIDI synthesiser](/tutorials/tutorial_synth_using_midi_input/)
- [Tutorial: Wavetable synthesis](/tutorials/tutorial_wavetable_synth/)
