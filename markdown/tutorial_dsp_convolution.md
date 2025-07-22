---
title: Add distortion through waveshaping and convolution
---
# Tutorial: Add distortion through waveshaping and convolution

Add grit to a synthesiser sound by creating harmonic distortion through waveshaping. Learn the basics of convolution to retrieve the sonic characteristics contained in an impulse response.

**LEVEL:** Advanced<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**Plugin Format:** VST, AU, Standalone<br/>
**CLASSES:** [dsp::ProcessorChain](https://docs.juce.com/master/classdsp_1_1ProcessorChain.html "This variadically-templated class lets you join together any number of processor classes into a singl..."), [dsp::Gain](https://docs.juce.com/master/classdsp_1_1Gain.html "Applies a gain to audio samples as single samples or AudioBlocks."), [dsp::Oscillator](https://docs.juce.com/master/classdsp_1_1Oscillator.html "Generates a signal based on a user-supplied function."), [dsp::Convolution](https://docs.juce.com/master/classdsp_1_1Convolution.html "Performs stereo partitioned convolution of an input signal with an impulse response in the frequency ..."), [dsp::WaveShaper](structdsp_1_1WaveShaper.html "Applies waveshaping to audio samples as single samples or AudioBlocks."), [dsp::Reverb](https://docs.juce.com/master/classdsp_1_1Reverb.html "Processor wrapper around juce::Reverb for easy integration into ProcessorChain."), [dsp::ProcessorDuplicator](structdsp_1_1ProcessorDuplicator.html "Converts a mono processor class into a multi-channel version by duplicating it and applying multichan...")

> [!WARNING]
> This project requires a compiler that supports C++14 features. Recent versions of Xcode and Visual Studio include this support.

# Getting started

This tutorial leads on from [Tutorial: Introduction to DSP](/tutorials/tutorial_dsp_introduction/). If you haven\'t done so already, you should read that tutorial first.

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/DSPConvolutionTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/DSPConvolutionTutorial.zip) . Unzip the project and open the first header file in the Projucer.

> [!WARNING]
> If using the PIP version of this project, please make sure to copy the `Resources` folder into the generated Projucer project.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The project is conceived as a plugin but you can run it as a standalone application by selecting the proper deployment target in your IDE. In Xcode, you can change the target in the top left corner of the main window as shown in the following screenshot:

![](/_images/tutorial_dsp_introduction_standalone_screenshot1.png "Changing the deployment target in Xcode")

The demo project provides us with an on-screen MIDI keyboard in the top half of the plugin and a visual representation of the signal through an oscilloscope in the bottom half. Presently if a key is pressed, the plugin outputs a basic oscillator sound with some reverb added.

> [!TIP]
>Feel free to remove the reverb to clearly hear the changes in each steps of the tutorial by commenting out the reverb processor in the `AudioEngine` class.

![](/_images/tutorial_dsp_introduction_screenshot1.png "The demo project plugin window")

> [!TIP]
>If you have a MIDI controller, you can also choose to connect it instead of using the on-screen keyboard throughout this tutorial.

# Introduction

In this tutorial we are introducing two new DSP concepts that allow for signal processing in different ways: Waveshaping and Convolution.

Let's start by defining this DSP terminology.

## What is Waveshaping?

Waveshaping is the process in which a certain signal is shaped into a different one by means of a transfer function that is applied on the original signal. For instance a simple sine wave can be shaped into a different waveform by applying a mathematical function to it.

Waveshapers can be used to effectively create distortion by adding harmonic content to the original signal when applying a certain transfer function. As you may know, square and triangle waves for example are essentially sine waves with odd harmonics added to them and a sawtooth wave is a sine wave with odd and even harmonics combined.

Knowing this fact, one way of creating distortion is to bring the shape of a sine wave closer to that of a square wave. So how can we achieve that using a transfer function?

Consider for instance a simple sine wave `sin(x)` which could be plotted as follows:

![](/_images/tutorial_dsp_convolution_graph1.png "Sine wave")

By applying the signum transfer function onto the sine wave, a function that essentially outputs the sign of the inputted number, we end up with `sgn(sin(x))` which perfectly represents the square wave like so:

![](/_images/tutorial_dsp_convolution_graph2.png "Square wave with signum function")

This perfect waveform however presents problems as it creates a harsh distortion referred to as hard-clipping, due to the hard edges on the curve. This kind of waveform is also \"too perfect\" to be recreated in the analog domain and therefore does not sound like the square wave created by most analog synthesisers.

In order to create a more gentle kind of distortion called soft-clipping, we can use a hyperbolic tangent transfer function `tanh(sin(x))` which outputs a signal that looks almost like a sine wave but with a rounder curve as shown here:

![](/_images/tutorial_dsp_convolution_graph3.png "Square wave with tanh function")

Then, in order to reach a square-like shape, we can boost the signal into clipping before applying the transfer function `tanh(n\*sin(x))` , essentially truncating the top of the bell shape into a soft-edged square wave as seen below:

![](/_images/tutorial_dsp_convolution_graph4.png "Square wave with clipped tanh function")

As you can see, the possibilities of waveshaping can be endless as any type of transfer function can be applied to the incoming signal.

In JUCE, you can perform Waveshaping with the [dsp::WaveShaper](structdsp_1_1WaveShaper.html "Applies waveshaping to audio samples as single samples or AudioBlocks.") class included in the DSP module.

## What is Convolution?

Convolution consists in simulating the reverberation characteristics of a certain space by using a pre-recorded impulse response that describes the properties of the space in question. This process allows us to apply any type of acoustic profile to an incoming signal by convolving, essentially multiplying every sample of data against the impulse response samples to create the combined output.

Impulse responses are audio files that are generated by recording short impulses in the spaces to profile but they do not necessarily need to be actual physical spaces. For instance, we can capture the profile of a guitar amplifier by playing the impulse through the cabinet and recording its effect as shown here:

![](/_images/tutorial_dsp_convolution_graph5.png "A Guitar Amp impulse response")

The same was performed through a cassette recorder and the impulse response generated is the following:

![](/_images/tutorial_dsp_convolution_graph6.png "A Cassette Recorder impulse response")

When an incoming signal is convolved with the impulse response, the original dry signal will be transformed into a wet reverberated counterpart that retains the characteristics of both. For example, the output of a 100ms 440Hz sine wave convolved through the guitar amp impulse response shown above produces the following outcome:

![](/_images/tutorial_dsp_convolution_graph7.png "A sine wave through the Guitar Amp impulse response")

The same sine wave signal through the cassette recorder impulse response presented before ends up generating this waveform:

![](/_images/tutorial_dsp_convolution_graph8.png "A sine wave through the Cassette Recorder impulse response")

As you can see, the possibilities of convolution can be endless as any type of impulse response can be applied to the incoming signal.

In JUCE, you can perform Convolution with the [dsp::Convolution](https://docs.juce.com/master/classdsp_1_1Convolution.html "Performs stereo partitioned convolution of an input signal with an impulse response in the frequency ...") class included in the DSP module.

# Integrate the Waveshaper

In the `Distortion` class, add a juce::dsp::ProcessorChain with a juce::dsp::WaveShaper processor as a template argument [1]. Also define an enum with the processor index [2] to be able to clearly refer to the corresponding process via its index later on.

```cpp
private:
//==============================================================================
enum {
    waveshaperIndex // [2]
};

juce::dsp::ProcessorChain<juce::dsp::WaveShaper<Type>> processorChain; // [1]
};
```

In the `reset()` function, call the reset function of the waveshaper in the processor chain [3].

```cpp
void reset() noexcept
{
    processorChain.reset(); // [3]
}
```

In the `prepare()` function, call the prepare function of the waveshaper in the processor chain [4].

```cpp
void prepare (const juce::dsp::ProcessSpec& spec)
{
    processorChain.prepare (spec); // [4]
}
```

Now we want to define the transfer function that the waveshaper will use to shape the incoming signal. As described in the introduction section of this tutorial, let's start with a hard-clipping function first.

In the constructor, get a reference to the WaveShaper by supplying the index of the process and use the `processorChain.get<>()` method [5]. Let's initialise the waveshaper using a lambda function that limits the values to a range of `-0.1 .. 0.1` [6].

```cpp
public:
//==============================================================================
Distortion()
{
    auto& waveshaper = processorChain.template get<waveshaperIndex>(); // [5]
    waveshaper.functionToUse = [] (Type x) {
        return juce::jlimit (Type (-0.1), Type (0.1), x); // [6]
    };
}
```

In the `process()` function, we can call the process function of the waveshaper in the processor chain [7].

```cpp
template <typename ProcessContext>
void process (const ProcessContext& context) noexcept
{
    processorChain.process (context); // [7]
}
```

If we run this code after implementing the above changes in the `Distortion` class, we should be able to hear the effects of the waveshaper on the oscillator signal.

![](/_images/tutorial_dsp_convolution_screenshot1.png "Waveshaping the oscillator signal")

> [!NOTE]
> Exercise: Change the amount of limiting that occurs in the transfer function by restricting the values between `-0.5 .. 0.5` . Do you perceive a difference in sound?

# Change the transfer function

Let's make the clipping of our waveshaper a little softer by changing the transfer function to a hyperbolic tangent.

Add two `juce::dsp::Gain` processors to the processor chain with the `juce::dsp::WaveShaper` in between them [1] and add the corresponding indices to the enum as pre-gain [2] and post-gain [3]. These will allow us to adjust the level of the signal going into the waveshaper and control the level coming out of it, thus affecting the behaviour of the transfer function.

```cpp
private:
//==============================================================================
enum {
    preGainIndex, // [2]
    waveshaperIndex,
    postGainIndex // [3]
};

juce::dsp::ProcessorChain<juce::dsp::Gain<Type>, juce::dsp::WaveShaper<Type>, juce::dsp::Gain<Type>> processorChain; // [1]
};
```

In the constructor, change the transfer function to a hyperbolic tangent [4] as described in the introduction of this tutorial:

```cpp
public:
//==============================================================================
Distortion()
{
    auto& waveshaper = processorChain.template get<waveshaperIndex>();
    waveshaper.functionToUse = [] (Type x) {
        return std::tanh (x); // [4]
    };

    auto& preGain = processorChain.template get<preGainIndex>(); // [5]
    preGain.setGainDecibels (30.0f); // [6]

    auto& postGain = processorChain.template get<postGainIndex>(); // [7]
    postGain.setGainDecibels (-20.0f); // [8]
}
```

Here we also retrieve a reference to the pre-gain processor [5] and boost the signal going into the waveshaper by 30dB [6]. Then we get a reference to the post-gain processor [7] and trim down the level coming from the waveshaper by 20dB [8].

Running the program should give us a different distorted sound.

![](/_images/tutorial_dsp_convolution_screenshot2.png "Waveshaping with the hyperbolic tangent function")

> [!NOTE]
> Exercise: Change the transfer function used in the waveshaper to the signum function. Do you perceive a difference in sound?

# Adjust the filter

One thing you may have noticed is that when distorting low frequency content with the waveshaper, the distorted sound can get muddy very quickly. We can alleviate this problem by introducing a high-pass filter before processing the signal with the waveshaper.

Add a `juce::dsp::ProcessorDuplicator` with the `juce::dsp::IIR::Filter` and `juce::dsp::IIR::Coefficients` classes as template arguments [1] in order to easily convert the mono filter into a multi-channel version. To simplify the long class names, we use shorter names with the \"using\" keyword and add the corresponding index in the enum [2].

```cpp
private:
//==============================================================================
enum {
    filterIndex, // [2]
    preGainIndex,
    waveshaperIndex,
    postGainIndex
};

using Filter = juce::dsp::IIR::Filter<Type>;
using FilterCoefs = juce::dsp::IIR::Coefficients<Type>;

juce::dsp::ProcessorChain<juce::dsp::ProcessorDuplicator<Filter, FilterCoefs>,
    juce::dsp::Gain<Type>,
    juce::dsp::WaveShaper<Type>,
    juce::dsp::Gain<Type>>
    processorChain;
};
```

In the `prepare()` function, get a reference to the filter processor [3] and specify the cut-off frequency of the high-pass filter to 1kHz by calling the `makeFirstOrderHighPass()` function [4]:

```cpp
void prepare (const juce::dsp::ProcessSpec& spec)
{
    auto& filter = processorChain.template get<filterIndex>(); // [3]
    filter.state = FilterCoefs::makeFirstOrderHighPass (spec.sampleRate, 1000.0f); // [4]

    processorChain.prepare (spec);
}
```

The signal should now have its lower frequencies attenuated with a clearer sound.

![](/_images/tutorial_dsp_convolution_screenshot3.png "Filtering before the waveshaper")

> [!NOTE]
> Exercise: Experiment with different types of filters before waveshaping and notice how the harmonic content of the distorted signal changes.

# Implement the cabinet simulator

Let's give even more character to our sound by using convolution to simulate a guitar cabinet.

In the `CabSimulator` class, add a `juce::dsp::Convolution` processor to the processor chain [1] and add its corresponding index in the enum [2].

```cpp
private:
//==============================================================================
enum {
    convolutionIndex // [2]
};

juce::dsp::ProcessorChain<juce::dsp::Convolution> processorChain;
};
```

In the `reset()` function, call the reset function of the convolver in the processor chain [3].

```cpp
void reset() noexcept
{
    processorChain.reset(); // [3]
}
```

In the `prepare()` function, call the prepare function of the convolver in the processor chain [4].

```cpp
private:
//==============================================================================
enum {
    convolutionIndex // [2]
};

juce::dsp::ProcessorChain<juce::dsp::Convolution> processorChain;
};
```

Now we want to specify the impulse response that the convolver will use to reverberate the incoming signal. As presented in the introduction section of this tutorial, let's load the convolution processor with the guitar amp impulse response included in the `Resources` folder of the project.

In the constructor, get a reference to the Convolution processor by supplying the index of the process and use the `processorChain.get<>()` method [5]. Let's initialise the convolver with the guitar amp impulse response by loading the audio file from the `Resources` folder [6].

```cpp
CabSimulator()
{
    auto dir = juce::File::getCurrentWorkingDirectory();

    int numTries = 0;

    while (!dir.getChildFile ("Resources").exists() && numTries++ < 15)
        dir = dir.getParentDirectory();

    auto& convolution = processorChain.template get<convolutionIndex>(); // [5]

    convolution.loadImpulseResponse (dir.getChildFile ("Resources").getChildFile ("guitar_amp.wav"),
        juce::dsp::Convolution::Stereo::yes,
        juce::dsp::Convolution::Trim::no,
        1024); // [6]
}
```

> [!WARNING]
> Make sure that the \"guitar_amp.wav\" file exists in the `Resources` folder of your project.

In the `process()` function, we can call the process function of the convolver in the processor chain [7].

```cpp
template <typename ProcessContext>
void process (const ProcessContext& context) noexcept
{
    processorChain.process (context); // [7]
}
```

In the `Distortion` class, remove the gain trim or set its level to 0dB of attenuation [8] as the signal will get attenuated naturally through the convolution process which happens after the distortion in our signal chain:

```cpp
public:
//==============================================================================
Distortion()
{
    auto& waveshaper = processorChain.template get<waveshaperIndex>();
    waveshaper.functionToUse = [] (Type x) {
        return std::tanh (x);
    };

    auto& preGain = processorChain.template get<preGainIndex>();
    preGain.setGainDecibels (30.0f);

    auto& postGain = processorChain.template get<postGainIndex>();
    postGain.setGainDecibels (0.0f); // [8]
}
```

Let's run the program and see how it sounds like.

![](/_images/tutorial_dsp_convolution_screenshot4.png "Guitar amp simulation with convolution")

> [!NOTE]
> Exercise: Load the impulse response of the cassette recorder included in the `Resources` folder and notice how drastically the convolved sound changes.

<!-- -->

> [!TIP]
>The source code for this modified version of the code can be found in the `DSPConvolutionTutorial_02.h` file of the demo project.

# Summary

In this tutorial, we have learnt how to incorporate waveshaping and convolution. In particular, we have:

- Learnt the basics of waveshaping and convolution.
- Integrated a hard-clipping waveshaper to create distortion.
- Modified the waveshaper transfer curve into a hyperbolic tangent.
- Implemented a cabinet simulator with convolution techniques.

> [!NOTE]
> Jump back to part 1 of this tutorial to understand oscillators and filters here: [Tutorial: Introduction to DSP](/tutorials/tutorial_dsp_introduction/)

<!-- -->

> [!TIP]
> Check out part 3 of this tutorial to add delay lines here: [Tutorial: Create a string model with delay lines](/tutorials/tutorial_dsp_delay_line/)

# See also

- [Tutorial: The fast Fourier transform](/tutorials/tutorial_simple_fft/)
- [Tutorial: Visualise the frequencies of a signal in real time](/tutorials/tutorial_spectrum_analyser/)
- [Tutorial: Optimisation using the SIMDRegister class](/tutorials/tutorial_simd_register_optimisation/)
- [Tutorial: Introduction to DSP](/tutorials/tutorial_dsp_introduction/)
- [Tutorial: Create a string model with delay lines](/tutorials/tutorial_dsp_delay_line/)
