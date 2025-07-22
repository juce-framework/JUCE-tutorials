---
title: Processing audio input
---
# Tutorial: Processing audio input

This tutorial shows how to process audio input and pass it to the audio output.

**LEVEL:** Beginner<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**CLASSES:** [Random](https://docs.juce.com/master/classRandom.html "A random number generator."), [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class."), [AudioBuffer](https://docs.juce.com/master/classAudioBuffer.html "A multi-channel buffer containing floating point audio samples.")

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/ProcessingAudioInputTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/ProcessingAudioInputTutorial.zip) . Unzip the project and open the first header file in the Projucer.

> [!TIP]
>If your operating system requires you to request permission to access the microphone (currently iOS, Android and macOS Mojave) then you will need to set the corresponding option under the relevant exporter in the Projucer and resave the project.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project modulates an incoming signal with white noise. The level of the white noise can be changed which affects the level of the overall output (see [Tutorial: Control audio levels](/tutorials/tutorial_synth_level_control/) for the technique used to generate the white noise). The result is a very \"fuzzy\" version of the input signal.

> [!WARNING]
> Be careful to avoid feedback when running the application (although the effect can be quite interesting!).

It's probably best to use a separate microphone and headphones. Of course, you will need some kind of audio input device for the project to work correctly.

# Audio input

This tutorial uses the [AudioAppComponent](https://docs.juce.com/master/classAudioAppComponent.html "A base class for writing audio apps that stream from the audio i/o devices.") class as the basis for the demo project application. In other tutorials we generate audio within the `getNextAudioBlock()` function --- see [Tutorial: Build a white noise generator](/tutorials/tutorial_simple_synth_noise/), [Tutorial: Control audio levels](/tutorials/tutorial_synth_level_control/), and [Tutorial: Build a sine wave synthesiser](/tutorials/tutorial_sine_synth/). In this tutorial we read the audio input and output some audio too. In the `MainContentComponent` constructor we request two audio inputs and two audio outputs:

```cpp
setAudioChannels (2, 2);
```

> [!TIP]
>The actual number of available inputs or outputs may be fewer than the number we request.

## Reusing buffers

It is important to know that the input and output buffers are not completely separate. The same buffer is used for the input and output. You can test this by temporarily commenting out all of the code in the `getNextAudioBlock()` function. If you then run the application, the audio input will be passed directly to the output. In the `getNextAudioBlock()` function, the number of channels in the [AudioSampleBuffer](https://docs.juce.com/master/classAudioBuffer.html) object within the `bufferToFill` struct may be larger than the number input channels, the number of output channels, or both. It is important to access only the data that refers to the number of input and output channels that you have requested, _and_ that are available. In particular, if you have more input channels than output channels you _must not_ modify the channels that should contain read-only data.

## Getting active channels

In the `getNextAudioBlock()` function we obtain [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") objects that represent the list of active input and output channels as a _bitmask_ (this is similar to the [std::bitset](http://www.cplusplus.com/reference/bitset/bitset/) class or using a [std::vector\<bool\>](http://www.cplusplus.com/reference/vector/vector-bool/) object). In these [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") objects the channels are represented by a 0 (inactive) or 1 (active) in the bits comprising the [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") value.

> [!TIP]
>See [Tutorial: The BigInteger class](/tutorials/tutorial_big_integer/) for other operations that can be performed on [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") objects.

```cpp
void getNextAudioBlock (const juce::AudioSourceChannelInfo& bufferToFill) override
{
    auto* device = deviceManager.getCurrentAudioDevice();
    auto activeInputChannels = device->getActiveInputChannels();
    auto activeOutputChannels = device->getActiveOutputChannels();
```

To work out the maximum number of channels over which we need to iterate, we can inspect the bits in the [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") objects to find the highest numbered bit. The maximum number of channels will be one more than this.

```cpp
auto maxInputChannels = activeInputChannels.getHighestBit() + 1;
auto maxOutputChannels = activeOutputChannels.getHighestBit() + 1;
```

Then obtain the desired level from our level slider and move on to process the audio by processing each output channel, one at a time. If the maximum number of input channels is zero (which could happen even though we requested two channels if our hardware has no audio inputs) then we must not try to process the audio. In this case we simply zero the output channel buffer (to output silence). Individual output channels may also be inactive, so we check the state of the channel and also output silence for that channel if it is inactive:

```cpp
auto level = (float) levelSlider.getValue();

for (auto channel = 0; channel < maxOutputChannels; ++channel)
{
    if ((!activeOutputChannels[channel]) || maxInputChannels == 0)
    {
        bufferToFill.buffer->clear (channel, bufferToFill.startSample, bufferToFill.numSamples);
    }
```

Then we go on to process the input data through to the output:

```cpp
else
{
    auto actualInputChannel = channel % maxInputChannels; // [1]

    if (!activeInputChannels[channel]) // [2]
    {
        bufferToFill.buffer->clear (channel, bufferToFill.startSample, bufferToFill.numSamples);
    }
    else // [3]
    {
        auto* inBuffer = bufferToFill.buffer->getReadPointer (actualInputChannel,
            bufferToFill.startSample);
        auto* outBuffer = bufferToFill.buffer->getWritePointer (channel, bufferToFill.startSample);

        for (auto sample = 0; sample < bufferToFill.numSamples; ++sample)
        {
            auto noise = (random.nextFloat() * 2.0f) - 1.0f;
            outBuffer[sample] = inBuffer[sample] + (inBuffer[sample] * noise * level);
        }
    }
}
}
}
```

The code should be reasonably self-explanatory but here are a few highlights:

- [1]: We may have requested more output channels than input channels, so our app needs to make a decision about what to do about these extra outputs. In this example we simply repeat the input channels if we have more outputs than inputs. (To do this we use the modulo operator to \"wrap\" our access to the input channels based on the number of input channels available.) In other applications it may be more appropriate to output silence for higher numbered channels where there are more output channels than input channels.
- [2]: Individual input channels may be inactive so we output silence in this case too.
- [3]: This final block _actually_ does the processing! Here we get pointers to the input and output buffer samples and add some noise to the input samples.

> [!NOTE]
> Exercise: In this example we don\'t smooth the amplitude changes as we do in [Tutorial: Build a sine wave synthesiser](/tutorials/tutorial_sine_synth/). This is partly to keep the example simple, but you probably wouldn\'t hear any additional glitches due to the crackling effect anyway. Modify the code to simply output the input channels, scaled by the level slider value, but also smooth out the level changes so that there are no glitches.

# Summary

In this tutorial we have introduced processing audio from an audio input in a JUCE application. In particular, you should now know:

- How to set up an audio application to be able to access audio from the computer's audio input hardware.
- That input and output buffers are shared.
- How to deal with active and inactive channels.
- How to reason about what to do where there are different numbers of input and output channels.

# See also

- [Tutorial: The AudioDeviceManager class](/tutorials/tutorial_audio_device_manager/)
- [Tutorial: Build a white noise generator](/tutorials/tutorial_simple_synth_noise/)
- [Tutorial: Control audio levels](/tutorials/tutorial_synth_level_control/)
- [Tutorial: Build a sine wave synthesiser](/tutorials/tutorial_sine_synth/)
