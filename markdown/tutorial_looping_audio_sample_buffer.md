# Tutorial: Looping audio using the AudioSampleBuffer class

This tutorial shows how to play and loop audio stored in an AudioSampleBuffer object. This is a useful basis for sampling applications that manipulate recorded audio data.

**LEVEL:** Intermediate<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**CLASSES:** [AudioBuffer](https://docs.juce.com/master/classAudioBuffer.html "A multi-channel buffer containing floating point audio samples."), [AudioFormatReader](https://docs.juce.com/master/classAudioFormatReader.html "Reads samples from an audio file stream."), [AudioAppComponent](https://docs.juce.com/master/classAudioAppComponent.html "A base class for writing audio apps that stream from the audio i/o devices.")

# Getting started

> [!TIP]
>This tutorial assumes you have already completed [Tutorial: Build a white noise generator](/tutorials/tutorial_simple_synth_noise/) and [Tutorial: Build an audio player](/tutorials/tutorial_playing_sound_files/). If not, you should look at these first.

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/LoopingAudioSampleBufferTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/LoopingAudioSampleBufferTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project for this tutorial allows the user to open a sound file, read the whole file into an [AudioSampleBuffer](group__juce__audio__basics-buffers.html#gab339ebab0d3b10c91c0d47c8fd2e50d2) object, and play it in a loop. In [Tutorial: Build an audio player](/tutorials/tutorial_playing_sound_files/) we played sound files using an [AudioFormatReaderSource](https://docs.juce.com/master/classAudioFormatReaderSource.html "A type of AudioSource that will read from an AudioFormatReader.") object connected to an [AudioTransportSource](https://docs.juce.com/master/classAudioTransportSource.html "An AudioSource that takes a PositionableAudioSource and allows it to be played, stopped,...") object to play the sound. Looping is possible using this method by enabling enabling the [AudioFormatReaderSource](https://docs.juce.com/master/classAudioFormatReaderSource.html "A type of AudioSource that will read from an AudioFormatReader.") object's looping flag --- using the [AudioFormatReaderSource::setLooping()](https://docs.juce.com/master/classAudioFormatReaderSource.html#a15d8af211ce8cfcbc0c4aaac143b303e "Toggles loop-mode.") function.

All of the code relevant to the discussion in this tutorial is in the `MainContentComponent` class of the demo project.

# Loading sample data into memory

There are many cases where it is probably better to use the built-in classes for sound file playback. There may be occasions where you need to do this yourself and this tutorial gives an introduction to some of the techniques. Sampler applications commonly load the sound file data into memory like this, especially when the sounds are relatively short (see the [SamplerSound](https://docs.juce.com/master/classSamplerSound.html "A subclass of SynthesiserSound that represents a sampled audio clip.") class for an example). Synthesising sounds can also be achieved by storing a wavetable in an [AudioSampleBuffer](group__juce__audio__basics-buffers.html#gab339ebab0d3b10c91c0d47c8fd2e50d2) object and looping it at an appropriate rate to produce the required musical pitch. This is explored in [Tutorial: Wavetable synthesis](/tutorials/tutorial_wavetable_synth/).

This tutorial also highlights some of the potential multi-threading issues you may encounter when combining access to files, and audio processing on the audio thread. Some of these problems seem simple on the surface but often require carefully applied techniques in order to avoid crashes and audio glitches. These techniques are explored further in [Tutorial: Looping audio using the AudioSampleBuffer class (advanced)](/tutorials/tutorial_looping_audio_sample_buffer_advanced/).

## Why the length limit?

The demo project limits the length of the sound file you can load to less than 2 seconds. This limit is rather arbitrary, but this is broadly for two reasons:

1.  If the whole file is very large then your computer might run out of physical memory. In a real application, of course, you would be able to use a much higher limit. A 2-second stereo audio file, at a sample rate of 44.1kHz, loaded into an [AudioSampleBuffer](group__juce__audio__basics-buffers.html#gab339ebab0d3b10c91c0d47c8fd2e50d2) object, will only occupy 705,600 bytes of memory. (See notes)
2.  Loading even quite short files doesn\'t take a trivial amount of time.

Regarding point 1: if we exceed the amount of physical memory the computer has, it may start to use virtual memory (that is, secondary storage such as a hard drive). This rather defeats the purpose of loading the data into memory in the first place! Of course the operation may just fail on some devices if it runs out of memory.

Regarding point 2: we keep the example simple by loading the audio data directly in, after the [FileChooser::browseForFileToOpen()](https://docs.juce.com/master/classFileChooser.html#a546ef74bcd139b67a90e4459cd591d21 "Shows a dialog box to choose a file to open.") function has returned the file selected by the user. This means that the _message thread_ will be blocked until all of the audio has been read in from disk into the [AudioSampleBuffer](group__juce__audio__basics-buffers.html#gab339ebab0d3b10c91c0d47c8fd2e50d2) object. Even with short sounds we should really do this on a background thread to keep the user interface as responsive as possible for the user. For long sounds the delay and unresponsiveness will be very noticeable. Adding another (background) thread would add to the complexity of this example. See [Tutorial: Looping audio using the AudioSampleBuffer class (advanced)](/tutorials/tutorial_looping_audio_sample_buffer_advanced/) for example of how to load files on a background thread in this way.

> [!NOTE]
> Exercise: To keep it simple, demo project doesn\'t report an error if you try to load a longer file --- it just fails. Adding error reporting like this is left for you as an additional exercise.

## Reading the sound file

When the user clicks the **Open...** button they are presented with a file chooser. The whole file is then read into an [AudioSampleBuffer](group__juce__audio__basics-buffers.html#gab339ebab0d3b10c91c0d47c8fd2e50d2) member `fileBuffer` in our `MainContentComponent` class.

```cpp
void openButtonClicked()
{
    shutdownAudio(); // [1]

    chooser = std::make_unique<juce::FileChooser> ("Select a Wave file shorter than 2 seconds to play...",
        juce::File {},
        "*.wav");
    auto chooserFlags = juce::FileBrowserComponent::openMode
                        | juce::FileBrowserComponent::canSelectFiles;

    chooser->launchAsync (chooserFlags, [this] (const juce::FileChooser& fc) {
        auto file = fc.getResult();

        if (file == juce::File {})
            return;

        std::unique_ptr<juce::AudioFormatReader> reader (formatManager.createReaderFor (file)); // [2]

        if (reader.get() != nullptr)
        {
            auto duration = (float) reader->lengthInSamples / reader->sampleRate; // [3]

            if (duration < 2)
            {
                fileBuffer.setSize ((int) reader->numChannels, (int) reader->lengthInSamples); // [4]
                reader->read (&fileBuffer, // [5]
                    0, // [5.1]
                    (int) reader->lengthInSamples, // [5.2]
                    0, // [5.3]
                    true, // [5.4]
                    true); // [5.5]
                position = 0; // [6]
                setAudioChannels (0, (int) reader->numChannels); // [7]
            }
            else
            {
                // handle the error that the file is 2 seconds or longer..
            }
        }
    });
}
```

- [1]: Notice that we shut down the audio system for the [AudioAppComponent](https://docs.juce.com/master/classAudioAppComponent.html "A base class for writing audio apps that stream from the audio i/o devices.") object each time we open a new file. This is to avoid some of the multithreading issues hinted at already. Once the audio system is shut down, there is no danger that our `getNextAudioBlock()` function will be called on the _audio thread_ while we are still within the call to the [Button::onClick](https://docs.juce.com/master/classButton.html#a30b76ab312dc7f66e67596ae20540ec2 "You can assign a lambda to this callback object to have it called when the button is clicked.") lambda function (which will have called this `openButtonClicked()` function from the _message thread_).
- [2]: Here we create the [AudioFormatReader](https://docs.juce.com/master/classAudioFormatReader.html "Reads samples from an audio file stream.") object using the [AudioFormatManager](https://docs.juce.com/master/classAudioFormatManager.html "A class for keeping a list of available audio formats, and for deciding which one to use to open a gi...") object. Notice that we store this in a std::unique_ptr object as we need to manage this object ourselves. (In [Tutorial: Build an audio player](/tutorials/tutorial_playing_sound_files/) we pass the [AudioFormatReader](https://docs.juce.com/master/classAudioFormatReader.html "Reads samples from an audio file stream.") object to the [AudioFormatReaderSource](https://docs.juce.com/master/classAudioFormatReaderSource.html "A type of AudioSource that will read from an AudioFormatReader.") object to manage for for us.) This operation may fail to create the reader object, therefore we have to check that the `reader` pointer is not a `nullptr` value on the next line.
- [3]: This is where we calculate the duration of the sound file by dividing the length of the file in samples by its sample rate. We check that this is less that 2 seconds on the next line.
- [4]: Here we resize the [AudioSampleBuffer](group__juce__audio__basics-buffers.html#gab339ebab0d3b10c91c0d47c8fd2e50d2) object by calling the [AudioSampleBuffer::setSize()](https://docs.juce.com/master/classAudioBuffer.html#a4434de94aa03d7db6d7ef06977ddf0ac) function using the number of channels and length from the [AudioFormatReader](https://docs.juce.com/master/classAudioFormatReader.html "Reads samples from an audio file stream.") object.
- [5]: This reads the audio data from the [AudioFormatReader](https://docs.juce.com/master/classAudioFormatReader.html "Reads samples from an audio file stream.") object into our [AudioSampleBuffer](group__juce__audio__basics-buffers.html#gab339ebab0d3b10c91c0d47c8fd2e50d2) `fileBuffer` member using the [AudioFormatReader::read()](https://docs.juce.com/master/classAudioFormatReader.html#ad180e2b06c2a10c0ca399c3231b155c0) function. The arguments are:
  - [5.1]: The destination start sample in the [AudioSampleBuffer](group__juce__audio__basics-buffers.html#gab339ebab0d3b10c91c0d47c8fd2e50d2) object where the data will start to be written.
  - [5.2]: The number of samples to read.
  - [5.3]: The start samples in the [AudioFormatReader](https://docs.juce.com/master/classAudioFormatReader.html "Reads samples from an audio file stream.") object where reading will start.
  - [5.4]: For stereo (or other two-channel) files this flag indicates whether to read the left channel.
  - [5.5]: For stereo files this flag indicates whether to read the right channel.
- [6]: We need to store the most recent read position within our buffer as we play it. This resets our `position` member to zero.
- [7]: This starts the audio system back up again. Here we have an opportunity to use the number of channels in the sound file to try and configure our audio device with the same number of channels.

# Processing the audio

In the `getNextAudioBlock()` function the appropriate number of samples is read from our `fileBuffer` [AudioSampleBuffer](group__juce__audio__basics-buffers.html#gab339ebab0d3b10c91c0d47c8fd2e50d2) member and written out the the [AudioSampleBuffer](group__juce__audio__basics-buffers.html#gab339ebab0d3b10c91c0d47c8fd2e50d2) object in the [AudioSourceChannelInfo](structAudioSourceChannelInfo.html "Used by AudioSource::getNextAudioBlock().") struct.

While reading the audio data from the file we keep track of the current read position using the `position` member (being careful to update it after all the channels of the audio have been processed for the specified block of samples):

```cpp
void getNextAudioBlock (const juce::AudioSourceChannelInfo& bufferToFill) override
{
    auto numInputChannels = fileBuffer.getNumChannels();
    auto numOutputChannels = bufferToFill.buffer->getNumChannels();

    auto outputSamplesRemaining = bufferToFill.numSamples; // [8]
    auto outputSamplesOffset = bufferToFill.startSample; // [9]

    while (outputSamplesRemaining > 0)
    {
        auto bufferSamplesRemaining = fileBuffer.getNumSamples() - position; // [10]
        auto samplesThisTime = juce::jmin (outputSamplesRemaining, bufferSamplesRemaining); // [11]

        for (auto channel = 0; channel < numOutputChannels; ++channel)
        {
            bufferToFill.buffer->copyFrom (channel, // [12]
                outputSamplesOffset, // [12.1]
                fileBuffer, // [12.2]
                channel % numInputChannels, // [12.3]
                position, // [12.4]
                samplesThisTime); // [12.5]
        }

        outputSamplesRemaining -= samplesThisTime; // [13]
        outputSamplesOffset += samplesThisTime; // [14]
        position += samplesThisTime; // [15]

        if (position == fileBuffer.getNumSamples())
            position = 0; // [16]
    }
}
```

- [8]: The `outputSamplesRemaining` variable stores the total number of samples that the `getNextAudioBlock()` function needs to output taking a copy from the [AudioSourceChannelInfo](structAudioSourceChannelInfo.html "Used by AudioSource::getNextAudioBlock().") struct. We use this to check if we need to exit the `while()` loop that starts on the next line.
- [9]: We also take a copy of the [AudioSourceChannelInfo::startSample](structAudioSourceChannelInfo.html#a42e88ccc05d4893015e2415785390259 "The first sample in the buffer from which the callback is expected to write data.") value to use as our offset within the destination buffer.
- [10]: Here we calculate how many samples are left in the buffer from which we are reading.
- [11]: For this pass of the `while()` loop we need to output the smaller of the remaining samples for this call to the `getNextAudioBlock()` function and the remaining samples in the buffer --- using the [jmin()](group__juce__core-maths.html#gae8394cdf11279b266f4aa741758c1669) function. If this is less than the total number of samples for this call to the `getNextAudioBlock()` function, then there will be one more pass of the `while()` loop, before exiting.
- [12]: For each output channel we use the [AudioSampleBuffer::copyFrom()](https://docs.juce.com/master/classAudioBuffer.html#a9ec751bfa23564c011bf3940ca17b743) function to copy sections of data from one channel of one buffer to a channel of another buffer. Here we specify the destination channel index.
  - [12.1]: This is the sample offset within the destination buffer.
  - [12.2]: This is the source [AudioSampleBuffer](group__juce__audio__basics-buffers.html#gab339ebab0d3b10c91c0d47c8fd2e50d2) object from which to copy.
  - [12.3]: This is the channel index of the source buffer. In case the source buffer has fewer channels than our destination buffer we use this modulo calculation. For example, a mono source buffer will mean that this always results in zero, copying the same data to each of the output channels.
  - [12.4]: This is the position to start reading from in the source buffer.
  - [12.5]: The number of samples to read that we calculated earlier.
- [13]: Now deduct the number of samples we just processed from the `outputSamplesRemaining` variable.
- [14]: Increment the `outputSamplesOffset` by the same amount in case we have another pass of the `while()` loop.
- [15]: Offset our `position` member by the same amount too.
- [16]: Finally, check if the `position` member reached the end of the `fileBuffer` [AudioSampleBuffer](group__juce__audio__basics-buffers.html#gab339ebab0d3b10c91c0d47c8fd2e50d2) object and reset it to zero to form the loop if necessary.

> [!NOTE]
> Exercise: Add a level slider to control the audio playback level of the audio file (see [Tutorial: Control audio levels](/tutorials/tutorial_synth_level_control/)). You can use the [AudioSampleBuffer::applyGain()](https://docs.juce.com/master/classAudioBuffer.html#a9ffc61d339e455d4bddc7cf055a63ee3) or [AudioSampleBuffer::applyGainRamp()](https://docs.juce.com/master/classAudioBuffer.html#ab0542e5b626b36087f0054e795695682) functions to apply the gain to the data in an [AudioSampleBuffer](group__juce__audio__basics-buffers.html#gab339ebab0d3b10c91c0d47c8fd2e50d2) object.

# Multithreading issues

As discussed previously, this tutorial avoids multithreading issues by shutting down and restarting audio each time the user clicks the **Open...** button. But what if we didn\'t do this --- what could happen? There many things that could go wrong, all of which have to do with the fact that both the `getNextAudioBlock()` and `openButtonClicked()` functions could be running at the same time in different threads. Here are some examples:

- Let's say that the application is already playing an audio file and the user clicks the **Open...** button and chooses a new file. Suppose the audio thread interrupts this function between [4] and [5]. The buffer has been resized but the data hasn\'t been written to the buffer. The buffer may still contain audio data from the previous file but it depends whether the memory for the buffer needed to be moved when it was resized. In any case we\'ll probably get a glitch.
- It's possible that the `getNextAudioBlock()` function could be interrupted by code in the `openButtonClicked()` function. Suppose this happens just after [11]and that the `openButtonClicked()` function has just reached [4]. The buffer might be resized to be shorter than it was but we already calculated our starting point a few lines earlier. This could lead to a memory access error and the application could crash.
- The `getNextAudioBlock()` function could be interrupted while calling the [AudioSampleBuffer::copyFrom()](https://docs.juce.com/master/classAudioBuffer.html#a9ec751bfa23564c011bf3940ca17b743) function. Again depending in the implementation of this we could end up accessing memory that we shouldn\'t.

> [!WARNING]
> There are a number of other things that could go wrong. You may be familiar with using a [critical section](https://en.wikipedia.org/wiki/Critical_section) to synchronise memory access between threads. This is just one possible solution but care should be taken using a [critical section](https://en.wikipedia.org/wiki/Critical_section) in audio code as it can lead to [priority inversion](https://en.wikipedia.org/wiki/Priority_inversion) which could cause audio drop outs. We look at a solution that avoids critical sections in [Tutorial: Looping audio using the AudioSampleBuffer class (advanced)](/tutorials/tutorial_looping_audio_sample_buffer_advanced/).

# Notes

Two seconds of stereo audio at 44.1kHz would use 705,600 bytes in an AudioSampleBuffer object because there are:

- 2 channels
- 2 seconds
- 44,100 samples
- 4 bytes-per-sample (using the `float` type)

Multiply these together and the result is: `2 x 2 x 44100 x 4 = 705600`

# Summary

In this tutorial we have introduced:

- How to read audio data directly from a sound file.
- How to copy the data into a buffer for playback.
- The basis for simple sampler applications and synthesisers using wavetable buffers.
- Some of the potential multithreading issues that exist in audio applications.

# See also

- [Tutorial: Build an audio player](/tutorials/tutorial_playing_sound_files/)
- [Tutorial: Processing audio input](/tutorials/tutorial_processing_audio_input/)
- [Tutorial: Draw audio waveforms](/tutorials/tutorial_audio_thumbnail/)
- [Tutorial: Looping audio using the AudioSampleBuffer class (advanced)](/tutorials/tutorial_looping_audio_sample_buffer_advanced/)
