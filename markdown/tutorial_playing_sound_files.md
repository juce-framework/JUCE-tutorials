---
title: Build an audio player
---
# Tutorial: Build an audio player

This tutorial covers how to open and play sound files. This includes some important classes for handling sound files in JUCE.

**LEVEL:** Intermediate<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**CLASSES:** [AudioFormatManager](https://docs.juce.com/master/classAudioFormatManager.html "A class for keeping a list of available audio formats, and for deciding which one to use to open a gi..."), [AudioFormatReader](https://docs.juce.com/master/classAudioFormatReader.html "Reads samples from an audio file stream."), [AudioFormatReaderSource](https://docs.juce.com/master/classAudioFormatReaderSource.html "A type of AudioSource that will read from an AudioFormatReader."), [AudioTransportSource](https://docs.juce.com/master/classAudioTransportSource.html "An AudioSource that takes a PositionableAudioSource and allows it to be played, stopped,..."), [FileChooser](https://docs.juce.com/master/classFileChooser.html "Creates a dialog box to choose a file or directory to load or save."), [ChangeListener](https://docs.juce.com/master/classChangeListener.html "Receives change event callbacks that are sent out by a ChangeBroadcaster."), [File](https://docs.juce.com/master/classFile.html "Represents a local file or directory."), [FileChooser](https://docs.juce.com/master/classFileChooser.html "Creates a dialog box to choose a file or directory to load or save.")

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/PlayingSoundFilesTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/PlayingSoundFilesTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project presents a three-button interface for controlling the playback of a sound file. The three buttons are:

- A button to present a file chooser to the user for them select the sound file.
- A button to play the sound.
- A button to stop the sound.

The interface is shown in the following screenshot:

![](/_images/tutorial_playing_sound_files_screenshot1.png "A three-button interface to control sound file playback.")

# Helpful classes

## The AudioSource class

While we can generate audio sample-by-sample in the `getNextAudioBlock()` of the Audio Application template, there are some built-in tools for generating and processing audio. These allow us to link together high-level building blocks to form powerful audio applications without having to process each and every sample of audio within our application code (JUCE does this on our behalf). These building blocks are based on the [AudioSource](https://docs.juce.com/master/classAudioSource.html "Base class for objects that can produce a continuous stream of audio.") class. In fact, if you have followed any of the tutorials based on the [AudioAppComponent](https://docs.juce.com/master/classAudioAppComponent.html "A base class for writing audio apps that stream from the audio i/o devices.") class --- for example, [Tutorial: Build a white noise generator](/tutorials/tutorial_simple_synth_noise/) --- then you have been making use of the [AudioSource](https://docs.juce.com/master/classAudioSource.html "Base class for objects that can produce a continuous stream of audio.") class already. The [AudioAppComponent](https://docs.juce.com/master/classAudioAppComponent.html "A base class for writing audio apps that stream from the audio i/o devices.") class itself inherits from the [AudioSource](https://docs.juce.com/master/classAudioSource.html "Base class for objects that can produce a continuous stream of audio.") class and, importantly, contains an [AudioSourcePlayer](https://docs.juce.com/master/classAudioSourcePlayer.html "Wrapper class to continuously stream audio from an audio source to an AudioIODevice.") object that streams the audio between the [AudioAppComponent](https://docs.juce.com/master/classAudioAppComponent.html "A base class for writing audio apps that stream from the audio i/o devices.") and the audio hardware device. We can simply generate the audio samples directly in the `getNextAudioBlock()` function but we can instead chain a number of [AudioSource](https://docs.juce.com/master/classAudioSource.html "Base class for objects that can produce a continuous stream of audio.") objects together to form series of processes. We make use of this feature in this tutorial.

## Audio formats

JUCE provides number of tools for reading and writing sound files in a number of formats. In this tutorial we make use of several of these, in particular we use the following classes:

- [AudioFormatManager](https://docs.juce.com/master/classAudioFormatManager.html "A class for keeping a list of available audio formats, and for deciding which one to use to open a gi..."): This class contains a list of audio formats (such as WAV, AIFF, Ogg Vorbis, and so on) and can create suitable objects for reading audio data from these formats.
- [AudioFormatReader](https://docs.juce.com/master/classAudioFormatReader.html "Reads samples from an audio file stream."): This class handles the low-level file reading operations on the audio file and allows us to read audio in a consistent format (generally this means arrays of `float` values). When an [AudioFormatManager](https://docs.juce.com/master/classAudioFormatManager.html "A class for keeping a list of available audio formats, and for deciding which one to use to open a gi...") object is asked to open a particular file, it creates instances of this class.
- [AudioFormatReaderSource](https://docs.juce.com/master/classAudioFormatReaderSource.html "A type of AudioSource that will read from an AudioFormatReader."): This is a subclass of the [AudioSource](https://docs.juce.com/master/classAudioSource.html "Base class for objects that can produce a continuous stream of audio.") class. It can read audio data from an [AudioFormatReader](https://docs.juce.com/master/classAudioFormatReader.html "Reads samples from an audio file stream.") object and render the audio via its `getNextAudioBlock()` function.
- [AudioTransportSource](https://docs.juce.com/master/classAudioTransportSource.html "An AudioSource that takes a PositionableAudioSource and allows it to be played, stopped,..."): This class is another subclass of the [AudioSource](https://docs.juce.com/master/classAudioSource.html "Base class for objects that can produce a continuous stream of audio.") class. It can control the playback of an [AudioFormatReaderSource](https://docs.juce.com/master/classAudioFormatReaderSource.html "A type of AudioSource that will read from an AudioFormatReader.") object. This control includes starting and stopping the playback of the [AudioFormatReaderSource](https://docs.juce.com/master/classAudioFormatReaderSource.html "A type of AudioSource that will read from an AudioFormatReader.") object. It can also perform sample rate conversion and it can buffer audio ahead of time if we wish.

# Putting it together

We will now bring together these classes along with suitable user interface classes to make our sound file playing application. It is useful at this point to think about the various phases --- or _transport states_ --- of playing an audio file. Once the audio file is loaded we can consider these four possible states:

- _Stopped_: Audio playback is stopped and ready to be started.
- _Starting_: Audio playback hasn\'t yet started but it has been told to start.
- _Playing_: Audio is playing.
- _Stopping_: Audio is playing but playback has been told to stop, after this it will return to the _Stopped_ state.

To represent these states, we create an `enum` within our `MainContentComponent` class:

```cpp
enum TransportState {
    Stopped,
    Starting,
    Playing,
    Stopping
};
```

## Initialising the interface

In the constructor for our `MainContentComponent` class, we configure the three buttons:

```cpp
MainContentComponent()
    : state (Stopped)
{
    addAndMakeVisible (&openButton);
    openButton.setButtonText ("Open...");
    openButton.onClick = [this] { openButtonClicked(); };

    addAndMakeVisible (&playButton);
    playButton.setButtonText ("Play");
    playButton.onClick = [this] { playButtonClicked(); };
    playButton.setColour (juce::TextButton::buttonColourId, juce::Colours::green);
    playButton.setEnabled (false);

    addAndMakeVisible (&stopButton);
    stopButton.setButtonText ("Stop");
    stopButton.onClick = [this] { stopButtonClicked(); };
    stopButton.setColour (juce::TextButton::buttonColourId, juce::Colours::red);
    stopButton.setEnabled (false);
```

Notice in particular that we disable the **Play** and **Stop** buttons initially. The **Play** button is enabled once a valid file is loaded. We can see here that we have assigned a lambda function to the [Button::onClick](https://docs.juce.com/master/classButton.html#a30b76ab312dc7f66e67596ae20540ec2 "You can assign a lambda to this callback object to have it called when the button is clicked.") helper objects for each of these three buttons (see [Tutorial: Listeners and Broadcasters](/tutorials/tutorial_listeners_and_broadcasters/)). We also initialise our transport state in the constructor's initialiser list.

## Other initialisation

In addition to the three [TextButton](https://docs.juce.com/master/classTextButton.html "A button that uses the standard lozenge-shaped background with a line of text on it.") objects we have four other members of our `MainContentComponent` class:

```cpp
juce::AudioFormatManager formatManager;
std::unique_ptr<juce::AudioFormatReaderSource> readerSource;
juce::AudioTransportSource transportSource;
TransportState state;
```

Here we see the [AudioFormatManager](https://docs.juce.com/master/classAudioFormatManager.html "A class for keeping a list of available audio formats, and for deciding which one to use to open a gi..."), [AudioFormatReaderSource](https://docs.juce.com/master/classAudioFormatReaderSource.html "A type of AudioSource that will read from an AudioFormatReader."), and [AudioTransportSource](https://docs.juce.com/master/classAudioTransportSource.html "An AudioSource that takes a PositionableAudioSource and allows it to be played, stopped,...") classes mentioned earlier.

In the `MainContentComponent` constructor we need to initialise the [AudioFormatManager](https://docs.juce.com/master/classAudioFormatManager.html "A class for keeping a list of available audio formats, and for deciding which one to use to open a gi...") object to register a list of standard formats [1]:

```cpp
formatManager.registerBasicFormats(); // [1]
```

As a minimum this will enable the [AudioFormatManager](https://docs.juce.com/master/classAudioFormatManager.html "A class for keeping a list of available audio formats, and for deciding which one to use to open a gi...") object to create readers for the WAV and AIFF formats. Other formats may be available depending on the platform and the options enabled in the `juce_audio_formats` module within the Projucer project as shown in the following screenshot:

![](/_images/tutorial_playing_sound_files_screenshot2.png "The juce_audio_formats module options showing audio format options.")

In the `MainContentComponent` constructor we also add our `MainContentComponent` object as a listener [2] to the [AudioTransportSource](https://docs.juce.com/master/classAudioTransportSource.html "An AudioSource that takes a PositionableAudioSource and allows it to be played, stopped,...") object so that we can respond to changes in its state (for example, when it stops):

```cpp
transportSource.addChangeListener (this); // [2]
```

> [!TIP]
>The function name is `addChangeListener()` in this case, rather than simply `addListener()` as it is with many other listener classes in JUCE.

## Responding to AudioTransportSource changes

When changes in the transport are reported, the `changeListenerCallback()` function will be called. This will be called asynchronously on the message thread:

```cpp
void changeListenerCallback (juce::ChangeBroadcaster* source) override
{
    if (source == &transportSource)
    {
        if (transportSource.isPlaying())
            changeState (Playing);
        else
            changeState (Stopped);
    }
}
```

You can see that this just calls a member function `changeState()` .

## Changing states

The changing of the transport state is localised into this single function `changeState()` . This helps keep all of the logic for this functionality in one place. This function updates the `state` member and triggers any changes to other objects that need to take place when in this new state.

> [!TIP]
>More experienced readers may wish to use the [state design pattern](https://en.wikipedia.org/wiki/State_pattern) as an alternative way of structuring this code.

```cpp
void changeState (TransportState newState)
{
    if (state != newState)
    {
        state = newState;

        switch (state)
        {
            case Stopped: // [3]
                stopButton.setEnabled (false);
                playButton.setEnabled (true);
                transportSource.setPosition (0.0);
                break;

            case Starting: // [4]
                playButton.setEnabled (false);
                transportSource.start();
                break;

            case Playing: // [5]
                stopButton.setEnabled (true);
                break;

            case Stopping: // [6]
                transportSource.stop();
                break;
        }
    }
}
```

- [3]: When the transport returns to the _Stopped_ state it disables the **Stop** button, enables the **Play** button, and resets the transport position back to the start of the file.
- [4]: The _Starting_ state is triggered by the user clicking the **Play** button, this tells the [AudioTransportSource](https://docs.juce.com/master/classAudioTransportSource.html "An AudioSource that takes a PositionableAudioSource and allows it to be played, stopped,...") object to start playing. At this point we disable the **Play** button too.
- [5]: The _Playing_ state is triggered by the [AudioTransportSource](https://docs.juce.com/master/classAudioTransportSource.html "An AudioSource that takes a PositionableAudioSource and allows it to be played, stopped,...") object reporting a change via the `changeListenerCallback()` function. Here we enable the **Stop** button.
- [6]: The _Stopping_ state is triggered by the user clicking the **Stop** button, so we tell the [AudioTransportSource](https://docs.juce.com/master/classAudioTransportSource.html "An AudioSource that takes a PositionableAudioSource and allows it to be played, stopped,...") object to stop.

## Processing the audio

The audio processing in this demo project is very straightforward: we simply hand off the processing to the [AudioTransportSource](https://docs.juce.com/master/classAudioTransportSource.html "An AudioSource that takes a PositionableAudioSource and allows it to be played, stopped,...") object by passing it the [AudioSourceChannelInfo](structAudioSourceChannelInfo.html "Used by AudioSource::getNextAudioBlock().") struct that we have been passed via the [AudioAppComponent](https://docs.juce.com/master/classAudioAppComponent.html "A base class for writing audio apps that stream from the audio i/o devices.") class:

```cpp
void getNextAudioBlock (const juce::AudioSourceChannelInfo& bufferToFill) override
{
    if (readerSource.get() == nullptr)
    {
        bufferToFill.clearActiveBufferRegion();
        return;
    }

    transportSource.getNextAudioBlock (bufferToFill);
}
```

Notice that we check if there is a valid [AudioFormatReaderSource](https://docs.juce.com/master/classAudioFormatReaderSource.html "A type of AudioSource that will read from an AudioFormatReader.") object first and simply zero the output if not (using the convenient [AudioSourceChannelInfo::clearActiveBufferRegion()](structAudioSourceChannelInfo.html#a6246ca9f1f6ef86dab31fc0eeaacb7a3 "Convenient method to clear the buffer if the source is not producing any data.") function). The [AudioFormatReaderSource](https://docs.juce.com/master/classAudioFormatReaderSource.html "A type of AudioSource that will read from an AudioFormatReader.") member is stored in a std::unique_ptr object because we need to create these objects dynamically based on the user's actions. It also allows us to check for `nullptr` for invalid objects.

We also need to remember to pass the `prepareToPlay()` callback to any other [AudioSource](https://docs.juce.com/master/classAudioSource.html "Base class for objects that can produce a continuous stream of audio.") objects we are using:

```cpp
void prepareToPlay (int samplesPerBlockExpected, double sampleRate) override
{
    transportSource.prepareToPlay (samplesPerBlockExpected, sampleRate);
}
```

And the `releaseResources()` callback too:

```cpp
void releaseResources() override
{
    transportSource.releaseResources();
}
```

## Opening a file

To open a file we pop up a [FileChooser](https://docs.juce.com/master/classFileChooser.html "Creates a dialog box to choose a file or directory to load or save.") object in response to the **Open...** button being clicked:

```cpp
void openButtonClicked()
{
    chooser = std::make_unique<juce::FileChooser> ("Select a Wave file to play...",
        juce::File {},
        "*.wav"); // [7]
    auto chooserFlags = juce::FileBrowserComponent::openMode
                        | juce::FileBrowserComponent::canSelectFiles;

    chooser->launchAsync (chooserFlags, [this] (const juce::FileChooser& fc) // [8]
        {
            auto file = fc.getResult();

            if (file != juce::File {}) // [9]
            {
                auto* reader = formatManager.createReaderFor (file); // [10]

                if (reader != nullptr)
                {
                    auto newSource = std::make_unique<juce::AudioFormatReaderSource> (reader, true); // [11]
                    transportSource.setSource (newSource.get(), 0, nullptr, reader->sampleRate); // [12]
                    playButton.setEnabled (true); // [13]
                    readerSource.reset (newSource.release()); // [14]
                }
            }
        });
}
```

- [7]: Create the [FileChooser](https://docs.juce.com/master/classFileChooser.html "Creates a dialog box to choose a file or directory to load or save.") object with a short message and allow the user to select only `.wav` files.
- [8]: Pop up the [FileChooser](https://docs.juce.com/master/classFileChooser.html "Creates a dialog box to choose a file or directory to load or save.") object.
- [9]: If `if()` will succeed if the user actually selects a file (rather than cancelling)
- [10]: The [AudioFormatManager::createReaderFor()](https://docs.juce.com/master/classAudioFormatManager.html#a4eed7ecbb353d3d85154182de5057221 "Searches through the known formats to try to create a suitable reader for this file.") function is used attempt to create a reader for this particular file. This will return the `nullptr` value if it fails (for example the file is not an audio format the [AudioFormatManager](https://docs.juce.com/master/classAudioFormatManager.html "A class for keeping a list of available audio formats, and for deciding which one to use to open a gi...") object can handle).
- [11]: We create a new [AudioFormatReaderSource](https://docs.juce.com/master/classAudioFormatReaderSource.html "A type of AudioSource that will read from an AudioFormatReader.") object using the reader we just created. The second argument `true` indicates that we want the [AudioFormatReaderSource](https://docs.juce.com/master/classAudioFormatReaderSource.html "A type of AudioSource that will read from an AudioFormatReader.") object to manage the [AudioFormatReader](https://docs.juce.com/master/classAudioFormatReader.html "Reads samples from an audio file stream.") object and delete it when it is no longer needed. We store the [AudioFormatReaderSource](https://docs.juce.com/master/classAudioFormatReaderSource.html "A type of AudioSource that will read from an AudioFormatReader.") object in a temporary std::unique_ptr object to avoid deleting a previously allocated [AudioFormatReaderSource](https://docs.juce.com/master/classAudioFormatReaderSource.html "A type of AudioSource that will read from an AudioFormatReader.") prematurely on subsequent commands to open a file.
- [12]: The [AudioFormatReaderSource](https://docs.juce.com/master/classAudioFormatReaderSource.html "A type of AudioSource that will read from an AudioFormatReader.") object is connected to our [AudioTransportSource](https://docs.juce.com/master/classAudioTransportSource.html "An AudioSource that takes a PositionableAudioSource and allows it to be played, stopped,...") object that is being used in our `getNextAudioBlock()` function. In case the sample rate of the file doesn\'t match the hardware sample rate we pass this in as the fourth argument, which we obtain from the [AudioFormatReader](https://docs.juce.com/master/classAudioFormatReader.html "Reads samples from an audio file stream.") object. See [Notes](#tutorial_playing_sound_files_notes) for more information on the second and third arguments. The [AudioTransportSource](https://docs.juce.com/master/classAudioTransportSource.html "An AudioSource that takes a PositionableAudioSource and allows it to be played, stopped,...") source will handle any sample rate conversion that is necessary.
- [13]: The **Play** button is enabled so that the user can click on it.
- [14]: Since the [AudioTransportSource](https://docs.juce.com/master/classAudioTransportSource.html "An AudioSource that takes a PositionableAudioSource and allows it to be played, stopped,...") should now be using our newly allocated [AudioFormatReaderSource](https://docs.juce.com/master/classAudioFormatReaderSource.html "A type of AudioSource that will read from an AudioFormatReader.") object we can safely store the [AudioFormatReaderSource](https://docs.juce.com/master/classAudioFormatReaderSource.html "A type of AudioSource that will read from an AudioFormatReader.") object in our `readerSource` member. (As mentioned in [Processing the audio](#tutorial_playing_sound_files_processing_the_audio) above.) To do this we must transfer ownership from the local `newSource` variable by using std::unique_ptr::release().

> [!TIP]
>Storing the newly allocated [AudioFormatReaderSource](https://docs.juce.com/master/classAudioFormatReaderSource.html "A type of AudioSource that will read from an AudioFormatReader.") object in a temporary std::unique_ptr object has the added benefit of being exception-safe. An exception could be thrown during the function call [AudioTransportSource::setSource()](https://docs.juce.com/master/classAudioTransportSource.html#a20a6a91c4505e77207cadef7ddc4dc5c "Sets the reader that is being used as the input source."), in which case the std::unique_ptr object will delete the [AudioFormatReaderSource](https://docs.juce.com/master/classAudioFormatReaderSource.html "A type of AudioSource that will read from an AudioFormatReader.") object that is no longer needed. If a raw pointer had been used at this point to store [AudioFormatReaderSource](https://docs.juce.com/master/classAudioFormatReaderSource.html "A type of AudioSource that will read from an AudioFormatReader.") object then there could be a memory leak since the pointer would be left dangling if the exception is thrown.

## Playing and stopping the file

Since we have already set up the code to actually play the file, we need only call our `changeState()` function with the appropriate argument to play the file. When the **Play** button is clicked, we do the following:

```cpp
void playButtonClicked()
{
    changeState (Starting);
}
```

Stopping the file is similarly straightforward, when the the **Stop** button is clicked:

```cpp
void stopButtonClicked()
{
    changeState (Stopping);
}
```

> [!NOTE]
> Exercise: Change the third (`filePatternsAllowed` ) argument when creating the [FileChooser](https://docs.juce.com/master/classFileChooser.html "Creates a dialog box to choose a file or directory to load or save.") object to allow the application to load AIFF files too. The file patterns can be separated by a semicolon so this should be `"*.wav;*.aif;*.aiff"` to allow for the two common file extensions for this format.

# Adding pause functionality

We will now walk through some steps to add a _pause_ functionality to the application. Here we will make the **Play** button become a **Pause** button while the file is playing (instead of just disabling it). We will also make the **Stop** button become a **Return to zero** button while the sound file is paused.

First of all we need to add two states _Pausing_ and _Paused_ to our `TransportState` enum:

```cpp
enum TransportState {
    Stopped,
    Starting,
    Playing,
    Pausing,
    Paused,
    Stopping
};
```

Our `changeState()` function needs to handle the two new states and the code for the other states needs to be updated too:

```cpp
void changeState (TransportState newState)
{
    if (state != newState)
    {
        state = newState;

        switch (state)
        {
            case Stopped:
                playButton.setButtonText ("Play");
                stopButton.setButtonText ("Stop");
                stopButton.setEnabled (false);
                transportSource.setPosition (0.0);
                break;

            case Starting:
                transportSource.start();
                break;

            case Playing:
                playButton.setButtonText ("Pause");
                stopButton.setButtonText ("Stop");
                stopButton.setEnabled (true);
                break;

            case Pausing:
                transportSource.stop();
                break;

            case Paused:
                playButton.setButtonText ("Resume");
                stopButton.setButtonText ("Return to Zero");
                break;

            case Stopping:
                transportSource.stop();
                break;
        }
    }
}
```

We enable and disable the buttons appropriately, and update the button text correctly in each state.

Notice that we actually stop the transport when asked to pause in the _Pausing_ state. In the `changeListenerCallback()` function, we need to change the logic to move to the correct state depending on whether a pause or stop request was made:

```cpp
void changeListenerCallback (juce::ChangeBroadcaster* source) override
{
    if (source == &transportSource)
    {
        if (transportSource.isPlaying())
            changeState (Playing);
        else if ((state == Stopping) || (state == Playing))
            changeState (Stopped);
        else if (Pausing == state)
            changeState (Paused);
    }
}
```

We need to change the code when the **Play** button is clicked:

```cpp
void playButtonClicked()
{
    if ((state == Stopped) || (state == Paused))
        changeState (Starting);
    else if (state == Playing)
        changeState (Pausing);
}
```

And when the **Stop** button is clicked:

```cpp
void stopButtonClicked()
{
    if (state == Paused)
        changeState (Stopped);
    else
        changeState (Stopping);
}
```

And that's it: you should be able to build and run the application now.

> [!TIP]
>The source code for this modified version of the application can be found in the `PlayingSoundFilesTutorial_02.h` file of the demo project.

<!-- -->

> [!NOTE]
> Exercise: Add a [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") object to the interface that displays the current time position of the [AudioTransportSource](https://docs.juce.com/master/classAudioTransportSource.html "An AudioSource that takes a PositionableAudioSource and allows it to be played, stopped,...") object. You can use the [AudioTransportSource::getCurrentPosition()](https://docs.juce.com/master/classAudioTransportSource.html#a1984ac3a6eafb12d5312c3eab5615810 "Returns the position that the next data block will be read from.") function to obtain this position. You will also need to make the `MainContentComponent` class inherit from the [Timer](https://docs.juce.com/master/classTimer.html "Makes repeated callbacks to a virtual method at a specified time interval.") class and perform periodic updates in your `timerCallback()` function to update the label. You could even use the [RelativeTime](https://docs.juce.com/master/classRelativeTime.html "A relative measure of time.") class to convert the raw time in seconds to a more useful format in minutes, seconds, and milliseconds.

<!-- -->

> [!TIP]
>The source code for this exercise can be found in the `PlayingSoundFilesTutorial_03.h` file of the demo project.

# Summary

In this tutorial we have introduced the reading and playing of sound files. In particular we have covered the following things:

- Using the [AudioFormatManager](https://docs.juce.com/master/classAudioFormatManager.html "A class for keeping a list of available audio formats, and for deciding which one to use to open a gi...") class to create [AudioFormatReader](https://docs.juce.com/master/classAudioFormatReader.html "Reads samples from an audio file stream.") objects for common audio formats.
- Using various [AudioSource](https://docs.juce.com/master/classAudioSource.html "Base class for objects that can produce a continuous stream of audio.") classes to create and connecting them together. In particular:
  - the [AudioFormatReaderSource](https://docs.juce.com/master/classAudioFormatReaderSource.html "A type of AudioSource that will read from an AudioFormatReader.") class for reading and playing audio from [AudioFormatReader](https://docs.juce.com/master/classAudioFormatReader.html "Reads samples from an audio file stream.") objects; and
  - the [AudioTransportSource](https://docs.juce.com/master/classAudioTransportSource.html "An AudioSource that takes a PositionableAudioSource and allows it to be played, stopped,...") class for controlling the playback of an [AudioFormatReaderSource](https://docs.juce.com/master/classAudioFormatReaderSource.html "A type of AudioSource that will read from an AudioFormatReader.") object.
- We also looked one way of managing the state of audio file playback.

# Notes

The second and third arguments to the [AudioTransportSource::setSource()](https://docs.juce.com/master/classAudioTransportSource.html#a20a6a91c4505e77207cadef7ddc4dc5c "Sets the reader that is being used as the input source.") function allow you to control look ahead buffering on a background thread. The second argument is the buffer size to use and the third argument is a pointer to a [TimeSliceThread](https://docs.juce.com/master/classTimeSliceThread.html "A thread that keeps a list of clients, and calls each one in turn, giving them all a chance to run so...") object, which is used for the background processing. In this example we use a zero buffer size and a `nullptr` value for the thread object, which is the default.

# See also

- [Tutorial: Listeners and Broadcasters](/tutorials/tutorial_listeners_and_broadcasters/)
- [Tutorial: Build a white noise generator](/tutorials/tutorial_simple_synth_noise/)
- [Tutorial: Build a sine wave synthesiser](/tutorials/tutorial_sine_synth/)
- [Tutorial: Draw audio waveforms](/tutorials/tutorial_audio_thumbnail/)
