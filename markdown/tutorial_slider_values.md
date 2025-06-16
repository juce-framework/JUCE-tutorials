# Tutorial: The Slider class

This tutorial introduces the [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value.") class, shows how to respond to slider movements, and how to obtain values from a slider. The tutorial also introduces some essential customisation techniques for displaying values with a slider.

**LEVEL:** Beginner<br/>
**PLATFORMS:** Windows, macOS, Linux, iOS, Android<br/>
**CLASSES:** [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value."), [Slider::Listener](https://docs.juce.com/master/classSlider.html#a1977aeac9b4363e8ed0cac0ac103055a), [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.")

# Getting started

> [!TIP]
>This tutorial leads on from [Tutorial: Listeners and Broadcasters](/tutorials/tutorial_listeners_and_broadcasters/), which you should have read and understood first.

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/SliderValuesTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/SliderValuesTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project shows two linear horizontal sliders. One slider is labelled **Frequency** and the other is labelled **Duration** as shown in the following screenshot:

![](/_images/tutorial_slider_values_screenshot1.png "The demo project user interface showing two sliders and their values.")

The idea is that both sliders essentially display the same underlying value since frequency ( *f* ) is the reciprocal of duration ( *d* ):

*f* = <sup>1</sup>⁄<sub>d</sub>

When either of the sliders are moved, the other one updates to reflect the change.

# The JUCE Slider class.

This tutorial shows how to create the sliders, configure their range, listen for changes in value, and update the slider value programmatically. You will notice in the demo application, when it runs, that both sliders include a text box, and this text box also includes the units for frequency (**Hz** , Hertz) and duration (**s** , seconds).

## Adding the sliders

The sliders have been added as private members to our `MainContentComponent` class:

```cpp
private:
    juce::Slider frequencySlider;
    juce::Label frequencyLabel;
    juce::Slider durationSlider;
    juce::Label durationLabel;

    JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR (MainContentComponent)
};
```

Notice that we also add a [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") object for each [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value.") object. These are to display the text **Frequency** and **Duration** to the left of the sliders. The boxes immediately to the left of the slider controls, which show the current slider values, are actually part of the [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value.") objects.

We have also added the [Slider::Listener](https://docs.juce.com/master/classSlider.html#a1977aeac9b4363e8ed0cac0ac103055a) class as base class, so that we can register our class to receive slider changes

```cpp
class MainContentComponent : public juce::Component,
                             public juce::Slider::Listener
{
public:
```

In our `MainContentComponent` constructor, we add the sliders as child components (see [Tutorial: Parent and child components](/tutorials/tutorial_component_parents_children/)), make them visible, and configure the range of values that the slider can represent. First we configure the `frequencySlider` member:

```cpp
MainContentComponent()
{
    addAndMakeVisible (frequencySlider);
    frequencySlider.setRange (50, 5000.0); // [1]
    frequencySlider.setTextValueSuffix (" Hz"); // [2]
    frequencySlider.addListener (this); // [3]
```

- [1]: The range of the slider is set using the [Slider::setRange()](https://docs.juce.com/master/classSlider.html#a9fbc23e67deb4a18d172c357a7cff14c "Sets the limits that the slider's value can take.") function.
- [2]: We add a suffix to the text display in the slider's text box to show the value's units.
- [3]: We add our `MainContentComponent` object as a listener to the slider.

The corresponding label is set up as follows:

```cpp
addAndMakeVisible (frequencyLabel);
frequencyLabel.setText ("Frequency", juce::dontSendNotification);
frequencyLabel.attachToComponent (&frequencySlider, true); // [4]
```

The [Label::attachToComponent()](https://docs.juce.com/master/classLabel.html#a3c2397c0da1249f9e27e2279e0f2d4eb "Makes this label "stick to" another component.") function [4] is really useful for placing a label adjacent to another component. The second argument, `true` , positions the label to the left of the other component (`false` would position it above). As we will see shortly, this avoids us having to position the labels manually in the `MainContentComponent::resized()` function.

The `durationSlider` and the `durationLabel` members are set up similarly, but the range of this slider is set to be the reciprocal of the range of the `frequencySlider` member:

```cpp
addAndMakeVisible (durationSlider);
durationSlider.setRange (1.0 / frequencySlider.getMaximum(),
    1.0 / frequencySlider.getMinimum());
durationSlider.setTextValueSuffix (" s");
durationSlider.addListener (this);

addAndMakeVisible (durationLabel);
durationLabel.setText ("Duration", juce::dontSendNotification);
durationLabel.attachToComponent (&durationSlider, true);
```

## Positioning the slider

The sliders are positioned in the `MainContentComponent::resized()` function. Since we used the [Label::attachToComponent()](https://docs.juce.com/master/classLabel.html#a3c2397c0da1249f9e27e2279e0f2d4eb "Makes this label "stick to" another component.") function to attach the labels to the sliders, these are positioned to the left of the sliders automatically.

```cpp
void resized() override
{
    auto sliderLeft = 120;
    frequencySlider.setBounds (sliderLeft, 20, getWidth() - sliderLeft - 10, 20);
    durationSlider.setBounds (sliderLeft, 50, getWidth() - sliderLeft - 10, 20);
}
```

## Responding to slider changes

The following code makes the listeners of the sliders react to changes in the sliders\' values.

```cpp
void sliderValueChanged (juce::Slider* slider) override
{
    if (slider == &frequencySlider)
        durationSlider.setValue (1.0 / frequencySlider.getValue(), juce::dontSendNotification);
    else if (slider == &durationSlider)
        frequencySlider.setValue (1.0 / durationSlider.getValue(), juce::dontSendNotification);
}
```

This is the [Slider::Listener::sliderValueChanged()](https://docs.juce.com/master/classSliderListener.html#aa43e257ace7dd4a61399b30cb3285e75 "Called when the slider's value is changed.") function that we must override if we add [Slider::Listener](https://docs.juce.com/master/classSlider.html#a1977aeac9b4363e8ed0cac0ac103055a) as a base class. Here we simply pass the reciprocal of the slider to the other slider by calling the [Slider::setValue()](https://docs.juce.com/master/classSlider.html#a430a5c4e56b39dd622f5800f787e0822 "Changes the slider's current value.") function. We also tell the slider not to broadcast its change. This is because there is the potential for an infinite feedback loop to occur in cases such as this, where two sliders depend upon each other. The [dontSendNotification](group__juce__events-messages.html#gga25a31a4bdc02a0d0adb743e26dd14ba9a057928146704d53dfee6f4ac33c92629) value breaks this potential loop. Assuming the arithmetic is accurate, and the the conversions in both directions produce identical results, then this shouldn\'t be needed. This is because the slider will only broadcast to its listeners if the value has actually changed. (Problems can occur where there are slight rounding errors in the conversions in situations like this.) You can try omitting the [dontSendNotification](group__juce__events-messages.html#gga25a31a4bdc02a0d0adb743e26dd14ba9a057928146704d53dfee6f4ac33c92629) value which causes the default behaviour where the slider _will_ broadcast changes. You really need to think carefully about whether to use [dontSendNotification](group__juce__events-messages.html#gga25a31a4bdc02a0d0adb743e26dd14ba9a057928146704d53dfee6f4ac33c92629), or not, for specific use-cases in your own applications.

## Setting the initial value

In the constructor the `frequencySlider` slider is set to a value of 500. This in turn will cause the `durationSlider` slider to update, since we omit the [dontSendNotification](group__juce__events-messages.html#gga25a31a4bdc02a0d0adb743e26dd14ba9a057928146704d53dfee6f4ac33c92629) value this time:

```cpp
frequencySlider.setValue (500.0); // [5]
```

# Some customisations

There are couple of simple customisations we can add here to make the interface more effective.

## Making the text box wider

The text box for the `durationSlider` slider, in particular, needs many digits to display its value satisfactorily. To do this we can use the [Slider::setTextBoxStyle()](https://docs.juce.com/master/classSlider.html#a5bc748a21e72fe14153bc9fe5ac03e77 "Changes the location and properties of the text-entry box.") function. Add the following two lines of code to the `MainContentComponent` constructor:

```cpp
frequencySlider.setTextBoxStyle (juce::Slider::TextBoxLeft, false, 160, frequencySlider.getTextBoxHeight());
durationSlider.setTextBoxStyle (juce::Slider::TextBoxLeft, false, 160, durationSlider.getTextBoxHeight());
```

This sets the text box to be 160 pixels in each case (but maintaining the height by using the [Slider::getTextBoxHeight()](https://docs.juce.com/master/classSlider.html#a8258ee0e1222f3e02a696243a3468578 "Returns the height used for the text-box.") function).

![](/_images/tutorial_slider_values_screenshot2.png "The sliders with wider text boxes.")

## Skewing the slider values

By default, the slider track is linear in the sense that the slider's value is proportional to the position of the slider thumb along the slider track. It is clear from manipulating the interface that this doesn\'t quite feel right. We can adjust the slider _skew_ to make the slider track logarithmic. To do this we can use the [Slider::setSkewFactorFromMidPoint()](https://docs.juce.com/master/classSlider.html#a2d948c662c1f636810d3125d8b3f0d47 "Sets up a skew factor to alter the way values are distributed.") function. Try this out by adding the following two lines of code to the `MainContentComponent` constructor after the sliders have been configured:

```cpp
frequencySlider.setSkewFactorFromMidPoint (500);
durationSlider.setSkewFactorFromMidPoint (0.002);
```

This places a value of 500 at the mid-point of the slider track for the `frequencySlider` slider, and 0.002 for the `durationSlider` slider. Effectively, the sliders will now seem to move equally but in opposite directions. A non-linear slider track like this works well for parameters such as time and frequency where we tend to want finer control over smaller values but need less fine control over larger values.

> [!TIP]
>The completed code for this section can be found in the `SliderValuesTutorial_02.h` file of the demo project for this tutorial.

<!-- -->

> [!NOTE]
> Exercise: Try out different values for the calls to the [Slider::setSkewFactorFromMidPoint()](https://docs.juce.com/master/classSlider.html#a2d948c662c1f636810d3125d8b3f0d47 "Sets up a skew factor to alter the way values are distributed.") function and try out different text box sizes. Have a look at the API reference for the [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value.") class and try out some other customisations.

## Simplifying the Slider callback

Instead of using the listeners and broadcasters paradigm as shown in this tutorial, we can simplify slider callbacks using lambda functions from the latest C++ standards. This works especially well for simple callbacks that don\'t require complex implementations.

First, let's remove the inheritance from the [Slider::Listener](https://docs.juce.com/master/classSlider.html#a1977aeac9b4363e8ed0cac0ac103055a) class and restore the MainContentComponent class definition like this:

```cpp
class MainContentComponent : public juce::Component
{
public:
```

Then, instead of adding the MainContentComponent as a listener to the [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value."), assign a lambda function to the [Slider::onValueChange](https://docs.juce.com/master/classSlider.html#a680d007f6a824a28a60aa05b4045e794 "You can assign a lambda to this callback object to have it called when the slider value is changed.") helper object as follows:

```cpp
MainContentComponent()
{
    addAndMakeVisible (frequencySlider);
    frequencySlider.setRange (50, 5000.0);
    frequencySlider.setTextValueSuffix (" Hz");
    frequencySlider.onValueChange = [this] { durationSlider.setValue (1.0 / frequencySlider.getValue(), juce::dontSendNotification); };

    addAndMakeVisible (frequencyLabel);
    frequencyLabel.setText ("Frequency", juce::dontSendNotification);
    frequencyLabel.attachToComponent (&frequencySlider, true);

    addAndMakeVisible (durationSlider);
    durationSlider.setRange (1.0 / frequencySlider.getMaximum(),
        1.0 / frequencySlider.getMinimum());
    durationSlider.setTextValueSuffix (" s");
    durationSlider.onValueChange = [this] { frequencySlider.setValue (1.0 / durationSlider.getValue(), juce::dontSendNotification); };
```

This tells the [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value.") object which function to call when the [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value.") value is changed by the user.

> [!TIP]
>The implementation of the code can be found in the `SliderValuesTutorial_03.h` file of the demo project for this tutorial.

# Summary

In this tutorial we have introduced the [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value.") class. In particular we have learned:

- How to set up a slider to operate over a specific range.
- How to respond to slider value changes.
- How to configure the slider skew to make it use a logarithmic scale.

# See also

- [Tutorial: Listeners and Broadcasters](/tutorials/tutorial_listeners_and_broadcasters/)
- [Tutorial: The ComboBox class](/tutorials/tutorial_combo_box/)
- [Tutorial: The Label class](/tutorials/tutorial_label/)
- [Tutorial: Radio buttons and checkboxes](/tutorials/tutorial_radio_buttons_checkboxes/)
