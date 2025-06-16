# Tutorial: Optimisation using the SIMDRegister class

Take advantage of the processor's parallelism to perform single instruction multiple data calculations. Optimise your audio applications without introducing concurrency.

**LEVEL:** Advanced<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**CLASSES:** [dsp::SIMDRegister](structdsp_1_1SIMDRegister.html "A wrapper around the platform's native SIMD register type."), [dsp::IIR](namespacedsp_1_1IIR.html "Classes for IIR filter processing."), [dsp::ProcessorDuplicator](structdsp_1_1ProcessorDuplicator.html "Converts a mono processor class into a multi-channel version by duplicating it and applying multichan..."), AudioDataConverters, [dsp::AudioBlock](https://docs.juce.com/master/classdsp_1_1AudioBlock.html "Minimal and lightweight data-structure which contains a list of pointers to channels containing some ..."), [HeapBlock](https://docs.juce.com/master/classHeapBlock.html "Very simple container class to hold a pointer to some data on the heap.")

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/SIMDRegisterTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/SIMDRegisterTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project can play a loaded audio file through an IIR filter in order to be processed and altered when auditioned. The purpose of this optimisation is to see how much CPU power we can alleviate using SIMD instruction sets on the same IIR filter.

![](/_images/tutorial_simd_register_optimisation_screenshot1.png "The demo project window")

> [!TIP]
>The code presented here is broadly similar to the **SIMDRegisterDemo** from the DSP Demo.

# SIMD Instructions

SIMD stands for \"Single Instruction Multiple Data\" and refers to the way modern CPUs can apply a single instruction to a set of data by loading numbers into multiple registers and performing the same calculation all at once. In the world of digital signal processing, this type of parallelism is favoured over other types such as MIMD (Multiple Instruction Multiple Data) because concurrency becomes an issue on an audio level. Making sure that the audio thread is not fighting over its data with other threads is paramount and the order of instructions should be kept in the same order in most cases when processing audio.

SIMD operates on vectors of data streams instead of individual data which makes it even more suitable for audio processing as we are used to receiving blocks of data from the audio buffer. SIMD also thrives when we need to apply the same scalar operation over multiple data points which is something that is very common in DSP algorithms.

The process of optimising general code is usually done by the compiler automatically nowadays but the vectorisation of DSP algorithms is not always trivial. Compilers are not always able to understand humanly what the algorithm is trying to do in order to optimise correctly. Therefore, this task is usually performed manually and the SIMDRegister class is a handy tool to do this in JUCE.

The SIMDRegister class is convenient because it handles different processor types for you. Depending on the CPU, the size and number of registers can vary and it can quickly become difficult to account for all CPU vendors. This is all handled by the SIMDRegister class and all we need to do is to specify which sets of instructions we want to vectorise in our algorithms.

Using the SIMDRegister class is relatively straightforward and it essentially acts as a drop-in replacement for primitive types. Let's take a look at a simple example code such as this one:

```cpp
float calculateDSPEffect (float x,
    float y)
{
    auto z = x + (y * 2.0f);

    return z;
}
```

This can be easily vectorised by simply wrapping the primitive types with the SIMDRegister class:

```cpp
SIMDRegister<float> calculateDSPEffect (SIMDRegister<float> x,
    SIMDRegister<float> y)
{
    auto z = x + (y * 2.0f);

    return z;
}
```

In DSP code, conditional statements are very slow and branching should be generally avoided as much as possible. Therefore the following example is a good candidate for SIMD optimisation:

```cpp
float calculateDSPEffect (float x,
    float y)
{
    auto z = (x > y ? x + (y * 2.0f) : y);

    return z;
}
```

Fortunately, the SIMDRegister class provides us with bit masks that allow us to select the correct result as follows:

```cpp
SIMDRegister<float> calculateDSPEffect (SIMDRegister<float> x,
    SIMDRegister<float> y)
{
    auto mask = SIMDRegister<float>::greaterThan (x, y);
    auto z = ((x + (y * 2.0f)) & mask) + (y & (~mask));

    return z;
}
```

For the purpose of this tutorial we will optimise an IIR filter using SIMD, so let's start by taking a look at the IIR filter implementation.

# The IIR Filter

In the `SIMDTutorialFilter` class, we first define member variables such as parameters for our filter as shown here:

```cpp
dsp::ProcessorDuplicator<dsp::IIR::Filter<float>, dsp::IIR::Coefficients<float>> iir;

ChoiceParameter typeParam { { "Low-pass", "High-pass", "Band-pass" }, 1, "Type" };
SliderParameter cutoffParam { { 20.0, 20000.0 }, 0.5, 440.0f, "Cutoff", "Hz" };
SliderParameter qParam { { 0.3, 20.0 }, 0.5, 0.7, "Q" };

std::vector<DSPParameterBase*> parameters { &typeParam, &cutoffParam, &qParam };
double sampleRate = 0.0;
};
```

Defining the IIR filter object within a ProcessorDuplicator allows us to convert our mono processor into a multi-channel one automatically by not worrying about calling the prepare(), process() and reset() functions on each channels individually. We also define the parameters of the filter such as the type of pass filter, the cutoff frequency and the sharpness Q of the filter.

In the updateParameters() function, we make sure that the parameters of the filter are updated when the on-screen controls are modified:

```cpp
void updateParameters()
{
    if (sampleRate != 0.0)
    {
        auto cutoff = static_cast<float> (cutoffParam.getCurrentValue());
        auto qVal = static_cast<float> (qParam.getCurrentValue());

        switch (typeParam.getCurrentSelectedID())
        {
            case 1:
                *iir.state = *dsp::IIR::Coefficients<float>::makeLowPass (sampleRate, cutoff, qVal);
                break;
            case 2:
                *iir.state = *dsp::IIR::Coefficients<float>::makeHighPass (sampleRate, cutoff, qVal);
                break;
            case 3:
                *iir.state = *dsp::IIR::Coefficients<float>::makeBandPass (sampleRate, cutoff, qVal);
                break;
            default:
                break;
        }
    }
}
```

Every time a parameter is modified, we create a new state for the IIR filter with a new set of coefficients depending on the sample rate, cutoff frequency and Q. The DSP module provides us with handy coefficients for our three filter types by using the makeLowPass(), makeHighPass() and makeBandPass() functions respectively.

In the prepare() function, we set the sample rate from the ProcessSpec object, set the IIR filter coefficients for the default case of a low pass filter and prepare the filter using the prepare() function with information on the processing context:

```cpp
void prepare (const dsp::ProcessSpec& spec)
{
    sampleRate = spec.sampleRate;

    iir.state = dsp::IIR::Coefficients<float>::makeLowPass (sampleRate, 440.0);
    iir.prepare (spec);
}
```

Processing the audio file with the filter is trivial where, in the process() function we call the process() function on the filter with a context where a single block is used for both the input and output:

```cpp
void process (const dsp::ProcessContextReplacing<float>& context)
{
    iir.process (context);
}
```

Finally, we reset the filter by calling reset on the filter in the reset() function:

```cpp
void reset()
{
    iir.reset();
}
```

Let's start optimising this IIR filter now.

# The SIMD-Optimised IIR Filter

Before optimising the code of our IIR Filter, we need to ensure that SIMD is available on our system. Use the `JUCE_USE_SIMD` macro to check whether you are developing on a SIMD machine by wrapping the whole filter implementation like so:

```cpp
#if JUCE_USE_SIMD

//==============================================================================
template <typename T>
static T* toBasePointer (dsp::SIMDRegister<T>* r) noexcept
{
    return reinterpret_cast<T*> (r);
}

constexpr auto registerSize = dsp::SIMDRegister<float>::size();

struct SIMDTutorialFilter
{
```

```cpp
};

#endif
```

Let's first define member variables for the IIR filter as well as AudioBlock and [HeapBlock](https://docs.juce.com/master/classHeapBlock.html "Very simple container class to hold a pointer to some data on the heap.") objects to facilitate the processing at the bottom of our `SIMDTutorialFilter` class:

```cpp
dsp::IIR::Coefficients<float>::Ptr iirCoefficients; // [1]
std::unique_ptr<dsp::IIR::Filter<dsp::SIMDRegister<float>>> iir;

dsp::AudioBlock<dsp::SIMDRegister<float>> interleaved; // [2]
dsp::AudioBlock<float> zero;

juce::HeapBlock<char> interleavedBlockData, zeroData; // [3]

ChoiceParameter typeParam { { "Low-pass", "High-pass", "Band-pass" }, 1, "Type" };
SliderParameter cutoffParam { { 20.0, 20000.0 }, 0.5, 440.0f, "Cutoff", "Hz" };
SliderParameter qParam { { 0.3, 20.0 }, 0.5, 0.7, "Q" };

std::vector<DSPParameterBase*> parameters { &typeParam, &cutoffParam, &qParam };
double sampleRate = 0.0;
```

Define the IIR coefficients as a pointer and the filter as a unique pointer using the SIMDRegister class to wrap the sample type [1]. Create an AudioBlock to store interleaved data using the SIMDRegister class to wrap the sample type and another AudioBlock for zero data used later to store the output block [2]. Allocate [HeapBlock](https://docs.juce.com/master/classHeapBlock.html "Very simple container class to hold a pointer to some data on the heap.") objects to hold the corresponding AudioBlock objects and some channel pointers with the size of the number of elements in a SIMDRegister vector [3].

In the prepare() function, set the sample rate as before and calculate the default coefficients for the filter [4]. Reset the filter by instantiating a new IIR filter with a SIMDRegister wrapper around the sample type and the coefficients defined earlier [5] as follows:

```cpp
void prepare (const dsp::ProcessSpec& spec)
{
    sampleRate = spec.sampleRate; // [4]

    iirCoefficients = dsp::IIR::Coefficients<float>::makeLowPass (sampleRate, 440.0f);
    iir.reset (new dsp::IIR::Filter<dsp::SIMDRegister<float>> (iirCoefficients)); // [5]

    interleaved = dsp::AudioBlock<dsp::SIMDRegister<float>> (interleavedBlockData, 1, spec.maximumBlockSize);
    zero = dsp::AudioBlock<float> (zeroData, dsp::SIMDRegister<float>::size(), spec.maximumBlockSize); // [6]

    zero.clear();

    auto monoSpec = spec;
    monoSpec.numChannels = 1;
    iir->prepare (monoSpec); // [7]
}
```

Create the AudioBlock objects for the interleaved data and the zero data by allocating the corresponding [HeapBlock](https://docs.juce.com/master/classHeapBlock.html "Very simple container class to hold a pointer to some data on the heap.") objects defined earlier [6]. The interleaved data block only need one channel and the maximum block size is retrieved from the context information. The zero data block takes the size of the SIMDRegister vector and is cleared before processing. The filter is prepared by reducing the number of channels to mono on the present context information [7] as the multi-channel samples will be interleaved later and processed as one channel.

Finally, in the process() function we will interleave the samples for optimised processing like follows:

```cpp
void process (const dsp::ProcessContextReplacing<float>& context)
{
    jassert (context.getInputBlock().getNumSamples() == context.getOutputBlock().getNumSamples());
    jassert (context.getInputBlock().getNumChannels() == context.getOutputBlock().getNumChannels());

    const auto& input = context.getInputBlock(); // [9]
    const auto numSamples = (int) input.getNumSamples();

    auto inChannels = prepareChannelPointers (input); // [10]

    using Format = juce::AudioData::Format<juce::AudioData::Float32, juce::AudioData::NativeEndian>;

    juce::AudioData::interleaveSamples (juce::AudioData::NonInterleavedSource<Format> {
                                            inChannels.data(),
                                            registerSize,
                                        },
        juce::AudioData::InterleavedDest<Format> { toBasePointer (interleaved.getChannelPointer (0)), registerSize },
        numSamples); // [11]

    iir->process (dsp::ProcessContextReplacing<dsp::SIMDRegister<float>> (interleaved)); // [12]

    auto outChannels = prepareChannelPointers (context.getOutputBlock()); // [13]

    juce::AudioData::deinterleaveSamples (juce::AudioData::InterleavedSource<Format> { toBasePointer (interleaved.getChannelPointer (0)), registerSize },
        juce::AudioData::NonInterleavedDest<Format> { outChannels.data(), registerSize },
        numSamples); // [14]
}
```

- [8]: First, make sure that the number of samples and the number of channels is the same for the input and output blocks.
- [9]: Next, retrieve the input block and the number of samples to process.
- [10]: For every channel in a SIMDRegister, check whether the channel is an input channel and copy the channel pointer into the corresponding [HeapBlock](https://docs.juce.com/master/classHeapBlock.html "Very simple container class to hold a pointer to some data on the heap."). Otherwise, it means that it is an output channel and we copy the zero data channel pointer.
- [11]: Now we interleave all the samples for the different channels by copying from the channel pointers [HeapBlock](https://docs.juce.com/master/classHeapBlock.html "Very simple container class to hold a pointer to some data on the heap.") into the interleaved AudioBlock and specifying the number of samples and the number of channels as the SIMDRegister size.
- [12]: Process the audio with the filter using the interleaved data in a single block context with a SIMDRegister wrapper on the sample type.
- [13]: Then, for every input channel, copy the output block channel pointer into the corresponding [HeapBlock](https://docs.juce.com/master/classHeapBlock.html "Very simple container class to hold a pointer to some data on the heap.").
- [14]: Finally, we deinterleave all the samples for the different channels by copying from the interleaved AudioBlock into the channel pointers [HeapBlock](https://docs.juce.com/master/classHeapBlock.html "Very simple container class to hold a pointer to some data on the heap.") and specifying the number of samples and the number of channels as the SIMDRegister size.

The reset() function of the filter remains the same in both cases and the optimisation is complete.

We just have to update the updateParameters() function to account for the new coefficients pointer as follows:

```cpp
void updateParameters()
{
    if (sampleRate != 0.0)
    {
        auto cutoff = static_cast<float> (cutoffParam.getCurrentValue());
        auto qVal = static_cast<float> (qParam.getCurrentValue());

        switch (typeParam.getCurrentSelectedID())
        {
            case 1:
                *iirCoefficients = *dsp::IIR::Coefficients<float>::makeLowPass (sampleRate, cutoff, qVal);
                break;
            case 2:
                *iirCoefficients = *dsp::IIR::Coefficients<float>::makeHighPass (sampleRate, cutoff, qVal);
                break;
            case 3:
                *iirCoefficients = *dsp::IIR::Coefficients<float>::makeBandPass (sampleRate, cutoff, qVal);
                break;
            default:
                break;
        }
    }
}
```

> [!TIP]
>The source code for this modified version of the code can be found in the `SIMDRegisterTutorial_02.h` file of the demo project.

# Summary

In this tutorial, we have learnt how to optimise DSP code using the SIMDRegister class. In particular, we have:

- Learnt the advantages of SIMD instructions.
- Processed a sound file through an IIR filter.
- Optimised the IIR filter using the SIMDRegister class.

# See also

- [Tutorial: Draw audio waveforms](/tutorials/tutorial_audio_thumbnail/)
- [Tutorial: Build an audio player](/tutorials/tutorial_playing_sound_files/)
- [Tutorial: Introduction to DSP](/tutorials/tutorial_dsp_introduction/)
- [Tutorial: Looping audio using the AudioSampleBuffer class](/tutorials/tutorial_looping_audio_sample_buffer/)
- [Tutorial: Looping audio using the AudioSampleBuffer class (advanced)](/tutorials/tutorial_looping_audio_sample_buffer_advanced/)
