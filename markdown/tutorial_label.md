# Tutorial: The Label class

This tutorial introduces the [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") class, which is a component for displaying text. A [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") component can also be set to be editable so it's really useful for displaying text and simple text entry.

**LEVEL:** Beginner<br/>
**PLATFORMS:** Windows, macOS, Linux, iOS, Android<br/>
**CLASSES:** [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked."), [TextEditor](https://docs.juce.com/master/classTextEditor.html "An editable text box."), [Font](https://docs.juce.com/master/classFont.html "Represents a particular font, including its size, style, etc."), [Colours](namespaceColours.html "Contains a set of predefined named colours (mostly standard HTML colours)")

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/LabelTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/LabelTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project includes a number of labels. Some of these are for displaying text, but one of them is for entering text. Run the demo project and it should look like the following screenshot:

![](/_images/tutorial_label_screenshot1.png "The initial appearance of the application")

When you click on the label with the white background it becomes editable, the label shows a subtle border, and a caret appears. This is shown in the following screenshot.

![](/_images/tutorial_label_screenshot2.png "The application with the text entry label focused")

Text can then be entered into this label as shown below:

![](/_images/tutorial_label_screenshot3.png "Entering text into the text entry label")

When you hit return or click away from the label it goes out of its editable state. This commits a change that is broadcasted to its listeners. In this case we convert the text to uppercase and display it in another label:

![](/_images/tutorial_label_screenshot4.png "The text is converted to uppercase and displayed in another label")

This illustrates several different uses of the [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") class for both displaying and entering text. There are actually five labels here that are used in slightly different ways:

- The green label at the top that is used as a heading.
- The two orange labels that are used to display other fixed text.
- The two rectangular boxes that are used to display dynamically changing text (one of which is to allow text entry from the user).

The second of the rectangular boxes is very slightly greyed out to distinguish it from the text entry box.

# Displaying text

This tutorial covers most of the main features of the [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") class. The most fundamental feature of a label is that it displays some text! While you can draw your own text in your component's `paint()` function (see [Tutorial: The Graphics class](/tutorials/tutorial_graphics_class/)) it is usually much more convenient to manage the layout of text using [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") objects (or [TextEditor](https://docs.juce.com/master/classTextEditor.html "An editable text box.") objects in some cases).

As you would expect, the text within a [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") object can be displayed in any font, font size, bold, italics, and so on. The text can also be justified (aligned) within the component bounds to the left, right, centre, top, bottom, and using various other options.

> [!TIP]
>The [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") class can only contain a single run of text in the same font, size, and justification. To display multiple runs of text in different styles (as you would find in a word processor application) see the [TextEditor](https://docs.juce.com/master/classTextEditor.html "An editable text box.") class. The [TextEditor](https://docs.juce.com/master/classTextEditor.html "An editable text box.") class can also include horizontal and vertical scroll bars to display large amounts of text.

If the bounds of the component are too small to display all of the text requested then the [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") class will try a few things to make the text fit. First, it will make the glyphs of the font a little narrower. If the glyphs are too narrow then the text will be unreadable, so beyond a certain point it gives up. In this case it truncates the text showing an ellipsis (...) at the end to show that this has been done. You can try this now by typing in the text \"The quick brown fox jumps over the lazy dog\" in the text entry label. In fact you can copy the text (\"CMD-C\" to copy on Mac OS X or \"Ctrl-C\" on Windows) from this page and paste it into the label using keyboard shortcuts (\"CMD-V\" to paste on Mac OS X or \"Ctrl-V\" on Windows). This capability is built in without you needing to add any code!

The result should be something like this:

![](/_images/tutorial_label_screenshot5.png "Two ways that the Label class copes with too much text")

The original text just about fits but it is slightly squashed in the horizontal axis. The uppercase version is going to be a little wider since uppercase glyphs are usually wider than lowercase glyphs. The [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") class decided that the uppercase version is too wide for it to squash the text in the horizontal axis and keep it readable. In this case it has truncated the text at the \"L\" in \"LAZY\" and added the ellipsis.

> [!TIP]
>You can change the amount by which the text will be scaled using the [Label::setMinimumHorizontalScale()](https://docs.juce.com/master/classLabel.html#a3529019ec38b097daec0fb97fe7218d1 "Specifies the minimum amount that the font can be squashed horizontally before it starts using ellips...") function for each [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") object. Using a value of `1.0f` will disable the scaling. You can measure the height required for a single line in a particular font using the [Font::getHeight()](https://docs.juce.com/master/classFont.html#a96e4ccc975ce4180c10107aa77ea8c7d "Returns the total height of this font, in pixels.") function. To measure the width that an entire string would require to fit on a single line you can use the [Font::getStringWidth()](https://docs.juce.com/master/classFont.html#a23539f1990fd39341b84566a413c051d "Returns the total width of a string as it would be drawn using this font.") function.

## Basic label configuration

In the `MainContentComponent` constructor each of the labels in our application is configured. First we set up the title with a 16-point bold font, the text it should contain, the colour of the text, and we justify the text to the centre.

Add the label to the parent component:

```cpp
addAndMakeVisible (titleLabel);
```

Set the font for the label:

```cpp
titleLabel.setFont (juce::Font (16.0f, juce::Font::bold));
```

Set the text to be displayed within the label:

```cpp
titleLabel.setText ("Click in the white box to enter some text...", juce::dontSendNotification);
```

Set the colour of the label text:

```cpp
titleLabel.setColour (juce::Label::textColourId, juce::Colours::lightgreen);
```

Justify the display of the label text:

```cpp
titleLabel.setJustificationType (juce::Justification::centred);
```

> [!TIP]
>The [Justification::centred](https://docs.juce.com/master/classJustification.html#a1f8c07756c56fe8f31ed44964e51bfbcaf9d9adde20dce1aa38ff9a69be2b4384 "Indicates that the item should be centred vertically and horizontally.") value centres the text in both vertical and horizontal axes. Have a look at the [Justification](https://docs.juce.com/master/classJustification.html "Represents a type of justification to be used when positioning graphical items.") class for more options.

## Attaching a label to another component

As its name suggests, a [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") component is often used as a label for another component. In this case a [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") object can be _attached_ to another component. Once a label is attached only the _owner_ component needs to be positioned. The attached label will follow its owner around the parent component. In our case we want the `inputLabel` object (that displays the text **Text input:** ) to be attached to the `inputText` object. The second argument to the [Label::attachToComponent()](https://docs.juce.com/master/classLabel.html#a3c2397c0da1249f9e27e2279e0f2d4eb "Makes this label "stick to" another component.") function states whether the label should be attached to the left or above the owner. The `true` argument means that it should be attached to the left:

```cpp
addAndMakeVisible (inputLabel);
inputLabel.setText ("Text input:", juce::dontSendNotification);
inputLabel.attachToComponent (&inputText, true);
inputLabel.setColour (juce::Label::textColourId, juce::Colours::orange);
inputLabel.setJustificationType (juce::Justification::right);
```

The labels to display the uppercase text are set up similarly:

```cpp
addAndMakeVisible (uppercaseLabel);
uppercaseLabel.setText ("Uppercase:", juce::dontSendNotification);
uppercaseLabel.attachToComponent (&uppercaseText, true);
uppercaseLabel.setColour (juce::Label::textColourId, juce::Colours::orange);
uppercaseLabel.setJustificationType (juce::Justification::right);

addAndMakeVisible (uppercaseText);
uppercaseText.setColour (juce::Label::backgroundColourId, juce::Colours::darkblue);
```

In the `resized()` function we need to position only three of our five labels:

```cpp
void resized() override
{
    titleLabel.setBounds (10, 10, getWidth() - 20, 30);
    inputText.setBounds (100, 50, getWidth() - 110, 20);
    uppercaseText.setBounds (100, 80, getWidth() - 110, 20);
}
```

## Making a label editable

The final label that we set up in the `MainContentComponent` constructor is the editable text field.

```cpp
addAndMakeVisible (inputText);
inputText.setEditable (true);
inputText.setColour (juce::Label::backgroundColourId, juce::Colours::darkblue);
inputText.onTextChange = [this] { uppercaseText.setText (inputText.getText().toUpperCase(), juce::dontSendNotification); };
```

> [!TIP]
>The default behaviour when setting the label to editable using the [Label::setEditable()](https://docs.juce.com/master/classLabel.html#a9342a397932d82bb1d196ebf9a8969d8 "Makes the label turn into a TextEditor when clicked.") function is to show the editor on a single click. The label can be set up to be editable on a double-click instead. If the user clicks off the label after entering some text then this will be confirmed even if the user doesn\'t hit the \"Return\" key. You can also changes this to make sure that users must hit the \"Return\" key to confirm data entry, if you prefer. See the API reference for this function for more information.

<!-- -->

> [!NOTE]
> Exercise: Add a button to the interface and use this to clear the text entry label. (See [Tutorial: Listeners and Broadcasters](/tutorials/tutorial_listeners_and_broadcasters/).)

## Responding to changes

The lambda function is the callback for the [Label::onTextChange](https://docs.juce.com/master/classLabel.html#a5ba80ea5ea965a001e2eacf75fc0314b "You can assign a lambda to this callback object to have it called when the label text is changed.") helper object. Here we get the entered text, convert it to uppercase using the [String](https://docs.juce.com/master/classString.html "The JUCE String class!") class, and set the text of the `uppercaseText` label.

```cpp
inputText.onTextChange = [this] { uppercaseText.setText (inputText.getText().toUpperCase(), dontSendNotification); };
```

> [!TIP]
>We can use `sendNotification` instead of `dontSendNotification` when calling [Label::setText()](https://docs.juce.com/master/classLabel.html#a3f0ca22cb63e924d3db23da48c210790 "Changes the label text."). This will cause a label's listeners to be notified of the change (if the text is different from its current contents) in addition to the user changing the text from the user interface.

<!-- -->

> [!NOTE]
> Exercise: Add another label that displays the text entered in lowercase.

# Other customisations

There are a few other customisations that can be made. In particular labels can display multiline text. You may also want to customise the editor for editable labels.

## Displaying multiline text

While the [TextEditor](https://docs.juce.com/master/classTextEditor.html "An editable text box.") class is more flexible for large amounts of text, the [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") class can display multiple lines of text. To do this, you just need to ensure that the height of the component is large enough to display more than one line of text.

To illustrate this, add the following member to the `MainContentComponent` class:

```cpp
juce::Label infoLabel;
```

And add the following code to the `MainContentComponent` constructor:

```cpp
addAndMakeVisible (infoLabel);
juce::String infoText;
infoText << "This simple application takes some text input from the user, ";
infoText << "converts it to uppercase, and displays this in another label. ";
infoText << "The application demonstrates a number of useful features of the Label class.";
infoLabel.setText (infoText, juce::dontSendNotification);
infoLabel.setColour (juce::Label::backgroundColourId, juce::Colours::darkblue);
```

Finally, set the bounds of the `infoLabel` component in the `resized()` function:

```cpp
infoLabel.setBounds (10, 110, getWidth() - 20, getHeight() - 120);
```

> [!TIP]
>You might need to adjust the bounds a little if you added a button and another label in the earlier exercises.

Run the application and it should look something like this:

![](/_images/tutorial_label_screenshot6.png "Displaying multiline text")

## Changing the editor appearance

Some of the editor features can be set via the [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") class itself. When the editor is created the [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") class copies the font style and colours to the editor. Some of the [Label::ColourIds](https://docs.juce.com/master/classLabel.html#a41756012394513222e1323bb432fa701 "A set of colour IDs to use to change the colour of various aspects of the label.") values relate to the editor. For example, to change the border around the label when it is being edited you can use the [Label::outlineWhenEditingColourId](https://docs.juce.com/master/classLabel.html#a41756012394513222e1323bb432fa701a6c7adfce6c830c9faa94efd3445f78cf "An optional border colour when the label is being edited.") value for the Component::setColour() function:

```cpp
addAndMakeVisible (inputText);
inputText.setEditable (true);
inputText.setColour (juce::Label::backgroundColourId, juce::Colours::darkblue);
inputText.setColour (juce::Label::outlineWhenEditingColourId, juce::Colours::orangered);
```

Other customisations can be achieved by implementing the [Label::onEditorShow](https://docs.juce.com/master/classLabel.html#ae8e66977501dc15f678cc1b80fa218ce "You can assign a lambda to this callback object to have it called when the label's editor is shown.") helper object. For example, you could make the text italic when being edited like this:

```cpp
inputText.onEditorShow = [this] {
    auto* editor = inputText.getCurrentTextEditor();

    auto editorFont = editor->getFont();
    editorFont.setItalic (true);
    editor->setFont (editorFont);
};
```

> [!TIP]
>In practice --- especially for large-scale applications --- you would probably use the [LookAndFeel](https://docs.juce.com/master/classLookAndFeel.html "LookAndFeel objects define the appearance of all the JUCE widgets, and subclasses can be used to appl...") class to customise the appearance of all of your components for consistency across your application.

# Summary

This tutorial has examined the [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") class. While it is very easy to use, there are a number of useful features that make it powerful. We have covered:

- Changing the font of the text displayed in the label.
- Changing the colours of the text displayed in the label.
- Using a label as a simple text editor and customising its appearance.
- Displaying multiline text.

# See also

- [Tutorial: Colours in JUCE](/tutorials/tutorial_colours/)
- [Tutorial: The Graphics class](/tutorials/tutorial_graphics_class/)
- [Tutorial: Listeners and Broadcasters](/tutorials/tutorial_listeners_and_broadcasters/)
- [Tutorial: The TableListBox class](/tutorials/tutorial_table_list_box/)
