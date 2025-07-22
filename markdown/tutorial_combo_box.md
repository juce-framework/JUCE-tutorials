---
title: The ComboBox class
---
# Tutorial: The ComboBox class

This tutorial introduces the [ComboBox](https://docs.juce.com/master/classComboBox.html "A component that lets the user choose from a drop-down list of choices.") class, which is a component for displaying lists of items to the user. The contents of a [ComboBox](https://docs.juce.com/master/classComboBox.html "A component that lets the user choose from a drop-down list of choices.") object can be modified dynamically, and can be used for text input, too.

**LEVEL:** Beginner<br/>
**PLATFORMS:** Windows, macOS, Linux, iOS, Android<br/>
**CLASSES:** [ComboBox](https://docs.juce.com/master/classComboBox.html "A component that lets the user choose from a drop-down list of choices."), [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked."), [Font](https://docs.juce.com/master/classFont.html "Represents a particular font, including its size, style, etc."), [Colour](https://docs.juce.com/master/classColour.html "Represents a colour, also including a transparency value."), [Colours](namespaceColours.html "Contains a set of predefined named colours (mostly standard HTML colours)")

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/ComboBoxTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/ComboBoxTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project displays a piece of text at the top of the window within a [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") component (see [Tutorial: The Label class](/tutorials/tutorial_label/)). A [ComboBox](https://docs.juce.com/master/classComboBox.html "A component that lets the user choose from a drop-down list of choices.") component contains the items **Plain** , **Bold** , and **Italic** . The user can select one of these items to change the style of the text within the label.

![](/_images/tutorial_combo_box_screenshot1.png "Selecting font styles")

# The ComboBox class

This tutorial introduces many of the features of the [ComboBox](https://docs.juce.com/master/classComboBox.html "A component that lets the user choose from a drop-down list of choices.") class. A [ComboBox](https://docs.juce.com/master/classComboBox.html "A component that lets the user choose from a drop-down list of choices.") component contains a list of text strings. Each of these text strings is associated with an ID number (an `int` value). You can query which item is currently selected, either by:

- retrieving the currently selected ID using the [ComboBox::getSelectedId()](https://docs.juce.com/master/classComboBox.html#ab6232527104faad901ba0fa1380cd8ec "Returns the ID of the item that's currently shown in the box.") function; or
- requesting the currently displayed text using the [ComboBox::getText()](https://docs.juce.com/master/classComboBox.html#a420630894c696545c29271f557f7614d "Returns the text that is currently shown in the combo-box's text field.") function.

The [ComboBox](https://docs.juce.com/master/classComboBox.html "A component that lets the user choose from a drop-down list of choices.") class is also a broadcaster. To listen for changes you can register a [ComboBox::Listener](https://docs.juce.com/master/classComboBox_1_1Listener.html "A class for receiving events from a ComboBox.") class (see [Tutorial: Listeners and Broadcasters](/tutorials/tutorial_listeners_and_broadcasters/)) or alternatively use a Lambda function with the [ComboBox::onChange](https://docs.juce.com/master/classComboBox.html#a9cf2e20990541b9fbb539cd4a8e0ac4e "You can assign a lambda to this callback object to have it called when the selected ID is changed.") helper object.

Let's examine the demo project. In our `MainContentComponent` class we have four private members:

```cpp
juce::Label textLabel { {}, "The quick brown fox jumps over the lazy dog." };
juce::Font textFont { 12.0f };
juce::ComboBox styleMenu;
```

The [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") and [Font](https://docs.juce.com/master/classFont.html "Represents a particular font, including its size, style, etc.") objects are configured in the constructor:

```cpp
MainContentComponent()
{
    addAndMakeVisible (textLabel);
    textLabel.setFont (textFont);
```

## Adding items

Items can be added to a [ComboBox](https://docs.juce.com/master/classComboBox.html "A component that lets the user choose from a drop-down list of choices.") object one at a time using the [ComboBox::addItem()](https://docs.juce.com/master/classComboBox.html#a37491da45f1cbb74e47f145e5664d8bf "Adds an item to be shown in the drop-down list.") function. Here we add the \"Plain\", \"Bold\", and \"Italic\" items with ID numbers 1, 2, and 3 respectively:

```cpp
// add items to the combo-box
addAndMakeVisible (styleMenu);
styleMenu.addItem ("Plain", 1);
styleMenu.addItem ("Bold", 2);
styleMenu.addItem ("Italic", 3);

styleMenu.onChange = [this] { styleMenuChanged(); };
styleMenu.setSelectedId (1);

setSize (400, 200);
}
```

## Responding to changes

We could have registered our `MainContentComponent` object as a listener to get notified when the `styleMenu` [ComboBox](https://docs.juce.com/master/classComboBox.html "A component that lets the user choose from a drop-down list of choices.") object is changed by the user in the application. But in this case we implement the [ComboBox::onChange](https://docs.juce.com/master/classComboBox.html#a9cf2e20990541b9fbb539cd4a8e0ac4e "You can assign a lambda to this callback object to have it called when the selected ID is changed.") helper object to directly call the `styleMenuChanged()` function:

```cpp
styleMenu.onChange = [this] { styleMenuChanged(); };
```

The `styleMenuChanged()` function handles changes to the `styleMenu` object:

```cpp
void styleMenuChanged()
{
    switch (styleMenu.getSelectedId())
    {
        case 1:
            textFont.setStyleFlags (juce::Font::plain);
            break;
        case 2:
            textFont.setStyleFlags (juce::Font::bold);
            break;
        case 3:
            textFont.setStyleFlags (juce::Font::italic);
            break;
        default:
            break;
    }

    textLabel.setFont (textFont);
}
```

Here you can see that we set the `textFont` [Font](https://docs.juce.com/master/classFont.html "Represents a particular font, including its size, style, etc.") object appropriately based on the user's selection. Then we use this font to update the `textLabel` [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") object's font.

## Using item ID numbers

You can use any integer as an item ID _except_ zero. Zero has a special meaning. It is used to indicate that none of the items are selected (either an item hasn\'t been selected yet or the [ComboBox](https://docs.juce.com/master/classComboBox.html "A component that lets the user choose from a drop-down list of choices.") object is displaying some other custom text).

With a small selection of items using simple numbers like 1, 2, and 3 in your code is easy to manage. This would quickly become unmanageable as you develop your app further. In our case it would be clearer to use an `enum` . Let's add a private `enum` for our styles:

```cpp
enum FontStyles {
    stylePlain = 1,
    styleBold,
    styleItalic,
    numberOfStyles
};
```

Then we can use these values when setting up the [ComboBox](https://docs.juce.com/master/classComboBox.html "A component that lets the user choose from a drop-down list of choices.") object:

```cpp
addAndMakeVisible (styleMenu);

// add items to the combo-box
styleMenu.addItem ("Plain", 1);
styleMenu.addItem ("Bold", 2);
styleMenu.addItem ("Italic", 3);

styleMenu.onChange = [this] { styleMenuChanged(); };
styleMenu.setSelectedId (stylePlain);
```

And in our `styleMenuChanged()` function:

```cpp
void styleMenuChanged()
{
    switch (styleMenu.getSelectedId())
    {
        case stylePlain:
            textFont.setStyleFlags (juce::Font::plain);
            break;
        case styleBold:
            textFont.setStyleFlags (juce::Font::bold);
            break;
        case styleItalic:
            textFont.setStyleFlags (juce::Font::italic);
            break;
    }

    textLabel.setFont (textFont);
}
```

At the very least this make the code much more readable.

Other common strategies for using ID numbers are:

- Using an array to store data relevant to the items in the combo-box. You can use the ID as an index into the array although you will either need to leave index 0 unused or use an offset. For example, your IDs could start at 100 (100, 101, 102, and so on). ID 100 would associate with index 0 in the array and you can then easily convert between IDs and indices by adding or subtracting 100.
- Using different batches of integers in different ranges for each of the combo-box in your app. For example, you could use IDs 100, 101, 102... for one combo-box, 200, 201, 202... for another, and so on.
- Using a hash of the item text. You could use [String::hashCode()](https://docs.juce.com/master/classString.html#ad8ca5cb11984df03ff9697442da5ae4b "Generates a probably-unique 32-bit hashcode from this string.") to get a _probably_ unique hashcode for the string. But, be aware that this may return zero (using this function on an empty string returns a hashcode of zero).

## Sections and dividers

The list of items in a combo-box can contain dividers and section headings. This is especially useful in very long lists. Let's add a combo-box to our app that changes the text colour. First, let's add another `enum` for our colours:

```cpp
enum TextColours {
    black = 1,
    white,
    red,
    darkred,
    indianred,
    green,
    darkgreen,
    lightgreen,
    blue,
    darkblue,
    lightblue,
    numberOfColours
};
```

And a new member to our `MainContentComponent` class:

```cpp
juce::Label textLabel { {}, "The quick brown fox jumps over the lazy dog." };
juce::Font textFont { 12.0f };
juce::ComboBox styleMenu;
juce::ComboBox coloursMenu;
```

In the `MainContentComponent` constructor we need to add the code to set up the new combo-box. Here we introduce two new functions, [ComboBox::addSeparator()](https://docs.juce.com/master/classComboBox.html#af53bb419bb136f11f6f0342702d8d902 "Adds a separator line to the drop-down list.") and [ComboBox::addSectionHeading()](https://docs.juce.com/master/classComboBox.html#ad376928ef517800c42628399e1a80990 "Adds a heading to the drop-down list, so that you can group the items into different sections."):

```cpp
addAndMakeVisible (coloursMenu);

coloursMenu.addItem ("Black", black);
coloursMenu.addItem ("White", white);
coloursMenu.addSeparator();

coloursMenu.addSectionHeading ("Reds");
coloursMenu.addItem ("Red", red);
coloursMenu.addItem ("Dark Red", darkred);
coloursMenu.addItem ("Indian Red", indianred);
coloursMenu.addSeparator();

coloursMenu.addSectionHeading ("Greens");
coloursMenu.addItem ("Green", green);
coloursMenu.addItem ("Dark Green", darkgreen);
coloursMenu.addItem ("Light Green", lightgreen);
coloursMenu.addSeparator();

coloursMenu.addSectionHeading ("Blues");
coloursMenu.addItem ("Blue", blue);
coloursMenu.addItem ("Dark Blue", darkblue);
coloursMenu.addItem ("Light Blue", lightblue);

coloursMenu.onChange = [this] { coloursMenuChanged(); };
coloursMenu.setSelectedId (black);

coloursMenu.setEditableText (true);

setSize (400, 200);
}
```

You can use separators without section headings and section headings without separators but they do work well together.

> [!TIP]
>Separators and section headings can\'t be selected and don\'t have associated ID numbers.

We need to implement a new function to handle the changes to the `coloursMenu` object:

```cpp
void coloursMenuChanged()
{
    juce::Colour textColour;

    switch (coloursMenu.getSelectedId())
    {
        case black:
            textColour = Colours::black;
            break;
        case white:
            textColour = Colours::white;
            break;

        case red:
            textColour = Colours::red;
            break;
        case darkred:
            textColour = Colours::darkred;
            break;
        case indianred:
            textColour = Colours::indianred;
            break;

        case green:
            textColour = Colours::green;
            break;
        case darkgreen:
            textColour = Colours::darkgreen;
            break;
        case lightgreen:
            textColour = Colours::lightgreen;
            break;

        case blue:
            textColour = Colours::blue;
            break;
        case darkblue:
            textColour = Colours::darkblue;
            break;
        case lightblue:
            textColour = Colours::lightblue;
            break;
    }

    textLabel.setColour (juce::Label::textColourId, textColour);
}
```

Don\'t forget to set the bounds of the `coloursMenu` object in the `resized()` function:

```cpp
void resized() override
{
    textLabel.setBounds (10, 10, getWidth() - 20, 20);
    styleMenu.setBounds (10, 40, getWidth() - 20, 20);
    coloursMenu.setBounds (10, 70, getWidth() - 20, 20);
}
```

Run the project and the window should look something like this:

![](/_images/tutorial_combo_box_screenshot2.png "Selecting colours and styles")

You can now change the colour of the label text.

## Text entry

The default behaviour of a combo-box is to allow the user to select only the items listed. But, you can make the combo-box editable to allow the user to enter other text. In the `MainContentComponent` constructor let's make our `coloursMenu` object editable:

```cpp
coloursMenu.setEditableText (true);
```

Now let's add a function that takes the text from the `coloursMenu` object and treats the text as a hexadecimal colour code. The colour code is in the ARGB format (for example, \"ff888888\"\" would be an opaque mid-grey colour): \@snippet GUI/ComboBoxTutorial/ComboBoxTutorial_02.h handleColourText Notice that we convert the string to a hex value then convert it back to a string, setting the combo-box text. This is a simple way to filter the text entered. \@note A more sophisticated method to filter input text would be to implement a custom class that derives from the ComboBox class. Then you can override its Label::Listener::editorShown() virtual function to customise its text editor. You can use the TextEditor::setInputFilter() function to use a TextEditor::InputFilter object to filter the text entered before it even appears on the screen. In our \<code class=\"project_code\"\>coloursMenuChanged()\</code\> function we need to update the \<code class=\"code\"\>switch..case\</code\> to call our \<code class=\"project_code\"\>handleColourText()\</code\> function in the \<code class=\"code\"\>default\</code\> case: \@snippet GUI/ComboBoxTutorial/ComboBoxTutorial_02.h coloursMenuChanged \@image html tutorial_combo_box_screenshot3.png \"Entering custom text\" Run the application and you should be able to enter custom colours in the six-digit hex format (RGB). \<dl class=\"section attention\"\>\<dt\>Exercise\</dt\>\<dd\> Modify the code to add any custom hex colours to the \<code class=\"project_code\"\>coloursMenu\</code\> combo-box so that the user can choose these items again if they wish. \</dd\>\</dl\> \@subsection tutorial_combo_box_disabling_enabling Disabling and enabling items Items in a combo-box can be disabled and enabled (they are, of course, enabled by default). When they are disabled they will still be visible but shown greyed-out. Let's change the code so that none of the lighter versions colours (and white) can be selected if the bold style is selected. First, let's add a function to disable or enable the lighter colours using the ComboBox::setItemEnabled() function: \@snippet GUI/ComboBoxTutorial/ComboBoxTutorial_02.h setLightColoursEnabled Now let's refactor our \<code class=\"project_code\"\>styleMenuChanged()\</code\> function and make use of the \<code class=\"project_code\"\>setLightColoursEnabled()\</code\> function: \@snippet GUI/ComboBoxTutorial/ComboBoxTutorial_02.h styleMenuChanged Notice that in the \<code class=\"project_code\"\>setStyleBold()\</code\> function we disable the lighter colours and we need to enable them in the other cases. Run the application and you should now see that the lighter colours are disabled when the \<b class=\"onscreen_text\"\>Bold\</b\> style is selected. \@image html tutorial_combo_box_screenshot4.png \"Disabling combo-box items\" \<dl class=\"section attention\"\>\<dt\>Exercise\</dt\>\<dd\> One problem here is that you can select a lighter colour \<em\>then\</em\> choose the bold style. Modify the code to change the colour back to \<b class=\"onscreen_text\"\>black\</b\> if one of the lighter colours is selected when the user selects the \<b class=\"onscreen_text\"\>Bold\</b\> style. \</dd\>\</dl\> \@section tutorial_combo_box_notes Notes The final implementation of the code for this tutorial can be found in the \<code class=\"filename\"\>ComboBoxTutorial_02.h file of the demo project.

# Summary

This tutorial has introduced the [ComboBox](https://docs.juce.com/master/classComboBox.html "A component that lets the user choose from a drop-down list of choices.") class. After reading this tutorial you should be able to:

- Create a combo-box and add items to it.
- Manage ID numbers associated with the items in your combo-box.
- Respond to users changing the item selected in the combo-box.
- Allow custom text to be entered into the combo-box.
- Disable and enable items in the combo-box.

# See also

- [Tutorial: The Label class](/tutorials/tutorial_label/)
- [Tutorial: The TableListBox class](/tutorials/tutorial_table_list_box/)
- [Tutorial: Listeners and Broadcasters](/tutorials/tutorial_listeners_and_broadcasters/)
