# Tutorial: Build a sine wave synthesiser

This tutorial introduces simple sine wave synthesis. We show how to manage the state of a sine wave oscillator and write data to the audio output.

**LEVEL:** Intermediate<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**CLASSES:** [AudioAppComponent](https://docs.juce.com/master/classAudioAppComponent.html "A base class for writing audio apps that stream from the audio i/o devices."), [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value."), [MathConstants](structMathConstants.html "Commonly used mathematical constants.")

# Getting started

> [!TIP]
>This tutorial leads on from [Tutorial: Control audio levels](/tutorials/tutorial_synth_level_control/), which you should have read and understood first.

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/SineSynthTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/SineSynthTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project is based on the Audio Application template from the Projucer. It presents a single slider to the user to control the frequency of a sine wave.

# Generating a sine wave

This tutorial synthesises a sine wave using the standard library function `std::sin()` . In order to use this we need to maintain a state for our sine wave generation by storing the current _phase angle_ and the amount by which the phase angle needs to increment for each output sample. This size of this change per sample (\"delta\") is dependent on the sample rate of the output and the frequency of the sine wave we want to generate.

> [!WARNING]
> Most synthesis applications and plug-ins are likely not to use the `std::sin()` function as it probably isn\'t the most efficient technique. Commonly a [wavetable](https://en.wikipedia.org/wiki/Table-lookup_synthesis) would be used, see [Tutorial: Wavetable synthesis](/tutorials/tutorial_wavetable_synth/). Wavetables also allow for waveshapes other than sine waves, too.

## Maintaining our state

In our `MainContentComponent` class we store three `double` members [1]:

```cpp
double currentSampleRate = 0.0, currentAngle = 0.0, angleDelta = 0.0; // [1]
```

We have a simple function that updates the `angleDelta` member:

```cpp
void updateAngleDelta()
{
    auto cyclesPerSample = frequencySlider.getValue() / currentSampleRate; // [2]
    angleDelta = cyclesPerSample * 2.0 * juce::MathConstants<double>::pi; // [3]
}
```

- [2]: First we calculate the number of cycles that will need to complete for each output sample.
- [3]: Then this is multiplied by the length of a whole sine wave cycle, which is `2pi` radians.

Before this function can work correctly we need to know the output sample rate. This is because we need to know how frequently the samples are being generated, this is in order to know the amount of change that is needed per sample. We are passed the sample rate by the [AudioAppComponent::prepareToPlay()](https://docs.juce.com/master/classAudioAppComponent.html#a70634aa3ffaf6e7ff0d233e5933a063d "Tells the source to prepare for playing.") callback function:

```cpp
void prepareToPlay (int, double sampleRate) override
{
    currentSampleRate = sampleRate;
    updateAngleDelta();
}
```

Here we store a copy of the sample rate value and call our `updateAngleDelta()` function initially.

## Using the slider value

When the slider is moved while the app is running, we need to update the `angleDelta` member again:

```cpp
frequencySlider.onValueChange = [this] {
    if (currentSampleRate > 0.0)
        updateAngleDelta();
};
```

Here we check that the sample rate is valid, before calling the `updateAngleDelta()` function again.

## Outputting the audio

During the `getNextAudioBlock()` callback we need to generate the actual sine wave and write it to the output:

```cpp
void getNextAudioBlock (const juce::AudioSourceChannelInfo& bufferToFill) override
{
    auto level = 0.125f;
    auto* leftBuffer = bufferToFill.buffer->getWritePointer (0, bufferToFill.startSample);
    auto* rightBuffer = bufferToFill.buffer->getWritePointer (1, bufferToFill.startSample);

    for (auto sample = 0; sample < bufferToFill.numSamples; ++sample)
    {
        auto currentSample = (float) std::sin (currentAngle);
        currentAngle += angleDelta;
        leftBuffer[sample] = currentSample * level;
        rightBuffer[sample] = currentSample * level;
    }
}
```

For each output sample we calculate the sine function for the current angle, then increment the angle for the next sample. Notice that we bring the level down to `0.125` as a full scale sine wave will be very loud! We could (and perhaps should) _wrap_ the current angle value back to zero when it reaches `2pi` . Since larger values still return a valid value we can actually avoid this calculation. We get something like that shown in the following image:

![](/_images/tutorial_synth_sine_graph1.png "A full scale ±1.0 sine wave showing the phase angle in radians.")

## The slider configuration

You may have noticed that the slider value changes non-linearly (if not you should try this out now). These changes are, in fact, logarithmic. This gives us higher resolution for smaller values and lower resolution for larger values. When controlling a frequency value this is often appropriate (as musically, we hear equal changes in ratios between frequences rather than equal linear changes). This is configured by using the [Slider::setSkewFactorFromMidPoint()](https://docs.juce.com/master/classSlider.html#a2d948c662c1f636810d3125d8b3f0d47 "Sets up a skew factor to alter the way values are distributed.") function [4]. Our slider range is set to `50..5000` therefore setting the centre of the slider track to represent `500` would mean there is an equal musical interval between the slider minimum and the centre, and the centre and the slider maximum:

```cpp
MainContentComponent()
{
    addAndMakeVisible (frequencySlider);
    frequencySlider.setRange (50.0, 5000.0);
    frequencySlider.setSkewFactorFromMidPoint (500.0); // [4]
```

The _skew factor_ for the slider can be set directly using the [Slider::setSkewFactor()](https://docs.juce.com/master/classSlider.html#aba8fe3610e7198bd4c1f2804736bfbf2 "Sets up a skew factor to alter the way values are distributed.") function although it is often easier to think about what value you want at the mid-point.

> [!NOTE]
> Exercise: Add another slider to the application to control the level of the sine wave. Take care to keep the level well below `1.0` --- a maximum value of `0.25` should be fine.

# Smoothing frequency changes

You may notice --- especially in the higher frequencies --- that there are some audible, and probably unwanted, artifacts produced as the slider is moved. This is because the slider is actually changing in discrete steps, and when the slider is moved quickly then these steps are quite large. In addition to this the slider frequency is only updated for each audio block, therefore the precise effect of these changes will be dependent on the hardware block size.

## State members for smoothing

Let's add two members to our class, one to store the current frequency being used for synthesis, and another _target_ frequency that the user has requested by moving the slider. Then we can more slowly ramp between these values to remove the artifacts:

```cpp
double currentFrequency = 500.0, targetFrequency = 500.0; // [5]
```

We initialise these values at the same time [5]. We can also initialise the slider to the same value [6]:

```cpp
MainContentComponent()
{
    addAndMakeVisible (frequencySlider);
    frequencySlider.setRange (50.0, 5000.0);
    frequencySlider.setSkewFactorFromMidPoint (500.0);
    frequencySlider.setValue (currentFrequency, juce::dontSendNotification); // [6]
```

## Updating the synthesis code

The key to the way this algorithm works is to check whether the current and target values are the same or different. If they are the same, then we can simply use our original code as the `angleDelta` member doesn\'t need to change. If the current and target values are different, then we need to update the `angleDelta` member for each sample as we gradually move the current value closer to the target.

> [!WARNING]
> In this example we just use the number of samples in the output buffer as the duration of our ramp. This means that with very small buffer sizes you may still hear artifacts.

```cpp
void getNextAudioBlock (const juce::AudioSourceChannelInfo& bufferToFill) override
{
    auto level = 0.125f;
    auto* leftBuffer = bufferToFill.buffer->getWritePointer (0, bufferToFill.startSample);
    auto* rightBuffer = bufferToFill.buffer->getWritePointer (1, bufferToFill.startSample);

    auto localTargetFrequency = targetFrequency;

    if (!juce::approximatelyEqual (localTargetFrequency, currentFrequency)) // [7]
    {
        auto frequencyIncrement = (localTargetFrequency - currentFrequency) / bufferToFill.numSamples; // [8]

        for (auto sample = 0; sample < bufferToFill.numSamples; ++sample)
        {
            auto currentSample = (float) std::sin (currentAngle);
            currentFrequency += frequencyIncrement; // [9]
            updateAngleDelta(); // [10]
            currentAngle += angleDelta;
            leftBuffer[sample] = currentSample * level;
            rightBuffer[sample] = currentSample * level;
        }

        currentFrequency = localTargetFrequency;
    }
    else // [11]
    {
        for (auto sample = 0; sample < bufferToFill.numSamples; ++sample)
        {
            auto currentSample = (float) std::sin (currentAngle);
            currentAngle += angleDelta;
            leftBuffer[sample] = currentSample * level;
            rightBuffer[sample] = currentSample * level;
        }
    }
}
```

- [7]: Check if our target is different from the current value. Notice that we take a local copy of the target value, just in case the slider changes the value on the message thread while this function runs.
- [8]: Calculate the required increment per sample.
- [9]: Increment the current frequency.
- [10]: Update the `deltaAngle` member based on this new frequency.
- [11]: Otherwise just use the original code.

> [!TIP]
>The format of this code uses a typically pattern for [DSP](https://en.wikipedia.org/wiki/Digital_signal_processing) code. We avoid conditional statements in the inner `for()` loop, if possible. Instead, having the condition tested outside the loop, and we use two different, but quite similar loops depending upon whether the parameter is changing.

Finally, we need to update our [Slider::onValueChange](https://docs.juce.com/master/classSlider.html#a680d007f6a824a28a60aa05b4045e794 "You can assign a lambda to this callback object to have it called when the slider value is changed.") helper object so that it just updates the target value:

```cpp
frequencySlider.onValueChange = [this] { targetFrequency = frequencySlider.getValue(); };
```

And that's it! Try this now and the artifacts from the slider movement should have been removed.

> [!NOTE]
> Exercise: Add smoothing to the level slider control that you added in an earlier exercise.

# Notes

- Not wrapping the phase angle around `2pi` may not be ideal in all circumstances. If using `float` rather than `double` variables then there would be some innaccuracy to the calculations when the current angle value became very large. By not wrapping the phase at `2pi` using the `std::sin()` function performs reasonably well compared to a simple wavetable technnique. See [Tutorial: Wavetable synthesis](/tutorials/tutorial_wavetable_synth/) for an exploration of this.

# Summary

In this tutorial we have introduced some basic methods of synthesising and controlling a sine wave. We have looked at:

- The essential variables needed for maintaining the state of a sine wave oscillator.
- How to set these variable to produce the desired result.
- How to smooth out parameter changes to avoid audio artifacts.

# See also

- [Tutorial: The Slider class](/tutorials/tutorial_slider_values/)
- [Tutorial: Build a white noise generator](/tutorials/tutorial_simple_synth_noise/)
- [Tutorial: Control audio levels using decibels](/tutorials/tutorial_synth_db_level_control/)
- [Tutorial: Control audio levels](/tutorials/tutorial_synth_level_control/)
- [Tutorial: Build a MIDI synthesiser](/tutorials/tutorial_synth_using_midi_input/)
- [Tutorial: Wavetable synthesis](/tutorials/tutorial_wavetable_synth/)
