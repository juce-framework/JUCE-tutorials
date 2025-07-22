---
title: Draw audio waveforms
---
# Tutorial: Draw audio waveforms

This tutorial introduces the display of audio waveforms using the [AudioThumbnail](https://docs.juce.com/master/classAudioThumbnail.html "Makes it easy to quickly draw scaled views of the waveform shape of an audio file.") class. This provides an easy way of drawing any number of waveforms within your audio applications.

**LEVEL:** Intermediate<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**CLASSES:** [AudioThumbnail](https://docs.juce.com/master/classAudioThumbnail.html "Makes it easy to quickly draw scaled views of the waveform shape of an audio file."), [AudioThumbnailCache](https://docs.juce.com/master/classAudioThumbnailCache.html "An instance of this class is used to manage multiple AudioThumbnail objects."), [AudioFormatReader](https://docs.juce.com/master/classAudioFormatReader.html "Reads samples from an audio file stream."), [ChangeListener](https://docs.juce.com/master/classChangeListener.html "Receives change event callbacks that are sent out by a ChangeBroadcaster.")

# Getting started

> [!TIP]
>This tutorial leads on from [Tutorial: Build an audio player](/tutorials/tutorial_playing_sound_files/), which you should have read and understood first. It also assumes that you are familiar with the [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") class and the Component::paint() function for performing drawing within a component (see [Tutorial: The Graphics class](/tutorials/tutorial_graphics_class/)).

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/AudioThumbnailTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/AudioThumbnailTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project presents three buttons in the same way as [Tutorial: Build an audio player](/tutorials/tutorial_playing_sound_files/) (for opening, playing, and stopping a sound file).

There is also a rectangular area where the waveform from the sound file can be drawn. In its default state (with no sound file loaded) the application looks like this:

![](/_images/tutorial_audio_thumbnail_screenshot1.png "The demo project showing its initial state")

Once a sound file is loaded, the application looks like this:

![](/_images/tutorial_audio_thumbnail_screenshot2.png "The demo project showing a file opened and displayed using the AudioThumbnail class")

Drawing an audio waveform, especially for long files, generally involves storing a low resolution version of the audio data in a format that makes drawing the waveform efficient and also clear to the user. The [AudioThumbnail](https://docs.juce.com/master/classAudioThumbnail.html "Makes it easy to quickly draw scaled views of the waveform shape of an audio file.") class handles this low resolution version for you and it is created and updated when needed.

# Setting up the AudioThumbnail

The first important point is that the [AudioThumbnail](https://docs.juce.com/master/classAudioThumbnail.html "Makes it easy to quickly draw scaled views of the waveform shape of an audio file.") class **is not** a subclass of the [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") class. The [AudioThumbnail](https://docs.juce.com/master/classAudioThumbnail.html "Makes it easy to quickly draw scaled views of the waveform shape of an audio file.") class is used to perform the drawing of the audio waveform within the `paint()` function of another [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") object. The code below shows how to add this functionality based on the demo project from [Tutorial: Build an audio player](/tutorials/tutorial_playing_sound_files/).

## Additional objects

In our `MainContentComponent` class we need to add two members: an [AudioThumbnailCache](https://docs.juce.com/master/classAudioThumbnailCache.html "An instance of this class is used to manage multiple AudioThumbnail objects.") object and an [AudioThumbnail](https://docs.juce.com/master/classAudioThumbnail.html "Makes it easy to quickly draw scaled views of the waveform shape of an audio file.") object. The [AudioThumbnailCache](https://docs.juce.com/master/classAudioThumbnailCache.html "An instance of this class is used to manage multiple AudioThumbnail objects.") class is used to cache the necessary low resolution version of one or more audio files. This means, for example, if we close a file, open a new file, then return to open the first file, the [AudioThumbnailCache](https://docs.juce.com/master/classAudioThumbnailCache.html "An instance of this class is used to manage multiple AudioThumbnail objects.") will still contain the low resolution version of the first file and won\'t need to rescan and recalculate the data. Another useful feature is that [AudioThumbnailCache](https://docs.juce.com/master/classAudioThumbnailCache.html "An instance of this class is used to manage multiple AudioThumbnail objects.") objects can be shared between different instances of the [AudioThumbnail](https://docs.juce.com/master/classAudioThumbnail.html "Makes it easy to quickly draw scaled views of the waveform shape of an audio file.") class

```cpp
juce::TextButton openButton;
juce::TextButton playButton;
juce::TextButton stopButton;

std::unique_ptr<juce::FileChooser> chooser;

juce::AudioFormatManager formatManager; // [3]
std::unique_ptr<juce::AudioFormatReaderSource> readerSource;
juce::AudioTransportSource transportSource;
TransportState state;
juce::AudioThumbnailCache thumbnailCache; // [1]
juce::AudioThumbnail thumbnail; // [2]

JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR (MainContentComponent)
};
```

If statically allocated objects like this are used, it is important that the [AudioThumbnailCache](https://docs.juce.com/master/classAudioThumbnailCache.html "An instance of this class is used to manage multiple AudioThumbnail objects.") object [1] is listed before the [AudioThumbnail](https://docs.juce.com/master/classAudioThumbnail.html "Makes it easy to quickly draw scaled views of the waveform shape of an audio file.") object [2] since it is passed as an argument to the [AudioThumbnail](https://docs.juce.com/master/classAudioThumbnail.html "Makes it easy to quickly draw scaled views of the waveform shape of an audio file.") [constructor.](https://docs.juce.com/master/classAudioThumbnail.html#abfa7516538eb0518cddeec4c71cf329d) It is also important that the [AudioFormatManager](https://docs.juce.com/master/classAudioFormatManager.html "A class for keeping a list of available audio formats, and for deciding which one to use to open a gi...") object [3] is listed before the [AudioThumbnail](https://docs.juce.com/master/classAudioThumbnail.html "Makes it easy to quickly draw scaled views of the waveform shape of an audio file.") object for the same reason.

## Initialising the objects

In the initialiser list for the `MainContentComponent` constructor we set up these objects:

```cpp
MainContentComponent()
    : state (Stopped),
      thumbnailCache (5), // [4]
      thumbnail (512, formatManager, thumbnailCache) // [5]
{
```

- [4]: The [AudioThumbnailCache](https://docs.juce.com/master/classAudioThumbnailCache.html "An instance of this class is used to manage multiple AudioThumbnail objects.") objects must be constructed with the number of thumbnails to store.
- [5]: The [AudioThumbnail](https://docs.juce.com/master/classAudioThumbnail.html "Makes it easy to quickly draw scaled views of the waveform shape of an audio file.") object itself needs to be constructed by telling it how many source samples will be used to create a single thumbnail sample. This governs the resolution of the low resolution version. The other two arguments are the [AudioFormatManager](https://docs.juce.com/master/classAudioFormatManager.html "A class for keeping a list of available audio formats, and for deciding which one to use to open a gi...") and [AudioThumbnailCache](https://docs.juce.com/master/classAudioThumbnailCache.html "An instance of this class is used to manage multiple AudioThumbnail objects.") objects, as discussed above.

The [AudioThumbnail](https://docs.juce.com/master/classAudioThumbnail.html "Makes it easy to quickly draw scaled views of the waveform shape of an audio file.") class is also a type of [ChangeBroadcaster](https://docs.juce.com/master/classChangeBroadcaster.html "Holds a list of ChangeListeners, and sends messages to them when instructed.") class. We can register as a listener for changes [6](in our `MainContentComponent` constructor). These changes will be when the [AudioThumbnail](https://docs.juce.com/master/classAudioThumbnail.html "Makes it easy to quickly draw scaled views of the waveform shape of an audio file.") object has changed such that we need to update our drawing of the waveform.

```cpp
thumbnail.addChangeListener (this); // [6]
```

## Responding to changes

In our `changeListenerCallback()` function we need to determine whether the change is being broadcasted from the [AudioTransportSource](https://docs.juce.com/master/classAudioTransportSource.html "An AudioSource that takes a PositionableAudioSource and allows it to be played, stopped,...") object or the [AudioThumbnail](https://docs.juce.com/master/classAudioThumbnail.html "Makes it easy to quickly draw scaled views of the waveform shape of an audio file.") object:

```cpp
void changeListenerCallback (juce::ChangeBroadcaster* source) override
{
    if (source == &transportSource)
        transportSourceChanged();
    if (source == &thumbnail)
        thumbnailChanged();
}
```

The `transportSourceChanged()` function just contains our original code for responding to changes in the [AudioTransportSource](https://docs.juce.com/master/classAudioTransportSource.html "An AudioSource that takes a PositionableAudioSource and allows it to be played, stopped,...") object:

```cpp
void transportSourceChanged()
{
    changeState (transportSource.isPlaying() ? Playing : Stopped);
}
```

If it is the [AudioThumbnail](https://docs.juce.com/master/classAudioThumbnail.html "Makes it easy to quickly draw scaled views of the waveform shape of an audio file.") object that has changed, we call the Component::repaint() function. This will cause our `paint()` function to be called during the next screen drawing operation:

```cpp
void thumbnailChanged()
{
    repaint();
}
```

## Opening the file

When we open the sound file successfully we also need to pass the file to the [AudioThumbnail](https://docs.juce.com/master/classAudioThumbnail.html "Makes it easy to quickly draw scaled views of the waveform shape of an audio file.") object [7] within a [FileInputSource](https://docs.juce.com/master/classFileInputSource.html "A type of InputSource that represents a normal file.") object.

```cpp
void openButtonClicked()
{
    chooser = std::make_unique<juce::FileChooser> ("Select a Wave file to play...",
        juce::File {},
        "*.wav");
    auto chooserFlags = juce::FileBrowserComponent::openMode
                        | juce::FileBrowserComponent::canSelectFiles;

    chooser->launchAsync (chooserFlags, [this] (const juce::FileChooser& fc) {
        auto file = fc.getResult();

        if (file != juce::File {})
        {
            auto* reader = formatManager.createReaderFor (file);

            if (reader != nullptr)
            {
                auto newSource = std::make_unique<juce::AudioFormatReaderSource> (reader, true);
                transportSource.setSource (newSource.get(), 0, nullptr, reader->sampleRate);
                playButton.setEnabled (true);
                thumbnail.setSource (new juce::FileInputSource (file)); // [7]
                readerSource.reset (newSource.release());
            }
        }
    });
}
```

## Performing the drawing

In our `paint()` function, first we calculate the rectangle into which we will draw. Then we check how many channels the [AudioThumbnail](https://docs.juce.com/master/classAudioThumbnail.html "Makes it easy to quickly draw scaled views of the waveform shape of an audio file.") object contains, which tells us whether we have a file loaded or not:

```cpp
void paint (juce::Graphics& g) override
{
    juce::Rectangle<int> thumbnailBounds (10, 100, getWidth() - 20, getHeight() - 120);

    if (thumbnail.getNumChannels() == 0)
        paintIfNoFileLoaded (g, thumbnailBounds);
    else
        paintIfFileLoaded (g, thumbnailBounds);
}
```

If we have no file loaded then we display the message **No File Loaded** by passing our `paintIfNoFileLoaded()` function the [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") object and the bounds rectangle:

```cpp
void paintIfNoFileLoaded (juce::Graphics& g, const juce::Rectangle<int>& thumbnailBounds)
{
    g.setColour (juce::Colours::darkgrey);
    g.fillRect (thumbnailBounds);
    g.setColour (juce::Colours::white);
    g.drawFittedText ("No File Loaded", thumbnailBounds, juce::Justification::centred, 1);
}
```

The important part is next. If we _do_ have a file loaded we can draw the waveform:

```cpp
void paintIfFileLoaded (juce::Graphics& g, const juce::Rectangle<int>& thumbnailBounds)
{
    g.setColour (juce::Colours::white);
    g.fillRect (thumbnailBounds);

    g.setColour (juce::Colours::red); // [8]

    thumbnail.drawChannels (g, // [9]
        thumbnailBounds,
        0.0, // start time
        thumbnail.getTotalLength(), // end time
        1.0f); // vertical zoom
}
```

- [8]: Notice that we set the current colour for the [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") object. This will govern the colour used by the [AudioThumbnail](https://docs.juce.com/master/classAudioThumbnail.html "Makes it easy to quickly draw scaled views of the waveform shape of an audio file.") object to draw the waveform.
- [9]: We call the [AudioThumbnail::drawChannels()](https://docs.juce.com/master/classAudioThumbnail.html#a8fc354b03e88e66771ef3b9647fdd9fa "Draws the waveforms for all channels in the thumbnail.") function passing it the [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") object with which to draw, the rectangle into which it should draw, the start and end times (in seconds), and the vertical zoom factor. Here we use the [AudioThumbnail::getTotalLength()](https://docs.juce.com/master/classAudioThumbnail.html#a76892e0159356999dfe317fa1c5e044b "Returns the length of the audio file, in seconds.") function to get the file duration so that we can draw the whole file. (We could have obtained the length from our [AudioTransportSource](https://docs.juce.com/master/classAudioTransportSource.html "An AudioSource that takes a PositionableAudioSource and allows it to be played, stopped,...") object using the [AudioTransportSource::getLengthInSeconds()](https://docs.juce.com/master/classAudioTransportSource.html#aa2896785ea5864f2cd961676bfbe21b7 "Returns the stream's length in seconds.") function to get the same result.)

This covers all the basic points for using an [AudioThumbnail](https://docs.juce.com/master/classAudioThumbnail.html "Makes it easy to quickly draw scaled views of the waveform shape of an audio file.") object.

> [!NOTE]
> Exercise: In practice you will commonly want to display only certain regions of the sound files. It should be clear from the [AudioThumbnail::drawChannels()](https://docs.juce.com/master/classAudioThumbnail.html#a8fc354b03e88e66771ef3b9647fdd9fa "Draws the waveforms for all channels in the thumbnail.") function how simple this is to implement using JUCE. Try modifying the code to display only a specific region of the file.

# Adding a time position marker

In this section we will walk you through adding a vertical line to the display that will show the current time position of the file playback.

## Adding a timer

First of all we need to add the [Timer](https://docs.juce.com/master/classTimer.html "Makes repeated callbacks to a virtual method at a specified time interval.") class to our list of base classes [10]:

```cpp
class MainContentComponent : public juce::AudioAppComponent,
                             public juce::ChangeListener,
                             private juce::Timer // [10]
{
public:
```

Then we need to make the timer callback repaint our component. Make sure this code is added to the `private` section as you will notice we inherited privately from the [Timer](https://docs.juce.com/master/classTimer.html "Makes repeated callbacks to a virtual method at a specified time interval.") class:

```cpp
void timerCallback() override
{
    repaint();
}
```

In the `MainContentComponent` constructor we need to start the timer [11]--- every 40ms should be sufficient:

```cpp
startTimer (40); // [11]
}
```

<p class="callout-note"> Exercise:
In fact you could delay starting the timer, by starting it once the file is successfully opened.
</p

## Drawing the position line

Finally, to draw the line we need to calculate the position of the line and draw it _after_ drawing the thumbnail:

```cpp
void paintIfFileLoaded (juce::Graphics& g, const juce::Rectangle<int>& thumbnailBounds)
{
    g.setColour (juce::Colours::white);
    g.fillRect (thumbnailBounds);

    g.setColour (juce::Colours::red);

    auto audioLength = (float) thumbnail.getTotalLength(); // [12]
    thumbnail.drawChannels (g, thumbnailBounds, 0.0, audioLength, 1.0f);

    g.setColour (juce::Colours::green);

    auto audioPosition = (float) transportSource.getCurrentPosition();
    auto drawPosition = (audioPosition / audioLength) * (float) thumbnailBounds.getWidth()
                        + (float) thumbnailBounds.getX(); // [13]
    g.drawLine (drawPosition, (float) thumbnailBounds.getY(), drawPosition, (float) thumbnailBounds.getBottom(), 2.0f); // [14]
}
```

- [12]: Store the length of the file in a variable since we need to use this value twice.
- [13]: The position is calculated as a proportion of the total length of the audio file. The position to draw the line needs to be based on the same proportion of the width of the rectangle that the thumbnail is drawn within. We need to offset the drawing position based on the x-coordinate of the rectangle.
- [14]: Here we draw a line that is 2 pixels wide between the top (y) and bottom of the rectangle.

And that's it: you should be able to build and run the application now.

> [!TIP]
>The source code for this modified version of the application can be found in the `AudioThumbnailTutorial_02.h` file of the demo project.

> [!WARNING]
> The problem with this example is that we force the component to be repainted every 40ms. While this may be acceptable for a simple example you will probably hit performance problems in more complex cases. Take a look at the exercise, below, for more on this.


> [!NOTE]
> Exercise: Separate the drawing into a separate child components (see [Tutorial: Parent and child components](/tutorials/tutorial_component_parents_children/)). You should have three components:

    -   A component that draws the audio waveform.
    -   A component that draws the playback position as a vertical line.
    -   The main parent component that contains these two child components (laid on top of each other).

    Not only will this make the code easier to follow but, if done correctly, it will be much more efficient since we can avoid redrawing the waveform every frame. You could also add functionality to change the playback position if the user clicks on the waveform.

> [!TIP]
>The source code for possible implementations of this exercise can be found in the `AudioThumbnailTutorial_03.h` and `AudioThumbnailTutorial_04.h` files of the demo project.

# Summary

This tutorial has introduced the [AudioThumbnail](https://docs.juce.com/master/classAudioThumbnail.html "Makes it easy to quickly draw scaled views of the waveform shape of an audio file.") class and how it can be integrated into an audio application. In particular we have covered:

- Initialising [AudioThumbnail](https://docs.juce.com/master/classAudioThumbnail.html "Makes it easy to quickly draw scaled views of the waveform shape of an audio file.") and [AudioThumbnailCache](https://docs.juce.com/master/classAudioThumbnailCache.html "An instance of this class is used to manage multiple AudioThumbnail objects.") objects.
- Using the [AudioThumbnail](https://docs.juce.com/master/classAudioThumbnail.html "Makes it easy to quickly draw scaled views of the waveform shape of an audio file.") class within a component.
- Structuring components so that content that is complex to draw isn\'t forced to be redrawn unnecessarily.

# See also

- [Tutorial: The Graphics class](/tutorials/tutorial_graphics_class/)
- [Tutorial: Parent and child components](/tutorials/tutorial_component_parents_children/)
- [Tutorial: Build an audio player](/tutorials/tutorial_playing_sound_files/)
