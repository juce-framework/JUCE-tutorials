---
title: Colours in JUCE
---
# Tutorial: Colours in JUCE

Specify and apply colours within your application in various ways.

**LEVEL:** Beginner<br/>
**PLATFORMS:** Windows, macOS, Linux, iOS, Android<br/>
**CLASSES:** [Colour](https://docs.juce.com/master/classColour.html "Represents a colour, also including a transparency value."), [Colours](namespaceColours.html "Contains a set of predefined named colours (mostly standard HTML colours)"), [LookAndFeel](https://docs.juce.com/master/classLookAndFeel.html "LookAndFeel objects define the appearance of all the JUCE widgets, and subclasses can be used to appl...")

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/ColoursTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/ColoursTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project displays some child components and performs some simple drawing commands, which are used to illustrate how colours are specified and applied to components in JUCE. The application should look similar to the following screenshot:

![](/_images/tutorial_colours_screenshot1.png "The colours demo application")

The first part of this tutorial looks at specifying colours more generally. This is illustrated by modifying the drawing code in the `paint()` function within the demo application. The second part of the tutorial shows how colours are specified for elements of the built-in component types (such as labels, sliders, and so on).

# Colours and general painting operations

JUCE specifies colours using red, green, blue, and alpha (transparency) values. This is, of course, a widely used method of specificy colours in computing, but all implementations are slightly different. In particular, JUCE provides some useful methods for manipulating colours, which can help you maintain a consistent colour palette for your application. First, let's look at the `paint()` function from the demo application:

```cpp
void paint (juce::Graphics& g) override
{
    g.fillAll (juce::Colours::lightgreen);
    g.setColour (juce::Colours::orange);

    auto centralArea = getLocalBounds().toFloat().reduced (10.0f);
    g.drawRoundedRectangle (centralArea, 5.0f, 3.0f);

    juce::Array<juce::Colour> colours { juce::Colours::red, juce::Colours::green, juce::Colours::blue };

    auto colourBarArea = centralArea.reduced (4.0f).withHeight (20.0f);
    auto colourArea = colourBarArea.withWidth (colourBarArea.getWidth() / (float) colours.size());

    for (auto colour : colours)
    {
        g.setColour (colour);
        g.fillRect (colourArea);

        colourArea.translate (colourArea.getWidth(), 0.0f);
    }
}
```

The first line fills the entire graphics context with a single colour (which means the entire component's bounds):

```cpp
g.fillAll (juce::Colours::lightgreen);
```

The next line sets the colour for future drawing operations for a given graphics context.

```cpp
g.setColour (juce::Colours::orange);
```

Then we define a slightly inset rectangle and draw a rounded rectangle, as a border, using the current colour:

```cpp
auto centralArea = getLocalBounds().toFloat().reduced (10.0f);
g.drawRoundedRectangle (centralArea, 5.0f, 3.0f);
```

Next, we set up an array of colours, which we will use to draw a row of different coloured rectangles.

```cpp
juce::Array<juce::Colour> colours { juce::Colours::red, juce::Colours::green, juce::Colours::blue };
```

To draw this row of coloured rectangles, we first define the area within which they will be placed:

```cpp
auto colourBarArea = centralArea.reduced (4.0f).withHeight (20.0f);
```

Then we define the area for the first coloured rectangle. This will be a proportion of the total width of the `colourBarArea` rectangle, divided by the number of colours that we are using:

```cpp
auto colourArea = colourBarArea.withWidth (colourBarArea.getWidth() / colours.size());
```

Finally, we iterate over the array of colours, fill the rectangle with the specified colour, and move the `colourArea` rectangle to the right for the next iteration:

```cpp
for (auto colour : colours)
{
    g.setColour (colour);
    g.fillRect (colourArea);

    colourArea.translate (colourArea.getWidth(), 0.0f);
}
```

In the next few examples we will demonstrate some methods for specifying colours by changing the colours added to the `colours` array.

## Specifying colours by name

As shown in the demo project and the code above, colours can be specified in JUCE using some constants in the [Colours](namespaceColours.html "Contains a set of predefined named colours (mostly standard HTML colours)") namespace.

> [!TIP]
>Have a look at the API documentation for the [Colours](namespaceColours.html "Contains a set of predefined named colours (mostly standard HTML colours)") namespace for a full list, which are mostly standard HTML colours.

In addition to the constants within the [Colours](namespaceColours.html "Contains a set of predefined named colours (mostly standard HTML colours)") namespace, you can use the [Colours::findColourForName()](namespaceColours.html#af4f192e53312f7dd625afc1ce029025f "Attempts to look up a string in the list of known colour names, and return the appropriate colour.") function, using a string to look up the desired colour name. For example, we could use the same red, green, and blue colours using the following code to fill our `colours` array:

```cpp
auto defaultColour = Colours::black;

juce::Array<juce::Colour> colours { juce::Colours::findColourForName ("red", defaultColour),
    juce::Colours::findColourForName ("green", defaultColour),
    juce::Colours::findColourForName ("blue", defaultColour) };
```

> [!TIP]
>We need to provide a default colour (in this case we just use black) just in case the search for the named colour fails.

The [Colours::findColourForName()](namespaceColours.html#af4f192e53312f7dd625afc1ce029025f "Attempts to look up a string in the list of known colour names, and return the appropriate colour.") function performs a case-insensitive search and trims whitespace from the start and end of the string, but not spaces within the string. For example, the following code will still work as expected, even though the colours are stored internally using all lowercase strings:

```cpp
auto defaultColour = juce::Colours::black;

juce::Array<juce::Colour> colours { juce::Colours::findColourForName ("DarkRed", defaultColour),
    juce::Colours::findColourForName ("DarkGreen", defaultColour),
    juce::Colours::findColourForName ("DarkBlue", defaultColour) };
```

This produces the following colours:

![](/_images/tutorial_colours_screenshot2.png "Dark red, green, and blue")

But including spaces within the colour name will fail, in our case returning a black colour in each case:

```cpp
auto defaultColour = Colours::black;

juce::Array<juce::Colour> colours { juce::Colours::findColourForName ("Dark Red", defaultColour),
    juce::Colours::findColourForName ("Dark Green", defaultColour),
    juce::Colours::findColourForName ("Dark Blue", defaultColour) };
```

It is straightforward to write your own functions to suit your needs in these instances. For example, you could write a function to remove all spaces from a string:

```cpp
static juce::String removeSpaces (const juce::String& text)
{
    return text.removeCharacters (" ");
}
```

And use that when passing a string to the [Colours::findColourForName()](namespaceColours.html#af4f192e53312f7dd625afc1ce029025f "Attempts to look up a string in the list of known colour names, and return the appropriate colour.") function:

```cpp
auto defaultColour = juce::Colours::black;

juce::Array<juce::Colour> colours { juce::Colours::findColourForName (removeSpaces ("Dark Red"), defaultColour),
    juce::Colours::findColourForName (removeSpaces ("Dark Green"), defaultColour),
    juce::Colours::findColourForName (removeSpaces ("Dark Blue"), defaultColour) };
```

## Specifying colours from values

[Colours](namespaceColours.html "Contains a set of predefined named colours (mostly standard HTML colours)") can also be specified using the raw red, green, blue, and alpha values. Here you can create a [Colour](https://docs.juce.com/master/classColour.html "Represents a colour, also including a transparency value.") object using either floating point values in the range 0.0-1.0, or integers (of type uint8) between 0-255. Using integers we can create the same red, green, and blue colours as follows:

```cpp
juce::Array<juce::Colour> colours { juce::Colour (255, 0, 0), // red
    juce::Colour (0, 128, 0), // green
    juce::Colour (0, 0, 255) }; // blue
```

> [!TIP]
>The standard \"green\" colour does not have the maximum value 255 in the green element of the colour.

Omitting the alpha value in this case sets the alpha value to the maximum (255) making the colour completely opaque.

We can also use a single hexadecimal value to specify a colour. In this case the order of the colour value elements is: alpha, red, green, and blue:

```cpp
juce::Array<juce::Colour> colours { juce::Colour (0xffff0000), // red
    juce::Colour (0xff008000), // green
    juce::Colour (0xff0000ff) }; // blue
```

> [!TIP]
>In this case we MUST specify the alpha value otherwise it will be set to zero (and therefore transparent).

We can also use floating point values using the [Colour::fromFloatRGBA()](https://docs.juce.com/master/classColour.html#a3992d5ea9bdaf0471bd6caa0595b0185 "Creates a colour using floating point red, green, blue and alpha values.") function:

```cpp
juce::Array<juce::Colour> colours { juce::Colour::fromFloatRGBA (1.0f, 0.0f, 0.0f, 1.0f), // red
    juce::Colour::fromFloatRGBA (0.0f, 0.5f, 0.0f, 1.0f), // green
    juce::Colour::fromFloatRGBA (0.0f, 0.0f, 1.0f, 1.0f) }; // blue
```

> [!TIP]
>A integer value of 128 is equivalent to a floating point value of around 0.501961. Therefore the green colours is not *quite* the same as the previous example, but 0.5 is close enough for this demonstration.

<!-- -->

> [!NOTE]
> Try out different colour values and review the results by running the application. You are not limited to adding three colours to the `colours` array, you can use any number of colours (greater than or equal to one).

## Hue, saturation, and brightness

[Colour](https://docs.juce.com/master/classColour.html "Represents a colour, also including a transparency value.") objects can also be initialised from hue, saturation, and brightness values. This is one way to generate different colours that share some perceptual qualities.

For example we could create a series of light and dark reds using the following code:

```cpp
juce::Array<juce::Colour> colours { juce::Colour::fromHSV (0.0f, // hue
                                        0.5f, // saturation
                                        0.3f, // brightness
                                        1.0f), // alpha
    juce::Colour::fromHSV (0.0f, 0.5f, 0.5f, 1.0f),
    juce::Colour::fromHSV (0.0f, 0.5f, 0.7f, 1.0f) };
```

Here the hue, saturation, and alpha values are constant for each colour (a hue of 0.0f should generate colours perceived as \"reds\"). The result is shown in the following screenshot:

![](/_images/tutorial_colours_screenshot3.png "Reds with saturation 0.5 and brightnesses 0.3, 0.5, and 0.7")

We can also obtain the hue, saturation, and brightness values from a [Colour](https://docs.juce.com/master/classColour.html "Represents a colour, also including a transparency value.") object. For example, if we wanted a series of purples of different brightnesses, we could use the following code:

```cpp
auto purpleHue = juce::Colours::purple.getHue();

juce::Array<juce::Colour> colours { juce::Colour::fromHSV (purpleHue, 0.5f, 0.3f, 1.0f),
    juce::Colour::fromHSV (purpleHue, 0.5f, 0.5f, 1.0f),
    juce::Colour::fromHSV (purpleHue, 0.5f, 0.7f, 1.0f) };
```

The result of this is shown in the following screenshot:

![](/_images/tutorial_colours_screenshot4.png "Purples with saturation 0.5 and brightnesses 0.3, 0.5, and 0.7")

## Manipulating colour values

We can also use existing colours to create new colours. For example, to make colours that are slightly brighter or darker than an existing colour we can use the [Colour::brighter()](https://docs.juce.com/master/classColour.html#a614c7699a1b47dcf324106d548238bac "Returns a brighter version of this colour.") or [Colour::darker()](https://docs.juce.com/master/classColour.html#a9910d9b8385825a87ea0d62d96115872 "Returns a darker version of this colour.") functions respectively:

```cpp
auto baseColour = juce::Colours::orange;

juce::Array<juce::Colour> colours { baseColour.darker(),
    baseColour,
    baseColour.brighter() };
```

Or you can blend between two colours using the [Colour::interpolatedWith()](https://docs.juce.com/master/classColour.html#ac1500fb05c7775db2e1b21f07cc28c0c "Returns a colour that lies somewhere between this one and another.") function:

```cpp
auto colour1 = juce::Colours::red;
auto colour2 = juce::Colours::purple;

juce::Array<juce::Colour> colours { colour1,
    colour1.interpolatedWith (colour2, 0.5f),
    colour2 };
```

The result of this is shown in the following screenshot:

![](/_images/tutorial_colours_screenshot5.png "Red and purple with an equal blend of red and purple in between")

Given one colour you can create another colour that will be clearly visible against another colour using the [Colour::contrasting()](https://docs.juce.com/master/classColour.html#a50e8a45cda60f6853cb74ed1ff1fb7d7 "Returns a colour that will be clearly visible against this colour.") function. This allows you to specify the amount of contrast using an argument:

```cpp
auto baseColour = juce::Colours::darkcyan;

juce::Array<juce::Colour> colours { baseColour,
    baseColour.contrasting (0.5f) };
```

You can even create a colour that is contrasting against two other colours:

```cpp
auto colour1 = juce;:Colours::lightblue;
auto colour2 = juce;:Colours::darkred;

juce::Array<juce::Colour> colours { colour1,
    juce::Colour::contrasting (colour1, colour2),
    colour2 };
```

There are various other manipulations that can be performed such as blending two colours taking into account the alpha channel of the overlaid colour using the [Colour::overlaidWith()](https://docs.juce.com/master/classColour.html#a3c6c94ff6f05b403b783acaf7c68b439 "Returns a colour that is the result of alpha-compositing a new colour over this one.") function.

# Specifying component colours

The previous section explored the use of colours when performing your own drawing operations with your component's `paint()` function. To customise the colours of the built-in components (such as sliders, labels, and so on) you need to use Component::setColour() or [LookAndFeel::setColour()](https://docs.juce.com/master/classLookAndFeel.html#a167a1e914771f32e433a0d45aaba45e3 "Registers a colour to be used for a particular purpose.") functions.

Essentially, each of the built-in [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") subclasses contains an `enum` that lists the various elements of the component that can have a specific colour. Each of these items is referred to as a _colour ID_. (The values of these colour IDs are unique across the JUCE library.) For example, the colour IDs for the [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") class are as follows (from [Label::ColourIds](https://docs.juce.com/master/classLabel.html#a41756012394513222e1323bb432fa701 "A set of colour IDs to use to change the colour of various aspects of the label.")):

- [Label::backgroundColourId](https://docs.juce.com/master/classLabel.html#a41756012394513222e1323bb432fa701a80bf4bb58a466a4debb994f3ee2c8cab "The background colour to fill the label with.") : The background colour to fill the label with.

<!-- -->

- [Label::textColourId](https://docs.juce.com/master/classLabel.html#a41756012394513222e1323bb432fa701a1f25238374bf53d43f8c73e68c8e937c "The colour for the text.") : The colour for the text.

<!-- -->

- [Label::outlineColourId](https://docs.juce.com/master/classLabel.html#a41756012394513222e1323bb432fa701ab67bf6af853bcef12b41b3e81f7d18bb "An optional colour to use to draw a border around the label.") : An optional colour to use to draw a border around the label.

Leave this transparent to not have an outline.

- [Label::backgroundWhenEditingColourId](https://docs.juce.com/master/classLabel.html#a41756012394513222e1323bb432fa701a6ccfd805ecc818013ea4bdde69d4258b "The background colour when the label is being edited.") : The background colour when the label is being edited.

<!-- -->

- [Label::textWhenEditingColourId](https://docs.juce.com/master/classLabel.html#a41756012394513222e1323bb432fa701adb6284340066b8c4d9ea389087c7d69d "The colour for the text when the label is being edited.") : The colour for the text when the label is being edited.

<!-- -->

- [Label::outlineWhenEditingColourId](https://docs.juce.com/master/classLabel.html#a41756012394513222e1323bb432fa701a6c7adfce6c830c9faa94efd3445f78cf "An optional border colour when the label is being edited.") : An optional border colour when the label is being edited.

Let's try changing some of these colours. If you look in the `MainContentComponent` constructor you will see a [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked."), a [TextEditor](https://docs.juce.com/master/classTextEditor.html "An editable text box."), a [TextButton](https://docs.juce.com/master/classTextButton.html "A button that uses the standard lozenge-shaped background with a line of text on it."), and two [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value.") objects added as child components. Add the line [1] as shown below to change the label's text colour to black:

```cpp
MainContentComponent()
{
    label.setColour (juce::Label::textColourId, Colours::black); // [1]
    label.setEditable (true);
    addAndMakeVisible (label);
    //...
```

The result should be similar to the following screenshot:

![](/_images/tutorial_colours_screenshot6.png "Showing a label with a customised text colour")

> [!NOTE]
> Look at the colour IDs for the [TextEditor](https://docs.juce.com/master/classTextEditor.html "An editable text box."), [TextButton](https://docs.juce.com/master/classTextButton.html "A button that uses the standard lozenge-shaped background with a line of text on it."), and [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value.") classes ([TextEditor::ColourIds](https://docs.juce.com/master/classTextEditor.html#aa805220923c93cd808a92fff0e3fb71d "A set of colour IDs to use to change the colour of various aspects of the editor."), [TextButton::ColourIds](https://docs.juce.com/master/classTextButton.html#afb6c1da587aebb63b58a2569bed70c2c "A set of colour IDs to use to change the colour of various aspects of the button."), and [Slider::ColourIds](https://docs.juce.com/master/classSlider.html#a1012002c53381ccc7c1fe7e604a75f44 "A set of colour IDs to use to change the colour of various aspects of the slider.")) and experiment with setting different colours for the child components in the demo application.

## Setting look-and-feel colours

It is very common for applications, or parts of applications, to require the same colour palette for all components of the same type. You may have found in the exercise in the previous section that you needed to repeat the calls to the Component::setColour() function for both sliders in order to give them the same appearance. One use of the [LookAndFeel](https://docs.juce.com/master/classLookAndFeel.html "LookAndFeel objects define the appearance of all the JUCE widgets, and subclasses can be used to appl...") class is to provide a single point where these colours can be specified. To illustrate this, return the `MainContentComponent` constructor back to its original state as shown below:

```cpp
MainContentComponent()
{
    label.setEditable (true);
    addAndMakeVisible (label);

    textEditor.setText ("This is a text editor.");
    addAndMakeVisible (textEditor);

    textButton.setClickingTogglesState (true);
    addAndMakeVisible (textButton);

    addAndMakeVisible (slider1);
    addAndMakeVisible (slider2);

    setSize (600, 210);
}
```

Now add the following line [2] to set the colour of the thumbs for both sliders:

```cpp
//...
getLookAndFeel().setColour (juce::Slider::thumbColourId, juce::Colours::red); // [2]
addAndMakeVisible (slider1);
addAndMakeVisible (slider2);
//...
```

This should produce a result similar to the following screenshot:

![](/_images/tutorial_colours_screenshot7.png "Customising the colour of multiple slider thumbs using a single line of code")

## Custom look-and-feel colours

You can also make a subclass of one of the [LookAndFeel](https://docs.juce.com/master/classLookAndFeel.html "LookAndFeel objects define the appearance of all the JUCE widgets, and subclasses can be used to appl...") classes ([LookAndFeel_V1](https://docs.juce.com/master/classLookAndFeel__V1.html "The original JUCE look-and-feel, as used back from 2002 to about 2007ish."), [LookAndFeel_V2](https://docs.juce.com/master/classLookAndFeel__V2.html "This LookAndFeel subclass implements the juce style from around 2008-12."), [LookAndFeel_V3](https://docs.juce.com/master/classLookAndFeel__V3.html "The latest JUCE look-and-feel style, as introduced in 2013."), or [LookAndFeel_V4](https://docs.juce.com/master/classLookAndFeel__V4.html "The latest JUCE look-and-feel style, as introduced in 2017.")) and customise specific colours in its constructor. To do this you could add the following class as a nested class of our `MainContentComponent` class:

```cpp
class CustomLookAndFeel : public juce::LookAndFeel_V4
{
public:
    CustomLookAndFeel()
    {
        setColour (juce::Slider::thumbColourId, juce::Colours::red);
    }
};
```

Add an instance of this class to our private member section [3]:

```cpp
private:
    CustomLookAndFeel lf; // [3]
    juce::Label label { {}, "This is some label text." };
```

And set the MainContentComponent class to use this look-and-feel in its constructor [4]:

```cpp
MainContentComponent()
{
    setLookAndFeel (&lf); // [4]

    label.setEditable (true);
```

> [!TIP]
>The changed code for this subsection can be found in the `ColoursTutorial_02.h` file of the demo project.

<!-- -->

> [!NOTE]
> Customise more colours in the `CustomLookAndFeel` constructor.

# Summary

In this tutorial we have looked at the following items that you can use in your own applications:

- How to use the [Colour](https://docs.juce.com/master/classColour.html "Represents a colour, also including a transparency value.") class and [Colours](namespaceColours.html "Contains a set of predefined named colours (mostly standard HTML colours)") namespace to specify colours in JUCE.
- Using colours in your component's `paint()` function when performing drawing operations.
- Specifying the colours for the elements of built-in components using colour IDs.
- Specifying colours across parts of your application or your whole application using the [LookAndFeel](https://docs.juce.com/master/classLookAndFeel.html "LookAndFeel objects define the appearance of all the JUCE widgets, and subclasses can be used to appl...") class.

# See also

- [Tutorial: The Random class](/tutorials/tutorial_random/)
- [Tutorial: Customise the look and feel of your app](/tutorials/tutorial_look_and_feel_customisation/)
- [Tutorial: Parent and child components](/tutorials/tutorial_component_parents_children/)
- [Tutorial: The Point, Line, and Rectangle classes](/tutorials/tutorial_point_line_rectangle/)
- [Tutorial: Advanced GUI layout techniques](/tutorials/tutorial_rectangle_advanced/)
