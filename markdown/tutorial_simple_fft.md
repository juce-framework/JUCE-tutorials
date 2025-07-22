---
title: The fast Fourier transform
---
# Tutorial: The fast Fourier transform

Learn how to display incoming audio data as a spectrogram by using the FFT class of the DSP module. Understand the benefits of using a Fast Fourier Transform.

**LEVEL:** Intermediate<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**CLASSES:** [dsp::FFT](https://docs.juce.com/master/classdsp_1_1FFT.html "Performs a fast fourier transform."), [Image](https://docs.juce.com/master/classImage.html "Holds a fixed-size bitmap."), [Colour](https://docs.juce.com/master/classColour.html "Represents a colour, also including a transparency value."), [FloatVectorOperations](https://docs.juce.com/master/classFloatVectorOperations.html "A collection of simple vector operations on arrays of floating point numbers, accelerated with SIMD i...")

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/SimpleFFTTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/SimpleFFTTutorial.zip) . Unzip the project and open the first header file in the Projucer.

> [!TIP]
>If your operating system requires you to request permission to access the microphone (currently iOS, Android and macOS Mojave) then you will need to set the corresponding option under the relevant exporter in the Projucer and resave the project.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

When completed, the demo project will display the incoming audio data as a three-dimensional spectrogram in the time (x-axis), frequency (y-axis) and amplitude (colour) domains. The values displayed on the screen will be updated 60 times a second and the window at any time frame may look something like this:

![](/_images/tutorial_simple_fft_screenshot1.png "The demo project window")

> [!TIP]
>The code presented here is broadly similar to the **SimpleFFTExample** from the JUCE Examples.

# The Fast Fourier Transform

A time or space domain signal can be converted to the frequency domain by using a transformation formula called the Fourier transform. A common efficient implementation of this transformation function is the Fast Fourier Transform or FFT, which is included in the JUCE DSP module and which we will use in this tutorial.

The FFT allows us to decompose an audio signal into its frequencies and represent the magnitude and phase information for each of these frequencies. Using its inverse function, we can revert the signal into its original domain thus making it really useful to process individual frequency components such as for filtering.

Since this tutorial only deals with displaying the audio data without actual processing for output, we focus on the forward FFT rather than the inverse FFT.

# Processing Audio Data

Currently our application does not display nor process any incoming audio signals so let's start by implementing the FFT.

## FFT Initialisation

In the `SpectrogramComponent` class, start by defining some useful constants for the FFT implementation:

```cpp
static constexpr auto fftOrder = 10; // [1]
static constexpr auto fftSize = 1 << fftOrder; // [2]

private:
```

- [1]: The FFT order designates the size of the FFT window and the number of points on which it will operate corresponds to 2 to the power of the order. In this case, let's use an order of 10 which will produce an FFT with 2 \^ 10 = 1024 points.
- [2]: To calculate the corresponding FFT size, we use the left bit shift operator which produces 1024 as binary number 10000000000.

Next, declare private member variables required for the FFT implementation as shown below:

```cpp
juce::dsp::FFT forwardFFT; // [3]
juce::Image spectrogramImage;

std::array<float, fftSize> fifo; // [4]
std::array<float, fftSize * 2> fftData; // [5]
int fifoIndex = 0; // [6]
bool nextFFTBlockReady = false; // [7]

JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR (SpectrogramComponent)
};
```

- [3]: Declare a [dsp::FFT](https://docs.juce.com/master/classdsp_1_1FFT.html "Performs a fast fourier transform.") object to perform the forward FFT on.
- [4]: The fifo float array of size 1024 will contain our incoming audio data in samples.
- [5]: The fftData float array of size 2048 will contain the results of our FFT calculations.
- [6]: This temporary index keeps count of the amount of samples in the fifo.
- [7]: This temporary boolean tells us whether the next FFT block is ready to be rendered.

Now let's initialise these variables in the member initialisation list of our constructor like so:

```cpp
SpectrogramComponent()
    : forwardFFT (fftOrder),
      spectrogramImage (juce::Image::RGB, 512, 512, true)
{
```

The FFT object has to be explicitly initialised with the correct order at this point.

In the overriden `getNextAudioBlock()` function, we simply push all the samples contained in our current audio buffer block to the fifo to be processed at a later time:

```cpp
void getNextAudioBlock (const juce::AudioSourceChannelInfo& bufferToFill) override
{
    if (bufferToFill.buffer->getNumChannels() > 0)
    {
        auto* channelData = bufferToFill.buffer->getReadPointer (0, bufferToFill.startSample);

        for (auto i = 0; i < bufferToFill.numSamples; ++i)
            pushNextSampleIntoFifo (channelData[i]);
    }
}
```

To push the sample into the fifo, implement the `pushNextSampleIntoFifo()` function as described below:

```cpp
void pushNextSampleIntoFifo (float sample) noexcept
{
    // if the fifo contains enough data, set a flag to say
    // that the next line should now be rendered..
    if (fifoIndex == fftSize) // [8]
    {
        if (!nextFFTBlockReady) // [9]
        {
            std::fill (fftData.begin(), fftData.end(), 0.0f);
            std::copy (fifo.begin(), fifo.end(), fftData.begin());
            nextFFTBlockReady = true;
        }

        fifoIndex = 0;
    }

    fifo[(size_t) fifoIndex++] = sample; // [9]
}
```

- [8]: If the fifo contains enough data in this case 1024 samples, we are ready to copy the data to the fftData array for it to be processed by the FFT. We also set a flag to say that the next line should now be rendered and always reset the index to 0 to start filling the fifo again.
- [9]: Every time this function gets called, a sample is stored in the fifo and the index is incremented.

The fifo data now occupies the first half of the FFT input array and is ready to be processed and displayed.

## Displaying the Spectrogram

In the `drawNextLineOfSpectrogram()` function, insert the pixel drawing implementation as explained below:

```cpp
void drawNextLineOfSpectrogram()
{
    auto rightHandEdge = spectrogramImage.getWidth() - 1;
    auto imageHeight = spectrogramImage.getHeight();

    // first, shuffle our image leftwards by 1 pixel..
    spectrogramImage.moveImageSection (0, 0, 1, 0, rightHandEdge, imageHeight); // [1]

    // then render our FFT data..
    forwardFFT.performFrequencyOnlyForwardTransform (fftData.data()); // [2]

    // find the range of values produced, so we can scale our rendering to
    // show up the detail clearly
    auto maxLevel = juce::FloatVectorOperations::findMinAndMax (fftData.data(), fftSize / 2); // [3]

    juce::Image::BitmapData bitmap { spectrogramImage, rightHandEdge, 0, 1, imageHeight, juce::Image::BitmapData::writeOnly }; // [4]

    for (auto y = 1; y < imageHeight; ++y) // [5]
    {
        auto skewedProportionY = 1.0f - std::exp (std::log ((float) y / (float) imageHeight) * 0.2f);
        auto fftDataIndex = (size_t) juce::jlimit (0, fftSize / 2, (int) (skewedProportionY * fftSize / 2));
        auto level = juce::jmap (fftData[fftDataIndex], 0.0f, juce::jmax (maxLevel.getEnd(), 1e-5f), 0.0f, 1.0f);

        bitmap.setPixelColour (0, y, juce::Colour::fromHSV (level, 1.0f, level, 1.0f)); // [6]
    }
}
```

- [1]: First, shuffle the image leftwards by 1 pixel using the `moveImageSection()` function on the [Image](https://docs.juce.com/master/classImage.html "Holds a fixed-size bitmap.") object. Specify the image section as the whole width minus one pixel and the whole height.
- [2]: Then, render the FFT data using the `performFrequencyOnlyForwardTransform()` function on the FFT object with the fftData array as an argument.
- [3]: Find the range of values produced, so that we can scale our rendering to show up the detail clearly. We can do so using the FloatVectorOperations::findMinAndMax() function.
- [4]: Create a BitmapData instance that refers to the rightmost column of pixels in the spectrogram image. When reading or writing to multiple pixels in an image, a BitmapData instance can be used to buffer pixel values internally, reading or writing them all at once. This approach is normally faster than using the member functions of [Image](https://docs.juce.com/master/classImage.html "Holds a fixed-size bitmap.") to access individual pixels.
- [5]: Now in the for loop for every pixel in the spectrogram height, calculate the level proportionally to the sample set. To do this, we first need to skew the y-axis to use a logarithmic scale to better represent our frequencies. We can then feed this scaling factor to retrieve the correct array index and use the amplitude value to map it to a range between `0.0 .. 1.0` .
- [6]: Finally set the appropriate pixel with the correct colour to display the FFT data.

As a final step, update the spectrogram using the timer callback function by calling the `drawNextLineOfSpectrogram()` only when the next FFT block is ready, reset the flag and update the GUI using the `repaint()` function:

```cpp
void timerCallback() override
{
    if (nextFFTBlockReady)
    {
        drawNextLineOfSpectrogram();
        nextFFTBlockReady = false;
        repaint();
    }
}
```

> [!NOTE]
> Exercise: Try to increase the resolution of the FFT and change the rate at which the spectrogram updates.

<!-- -->

> [!TIP]
>The source code for this modified version of the code can be found in the `SimpleFFTTutorial_02.h` file of the demo project.

# Summary

In this tutorial, we have learnt how to use an FFT function to display audio data in a spectrogram. In particular, we have:

- Learnt the basics of a fast fourier transform function.
- Processed audio sample by sample using a fifo.
- Displayed the data in an [Image](https://docs.juce.com/master/classImage.html "Holds a fixed-size bitmap.") object pixel by pixel.

# See also

- [Tutorial: Draw audio waveforms](/tutorials/tutorial_audio_thumbnail/)
- [Tutorial: Build an audio player](/tutorials/tutorial_playing_sound_files/)
- [Tutorial: Processing audio input](/tutorials/tutorial_processing_audio_input/)
- [Tutorial: Introduction to DSP](/tutorials/tutorial_dsp_introduction/)
- [Tutorial: Visualise the frequencies of a signal in real time](/tutorials/tutorial_spectrum_analyser/)
