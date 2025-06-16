# Tutorial: Wavetable synthesis

Incorporate wavetables to optimise your synthesiser oscillators. Manage the state of a sine wave oscillator using a wavetable and write data to the audio output.

**LEVEL:** Intermediate<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**CLASSES:** [AudioBuffer](https://docs.juce.com/master/classAudioBuffer.html "A multi-channel buffer containing floating point audio samples."), [AudioAppComponent](https://docs.juce.com/master/classAudioAppComponent.html "A base class for writing audio apps that stream from the audio i/o devices."), [Random](https://docs.juce.com/master/classRandom.html "A random number generator."), [MathConstants](structMathConstants.html "Commonly used mathematical constants.")

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/WavetableSynthTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/WavetableSynthTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project simply generates and outputs a stack of random sine wave harmonics through its stereo output. The user interface allows us to monitor the CPU usage by comparing the traditional implementation of an oscillator and the one that makes use of a wavetable.

In order to properly evaluate and compare the CPU usage of our different implementations, we are going to run our application in the **Release** configuration instead of the regular **Debug** configuration used during testing and development. By building the project in **Release** mode the compiler will be able to optimise the code as much as possible by removing assertions and comments from the code and inlining functions for example.

To change the build configuration in Xcode, first click on the deployment targets in the top left corner of the interface and navigate to **Edit Scheme...** as shown below:

![](/_images/tutorial_wavetable_synth_screenshot1.png "Editing the scheme")

In the pop-up window, select **Release** in the **Build Configuration** combo box as shown in the screenshot:

![](/_images/tutorial_wavetable_synth_screenshot2.png "Changing the build configuration")

Your application will now run after heavy compiler optimisations and the CPU usage should decrease significantly.

# Wavetables

Wavetable synthesis is a synthesis method that uses look-up tables that are pre-filled with periodic waveforms to generate oscillators without having to generate the same waveform for each sample calculated. The wavetable is initialised with periodic waveforms of your choice and the resolution of these waveforms can be specified. When retrieving the correct sample value to output, the value is found by interpolating between two wavetable samples if the number of samples in the table does not match with the number of samples in the audio buffer block and its corresponding requested frequency.

As an example, let's say that we want to look up a sine wave from the wavetable. We would first create a wavetable for a single cycle of a sine wave with a resolution of 128 sample points for instance. For each sample in the buffer block, we can then request the sine wave sample value by calculating the correct interpolated sample using a combination of the sample rate, the requested frequency to play, the resolution of the wavetable and the current phase or angle of the waveform.

Let's start with a simple sine wave oscillator implementation before diving into wavetables.

# Sine Wave Oscillator

> [!TIP]
>This section is covered in more detail in [Tutorial: Build a sine wave synthesiser](/tutorials/tutorial_sine_synth/) and if you need help with these steps please refer to that tutorial first.

In the `SineOscillator` class, we keep track of two member variables that store the current angle or phase in the waveform cycle and the angle delta to increment between every cycle depending on the frequency and the sample rate:

```cpp
class SineOscillator
{
public:
    SineOscillator() {}
    //...

private:
    float currentAngle = 0.0f, angleDelta = 0.0f;
};
```

The `setFrequency()` function allows us to calculate the angle delta by first dividing the frequency by the sample rate and multiplying the result by 2pi, the length of a cycle in radians:

```cpp
void setFrequency (float frequency, float sampleRate)
{
    auto cyclesPerSample = frequency / sampleRate;
    angleDelta = cyclesPerSample * juce::MathConstants<float>::twoPi;
}
```

The `getNextSample()` function gets called by the `getNextAudioBlock()` function of the [AudioSource](https://docs.juce.com/master/classAudioSource.html "Base class for objects that can produce a continuous stream of audio.") on every sample in the buffer to retrieve the sample value from the oscillator. Here we calculate the sample value using the `std::sin()` function by passing the current angle as an argument and updating the current angle by calling the helper function `updateAngle()` defined after:

```cpp
forcedinline float getNextSample() noexcept
{
    auto currentSample = std::sin (currentAngle);
    updateAngle();
    return currentSample;
}
```

The angle is updated by incrementing the current angle with the angle delta calculated previously when setting the frequency and by wrapping the value when the angle exceeds 2pi:

```cpp
forcedinline void updateAngle() noexcept
{
    currentAngle += angleDelta;
    if (currentAngle >= juce::MathConstants<float>::twoPi)
        currentAngle -= juce::MathConstants<float>::twoPi;
}
```

Now let's switch to the implementation of our `MainContentComponent` class.

We keep track of the overall level of our output and an array of oscillators as private member variables as shown here:

```cpp
class MainContentComponent : public juce::AudioAppComponent,
                             public juce::Timer
{
    //...

private:
    //...

    float level = 0.0f;
    juce::OwnedArray<SineOscillator> oscillators;
    //...
};
```

In the `prepareToPlay()` function, we have to initialise the oscillators and set their frequencies to play based on the sample rate as follows:

```cpp
void prepareToPlay (int, double sampleRate) override
{
    auto numberOfOscillators = 200; // [1]

    for (auto i = 0; i < numberOfOscillators; ++i)
    {
        auto* oscillator = new SineOscillator(); // [2]

        auto midiNote = juce::Random::getSystemRandom().nextDouble() * 36.0 + 48.0; // [3]
        auto frequency = 440.0 * pow (2.0, (midiNote - 69.0) / 12.0); // [4]

        oscillator->setFrequency ((float) frequency, (float) sampleRate); // [5]
        oscillators.add (oscillator);
    }

    level = 0.25f / (float) numberOfOscillators; // [6]
}
```

- [1]: First we define a large number of oscillators to evaluate the CPU load of such a number.
- [2]: For each oscillator, we instantiate a new `SineOscillator` object that generates a single sine wave voice.
- [3]: We also select a random midi note using the [Random](https://docs.juce.com/master/classRandom.html "A random number generator.") class by shifting the lowest possible note by 4 octaves and defining a range of 3 octaves starting from that lowest note.
- [4]: In order to calculate the frequency of that midi note, we use a simple mathematical formula to retrieve the scalar to multiply the frequency of A440 with. Since we know that the midi note number of A440 is 69, by subtracting the midi note by 69 we get the semitone distance from A440 that we can then plug into the following formula: `440 \* 2 \^ (d / 12)`
- [5]: Then, we set the frequency of the oscillator by passing the frequency and sample rate as arguments to the `setFrequency()` function. We also add the oscillator to the array of oscillators.
- [6]: Finally, we define the output level by dividing a quiet gain level by the number of oscillators to prevent clipping of the signal by summing such a large number of oscillator samples.

In the `getNextAudioBlock()` function we simply sum all the oscillator samples and write the result to the output buffers as shown below:

```cpp
void getNextAudioBlock (const juce::AudioSourceChannelInfo& bufferToFill) override
{
    auto* leftBuffer = bufferToFill.buffer->getWritePointer (0, bufferToFill.startSample); // [7]
    auto* rightBuffer = bufferToFill.buffer->getWritePointer (1, bufferToFill.startSample);

    bufferToFill.clearActiveBufferRegion();

    for (auto oscillatorIndex = 0; oscillatorIndex < oscillators.size(); ++oscillatorIndex)
    {
        auto* oscillator = oscillators.getUnchecked (oscillatorIndex); // [8]

        for (auto sample = 0; sample < bufferToFill.numSamples; ++sample)
        {
            auto levelSample = oscillator->getNextSample() * level; // [9]

            leftBuffer[sample] += levelSample; // [10]
            rightBuffer[sample] += levelSample;
        }
    }
}
```

- [7]: First, we retrieve the left and right channel pointers to write to the output buffers.
- [8]: For each oscillator in the array we retrieve a pointer to the oscillator instance.
- [9]: Then for each sample in the audio sample buffer we get the sine wave sample and trim the gain with the level variable.
- [10]: Finally we can add that sample value to the left and right channel samples and sum the signal with the other oscillators.

If we run the application now, we should be able to hear a random noise of stacked sine waves.

> [!NOTE]
> Exercise: Instead of generating random midi notes, find the midi notes of a certain chord and generate random notes from the chord.

# Wavetable Oscillator

Let's change the oscillator implementation to a wavetable synthesis method.

In the `MainContentComponent` class, add an AudioSampleBuffer as a member variable that will hold the wavetable values of our single sine wave cycle [1]. We also define the wavetable resolution as a constant of 128 samples using the bit shift operator [2]:

```cpp
private:
    juce::Label cpuUsageLabel;
    juce::Label cpuUsageText;

    const unsigned int tableSize = 1 << 7; // [2]
    float level = 0.0f;

    juce::AudioSampleBuffer sineTable; // [1]
    juce::OwnedArray<WavetableOscillator> oscillators;

    JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR (MainContentComponent)
};
```

Define a new function called `createWavetable()` that will be called in the `MainContentComponent` constructor before we start the audio processing.

```cpp
void createWavetable()
{
    sineTable.setSize (1, (int) tableSize);
    auto* samples = sineTable.getWritePointer (0); // [3]

    auto angleDelta = juce::MathConstants<double>::twoPi / (double) (tableSize - 1); // [4]
    auto currentAngle = 0.0;

    for (unsigned int i = 0; i < tableSize; ++i)
    {
        auto sample = std::sin (currentAngle); // [5]
        samples[i] = (float) sample;
        currentAngle += angleDelta;
    }
}
```

- [3]: In this function, initialise the AudioSampleBuffer by calling the `setSize()` method by specifying that we only need one channel and the number of samples equal to the table size, in our case a resolution of 128. Then retrieve the write pointer for that single channel buffer.
- [4]: Next, calculate the angle delta similarly to the previous section but this time using the table size and thus dividing the full 2pi cycle by 127.
- [5]: Now for each point in our wavetable, retrieve the sine wave value using the `std::sin()` function, assign the value to the buffer sample and increment the current angle by the delta value.

Add this function call in the `MainContentComponent` constructor as follows:

```cpp
MainContentComponent()
{
    cpuUsageLabel.setText ("CPU Usage", juce::dontSendNotification);
    cpuUsageText.setJustificationType (juce::Justification::right);
    addAndMakeVisible (cpuUsageLabel);
    addAndMakeVisible (cpuUsageText);

    createWavetable();

    setSize (400, 200);
    setAudioChannels (0, 2); // no inputs, two outputs
    startTimer (50);
}
```

The wavetable should now contain 128 samples of a full sine wave cycle.

In the for() loop of the `prepareToPlay()` function, change the below line to instantiate a `WavetableOscillator` object instead of a `SineOscillator` object:

```cpp
for (auto i = 0; i < numberOfOscillators; ++i)
{
    auto* oscillator = new WavetableOscillator (sineTable);
```

This constructor takes as an argument the wavetable to use for the sound generation and therefore, create a corresponding new `WavetableOscillator` class as shown below:

```cpp
class WavetableOscillator
{
public:
    WavetableOscillator (const juce::AudioSampleBuffer& wavetableToUse)
        : wavetable (wavetableToUse)
    {
        jassert (wavetable.getNumChannels() == 1);
    }
```

```cpp
private:
    const juce::AudioSampleBuffer& wavetable;
    float currentIndex = 0.0f, tableDelta = 0.0f;
};
```

Instead of keeping track of the current angle and the angle delta of the waveform cycle, define two member variables that store the current wavetable index and the angle delta of the wavetable. Also, define an AudioSampleBuffer variable to hold a reference to the wavetable to use.

The `setFrequency()` function of the `WavetableOscillator` class is fairly similar to the one implemented previously except that the angle delta is calculated using the size of the wavetable instead of the full cycle in radians of 2pi as follows:

```cpp
void setFrequency (float frequency, float sampleRate)
{
    auto tableSizeOverSampleRate = (float) wavetable.getNumSamples() / sampleRate;
    tableDelta = frequency * tableSizeOverSampleRate;
}
```

The `getNextSample()` function is where the interpolation between the wavetable values occur in order to get the correct sample value.

```cpp
forcedinline float getNextSample() noexcept
{
    auto tableSize = (unsigned int) wavetable.getNumSamples();

    auto index0 = (unsigned int) currentIndex; // [6]
    auto index1 = index0 == (tableSize - 1) ? (unsigned int) 0 : index0 + 1;

    auto frac = currentIndex - (float) index0; // [7]

    auto* table = wavetable.getReadPointer (0); // [8]
    auto value0 = table[index0];
    auto value1 = table[index1];

    auto currentSample = value0 + frac * (value1 - value0); // [9]

    if ((currentIndex += tableDelta) > (float) tableSize) // [10]
        currentIndex -= (float) tableSize;

    return currentSample;
}
```

- [6]: First, temporarily store the two indices of the wavetable that surround the sample value that we are trying to retrieve. If the higher index goes beyond the size of the wavetable then we wrap the value to the start of the table.
- [7]: Next, calculate the interpolation value as a fraction between the two indices by subtracting the actual current sample by the truncated lower index. This should give us a value between `0 .. 1` that defines the fraction.
- [8]: Then get a pointer to the AudioSampleBuffer and read the values at the two indices and store these values temporarily.
- [9]: The interpolated sample value can then be retrieved by using the standard interpolation formula and the fraction value calculated previously.
- [10]: Finally, increment the angle delta of the table and wrap the value around if the value exceeds the table size.

This implementation should give us the same output sound when we run the application.

> [!NOTE]
> Exercise: Modify the number of oscillators and observe the change in CPU usage.

<!-- -->

> [!TIP]
>The source code for this modified version of the code can be found in the `WavetableSynthTutorial_02.h` file of the demo project.

# Wrapping the Wavetable

If you paid close attention to the previous code, you may have noticed that we have one missing value in our wavetable. The final value is skipped as it wraps around to the first value which happens to be the same so let's fix that now.

In the `WavetableOscillator` constructor, assign the table size variable to hold the resolution of the wave table minus one and define that member variable appropriately as follows:

```cpp
class WavetableOscillator
{
public:
    WavetableOscillator (const juce::AudioSampleBuffer& wavetableToUse)
        : wavetable (wavetableToUse),
          tableSize (wavetable.getNumSamples() - 1)
    {
        jassert (wavetable.getNumChannels() == 1);
    }
```

```cpp
private:
    const juce::AudioSampleBuffer& wavetable;
    const int tableSize;
    float currentIndex = 0.0f, tableDelta = 0.0f;
};
```

The `setFrequency()` function needs to be updated using this variable and notice that the angle delta of the table will be slightly smaller:

```cpp
void setFrequency (float frequency, float sampleRate)
{
    auto tableSizeOverSampleRate = (float) tableSize / sampleRate;
    tableDelta = frequency * tableSizeOverSampleRate;
}
```

The `getNextSample()` function remains fairly similar except that we don\'t need to wrap the higher index anymore because we will increase the size of the table in the next step:

```cpp
forcedinline float getNextSample() noexcept
{
    auto index0 = (unsigned int) currentIndex;
    auto index1 = index0 + 1;
```

Here unlike before we set the resolution as one above the defined value and set the last sample as being the same as the first sample:

```cpp
void createWavetable()
{
    sineTable.setSize (1, (int) tableSize + 1);
    auto* samples = sineTable.getWritePointer (0);
```

```cpp
samples[tableSize] = samples[0];
}
```

This allows us to reduce the wrapping condition in the processing call and transfering the load to the `createWavetable()` function that only gets called once at the start of the application.

The result should sound the same as the previous section but notice the slight decrease in CPU usage.

> [!NOTE]
> Exercise: Can you find a way to further optimise this code? Every arithmetic operation in DSP counts towards performance so you should try to eliminate as many as possible.

<!-- -->

> [!TIP]
>The source code for this modified version of the code can be found in the `WavetableSynthTutorial_03.h` file of the demo project.

# Selecting the Harmonics

Instead of outputing a random sine wave sound, let's create a harmonious sine wave by explicitly setting its harmonics.

Modify the `createWavetable()` function to incorporate the harmonics in the wavetable samples of the sine wave as follows:

```cpp
void createWavetable()
{
    sineTable.setSize (1, (int) tableSize + 1);
    sineTable.clear();

    auto* samples = sineTable.getWritePointer (0);

    int harmonics[] = { 1, 3, 5, 6, 7, 9, 13, 15 };
    float harmonicWeights[] = { 0.5f, 0.1f, 0.05f, 0.125f, 0.09f, 0.005f, 0.002f, 0.001f }; // [1]

    jassert (juce::numElementsInArray (harmonics) == juce::numElementsInArray (harmonicWeights));

    for (auto harmonic = 0; harmonic < juce::numElementsInArray (harmonics); ++harmonic)
    {
        auto angleDelta = juce::MathConstants<double>::twoPi / (double) (tableSize - 1) * harmonics[harmonic]; // [2]
        auto currentAngle = 0.0;

        for (unsigned int i = 0; i < tableSize; ++i)
        {
            auto sample = std::sin (currentAngle);
            samples[i] += (float) sample * harmonicWeights[harmonic]; // [3]
            currentAngle += angleDelta;
        }
    }

    samples[tableSize] = samples[0];
}
```

- [1]: Define two arrays that respectively describe the indices of the odd harmonics and its corresponding weights.
- [2]: For each harmonic, calculate the angle delta by multiplying the full 2pi cycle by the harmonic order and dividing by the table size. This essentially multiplies the frequency generated by the harmonic order.
- [3]: For each sample in the table, retrieve the sine value from the current angle, add the value to the existing buffer sample by trimming the gain with the corresponding harmonic weight and increment the current angle by the delta value.

```cpp
void prepareToPlay (int, double sampleRate) override
{
    auto numberOfOscillators = 10;
```

Finally, reduce the number of oscillators to 10 in the `prepareToPlay()` function and listen to the result by running the application.

> [!NOTE]
> Exercise: Modify the harmonics to an even series and notice the change in timbre of the sound produced. What about an odd and even series?

<!-- -->

> [!WARNING]
> Since you are adding higher frequency components to the audio signal you need to watch out for aliasing effects! Dealing with these is beyond the scope of this tutorial, but reading about the Nyquist--Shannon sampling theorem and upsampling would be a good place to start.

<!-- -->

> [!TIP]
>The source code for this modified version of the code can be found in the `WavetableSynthTutorial_04.h` file of the demo project.

# Notes

In this tutorial we explored how to create a wavetable from a sine wave but you can essentially store any type of periodic waveform of your choice as long as the first sample matches the last one.

> [!NOTE]
> Exercise: Modify the `createWavetable()` function to generate and store different types of waveforms such as square, triangle or sawtooth waves.

# Summary

In this tutorial, we have learnt how to implement a wavetable synthesiser. In particular, we have:

- Converted our sine wave oscillator into a wavetable oscillator.
- Optimised consequently the CPU usage with hundreds of oscillators.
- Written random harmonics of the same oscillator to the audio output.
- Created a harmonious sound by choosing the harmonics and their weights.

# See also

- [Tutorial: Build a white noise generator](/tutorials/tutorial_simple_synth_noise/)
- [Tutorial: Build a sine wave synthesiser](/tutorials/tutorial_sine_synth/)
- [Tutorial: Control audio levels using decibels](/tutorials/tutorial_synth_db_level_control/)
- [Tutorial: Control audio levels](/tutorials/tutorial_synth_level_control/)
- [Tutorial: Build a MIDI synthesiser](/tutorials/tutorial_synth_using_midi_input/)
- [Tutorial: Looping audio using the AudioSampleBuffer class](/tutorials/tutorial_looping_audio_sample_buffer/)
- [Tutorial: Looping audio using the AudioSampleBuffer class (advanced)](/tutorials/tutorial_looping_audio_sample_buffer_advanced/)
