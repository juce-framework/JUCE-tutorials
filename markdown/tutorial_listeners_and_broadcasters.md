# Tutorial: Listeners and Broadcasters

This tutorial introduces a key concept in JUCE: the listener and broadcaster system. We look at this through implementing simple actions in response to button clicks.

**LEVEL:** Beginner<br/>
**PLATFORMS:** Windows, macOS, Linux, iOS, Android<br/>
**CLASSES:** [Button](https://docs.juce.com/master/classButton.html "A base class for buttons."), [TextButton](https://docs.juce.com/master/classTextButton.html "A button that uses the standard lozenge-shaped background with a line of text on it."), [Button::Listener](https://docs.juce.com/master/classButton_1_1Listener.html "Used to receive callbacks when a button is clicked."), [Time](https://docs.juce.com/master/classTime.html "Holds an absolute date and time.")

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/ListenersAndBroadcastersTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/ListenersAndBroadcastersTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project for this tutorial presents a simple user interface with a single button and a single label. The interface should look similar to the following screenshot:

![](/_images/tutorial_listeners_and_broadcasters_screenshot1.png "A basic button and label interface.")

The interface doesn\'t do anything in the state provided. We are going to add some code to make a click of the button cause the label to display the current date and time.

# Configuring the interface

The `MainContentComponent` class comprises two child components: a [TextButton](https://docs.juce.com/master/classTextButton.html "A button that uses the standard lozenge-shaped background with a line of text on it.") object and a [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") object. A [TextButton](https://docs.juce.com/master/classTextButton.html "A button that uses the standard lozenge-shaped background with a line of text on it.") object can display a button containing some specific text, and a [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") object can display a piece of text.

> [!TIP]
>The [TextButton](https://docs.juce.com/master/classTextButton.html "A button that uses the standard lozenge-shaped background with a line of text on it.") class implements one type of button. There are many types of [Button](https://docs.juce.com/master/classButton.html "A base class for buttons.") class available in JUCE. See the API reference documentation for the [ToggleButton](https://docs.juce.com/master/classToggleButton.html "A button that can be toggled on/off."), [ShapeButton](https://docs.juce.com/master/classShapeButton.html "A button that contains a filled shape."), [ImageButton](https://docs.juce.com/master/classImageButton.html "As the title suggests, this is a button containing an image."), [DrawableButton](https://docs.juce.com/master/classDrawableButton.html "A button that displays a Drawable."), and [ArrowButton](https://docs.juce.com/master/classArrowButton.html "A button with an arrow in it.") classes for more information.

The declaration for the `MainContentComponent` class is as follows:

```cpp
class MainContentComponent : public juce::Component
{
public:
    //==============================================================================
    MainContentComponent()
    {
        // ...
    }

    ~MainContentComponent()
    {
        // ...
    }

    void resized() override
    {
        // ...
    }

private:
    juce::TextButton checkTheTimeButton;
    juce::Label timeLabel;

    //==============================================================================
    JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR (MainContentComponent)
};
```

The button and the label are added to the `MainContentComponent` object and made visible in the `MainContentComponent` constructor:

```cpp
MainContentComponent()
{
    addAndMakeVisible (checkTheTimeButton);
    checkTheTimeButton.setButtonText ("Check the time...");

    addAndMakeVisible (timeLabel);
    timeLabel.setColour (juce::Label::backgroundColourId, juce::Colours::black);
    timeLabel.setColour (juce::Label::textColourId, juce::Colours::white);
    timeLabel.setJustificationType (juce::Justification::centred);

    setSize (600, 110);
}
```

Here we set the button text and configure a specific appearance for the label. This so that the label shows white text on a black background. By default, the label will not show any text.

## Adding a listener base class

In JUCE, buttons, sliders, and many other types of controls that may need to inform other objects about a change in their state are a type of _broadcaster_ object. In order to respond to changes in a broadcaster object, other classes need to be a _listener_ for that specific type of broadcaster. Listeners also need to be registered with at least one specific broadcaster object of that type. (The broadcaster-listener system in JUCE follows the [observer pattern](https://en.wikipedia.org/wiki/Observer_pattern) .) Many broadcaster objects contain a nested `Listener` class, from which we can inherit, in order to become a listener for that type of broadcaster. For example, the [Button](https://docs.juce.com/master/classButton.html "A base class for buttons.") class contains a nested class [Button::Listener](https://docs.juce.com/master/classButton_1_1Listener.html "Used to receive callbacks when a button is clicked.") for this purpose.

> [!TIP]
>The [Button::Listener](https://docs.juce.com/master/classButton_1_1Listener.html "Used to receive callbacks when a button is clicked.") class can be used to listen to any of the different button types including instances of the [TextButton](https://docs.juce.com/master/classTextButton.html "A button that uses the standard lozenge-shaped background with a line of text on it.") class as shown here.

To use the [Button::Listener](https://docs.juce.com/master/classButton_1_1Listener.html "Used to receive callbacks when a button is clicked.") class we need to add it as a base class. In our case, we need to add the [Button::Listener](https://docs.juce.com/master/classButton_1_1Listener.html "Used to receive callbacks when a button is clicked.") class as a base class of the `MainContentComponent` class [1]:

```cpp
class MainContentComponent : public juce::Component,
                             public juce::Button::Listener // [1]
{
public:
```

Custom classes can become listeners to different types of broadcaster by adding more listener base classes in the same way.

## Declaring the listener's callback

Usually each listener class has at least one [pure virtual](http://www.learncpp.com/cpp-tutorial/126-pure-virtual-functions-abstract-base-classes-and-interface-classes/) function. This is the function that will be called as a callback when the broadcaster object needs to broadcast its change. We must override this in order for the code to compile, and for us to use it.

> [!TIP]
>Listener classes often contain other virtual functions that may be overridden, but these are optional as they are needed in fewer cases. See the documentation for the [Slider::Listener](https://docs.juce.com/master/classSlider.html#a1977aeac9b4363e8ed0cac0ac103055a) class for an example.

The [pure virtual](http://www.learncpp.com/cpp-tutorial/126-pure-virtual-functions-abstract-base-classes-and-interface-classes/) function in the [Button::Listener](https://docs.juce.com/master/classButton_1_1Listener.html "Used to receive callbacks when a button is clicked.") class is the [Button::Listener::buttonClicked()](https://docs.juce.com/master/classButton_1_1Listener.html#a81499cef24b7189cd0d1581fd9dc9e14 "Called when the button is clicked.") function. We need to add its declaration [2] within our `MainContentComponent` class as shown here:

```cpp
MainContentComponent()
{
    // ...
}

~MainContentComponent()
{
    // ...
}

void resized() override
{
    // ...
}

void buttonClicked (juce::Button* button) override // [2]
{
}

// ...
```

## Implementing the listener callback

Now, let's implement the `MainContentComponent::buttonClicked()` function. Here, we are passed a pointer to the object that has broadcasted the change. We can then compare this pointer with other objects to determine which object it was:

```cpp
void buttonClicked (juce::Button* button) override // [2]
{
    if (button == &checkTheTimeButton) // [3]
    {
        auto currentTime = juce::Time::getCurrentTime(); // [4]

        auto includeDate = true;
        auto includeTime = true;
        auto currentTimeString = currentTime.toString (includeDate, includeTime); // [5]

        timeLabel.setText (currentTimeString, juce::dontSendNotification); // [6]
    }
}
```

- [3]: Here we compare the pointer passed to the function with the address of our button, to see if it matches. You should do this even if you have only one button, as we do here. It is safe to compare the pointer to the base [Button](https://docs.juce.com/master/classButton.html "A base class for buttons.") class passed to the function with instances of [Button](https://docs.juce.com/master/classButton.html "A base class for buttons.") subclasses, such as the [TextButton](https://docs.juce.com/master/classTextButton.html "A button that uses the standard lozenge-shaped background with a line of text on it.") class as shown here.
- [4]: This uses the [Time](https://docs.juce.com/master/classTime.html "Holds an absolute date and time.") class to get the current time from the operating system.
- [5]: This converts the [Time](https://docs.juce.com/master/classTime.html "Holds an absolute date and time.") object to a readable string. The two `bool` values allow some customisation of the output (see the documentation for the [Time::toString()](https://docs.juce.com/master/classTime.html#a330e0926141abd7ef42957bce8a9e054 "Returns a string version of this date and time, using this machine's local timezone.") function for more information).
- [6]: Here we update the text displayed within the label.

> [!TIP]
>The `dontSendNotification` argument [7] prevents the label from broadcasting this change to its listeners, should it have any. ([Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") objects can have listeners since they can be used to edit text, too.) In this case we know that it can\'t have any listeners (as it is our own private member) but it is good practice to be specific.

## Registering as a listener with the broadcaster

In order to receive the messages that are broadcast, we need to register our listener object with one or more broadcaster objects. In this case, we need to register with the [TextButton](https://docs.juce.com/master/classTextButton.html "A button that uses the standard lozenge-shaped background with a line of text on it.") object. Typically, this would be done within the constructor of the listener subclass [7]:

```cpp
MainContentComponent()
{
    addAndMakeVisible (checkTheTimeButton);
    checkTheTimeButton.setButtonText ("Check the time...");
    checkTheTimeButton.addListener (this); // [7]
```

> [!TIP]
>Most broadcaster objects have an `addListener()` function for this purpose (the [ChangeBroadcaster](https://docs.juce.com/master/classChangeBroadcaster.html "Holds a list of ChangeListeners, and sends messages to them when instructed.") object is an exception, it has the [ChangeBroadcaster::addChangeListener()](https://docs.juce.com/master/classChangeBroadcaster.html#ad68416fe79a94cd5c99519bdea6c2a06 "Registers a listener to receive change callbacks from this broadcaster.") function instead).

### Deregistering as a listener with the broadcaster

Broadcasters will also have a `removeListener()` function, too. For example, see the [Button::removeListener()](https://docs.juce.com/master/classButton.html#aa7016d2e4b8ab37a12c2736057eb28de "Removes a previously-registered button listener.") function. Since our button is owned by the same class that is performing the listening we don\'t really _need_ to remove the listener as the button will be destroyed at the same time as the listener. For completeness, we _could_ add this to our destructor:

```cpp
~MainContentComponent()
{
    checkTheTimeButton.removeListener (this);
}
```

> [!WARNING]
> Removing listeners appropriately is important if you set up more complex broadcaster-listener systems.

Build and run the application now. When you click the button it should display the time within the label.

![](/_images/tutorial_listeners_and_broadcasters_screenshot2.png "Using the button to display the current time.")

> [!TIP]
>The completed code for this section can be found in the `ListenersAndBroadcastersTutorial_02.h` file of the demo project for this tutorial.

<!-- -->

> [!NOTE]
> Exercise: Try changing the format of the text displayed. You can do this by changing the arguments passed to the [Time::toString()](https://docs.juce.com/master/classTime.html#a330e0926141abd7ef42957bce8a9e054 "Returns a string version of this date and time, using this machine's local timezone.") function. You could also change the code such that the label displays the number of milliseconds between button clicks, rather than the absolute time.

<!-- -->

> [!TIP]
>An example implementation of the code for this exercise can be found in the `ListenersAndBroadcastersTutorial_03.h` file of the demo project for this tutorial.

## Simplifying the Button callback

Instead of using the listeners and broadcasters paradigm as shown in this tutorial, we can simplify button callbacks using lambda functions from the latest C++ standards. This works especially well for simple callbacks that don\'t require complex implementations.

First, let's remove the inheritance from the [Button::Listener](https://docs.juce.com/master/classButton_1_1Listener.html "Used to receive callbacks when a button is clicked.") class and restore the MainContentComponent class definition like this:

```cpp
class MainContentComponent : public juce::Component
{
public:
```

Then, instead of adding the MainContentComponent as a listener to the [Button](https://docs.juce.com/master/classButton.html "A base class for buttons."), assign a lambda function to the [Button::onClick](https://docs.juce.com/master/classButton.html#a30b76ab312dc7f66e67596ae20540ec2 "You can assign a lambda to this callback object to have it called when the button is clicked.") helper object [8] as follows:

```cpp
MainContentComponent()
{
    addAndMakeVisible (checkTheTimeButton);
    checkTheTimeButton.setButtonText ("Check the time...");
    checkTheTimeButton.onClick = [this] { checkTime(); }; // [8]
```

This tells the [Button](https://docs.juce.com/master/classButton.html "A base class for buttons.") object which function to call when the [Button](https://docs.juce.com/master/classButton.html "A base class for buttons.") is clicked by the user.

Finally, rename the callback function to checkTime() [9] and remove the if() statement checking the pointers to the [Button](https://docs.juce.com/master/classButton.html "A base class for buttons.") objects as we don\'t need to check which [Button](https://docs.juce.com/master/classButton.html "A base class for buttons.") called the function anymore:

```cpp
void checkTime() // [9]
{
    auto currentTime = juce::Time::getCurrentTime();

    auto includeDate = true;
    auto includeTime = true;
    auto currentTimeString = currentTime.toString (includeDate, includeTime);

    timeLabel.setText (currentTimeString, juce::dontSendNotification);
}
```

> [!TIP]
>The implementation of the code can be found in the `ListenersAndBroadcastersTutorial_04.h` file of the demo project for this tutorial.

# Summary

In this tutorial we have introduced the basics of the broadcaster-listener system in JUCE. While we have focused on buttons in this tutorial, the same techniques can be applied to many areas of JUCE code. In particular we have learned:

- How to make one of our custom classes a listener-type object.
- How to add the listener callback function.
- How to register and deregister as a listener with a broadcaster object.
- How to access the current time using the [Time](https://docs.juce.com/master/classTime.html "Holds an absolute date and time.") class.
- How to simplify the callback with a lambda function.

# See also

- [Tutorial: The Label class](/tutorials/tutorial_label/)
- [Tutorial: The ComboBox class](/tutorials/tutorial_combo_box/)
- [Tutorial: Radio buttons and checkboxes](/tutorials/tutorial_radio_buttons_checkboxes/)
- [Tutorial: The Slider class](/tutorials/tutorial_slider_values/)
- [Tutorial: Build an audio player](/tutorials/tutorial_playing_sound_files/)
