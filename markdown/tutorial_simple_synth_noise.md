# Tutorial: Build a white noise generator

This tutorial introduces simple synthesis and audio output. This is key to developing an understanding of audio application (and plug-in) concepts in JUCE.

**LEVEL:** Intermediate<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**CLASSES:** [AudioAppComponent](https://docs.juce.com/master/classAudioAppComponent.html "A base class for writing audio apps that stream from the audio i/o devices."), [AudioSourceChannelInfo](structAudioSourceChannelInfo.html "Used by AudioSource::getNextAudioBlock()."), [AudioBuffer](https://docs.juce.com/master/classAudioBuffer.html "A multi-channel buffer containing floating point audio samples."), [Random](https://docs.juce.com/master/classRandom.html "A random number generator.")

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/SimpleSynthNoiseTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/SimpleSynthNoiseTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

This tutorial assumes that you are familiar with the basic principles of digital audio. In particular, you should know how audio signals are represented using sampling. You should also be familiar with the idea of the _sample rate_ in this context (which you may know as _sampling rate_, _sampling frequency_, or other similar terms).

# The demo project

The demo project was created using [The Projucer](https://juce.com/projucer) **Audio Application** template.

![](/_images/tutorial_simple_synth_noise_screenshot1.png "The Audio Application template shown highlighted within The Projucer.")

This is a useful starting point for your own audio applications in JUCE. The demo project synthesises white noise and plays it out of the target device's default audio hardware.

# The Audio Application template

This tutorial implements only audio output. Audio input and real-time audio processing of audio input data are explored in other tutorials. The Audio Application template is very similar to the GUI Application template, except that:

- The `MainContentComponent` class inherits from the [AudioAppComponent](https://docs.juce.com/master/classAudioAppComponent.html "A base class for writing audio apps that stream from the audio i/o devices.") class rather than the [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") class.
- The `juce_audio_utils` module is added to the project, in addition to the other audio-related modules that are added to projects by default.

The Audio Application template may be used for simple applications, such as the example provided here. It is also scalable for more complex applications, essentially any application that needs to interact directly with the target device's audio hardware. Creating audio plug-ins with JUCE is explored in other tutorials.

## The audio application lifecycle

The [AudioAppComponent](https://docs.juce.com/master/classAudioAppComponent.html "A base class for writing audio apps that stream from the audio i/o devices.") class is an abstract base class, it has three [pure virtual](http://www.learncpp.com/cpp-tutorial/126-pure-virtual-functions-abstract-base-classes-and-interface-classes/) functions that represent the lifecycle of our audio application that we _must_ implement in our derived class:

- [AudioAppComponent::prepareToPlay()](https://docs.juce.com/master/classAudioAppComponent.html#a70634aa3ffaf6e7ff0d233e5933a063d "Tells the source to prepare for playing."): This is called just before audio processing starts.
- [AudioAppComponent::releaseResources()](https://docs.juce.com/master/classAudioAppComponent.html#a588d9b990fc308774215548959e0fc50 "Allows the source to release anything it no longer needs after playback has stopped."): This is called when audio processing has finished.
- [AudioAppComponent::getNextAudioBlock()](https://docs.juce.com/master/classAudioAppComponent.html#aec5e36e694a5bd2edae903d1079166b1 "Called repeatedly to fetch subsequent blocks of audio data."): This is called each time the audio hardware needs a new block of audio data.

Of these three, the most important is perhaps [AudioAppComponent::getNextAudioBlock()](https://docs.juce.com/master/classAudioAppComponent.html#aec5e36e694a5bd2edae903d1079166b1 "Called repeatedly to fetch subsequent blocks of audio data."), since this is where you will either generate or process audio in your JUCE audio application. To understand how this works, we need to know a little about how modern computers generate audio. The audio hardware needs to generate a certain number of samples per channel for each second of audio. The CD-quality sample rate is 44.1kHz, which means there needs to be 44100 samples per second per channel sent to the audio hardware for playback. Rather than being passed to the audio hardware a single sample at a time, the samples are passed in buffers --- or blocks --- containing a certain number of samples. For example, at 44.1kHz and a block size of 441 our [AudioAppComponent::getNextAudioBlock()](https://docs.juce.com/master/classAudioAppComponent.html#aec5e36e694a5bd2edae903d1079166b1 "Called repeatedly to fetch subsequent blocks of audio data.") function would be called 100 times per second.

> [!TIP]
>The buffer size of 441 samples, above, is used to keep the arithmetic simple for the purposes of explanation. In practice, a buffer size of 441 would be unusual. Hardware buffer sizes would almost certainly be a even number and would tend to be powers-of-two (256, 512, 1024, and so on). It is important that your application is prepared to deal with *any* buffer size. For more information on the settings for sample rate and buffer sizes, see [Tutorial: The AudioDeviceManager class](/tutorials/tutorial_audio_device_manager/).

Essentially, our [AudioAppComponent::getNextAudioBlock()](https://docs.juce.com/master/classAudioAppComponent.html#aec5e36e694a5bd2edae903d1079166b1 "Called repeatedly to fetch subsequent blocks of audio data.") is servicing the _audio callback_ of the audio hardware. It is important to note that this function will be called from another thread (the _audio thread_ in most cases).

For our JUCE audio application to work correctly, there are two more important functions. This time we need to call them, rather than implement them:

- [AudioAppComponent::setAudioChannels()](https://docs.juce.com/master/classAudioAppComponent.html#a1cb6b457848fa80df7efc39cf095d465 "A subclass should call this from their constructor, to set up the audio."): We must call this to register the number of input and output channels we need. Typically, we do this in our constructor. In turn, this function triggers the start of audio processing in our application.
- [AudioAppComponent::shutdownAudio()](https://docs.juce.com/master/classAudioAppComponent.html#a40f1e657f048d02c5d065de566a84d92 "Shuts down the audio device and clears the audio source."): We must call this to shutdown the audio system. Typically, we do this in our destructor.

## Initialising the audio application

Let's now explore our simple implementation of a noise generator by examining the life cycle of an audio application in more detail. Our constructor needs to set up the size of the [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") object (see [Tutorial: The main component](/tutorials/tutorial_main_component/)). We also need to initialise at least one audio output:

```cpp
MainContentComponent()
{
    setSize (800, 600);
    setAudioChannels (0, 2); // no inputs, two outputs
}
```

As mentioned above, the call to the [AudioAppComponent::setAudioChannels()](https://docs.juce.com/master/classAudioAppComponent.html#a1cb6b457848fa80df7efc39cf095d465 "A subclass should call this from their constructor, to set up the audio.") function triggers the audio system to start up. In particular, it will call the `prepareToPlay()` function:

```cpp
void prepareToPlay (int samplesPerBlockExpected, double sampleRate) override
{
    juce::String message;
    message << "Preparing to play audio...\\n";
    message << " samplesPerBlockExpected = " << samplesPerBlockExpected << "\\n";
    message << " sampleRate = " << sampleRate;
    juce::Logger::getCurrentLogger()->writeToLog (message);
}
```

In this case we don\'t really need to do anything here, but as it is a [pure virtual](http://www.learncpp.com/cpp-tutorial/126-pure-virtual-functions-abstract-base-classes-and-interface-classes/) function we _must_ implement at least an empty function. Here we log some useful information that we can gain about the audio system on the target device at this point. The `samplesPerBlockExpected` argument, as its name suggests, is the size (in samples) of the buffers of audio that we can expect to be asked for each time a buffer of audio is requested in our `getNextAudioBlock()` function. This buffer size might vary between callbacks, but it is a good indication. The `sampleRate` argument tells us the current sample rate of the hardware. We would need this if we were doing something that is frequency-dependent, such as synthesising tones (see [Tutorial: Build a sine wave synthesiser](/tutorials/tutorial_sine_synth/)) or using equalisation. We would also need to know the sample rate if we were using delay effects. We don\'t need this information to generate noise.

## Generating audio data

Soon after this call to our `prepareToPlay()` function the audio thread will begin requesting blocks of audio via the [AudioAppComponent::getNextAudioBlock()](https://docs.juce.com/master/classAudioAppComponent.html#aec5e36e694a5bd2edae903d1079166b1 "Called repeatedly to fetch subsequent blocks of audio data.") function. This function is passed a single `bufferToFill` argument that is an [AudioSourceChannelInfo](structAudioSourceChannelInfo.html "Used by AudioSource::getNextAudioBlock().") struct. The [AudioSourceChannelInfo](structAudioSourceChannelInfo.html "Used by AudioSource::getNextAudioBlock().") struct contains a multichannel buffer of audio samples. It also contains two integer values that specify which region of this buffer should be processed on this call. In more detail, the [AudioSourceChannelInfo](structAudioSourceChannelInfo.html "Used by AudioSource::getNextAudioBlock().") contains the following members:

- `AudioSampleBuffer* buffer` : An AudioSampleBuffer object is a multichannel buffer of audio data that is essentially a multidimensional array of `float` values. When our `getNextAudioBlock()` function is called, this buffer contains any audio data from the target device's audio input (if we requested audio input). When our `getNextAudioBlock()` function returns, we must have filled the relevant section of the buffer with audio that we want to play.
- `int startSample` : This is the sample index within the buffer where our `getNextAudioBlock()` function should start reading or writing audio.
- `int numSamples` : This is the number of samples in the buffer that should be read or written.

Audio data stored as floating point values is very straightforward. Each sample of the audio signal is stored as a value that is nominally in the range ±1.0.

![](/_images/tutorial_simple_synth_noise_graph1.png "A full scale ±1.0 sine wave.")

At a peak level of ±1.0 like this the output level will be _very_ loud. In fact, this is the loudest the audio system will be able to generate without clipping. Typically, we will need to output audio that doesn\'t exceed this ±1.0 limit (although it is fine for intermediate stages of processing to go beyond this limit, as long as the final output is lower).

## The AudioSampleBuffer class

While the AudioSampleBuffer class is (at a very basic level) just a multichannel array of `float` values, it provides a useful set of functions for dealing with audio data. Many of these functions will be covered in later tutorials, but here we make use of the following functions:

- [AudioSampleBuffer::getNumChannels()](https://docs.juce.com/master/classAudioBuffer.html#a3a9ecde91bf5b96871ce3a474c1d831f): This returns the number of audio channels stored in the buffer. In this case the value should match the number of output channels we requested in our call to the [AudioAppComponent::setAudioChannels()](https://docs.juce.com/master/classAudioAppComponent.html#a1cb6b457848fa80df7efc39cf095d465 "A subclass should call this from their constructor, to set up the audio.") function earlier. (This value will always be the maximum of the number of input and output channels.)
- [AudioSampleBuffer::getWritePointer()](https://docs.juce.com/master/classAudioBuffer.html#a10cfe91eb4965895bc258cee02409d3b): This returns a pointer to the buffer of `float` values at a specific sample offset.

For our simple application to generate white noise, we need to fill the requested section of the buffer with random values. To do this we can iterate over the channels in the buffer, find the start sample within the buffer for that channel, and write the desired number of samples to the buffer:

```cpp
void getNextAudioBlock (const juce::AudioSourceChannelInfo& bufferToFill) override
{
    for (auto channel = 0; channel < bufferToFill.buffer->getNumChannels(); ++channel)
    {
        // Get a pointer to the start sample in the buffer for this audio output channel
        auto* buffer = bufferToFill.buffer->getWritePointer (channel, bufferToFill.startSample);

        // Fill the required number of samples with noise between -0.125 and +0.125
        for (auto sample = 0; sample < bufferToFill.numSamples; ++sample)
            buffer[sample] = random.nextFloat() * 0.25f - 0.125f;
    }
}
```

## Generating white noise using the Random class

Here we make use of the [Random](https://docs.juce.com/master/classRandom.html "A random number generator.") class in order to generate our random values (see [Tutorial: The Random class](/tutorials/tutorial_random/)). To generate white noise we need to generate uniformly distributed random numbers around zero. Here we generate random values between -0.125 and +0.125 by first calling the [Random::nextFloat()](https://docs.juce.com/master/classRandom.html#aec88d4e5cf44faaa038f6cfb41e96406 "Returns the next random floating-point number.") function. This generates values between 0 and 1. Then we multiply the result of this by 0.25 and subtract 0.125. (See [Tutorial: Control audio levels](/tutorials/tutorial_synth_level_control/) for more information on this process.) Notice that we didn\'t use the [Random::getSystemRandom()](https://docs.juce.com/master/classRandom.html#ad7d9d42dd0efbb68d569e975b0778518 "The overhead of creating a new Random object is fairly small, but if you want to avoid it,...") function to get the shared _system_ [Random](https://docs.juce.com/master/classRandom.html "A random number generator.") object, as shown in other tutorials ([Tutorial: The Random class](/tutorials/tutorial_random/)). This is because we are calling the [Random::nextFloat()](https://docs.juce.com/master/classRandom.html#aec88d4e5cf44faaa038f6cfb41e96406 "Returns the next random floating-point number.") function on the audio thread. We need to create our own [Random](https://docs.juce.com/master/classRandom.html "A random number generator.") object otherwise the values might get corrupted by other threads using that shared [Random](https://docs.juce.com/master/classRandom.html "A random number generator.") object. To achieve this, an instance of the [Random](https://docs.juce.com/master/classRandom.html "A random number generator.") class is added to our `MainContentComponent` class:

```cpp
private:
juce::Random random;

JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR (MainContentComponent)
};
```

## Shutting down the audio application

When our application is closed, our destructor will be called. At this point we should call the [AudioAppComponent::shutdownAudio()](https://docs.juce.com/master/classAudioAppComponent.html#a40f1e657f048d02c5d065de566a84d92 "Shuts down the audio device and clears the audio source.") function:

```cpp
~MainContentComponent() override
{
    shutdownAudio();
}
```

Calling the [AudioAppComponent::shutdownAudio()](https://docs.juce.com/master/classAudioAppComponent.html#a40f1e657f048d02c5d065de566a84d92 "Shuts down the audio device and clears the audio source.") function will, in turn, cause the [AudioAppComponent::releaseResources()](https://docs.juce.com/master/classAudioAppComponent.html#a588d9b990fc308774215548959e0fc50 "Allows the source to release anything it no longer needs after playback has stopped.") function to be called. This is a good place to dispose of resources, if we allocated any during the running of our audio process (for example, if we allocated memory or opened some files). In this case, we didn\'t need any additional resources and we just log the function call with a simple message:

```cpp
void releaseResources() override
{
    juce::Logger::getCurrentLogger()->writeToLog ("Releasing audio resources");
}
```

> [!NOTE]
> Exercise: Try changing the number of audio outputs. Notice that mono noise sounds subtly different from stereo noise. If you have a multichannel sound card, you may be able to generate more than two channels of noise. You could also try changing the level of the noise generated. For example, to generate noise at a level of 0.1 you would need to multiply the randomly generated values by 0.2 and subtract 0.1.

# Summary

This tutorial has introduced the [AudioAppComponent](https://docs.juce.com/master/classAudioAppComponent.html "A base class for writing audio apps that stream from the audio i/o devices.") class, used by the Audio Application template, in order to generate audio. We have covered the following topics:

- Initialising and shutting down the audio system.
- Writing audio data in response to an audio callback.
- Using the [AudioSourceChannelInfo](structAudioSourceChannelInfo.html "Used by AudioSource::getNextAudioBlock().") struct and the AudioSampleBuffer class.

# See also

- [Tutorial: The AudioDeviceManager class](/tutorials/tutorial_audio_device_manager/)
- [Tutorial: Build a sine wave synthesiser](/tutorials/tutorial_sine_synth/)
- [Tutorial: Control audio levels using decibels](/tutorials/tutorial_synth_db_level_control/)
- [Tutorial: Control audio levels](/tutorials/tutorial_synth_level_control/)
- [Tutorial: Build a MIDI synthesiser](/tutorials/tutorial_synth_using_midi_input/)
- [Tutorial: Wavetable synthesis](/tutorials/tutorial_wavetable_synth/)
