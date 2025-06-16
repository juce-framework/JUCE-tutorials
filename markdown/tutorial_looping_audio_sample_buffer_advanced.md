# Tutorial: Looping audio using the AudioSampleBuffer class (advanced)

This tutorial shows how to play and loop audio stored in an [AudioSampleBuffer](group__juce__audio__basics-buffers.html#gab339ebab0d3b10c91c0d47c8fd2e50d2) object using thread-safe techniques. A technique for loading the audio data on a background thread is also introduced.

**LEVEL:** Advanced<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**CLASSES:** [ReferenceCountedObject](https://docs.juce.com/master/classReferenceCountedObject.html "A base class which provides methods for reference-counting."), [ReferenceCountedArray](https://docs.juce.com/master/classReferenceCountedArray.html "Holds a list of objects derived from ReferenceCountedObject, or which implement basic reference-count..."), [Thread](https://docs.juce.com/master/classThread.html "Encapsulates a thread."), [AudioBuffer](https://docs.juce.com/master/classAudioBuffer.html "A multi-channel buffer containing floating point audio samples.")

# Getting started

This tutorial leads on from [Tutorial: Looping audio using the AudioSampleBuffer class](/tutorials/tutorial_looping_audio_sample_buffer/). If you haven\'t done so already, you should read that tutorial first.

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/LoopingAudioSampleBufferAdvancedTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/LoopingAudioSampleBufferAdvancedTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project implements similar behaviour to the demo project from [Tutorial: Looping audio using the AudioSampleBuffer class](/tutorials/tutorial_looping_audio_sample_buffer/). It allows the user to open an audio file that is loaded into a buffer and played in a loop. One major difference in this tutorial is that the audio system is kept running, rather than shutting it down each time we browse for a file. This is achieved by using some helpful classes for communicating between threads in a thread-safe manner.

# Thread-safe techniques

You should recall in [Tutorial: Looping audio using the AudioSampleBuffer class](/tutorials/tutorial_looping_audio_sample_buffer/) how we solved the potential problem of the audio thread and the message thread accessing potentially incomplete or corrupted data. Just before we browsed for a file we shut down the audio system. Then, once a file was selected, we opened the file and restarted the audio system. This is clearly an impractical and cumbersome method in a real application!

## Reference-counted objects

The [ReferenceCountedObject](https://docs.juce.com/master/classReferenceCountedObject.html "A base class which provides methods for reference-counting.") class is a useful tool for passing messages and data between threads. Here, we store our [AudioSampleBuffer](group__juce__audio__basics-buffers.html#gab339ebab0d3b10c91c0d47c8fd2e50d2) object and the playback position in a [ReferenceCountedObject](https://docs.juce.com/master/classReferenceCountedObject.html "A base class which provides methods for reference-counting.") class. To help with debugging, and to help illustrate how the class works, we also include `name` member (although this isn\'t strictly necessary for the class to function):

```cpp
class ReferenceCountedBuffer : public juce::ReferenceCountedObject
{
public:
    typedef juce::ReferenceCountedObjectPtr<ReferenceCountedBuffer> Ptr;

    ReferenceCountedBuffer (const juce::String& nameToUse,
        int numChannels,
        int numSamples)
        : name (nameToUse),
          buffer (numChannels, numSamples)
    {
        DBG (juce::String ("Buffer named '") + name + "' constructed. numChannels = " + juce::String (numChannels) + ", numSamples = " + juce::String (numSamples));
    }

    ~ReferenceCountedBuffer()
    {
        DBG (juce::String ("Buffer named '") + name + "' destroyed");
    }

    juce::AudioSampleBuffer* getAudioSampleBuffer()
    {
        return &buffer;
    }

    int position = 0;

private:
    juce::String name;
    juce::AudioSampleBuffer buffer;

    JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR (ReferenceCountedBuffer)
};
```

The `typedef` at the top of the class is an important part in implementing a [ReferenceCountedObject](https://docs.juce.com/master/classReferenceCountedObject.html "A base class which provides methods for reference-counting.") subclass. Rather than storing our `ReferenceCountedBuffer` object in a raw pointer, we store it in a `ReferenceCountedBuffer::Ptr` type. It is this that manages the reference count of the object (incrementing and decrementing as necessary) and its lifetime (deleting the object when the reference count reaches zero). We can also store an array of `ReferenceCountedBuffer` objects using the [ReferenceCountedArray](https://docs.juce.com/master/classReferenceCountedArray.html "Holds a list of objects derived from ReferenceCountedObject, or which implement basic reference-count...") class.

In our `MainContentComponent` class we store both an array and a single instance:

```cpp
juce::SpinLock mutex;
juce::ReferenceCountedArray<ReferenceCountedBuffer> buffers;
ReferenceCountedBuffer::Ptr currentBuffer;
```

The `buffers` member keeps hold of our buffers in the array until we are absolutely sure they are no longer needed by the audio thread. The `currentBuffer` member holds the currently selected buffer.

## Implementing the background thread

Our `MainContentComponent` class inherits from the [Thread](https://docs.juce.com/master/classThread.html "Encapsulates a thread.") class:

```cpp
class MainContentComponent : public juce::AudioAppComponent,
                             private juce::Thread
{
public:
```

This is used to implement our background thread. Our overridden [Thread::run()](https://docs.juce.com/master/classThread.html#aae90dfabab3e1776cf01a26e7ee3a620 "Must be implemented to perform the thread's actual code.") function is as follows:

```cpp
void run() override
{
    while (!threadShouldExit())
    {
        checkForBuffersToFree();
        wait (500);
    }
}
```

Here, we check whether there are any buffers to be freed, then our thread waits for 500ms or to be woken up (using the [Thread::notify()](https://docs.juce.com/master/classThread.html#a0b4d5a1ffaa35cc13f323a0524012a2b "Wakes up the thread.") function). Essentially, this means that the check will occur at least every 500ms. The `checkForBuffersToFree()` function searches through our `buffers` array to see if any buffers can be freed:

```cpp
void checkForBuffersToFree()
{
    for (auto i = buffers.size(); --i >= 0;) // [1]
    {
        ReferenceCountedBuffer::Ptr buffer (buffers.getUnchecked (i)); // [2]

        if (buffer->getReferenceCount() == 2) // [3]
            buffers.remove (i);
    }
}
```

- [1]: It is useful to remember to iterate over the array in reverse in these situations. It is easier to avoid corrupting the array index access if we remove items as we iterate over the array.
- [2]: This retains a copy of a buffer at the specified index.
- [3]: If the reference count at this point is equal to 2 then we know that the audio thread can\'t be using the buffer and we can remove it from the array. One of these two references will be in the `buffers` and the other will be in the local `buffer` variable. The removed buffer will delete itself as the `buffer` variable goes out of scope (as this will be the last remaining reference).

Of course, we need to start the thread as our application starts, which we do in our `MainContentComponent` constructor:

```cpp
startThread();
}
```

## Opening the file

Our `openButtonClicked()` function is similar to the `openButtonClicked()` function from [Tutorial: Looping audio using the AudioSampleBuffer class](/tutorials/tutorial_looping_audio_sample_buffer/) with some minor differences:

```cpp
void openButtonClicked()
{
    chooser = std::make_unique<juce::FileChooser> ("Select a Wave file shorter than 2 seconds to play...",
        juce::File {},
        "*.wav");
    auto chooserFlags = juce::FileBrowserComponent::openMode
                        | juce::FileBrowserComponent::canSelectFiles;

    chooser->launchAsync (chooserFlags, [this] (const juce::FileChooser& fc) {
        auto file = fc.getResult();

        if (file == juce::File {})
            return;

        std::unique_ptr<juce::AudioFormatReader> reader (formatManager.createReaderFor (file));

        if (reader != nullptr)
        {
            auto duration = (float) reader->lengthInSamples / reader->sampleRate;

            if (duration < 2)
            {
                ReferenceCountedBuffer::Ptr newBuffer = new ReferenceCountedBuffer (file.getFileName(),
                    (int) reader->numChannels,
                    (int) reader->lengthInSamples);

                reader->read (newBuffer->getAudioSampleBuffer(), 0, (int) reader->lengthInSamples, 0, true, true);

                {
                    const juce::SpinLock::ScopedLockType lock (mutex);
                    currentBuffer = newBuffer;
                }

                buffers.add (newBuffer);
            }
            else
            {
                // handle the error that the file is 2 seconds or longer..
            }
        }
    });
}
```

Here are the differences:

- Allocate a new instance of our `ReferenceCountedBuffer` class.
- Read the audio data into the [AudioSampleBuffer](group__juce__audio__basics-buffers.html#gab339ebab0d3b10c91c0d47c8fd2e50d2) object that it contains.
- Make it the current buffer.
- Add it to our array of buffers.

To clear the current buffer we can just set its value to `nullptr` :

```cpp
void clearButtonClicked()
{
    const juce::SpinLock::ScopedLockType lock (mutex);
    currentBuffer = nullptr;
}
```

## Playing the buffer

Our `getNextAudioBlock()` function is similar to the `getNextAudioBlock()` function from [Tutorial: Looping audio using the AudioSampleBuffer class](/tutorials/tutorial_looping_audio_sample_buffer/) except we need to access our current `ReferenceCountedBuffer` object and the [AudioSampleBuffer](group__juce__audio__basics-buffers.html#gab339ebab0d3b10c91c0d47c8fd2e50d2) object it contains.

```cpp
void getNextAudioBlock (const juce::AudioSourceChannelInfo& bufferToFill) override
{
    auto retainedCurrentBuffer = [&]() -> ReferenceCountedBuffer::Ptr // [4]
    {
        const juce::SpinLock::ScopedTryLockType lock (mutex);

        if (lock.isLocked())
            return currentBuffer;

        return nullptr;
    }();

    if (retainedCurrentBuffer == nullptr) // [5]
    {
        bufferToFill.clearActiveBufferRegion();
        return;
    }

    auto* currentAudioSampleBuffer = retainedCurrentBuffer->getAudioSampleBuffer(); // [6]
    auto position = retainedCurrentBuffer->position; // [7]

    auto numInputChannels = currentAudioSampleBuffer->getNumChannels();
    auto numOutputChannels = bufferToFill.buffer->getNumChannels();

    auto outputSamplesRemaining = bufferToFill.numSamples;
    auto outputSamplesOffset = 0;

    while (outputSamplesRemaining > 0)
    {
        auto bufferSamplesRemaining = currentAudioSampleBuffer->getNumSamples() - position;
        auto samplesThisTime = juce::jmin (outputSamplesRemaining, bufferSamplesRemaining);

        for (auto channel = 0; channel < numOutputChannels; ++channel)
        {
            bufferToFill.buffer->copyFrom (channel,
                bufferToFill.startSample + outputSamplesOffset,
                *currentAudioSampleBuffer,
                channel % numInputChannels,
                position,
                samplesThisTime);
        }

        outputSamplesRemaining -= samplesThisTime;
        outputSamplesOffset += samplesThisTime;
        position += samplesThisTime;

        if (position == currentAudioSampleBuffer->getNumSamples())
            position = 0;
    }

    retainedCurrentBuffer->position = position; // [8]
}
```

The important changes are:

- [4]: We retain a copy of the `currentBuffer` member. After this point in the function it doesn\'t matter if the `currentBuffer` member is changed on another thread since we have taken a local copy. Note that we use a try lock here, so that the audio thread doesn\'t get stuck waiting to access the `currentBuffer` in the case that another thread is currently modifying it.
- [5]: We output silence if the `currentBuffer` member was null when we took a copy.
- [6]: We access the [AudioSampleBuffer](group__juce__audio__basics-buffers.html#gab339ebab0d3b10c91c0d47c8fd2e50d2) object within the `ReferenceCountedBuffer` object.
- [7]: We get the current playback position for the buffer.
- [8]: After modifying the current playback position, we store it back in the `ReferenceCountedBuffer` object.

This algorithm ensures that `ReferenceCountedBuffer` objects aren\'t deleted on the the audio thread. It is not a good idea to allocate or free memory on the audio thread. The `ReferenceCountedBuffer` objects will only be deleted on our background thread.

# Reading the audio on the background thread

Our application still reads the audio data on the message thread. This is not ideal since this blocks the message thread and large files could take some time to load. In fact, we can also use our background thread to perform this task.

## Passing the file path to the background thread

First, add the following member to the `MainContentComponent` class:

```cpp
juce::CriticalSection pathMutex;
juce::String chosenPath;
```

Now change the `openButtonClicked()` function to _swap_ the full path of the file into this member. Swapping strings is not technically thread-safe, so we also need to take a lock to ensure that no other threads try to modify `chosenPath` while this thread is using it.

```cpp
void openButtonClicked()
{
    chooser = std::make_unique<juce::FileChooser> ("Select a Wave file shorter than 2 seconds to play...",
        juce::File {},
        "*.wav");
    auto chooserFlags = juce::FileBrowserComponent::openMode
                        | juce::FileBrowserComponent::canSelectFiles;

    chooser->launchAsync (chooserFlags, [this] (const juce::FileChooser& fc) {
        auto file = fc.getResult();

        if (file == juce::File {})
            return;

        auto path = file.getFullPathName();

        {
            const juce::ScopedLock lock (pathMutex);
            chosenPath.swapWith (path);
        }

        notify();
    });
}
```

Here we also wake up the background thread since we are going to call a function on the background thread to open the file.

## Accessing the path from the background thread

Our `run()` function should be updated as follows:

```cpp
void run() override
{
    while (!threadShouldExit())
    {
        checkForPathToOpen();
        checkForBuffersToFree();
        wait (500);
    }
}
```

The `checkForPathToOpen()` function checks the `chosenPath` member by swapping it into a local variable. Again, swapping is not thread-safe, so we must take a lock before accessing `chosenPath` .

```cpp
void checkForPathToOpen()
{
    juce::String pathToOpen;

    {
        const juce::ScopedLock lock (pathMutex);
        pathToOpen.swapWith (chosenPath);
    }

    if (pathToOpen.isNotEmpty())
    {
        juce::File file (pathToOpen);
        std::unique_ptr<juce::AudioFormatReader> reader (formatManager.createReaderFor (file));

        if (reader.get() != nullptr)
        {
            auto duration = (float) reader->lengthInSamples / reader->sampleRate;

            if (duration < 2)
            {
                ReferenceCountedBuffer::Ptr newBuffer = new ReferenceCountedBuffer (file.getFileName(),
                    (int) reader->numChannels,
                    (int) reader->lengthInSamples);

                reader->read (newBuffer->getAudioSampleBuffer(), 0, (int) reader->lengthInSamples, 0, true, true);

                {
                    const juce::SpinLock::ScopedLockType lock (mutex);
                    currentBuffer = newBuffer;
                }

                buffers.add (newBuffer);
            }
            else
            {
                // handle the error that the file is 2 seconds or longer..
            }
        }
    }
}
```

If the `pathToOpen` variable is an empty string then we know there isn\'t a new file to open. The remainder of the code in this function should be familiar to you.

Run the application again and it should still function correctly.

> [!TIP]
>The final code for this section can be found in the `LoopingAudioSampleBufferAdvancedTutorial_02.h` file of the demo project.

# Summary

This tutorial has introduced some useful techniques for passing data between threads, especially in an audio application. After reading this tutorial you should be able to:

- Implement a subclass of the [ReferenceCountedObject](https://docs.juce.com/master/classReferenceCountedObject.html "A base class which provides methods for reference-counting.") class.
- Maintain the lifetime of a [ReferenceCountedObject](https://docs.juce.com/master/classReferenceCountedObject.html "A base class which provides methods for reference-counting.") in a multi-threaded application.
- Implement a background thread to perform tasks such as deleting objects that are no longer needed and performing file reading operations.

# See also

- [Tutorial: Build an audio player](/tutorials/tutorial_playing_sound_files/)
- [Tutorial: Processing audio input](/tutorials/tutorial_processing_audio_input/)
- [Tutorial: Draw audio waveforms](/tutorials/tutorial_audio_thumbnail/)
- [Tutorial: Looping audio using the AudioSampleBuffer class](/tutorials/tutorial_looping_audio_sample_buffer/)
