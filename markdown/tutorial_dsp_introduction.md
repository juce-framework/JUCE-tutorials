# Tutorial: Introduction to DSP

Discover the realm of digital signal processing and audio buffer manipulation. Learn the basics of the JUCE DSP module and how you can incorporate its classes in your own audio application and plugins.

**LEVEL:** Advanced<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**Plugin Format:** VST, AU, Standalone<br/>
**CLASSES:** [dsp::ProcessorChain](https://docs.juce.com/master/classdsp_1_1ProcessorChain.html "This variadically-templated class lets you join together any number of processor classes into a singl..."), [dsp::Gain](https://docs.juce.com/master/classdsp_1_1Gain.html "Applies a gain to audio samples as single samples or AudioBlocks."), [dsp::Oscillator](https://docs.juce.com/master/classdsp_1_1Oscillator.html "Generates a signal based on a user-supplied function."), [dsp::LadderFilter](https://docs.juce.com/master/classdsp_1_1LadderFilter.html "Multi-mode filter based on the Moog ladder filter."), [dsp::Reverb](https://docs.juce.com/master/classdsp_1_1Reverb.html "Processor wrapper around juce::Reverb for easy integration into ProcessorChain.")

> [!WARNING]
> This project requires a compiler that supports C++14 features. Recent versions of Xcode and Visual Studio include this support.

# Getting started

Before reading this tutorial, make sure you understand the basics of synthesis and have been introduced to MPE. If you would like to find more about MPE, check out this tutorial [Tutorial: Build a multi-polyphonic synthesiser](/tutorials/tutorial_mpe_introduction/).

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/DSPIntroductionTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/DSPIntroductionTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The project is conceived as a plugin but you can run it as a standalone application by selecting the proper deployment target in your IDE. In Xcode, you can change the target in the top left corner of the main window as shown in the following screenshot:

![](/_images/tutorial_dsp_introduction_standalone_screenshot1.png "Changing the deployment target in Xcode")

The demo project provides us with an on-screen MIDI keyboard in the top half of the plugin and a visual representation of the signal through an oscilloscope in the bottom half. Presently if a key is pressed, the plugin does not output any sound unless we provide an implementation of an oscillator.

![](/_images/tutorial_dsp_introduction_screenshot1.png "The demo project plugin window")

> [!TIP]
>If you have a MIDI controller, you can also choose to connect it instead of using the on-screen keyboard throughout this tutorial.

# What is DSP?

Digital signal processing involves manipulating digital data to perform certain operations on the signal. In digital audio processing, we can deal with audio data in different domains:

- Time domain: one-dimensional signals where analysis is performed with respect to time.
- Space domain: multi-dimensional signals where analysis is performed with respect to a certain space.
- Frequency domain: a specific domain representing time or space in terms of frequencies.

## The Fast Fourier Transform or FFT

A time or space domain signal can be converted to the frequency domain by using a transformation formula called the Fourier transform. A common efficient implementation of this transformation function is the Fast Fourier Transform or FFT, which you may encounter in the JUCE DSP module.

The FFT allows us to decompose an audio signal into its frequencies and represent the magnitude and phase information for each of these frequencies. Using its inverse function, we can revert the signal into its original domain thus making it really useful to process individual frequency components such as for filtering.

## Finite/Infinite Impulse Response or FIR/IIR

There are two main digital filter designs in dsp:

- Finite impulse response filters (FIR): a stable design that processes each output sample as a function of its previous input samples. FIR filters can be linear phase and are often simpler to design but less efficient than IIR filters.
- Infinite impulse response filters (IIR): a possibly unstable design that processes each output sample as a function of its previous input and output samples. IIR filters use previous output samples thus creating internal feedback and are harder to design but can be more efficient than FIR filters.

Within these filter designs, there are a number of different transfer functions that dictate the sharpness of the filter and the amount of ripples that occur at the transition frequency. Many of these designs are inspired by analog filters and different transfer functions try to emulate different analog counterparts.

Some of the transfer functions that you can find in the JUCE DSP module are:

- FIR transfer functions: Window, Kaiser, Transition, Least Squares, Half-Band Equiripple.
- IIR transfer functions: Butterworth, Chebyshev type 1, Chebyshev type 2, Elliptic, Half-Band Polyphase Allpass.

If you are interested in these filter designs, you can find plenty of resources online that go more in depth about this topic but for the purpose of this tutorial, we have covered more than the basics to get us started.

# The signal processing lifecycle

Similarly to the audio application lifecycle of the [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.") with its `prepareToPlay()` and `getNextAudioBlock()` functions, we have to implement the `prepare()` and `renderNextBlock()` functions of our `AudioEngine` class derived from [MPESynthesiser](https://docs.juce.com/master/classMPESynthesiser.html "Base class for an MPE-compatible musical device that can play sounds.").

Each dsp processor also has to implement the following methods to ensure proper functioning:

- `prepare()` : called before the processing starts to set sample rate and block size.
- `process()` : processes the input and output buffers supplied in the processing context.
- `reset()` : resets the internal state of the processor with smoothing if necessary.

## The processor chain

A convenient template class of the DSP module is the `juce::dsp::ProcessorChain` which allows us to apply different processes in series by calling the `prepare()` , `process()` and `reset()` methods automatically one after the other.

We declare processors as template types like so:

```cpp
juce::dsp::ProcessorChain<juce::dsp::Oscillator<Type>, juce::dsp::Gain<Type>> processorChain;
```

We can then apply all our processes on the `processorChain` instance directly.

With some basic knowledge on how the dsp module works, let's start processing some signals!

# Creating an oscillator

In the `CustomOscillator` class, define a juce::dsp::ProcessorChain with juce::dsp::Oscillator and juce::dsp::Gain processors in this top-down order [1]. We want the gain processing to affect the output of the oscillator to be able to trim the level coming out. Also define an enum with processor indices [2] to be able to clearly refer to the corresponding process via its index later on.

```cpp
enum {
    oscIndex,
    gainIndex // [2]
};

juce::dsp::ProcessorChain<juce::dsp::Oscillator<Type>, juce::dsp::Gain<Type>> processorChain; // [1]
};
```

In the `prepare()` function, call the prepare functions of each processor in the processor chain sequentially [3].

```cpp
void prepare (const juce::dsp::ProcessSpec& spec)
{
    processorChain.prepare (spec); // [3]
}
```

In the `reset()` function, call the reset functions of each processor in the processor chain sequentially [4].

```cpp
void reset() noexcept
{
    processorChain.reset(); // [4]
}
```

Now we want to define the periodic function that the oscillator will use to generate the audio signal. As a simple example we will start with a sine wave.

In the constructor, get a reference to the Oscillator by supplying the index of the process and use the `processorChain.get<>()` method [5]. Let's initialise the oscillator using a lambda function and the `std::sin` function to provide the sine wave to the oscillator [6].

A lookup table will approximate expensive arithmetic operation depending on the number of supplied discrete points. In our case let's use 128 points.

```cpp
public:
//==============================================================================
CustomOscillator()
{
    auto& osc = processorChain.template get<oscIndex>(); // [5]
    osc.initialise ([] (Type x) { return std::sin (x); }, 128); // [6]
}
```

To set the frequency of the oscillator, we need to once again get a reference to it similarly to the previous step and call the `setFrequency()` method on it [7].

```cpp
void setFrequency (Type newValue, bool force = false)
{
    auto& osc = processorChain.template get<oscIndex>();
    osc.setFrequency (newValue, force); // [7]
}
```

Same process with the gain processor and its `setGainLinear()` method [8].

```cpp
void setLevel (Type newValue)
{
    auto& gain = processorChain.template get<gainIndex>();
    gain.setGainLinear (newValue); // [8]
}
```

In the `process()` function, we can call the process functions of each processor in the processor chain sequentially [9].

```cpp
template <typename ProcessContext>
void process (const ProcessContext& context) noexcept
{
    processorChain.process (context); // [9]
}
```

If we run this code after implementing the above changes in the `CustomOscillator` class, we should be able to hear a simple sine wave synthesiser using the JUCE DSP module.

![](/_images/tutorial_dsp_introduction_screenshot2.png "Sine wave synthesiser with the JUCE DSP module")

# Changing the oscillator waveform

Let's make our synthesiser a little more exciting by changing its oscillator waveform to a sawtooth.

Since we do not have access to a `std` version of a sawtooth function, we need to implement a manual mapping of values using the `jmap` function. To do this, map the range between `-Pi .. Pi` to `-1 .. 1` providing a linear ramp from -1 to 1. Since a sawtooth only has 2 breakpoints, we need only supply 2 discrete points to the lookup table.

```cpp
public:
//==============================================================================
CustomOscillator()
{
    auto& osc = processorChain.template get<oscIndex>();
    osc.initialise ([] (Type x) {
        return juce::jmap (x,
            Type (-juce::MathConstants<double>::pi),
            Type (juce::MathConstants<double>::pi),
            Type (-1),
            Type (1));
    },
        2);
}
```

Running the program should give us a different more aggressive sound.

![](/_images/tutorial_dsp_introduction_screenshot3.png "Sawtooth synthesiser with the JUCE DSP module")

> [!NOTE]
> Exercise: Initialise the oscillator with a triangle or square waveform and listen to its sound. Can you implement a white noise oscillator?

# Adding a second oscillator

Most analog synthesisers will have multiple oscillators and a common trick to get a thicker sound is to add a second oscillator with a slightly detuned frequency. So let's try that by modifying the `Voice` class.

Add a second CustomOscillator template type to the processor chain [1] and add its corresponding index in the enum [2].

```cpp
private:
//==============================================================================
juce::HeapBlock<char> heapBlock;
juce::dsp::AudioBlock<float> tempBlock;

enum {
    osc1Index,
    osc2Index, // [2]
    masterGainIndex
};

juce::dsp::ProcessorChain<CustomOscillator<float>, CustomOscillator<float>, juce::dsp::Gain<float>> processorChain; // [1]
//...
};
```

Let's set the frequency of the second oscillator to the currently played note and pitch it up by 1% in the `noteStarted()` function [3]. We can keep the velocity at the same lavel as the first oscillator [4].

```cpp
void noteStarted() override
{
    auto velocity = getCurrentlyPlayingNote().noteOnVelocity.asUnsignedFloat();
    auto freqHz = (float) getCurrentlyPlayingNote().getFrequencyInHertz();

    processorChain.get<osc1Index>().setFrequency (freqHz, true);
    processorChain.get<osc1Index>().setLevel (velocity);

    processorChain.get<osc2Index>().setFrequency (freqHz * 1.01f, true); // [3]
    processorChain.get<osc2Index>().setLevel (velocity); // [4]
}
```

Let's make sure that the detuned frequency remains the same when pitch bend is applied in the `notePitchbendChanged()` function [5].

```cpp
void notePitchbendChanged() override
{
    auto freqHz = (float) getCurrentlyPlayingNote().getFrequencyInHertz();
    processorChain.get<osc1Index>().setFrequency (freqHz);
    processorChain.get<osc2Index>().setFrequency (freqHz * 1.01f); // [5]
}
```

Let's run the program and see how it sounds like.

![](/_images/tutorial_dsp_introduction_screenshot4.png "Synthesiser with a second sawtooth oscillator")

> [!NOTE]
> Exercise: Add a third oscillator with its frequency pitched down by 1%. Does the sound get thicker?

# Adding a ladder filter

Let's introduce a filter design to our synthesiser. The ladder filter processor is inspired by a well-known analog design from the Moog synthesiser and this is the one we will use in our project. By now you should be familiar with the task of adding processors to the processor chain.

Add the `juce::dsp::LadderFilter` to the processor chain [1] and add its corresponding index to the enum [2] in the `Voice` class.

```cpp
juce::HeapBlock<char> heapBlock;
juce::dsp::AudioBlock<float> tempBlock;

enum {
    osc1Index,
    osc2Index,
    filterIndex, // [2]
    masterGainIndex
};

juce::dsp::ProcessorChain<CustomOscillator<float>, CustomOscillator<float>, juce::dsp::LadderFilter<float>, juce::dsp::Gain<float>> processorChain; // [1]
```

As previously explained, get the reference of the filter processor and set its cutoff frequency at 1kHz [3] and resonance at 0.7 [4].

```cpp
Voice()
{
    auto& masterGain = processorChain.get<masterGainIndex>();
    masterGain.setGainLinear (0.7f);

    auto& filter = processorChain.get<filterIndex>();
    filter.setCutoffFrequencyHz (1000.0f); // [3]
    filter.setResonance (0.7f); // [4]
```

The signal should now have its higher frequencies attenuated with a more muffled sound.

![](/_images/tutorial_dsp_introduction_screenshot5.png "Synthesiser with a ladder filter")

> [!NOTE]
> Exercise: Try different resonance values and cutoff frequencies and listen to the output. At the moment, the filter is a low-pass filter with 12dB/octave attenuation. Can you make it a high-pass filter with 24dB/octave attenuation?

# Modulating the signal with an LFO

Now that we are getting closer to a classic analog synth sound, what more could there be? A modulating LFO of course.

A low frequency oscillator acts as a control signal to another parameter that we want to modulate. Its frequency is usually very low and is below the human hearing range therefore we should not add the oscillator in the processor chain like we did for the previous oscillators. This time, declare the new Oscillator as a regular member variable [1] in the `Voice` class.

```cpp
static constexpr size_t lfoUpdateRate = 100;
size_t lfoUpdateCounter = lfoUpdateRate;
juce::dsp::Oscillator<float> lfo; // [1]
```

To produce a slow and smooth modulation change to the cutoff frequency of the ladder filter, initialise the LFO as a sine wave [2] with a rate of 3Hz [3] in the `Voice` constructor.

```cpp
lfo.initialise ([] (float x) { return std::sin (x); }, 128);
lfo.setFrequency (3.0f);
}
```

Since we do not need to update the LFO as often as the audio processing sample rate, divide the sample rate by the LFO update rate to set the LFO sample rate in the `prepare()` function [4]. In this case we decide to update the LFO a hundred times less frequently.

```cpp
void prepare (const juce::dsp::ProcessSpec& spec)
{
    tempBlock = juce::dsp::AudioBlock<float> (heapBlock, spec.numChannels, spec.maximumBlockSize);
    processorChain.prepare (spec);

    lfo.prepare ({ spec.sampleRate / lfoUpdateRate, spec.maximumBlockSize, spec.numChannels }); // [4]
}
```

In the following `for()` loop, we only modify the cutoff frequency every 100 samples. First call the `processSample()` function to process a single sample on the LFO [5] and then map its return value to the desired modulation range [6]. In this case we want to modulate the cutoff frequency from 100Hz to 2kHz. Finally, apply the new cutoff frequency to the ladder filter [7].

```cpp
void renderNextBlock (juce::AudioBuffer<float>& outputBuffer, int startSample, int numSamples) override
{
    auto output = tempBlock.getSubBlock (0, (size_t) numSamples);
    output.clear();

    for (size_t pos = 0; pos < (size_t) numSamples;)
    {
        auto max = juce::jmin ((size_t) numSamples - pos, lfoUpdateCounter);
        auto block = output.getSubBlock (pos, max);

        juce::dsp::ProcessContextReplacing<float> context (block);
        processorChain.process (context);

        pos += max;
        lfoUpdateCounter -= max;

        if (lfoUpdateCounter == 0)
        {
            lfoUpdateCounter = lfoUpdateRate;
            auto lfoOut = lfo.processSample (0.0f); // [5]
            auto curoffFreqHz = juce::jmap (lfoOut, -1.0f, 1.0f, 100.0f, 2000.0f); // [6]
            processorChain.get<filterIndex>().setCutoffFrequencyHz (curoffFreqHz); // [7]
        }
    }

    juce::dsp::AudioBlock<float> (outputBuffer)
        .getSubBlock ((size_t) startSample, (size_t) numSamples)
        .add (tempBlock);
}
```

You should now hear a UFO-type siren sound.

![](/_images/tutorial_dsp_introduction_screenshot6.png "Synthesiser with an LFO")

> [!NOTE]
> Exercise: Try to modulate different parameters such as filter resonance or oscillator frequency.

# Adding a simple reverb

At the moment if we play our synthesiser, you may have noticed that the sound is very dry so let's add a simple reverb to add depth to our signal. In order to apply the reverb to the entire synth sound, create an effects chain in the `AudioEngine` class and add the `juce::dsp::Reverb` template type to the effects chain [1] along with its index [2].

```cpp
enum {
    reverbIndex // [2]
};

juce::dsp::ProcessorChain<juce::dsp::Reverb> fxChain; // [1]
};
```

Call the `prepare()` function on the processor chain [3].

```cpp
void prepare (const juce::dsp::ProcessSpec& spec) noexcept
{
    setCurrentPlaybackSampleRate (spec.sampleRate);

    for (auto* v : voices)
        dynamic_cast<Voice*> (v)->prepare (spec);

    fxChain.prepare (spec); // [3]
}
```

In order to process the effects chain, we need to retrieve the correct AudioBlock from the [AudioBuffer](https://docs.juce.com/master/classAudioBuffer.html "A multi-channel buffer containing floating point audio samples.") to pass the context to the processing chain. First, convert the [AudioBuffer](https://docs.juce.com/master/classAudioBuffer.html "A multi-channel buffer containing floating point audio samples.") to a usable AudioBlock [4] and refer to the right portion of the samples to manipulate using the `getSubBlock()` method [5]. Now we can get the processing context from this AudioBlock [6] and process the effects chain with it [7].

```cpp
void renderNextSubBlock (juce::AudioBuffer<float>& outputAudio, int startSample, int numSamples) override
{
    MPESynthesiser::renderNextSubBlock (outputAudio, startSample, numSamples);

    auto block = juce::dsp::AudioBlock<float> (outputAudio); // [4]
    auto blockToUse = block.getSubBlock ((size_t) startSample, (size_t) numSamples); // [5]
    auto contextToUse = juce::dsp::ProcessContextReplacing<float> (blockToUse); // [6]
    fxChain.process (contextToUse); // [7]
}
```

The synth should now have a smooth reverb tail added to the end of the signal.

![](/_images/tutorial_dsp_introduction_screenshot7.png "Synthesiser with reverb")

> [!TIP]
>The source code for this modified version of the code can be found in the `DSPIntroductionTutorial_02.h` file of the demo project.

# Summary

In this tutorial, we have learnt how to manipulate the audio buffer and process the signal using the JUCE DSP module. In particular, we have:

- Created a wavetable synth using multiple oscillators.
- Played with different waveforms including sawtooth and sine waves.
- Implemented a filter and manipulated its cutoff frequency with an LFO.
- Added a simple reverb to incorporate spaciousness into the signal.

> [!TIP]
>Check out part 2 of this tutorial to add distortion and convolution here: [Tutorial: Add distortion through waveshaping and convolution](/tutorials/tutorial_dsp_convolution/)

> [!TIP]
> Skip to part 3 of this tutorial to add delay lines here: [Tutorial: Create a string model with delay lines](/tutorials/tutorial_dsp_delay_line/)

# See also

- [Tutorial: Build a white noise generator](/tutorials/tutorial_simple_synth_noise/)
- [Tutorial: Build a sine wave synthesiser](/tutorials/tutorial_sine_synth/)
- [Tutorial: Control audio levels using decibels](/tutorials/tutorial_synth_db_level_control/)
- [Tutorial: Control audio levels](/tutorials/tutorial_synth_level_control/)
- [Tutorial: Build a MIDI synthesiser](/tutorials/tutorial_synth_using_midi_input/)
- [Tutorial: Build a multi-polyphonic synthesiser](/tutorials/tutorial_mpe_introduction/)
- [Tutorial: The fast Fourier transform](/tutorials/tutorial_simple_fft/)
- [Tutorial: Visualise the frequencies of a signal in real time](/tutorials/tutorial_spectrum_analyser/)
- [Tutorial: Optimisation using the SIMDRegister class](/tutorials/tutorial_simd_register_optimisation/)
- [Tutorial: Add distortion through waveshaping and convolution](/tutorials/tutorial_dsp_convolution/)
- [Tutorial: Create a string model with delay lines](/tutorials/tutorial_dsp_delay_line/)
