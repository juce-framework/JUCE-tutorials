# Tutorial: Radio buttons and checkboxes

Use radio buttons and checkboxes to add selectable options to your application.

**LEVEL:** Beginner<br/>
**PLATFORMS:** Windows, macOS, Linux, iOS, Android<br/>
**CLASSES:** [ToggleButton](https://docs.juce.com/master/classToggleButton.html "A button that can be toggled on/off."), [Button](https://docs.juce.com/master/classButton.html "A base class for buttons."), [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked."), [String](https://docs.juce.com/master/classString.html "The JUCE String class!")

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/RadioButtonsAndCheckboxesTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/RadioButtonsAndCheckboxesTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project illustrates the use of radio buttons and checkboxes in JUCE. These are normally implemented using the [ToggleButton](https://docs.juce.com/master/classToggleButton.html "A button that can be toggled on/off.") class, although any JUCE [Button](https://docs.juce.com/master/classButton.html "A base class for buttons.") component can be used as an individual toggle button or as part of a \"radio group\". The demo application presents some headings (using the [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") class) and a series of [ToggleButton](https://docs.juce.com/master/classToggleButton.html "A button that can be toggled on/off.") components:

![](/_images/tutorial_radio_buttons_checkboxes_screenshot1.png "The demo application")

# Radio buttons and checkboxes

This tutorial examines the use of the [Button](https://docs.juce.com/master/classButton.html "A base class for buttons.") class for creating on/off style options. Normally you will use the [ToggleButton](https://docs.juce.com/master/classToggleButton.html "A button that can be toggled on/off.") class for this, but as mentioned above, any button can be set up to be a toggle-type button. The standard [ToggleButton](https://docs.juce.com/master/classToggleButton.html "A button that can be toggled on/off.") component comprises a piece of text and a \"bubble\" to the left of the text that either contains a tick, or not. (And this appearance can be customised, if you wish, using the [LookAndFeel](https://docs.juce.com/master/classLookAndFeel.html "LookAndFeel objects define the appearance of all the JUCE widgets, and subclasses can be used to appl...") class.) These two states can be seen in the screenshot in the previous section.

In the `MainContentComponent` constructor, the \"male\" and \"female\" buttons are set up as follows:

```cpp
addAndMakeVisible (maleButton);
addAndMakeVisible (femaleButton);
maleButton.onClick = [this] { updateToggleState (&maleButton, "Male"); };
femaleButton.onClick = [this] { updateToggleState (&femaleButton, "Female"); };
```

Here you can see that [ToggleButton](https://docs.juce.com/master/classToggleButton.html "A button that can be toggled on/off.") objects are configured in the same way as [TextButton](https://docs.juce.com/master/classTextButton.html "A button that uses the standard lozenge-shaped background with a line of text on it.") objects (as seen in [Tutorial: Listeners and Broadcasters](/tutorials/tutorial_listeners_and_broadcasters/)). To make these two toggle buttons mutually exclusive, we need to add them to the same \"radio group\". This will allow only one of these buttons to be toggled on at any one time (and toggling the other button on will, in turn, switch the other one off). A _radio group ID_ is a non-zero integer that is used to identify the group of buttons than must be mutually exclusive. In addition to this, all buttons with the same _radio group ID_ must be child components of the same parent component for this to work. (You would need to implement this mutually exclusive behaviour yourself if the buttons are within different parent components or different windows.)

We use an enumerated value (`GenderButtons` ) as our _radio group ID_ in this example:

```cpp
enum RadioButtonIds {
    GenderButtons = 1001
};
```

Then we use the [Button::setRadioGroupId()](https://docs.juce.com/master/classButton.html#a29adebc6ed27829f75b69c6ba40ef958 "Enables the button to act as a member of a mutually-exclusive group of 'radio buttons'.") function to set the _radio group ID_.

```cpp
maleButton.setRadioGroupId (GenderButtons);
femaleButton.setRadioGroupId (GenderButtons);
```

> [!TIP]
>Notice that the `setRadioGroupId()` function is a member of the [Button](https://docs.juce.com/master/classButton.html "A base class for buttons.") class, not just the [ToggleButton](https://docs.juce.com/master/classToggleButton.html "A button that can be toggled on/off.") class. Any button can be part of a radio group, but it probably only makes sense if that button is set up to expect to be toggled on and off (rather than simply intercept click events).

The other three [ToggleButton](https://docs.juce.com/master/classToggleButton.html "A button that can be toggled on/off.") objects are configured without adding them to a radio group:

```cpp
addAndMakeVisible (sportButton);
addAndMakeVisible (artButton);
addAndMakeVisible (filmButton);
sportButton.onClick = [this] { updateToggleState (&sportButton, "Sport"); };
artButton.onClick = [this] { updateToggleState (&artButton, "Art"); };
filmButton.onClick = [this] { updateToggleState (&filmButton, "Film"); };
```

> [!TIP]
>[Button](https://docs.juce.com/master/classButton.html "A base class for buttons.") objects can be removed from a radio group by calling the [Button::setRadioGroupId()](https://docs.juce.com/master/classButton.html#a29adebc6ed27829f75b69c6ba40ef958 "Enables the button to act as a member of a mutually-exclusive group of 'radio buttons'.") function with a zero argument.

## Responding to toggle state changes

Responding to buttons that can be toggled on or off is similar to responding to regular button clicks. We need to specify the function we want to call when the button is toggled by assigning a lambda function to the [Button::onClick](https://docs.juce.com/master/classButton.html#a30b76ab312dc7f66e67596ae20540ec2 "You can assign a lambda to this callback object to have it called when the button is clicked.") helper object (as you can see in the code snippets above).

The difference is that we also need to check the toggle state of the button in our updateToggleState() function. To do this we can call the [Button::getToggleState()](https://docs.juce.com/master/classButton.html#ae1723837340c174f0f212855d247b626 "Returns true if the button is 'on'.") function. Again this is a member of the [Button](https://docs.juce.com/master/classButton.html "A base class for buttons.") class, so this is valid for any button (but again, probably only makes sense if you expect the button to be toggled on and off, as it will return `false` in other cases).

```cpp
void updateToggleState (juce::Button* button, juce::String name)
{
    auto state = button->getToggleState();
```

In our example we just post the toggle changes to the logger as they happen:

```cpp
juce::String stateString = state ? "ON" : "OFF";

juce::Logger::outputDebugString (name + " Button changed to " + stateString);
}
```

Notice in particular, that when you toggle on either the \"Male\" or \"Female\" button that the other button turns \"off\" _and_ this is reported by the `updateToggleState()` function.

> [!NOTE]
> Add some additional [ToggleButton](https://docs.juce.com/master/classToggleButton.html "A button that can be toggled on/off.") objects, of course you can easily add more hobbies. But you could also add more options under the gender category, for example \"Prefer not to say\" or \"Other\".

## Using other buttons as toggles

As hinted at above, _any_ button can be used as a toggle button. To illustrate, let's convert all of the [ToggleButton](https://docs.juce.com/master/classToggleButton.html "A button that can be toggled on/off.") objects in the demo project into [TextButton](https://docs.juce.com/master/classTextButton.html "A button that uses the standard lozenge-shaped background with a line of text on it.") objects. The obvious thing that we need to do here is change the type of the member variables to the [TextButton](https://docs.juce.com/master/classTextButton.html "A button that uses the standard lozenge-shaped background with a line of text on it.") class:

```cpp
juce::Label genderLabel { {}, "I identify my gender as..." };
juce::ToggleButton maleButton { "Male" },
    femaleButton { "Female" };

juce::Label hobbiesLabel { {}, "My hobbies are..." };
juce::ToggleButton sportButton { "Sport" },
    artButton { "Art" },
    filmButton { "Film" };

//==============================================================================
JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR (MainContentComponent)
};
```

But in order to make the buttons toggle-able using mouse clicks, we need to call the [Button::setClickingTogglesState()](https://docs.juce.com/master/classButton.html#a675d6848c16e265ff334b2d51328d8d2 "This tells the button to automatically flip the toggle state when the button is clicked.") function. To do this, add the following code anywhere within the `MainContentComponent` constructor:

```cpp
maleButton.setClickingTogglesState (true);
femaleButton.setClickingTogglesState (true);
sportButton.setClickingTogglesState (true);
artButton.setClickingTogglesState (true);
filmButton.setClickingTogglesState (true);
```

Running this code will result in something that can be used like the original application but has a different appearance:

![](/_images/tutorial_radio_buttons_checkboxes_screenshot2.png "Using text buttons as toggle buttons")

> [!TIP]
>See [Tutorial: Colours in JUCE](/tutorials/tutorial_colours/) for guidance on customising the colours of the text button in its on and off states.

## Using text to display button toggle state

There may be cases where you want to change the text displayed depending on the button's toggle state, too. To do this, you can simply call the [Button::setButtonText()](https://docs.juce.com/master/classButton.html#a96f4185a0a716d10309f1081f0af2f91 "Changes the button's text.") function within your `updateToggleState()` function. To try this, revise the `updateToggleState()` function as follows:

```cpp
void updateToggleState (juce::Button* button, juce::String name)
{
    auto state = button->getToggleState();
    juce::String stateString = state ? "ON" : "OFF";
    juce::String selectedString = state ? " (selected)" : "";

    juce::Logger::outputDebugString (name + " Button changed to " + stateString);
    button->setButtonText (name + selectedString);
}
```

In this case, we just append the text \"(selected)\" to the normal button text if the button is in its toggled-on state.

> [!TIP]
>This technique will work with both [TextButton](https://docs.juce.com/master/classTextButton.html "A button that uses the standard lozenge-shaped background with a line of text on it.") components, as shown here, and [ToggleButton](https://docs.juce.com/master/classToggleButton.html "A button that can be toggled on/off.") components.

Now our application will looks similar to the following screenshot:

![](/_images/tutorial_radio_buttons_checkboxes_screenshot3.png "Changing button text based on toggle state")

# Summary

In this tutorial we have introduced the use of checkbox and radio buttons style controls. In particular you should be able to do the following things in your own applications:

- Use [ToggleButton](https://docs.juce.com/master/classToggleButton.html "A button that can be toggled on/off.") objects and respond to changes in state.
- Change a [ToggleButton](https://docs.juce.com/master/classToggleButton.html "A button that can be toggled on/off.") object's text, depending on its state.
- Group [ToggleButton](https://docs.juce.com/master/classToggleButton.html "A button that can be toggled on/off.") objects to form \"radio groups\" when only one button within a group may be on at any one time.
- Use other [Button](https://docs.juce.com/master/classButton.html "A base class for buttons.") components as toggles.

# See also

- [Tutorial: Listeners and Broadcasters](/tutorials/tutorial_listeners_and_broadcasters/)
- [Tutorial: Colours in JUCE](/tutorials/tutorial_colours/)
- [Tutorial: The Label class](/tutorials/tutorial_label/)
- [Tutorial: The ComboBox class](/tutorials/tutorial_combo_box/)
- [Tutorial: The Slider class](/tutorials/tutorial_slider_values/)
