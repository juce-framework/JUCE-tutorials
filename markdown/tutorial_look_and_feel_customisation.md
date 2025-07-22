---
title: Customise the look and feel of your app
---
# Tutorial: Customise the look and feel of your app

Customise the drawing of fundamental widgets in your application. Make a custom _skin_ for your application by drawing your own buttons, sliders, and other components.

**LEVEL:** Beginner<br/>
**PLATFORMS:** Windows, macOS, Linux, iOS, Android<br/>
**CLASSES:** [LookAndFeel](https://docs.juce.com/master/classLookAndFeel.html "LookAndFeel objects define the appearance of all the JUCE widgets, and subclasses can be used to appl..."), [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value."), [Button](https://docs.juce.com/master/classButton.html "A base class for buttons."), [Path](https://docs.juce.com/master/classPath.html "A path is a sequence of lines and curves that may either form a closed shape or be open-ended."), [AffineTransform](https://docs.juce.com/master/classAffineTransform.html "Represents a 2D affine-transformation matrix.")

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/LookAndFeelCustomisationTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/LookAndFeelCustomisationTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project creates a GUI with two buttons and two rotary sliders using the standard JUCE look-and-feel:

![](/_images/tutorial_look_and_feel_customisation_screenshot1.png "Standard look-and-feel buttons and sliders")

The [LookAndFeel](https://docs.juce.com/master/classLookAndFeel.html "LookAndFeel objects define the appearance of all the JUCE widgets, and subclasses can be used to appl...") class is fundamental to creating customised GUIs in JUCE. Using the [LookAndFeel](https://docs.juce.com/master/classLookAndFeel.html "LookAndFeel objects define the appearance of all the JUCE widgets, and subclasses can be used to appl...") class you can perform simple customisations such as changing the default colours of certain components. But you can also customise the drawing of many types of component. For example, this allows you to create buttons and sliders with a custom appearance.

# Customising colours

When a [LookAndFeel](https://docs.juce.com/master/classLookAndFeel.html "LookAndFeel objects define the appearance of all the JUCE widgets, and subclasses can be used to appl...") object is applied to a component, it is applied to that component _and_ its child components (see [Tutorial: Parent and child components](/tutorials/tutorial_component_parents_children/)) unless the child components have specifically had a different look-and-feel assigned.

One thing that you can do with the look-and-feel system is to override specific colours for elements of the standard JUCE components (see [Tutorial: Colours in JUCE](/tutorials/tutorial_colours/).) For example, if you add the following line to the `MainContentComponent` constructor, then _both_ dials will be red:

```cpp
getLookAndFeel().setColour (juce::Slider::thumbColourId, juce::Colours::red);
```

This should look something like the following screenshot:

![](/_images/tutorial_look_and_feel_customisation_screenshot2.png "Overriding look-and-feel colours")

To set the two dials differently we could make a new [LookAndFeel](https://docs.juce.com/master/classLookAndFeel.html "LookAndFeel objects define the appearance of all the JUCE widgets, and subclasses can be used to appl...") instance and apply that to only one of the dials. First add a [LookAndFeel_V4](https://docs.juce.com/master/classLookAndFeel__V4.html "The latest JUCE look-and-feel style, as introduced in 2017.") object as a member [1](this is the class that implements the default JUCE look-and-feel).

```cpp
private:
    juce::LookAndFeel_V4 otherLookAndFeel; // [1]
    juce::Slider dial1;
    juce::Slider dial2;
    juce::TextButton button1;
    juce::TextButton button2;
```

> [!NOTE]
> LookAndFeels must always be declared *before* the widgets and components consuming it! Otherwise they would be destructed while still in use by those classes.

Then change the line of code in the constructor, that we just added, to this:

```cpp
otherLookAndFeel.setColour (juce::Slider::thumbColourId, juce::Colours::red);
```

Let's use this look-and-feel _only_ for the first dial. Add this line of code to the `MainContentComponent` constructor:

```cpp
dial1.setLookAndFeel (&otherLookAndFeel);
```

This should now create a UI like the following screenshot:

![](/_images/tutorial_look_and_feel_customisation_screenshot3.png "Using different look-and-feel objects for different components")

Of course, in this simple example this approach offers no benefits compared to setting the [Slider::thumbColourId](https://docs.juce.com/master/classSlider.html#a1012002c53381ccc7c1fe7e604a75f44aee563b22e0033b0ed4f38e5296889835 "The colour to draw the thumb with.") colour on the slider objects directly. But your app may use multiple sliders for different purposes where you want sliders for one purpose to use one set of colours and sliders for other purposes to use different sets of colours. This approach allows you to change these colours globally as long as each slider is assigned the appropriate look-and-feel for its type.

The benefits of this approach are clearer once we start to customise the actual drawing code. In particular, we need to create a custom look-and-feel class.

# Custom look-and-feel

To customise the drawing of certain components we need to create a new class that inherits from the [LookAndFeel](https://docs.juce.com/master/classLookAndFeel.html "LookAndFeel objects define the appearance of all the JUCE widgets, and subclasses can be used to appl...") class. If you inherit directly from the [LookAndFeel](https://docs.juce.com/master/classLookAndFeel.html "LookAndFeel objects define the appearance of all the JUCE widgets, and subclasses can be used to appl...") class itself then you\'ll need to implement _all_ of the pure virtual functions. It's much more practical to inherit from one of the classes that already has all of these functions defined. Then you need override only the ones you need. Let's create a simple custom look-and-feel that has only this one colour change defined compared to the default look-and-feel.

First, remove this line from the constructor, which we added earlier:

```cpp
otherLookAndFeel.setColour (juce::Slider::thumbColourId, juce::Colours::red);
```

Now, add our new class, which inherits from the [LookAndFeel_V4](https://docs.juce.com/master/classLookAndFeel__V4.html "The latest JUCE look-and-feel style, as introduced in 2017.") class, before the `MainContentComponent` class:

```cpp
class OtherLookAndFeel : public juce::LookAndFeel_V4
{
public:
    OtherLookAndFeel()
    {
        setColour (juce::Slider::thumbColourId, juce::Colours::red);
    }
```

Before we run this code, change the class name of our `otherLookAndFeel` member to OtherLookAndFeel [2]:

```cpp
private:
OtherLookAndFeel otherLookAndFeel; // [2]
juce::Slider dial1;
juce::Slider dial2;
juce::TextButton button1;
juce::TextButton button2;
```

Build and run the application and the result should appear identical to the previous screenshot.

## Customising drawing

There are many functions in the [LookAndFeel](https://docs.juce.com/master/classLookAndFeel.html "LookAndFeel objects define the appearance of all the JUCE widgets, and subclasses can be used to appl...") class for many different types of components. The functions that are designated for a specific component type are easy to find as these are all declared within a nested class named `LookAndFeelMethods` within the relevant component class.

## Slider customisation

For example, take a look at the [Slider::LookAndFeelMethods](structSlider_1_1LookAndFeelMethods.html "This abstract base class is implemented by LookAndFeel classes to provide slider drawing functionalit...") within the JUCE API documentation. In this list you will notice a function named [Slider::LookAndFeelMethods::drawRotarySlider()](structSlider_1_1LookAndFeelMethods.html#ae63a9d8adce084cd5dbe02b960c73a9a).

Let's override this in our `OtherLookAndFeel` class. Add the declaration to the class:

```cpp
void drawRotarySlider (juce::Graphics& g, int x, int y, int width, int height, float sliderPos, const float rotaryStartAngle, const float rotaryEndAngle, juce::Slider&) override
```

Here you can see that we are passed the following data:

- `g` : The [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") context.
- `x` : The x coordinate of the top-left of the rectangle within which we should draw our rotary slider.
- `y` : The y coordinate of the top-left of the rectangle within which we should draw our rotary slider.
- `width` : The width of the rectangle within which we should draw our rotary slider.
- `height` : The height of the rectangle within which we should draw our rotary slider.
- `sliderPos` : The _position_ of the slider as a proportion in the range 0..1 (this is independent of the slider's actual range of values).
- `rotaryStartAngle` : The start angle of the dial rotation (in radians).
- `rotaryEndAngle` : The end angle of the dial rotation (in radians).
- `slider` : The [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value.") object itself.

> [!TIP]
>The x, y, width, and height arguments take into account the size and position of any *text box* that the slider may be using. This is why we can just access the slider position and size and use those values.

Now let's write the function body such that it draws a simple dial that is just a filled circle with a line representing the pointer of the dial. First, we will need some temporary variables to help with our calculations based on the values we have been passed:

```cpp
auto radius = (float) juce::jmin (width / 2, height / 2) - 4.0f;
auto centreX = (float) x + (float) width * 0.5f;
auto centreY = (float) y + (float) height * 0.5f;
auto rx = centreX - radius;
auto ry = centreY - radius;
auto rw = radius * 2.0f;
auto angle = rotaryStartAngle + sliderPos * (rotaryEndAngle - rotaryStartAngle);
```

> [!TIP]
>You can see that the final `angle` variable contains the angle at which the dial should point.

Now let's add code to fill in the colour of the dial and draw an outline:

```cpp
// fill
g.setColour (juce::Colours::orange);
g.fillEllipse (rx, ry, rw, rw);

// outline
g.setColour (juce::Colours::red);
g.drawEllipse (rx, ry, rw, rw, 1.0f);
```

To draw the pointer itself, first we\'ll use a [Path](https://docs.juce.com/master/classPath.html "A path is a sequence of lines and curves that may either form a closed shape or be open-ended.") object that we will translate and rotate into position by the required angle:

```cpp
juce::Path p;
auto pointerLength = radius * 0.33f;
auto pointerThickness = 2.0f;
p.addRectangle (-pointerThickness * 0.5f, -radius, pointerThickness, pointerLength);
p.applyTransform (juce::AffineTransform::rotation (angle).translated (centreX, centreY));
```

Then we fill this path to draw the pointer:

```cpp
// pointer
g.setColour (juce::Colours::yellow);
g.fillPath (p);
```

> [!TIP]
>The completed code for this section can be found in the `LookAndFeelCustomisationTutorial_02.h` file of the demo project for this tutorial.

<!-- -->

> [!NOTE]
> Exercise: Modify the drawing of the pointer. You could try different lengths, a slightly thicker but rounded rectangle, or draw an arrow.

This shows you only one simple customisation of one of the [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value.") look-and-feel methods. But the principle applies to the other methods. Perhaps the best approach for creating other customisations is to look at the existing implementation in the [LookAndFeel_V4](https://docs.juce.com/master/classLookAndFeel__V4.html "The latest JUCE look-and-feel style, as introduced in 2017.") or [LookAndFeel_V3](https://docs.juce.com/master/classLookAndFeel__V3.html "The latest JUCE look-and-feel style, as introduced in 2013.") classes and use this as a basis for your own code.

> [!TIP]
>The [LookAndFeel_V4](https://docs.juce.com/master/classLookAndFeel__V4.html "The latest JUCE look-and-feel style, as introduced in 2017.") class inherits from the [LookAndFeel_V3](https://docs.juce.com/master/classLookAndFeel__V3.html "The latest JUCE look-and-feel style, as introduced in 2013.") class and some methods are not redefined in the [LookAndFeel_V4](https://docs.juce.com/master/classLookAndFeel__V4.html "The latest JUCE look-and-feel style, as introduced in 2017.") class.

## Button customisation

Let's look at customising the buttons. First, let's set our `OtherLookAndFeel` class as the look-and-feel for our whole `MainContentComponent` by using this line in its constructor:

```cpp
setLookAndFeel (&otherLookAndFeel);
```

Let's also make sure that the [LookAndFeel](https://docs.juce.com/master/classLookAndFeel.html "LookAndFeel objects define the appearance of all the JUCE widgets, and subclasses can be used to appl...") object is not used on shutdown by the `MainContentComponent` anymore by supplying this line in its destructor:

```cpp
~MainContentComponent() override
{
    setLookAndFeel (nullptr);
}
```

This will, of course, mean that both of our dials take on the appearance we customised in the previous section. Now let's add the [Button::LookAndFeelMethods::drawButtonBackground()](structButton_1_1LookAndFeelMethods.html#a06f95e4c63f74cfe3af3f21698c9107a) function declaration:

```cpp
void drawButtonBackground (juce::Graphics& g, juce::Button& button, const juce::Colour& backgroundColour, bool, bool isButtonDown) override
```

Here, we are passed the following data:

- `g` : The [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") context.
- `button` : The [Button](https://docs.juce.com/master/classButton.html "A base class for buttons.") object itself.
- `backgroundColour` : The base background colour that should be used (which will have been chosen from the [LookAndFeel](https://docs.juce.com/master/classLookAndFeel.html "LookAndFeel objects define the appearance of all the JUCE widgets, and subclasses can be used to appl...") colours based on the toggle state of the button).
- `isMouseOverButton` : Whether the mouse pointer is within the bounds of the button.
- `isButtonDown` : Whether the mouse button is down.

Now, let's add the function body to make a really simple button background that simply fills the button rectangle with the background colour:

```cpp
auto buttonArea = button.getLocalBounds();
g.setColour (backgroundColour);
g.fillRect (buttonArea);
```

If you build and run this, it should look similar to the following screenshot:

![](/_images/tutorial_look_and_feel_customisation_screenshot4.png "Simple button")

If you interact with this, you will notice that the buttons do not respond visually to mouse pointer interaction. Let's implement a simple shadow effect. Change the `drawButtonBackground()` function to this:

```cpp
auto buttonArea = button.getLocalBounds();
auto edge = 4;

buttonArea.removeFromLeft (edge);
buttonArea.removeFromTop (edge);

// shadow
g.setColour (juce::Colours::darkgrey.withAlpha (0.5f));
g.fillRect (buttonArea);

auto offset = isButtonDown ? -edge / 2 : -edge;
buttonArea.translate (offset, offset);

g.setColour (backgroundColour);
g.fillRect (buttonArea);
```

The button will now appear to move as we click the button. Unfortunately, the text stays static, so we need to override the [Button::LookAndFeelMethods::drawButtonText()](structButton_1_1LookAndFeelMethods.html#a3c7f5449c273361a926f5b82b8547383 "Draws the text for a TextButton.") function to make this more believable. To write this function we\'ll start with a copy of the code from the [LookAndFeel_V2](https://docs.juce.com/master/classLookAndFeel__V2.html "This LookAndFeel subclass implements the juce style from around 2008-12.") class and add it to our OtherLookAndFeel class:

```cpp
void drawButtonText (juce::Graphics& g, juce::TextButton& button, bool isMouseOverButton, bool isButtonDown) override
{
    auto font = getTextButtonFont (button, button.getHeight());
    g.setFont (font);
    g.setColour (button.findColour (button.getToggleState() ? juce::TextButton::textColourOnId
                                                            : juce::TextButton::textColourOffId)
                     .withMultipliedAlpha (button.isEnabled() ? 1.0f : 0.5f));

    auto yIndent = juce::jmin (4, button.proportionOfHeight (0.3f));
    auto cornerSize = juce::jmin (button.getHeight(), button.getWidth()) / 2;

    auto fontHeight = juce::roundToInt (font.getHeight() * 0.6f);
    auto leftIndent = juce::jmin (fontHeight, 2 + cornerSize / (button.isConnectedOnLeft() ? 4 : 2));
    auto rightIndent = juce::jmin (fontHeight, 2 + cornerSize / (button.isConnectedOnRight() ? 4 : 2));
    auto textWidth = button.getWidth() - leftIndent - rightIndent;

    if (textWidth > 0)
        g.drawFittedText (button.getButtonText(),
            leftIndent,
            yIndent,
            textWidth,
            button.getHeight() - yIndent * 2,
            juce::Justification::centred,
            2);
}
```

We just need to change the offset at which the text is drawn to match the apparent movement in our `drawButtonBackground()` function. We need to change only the last few lines:

```cpp
auto textWidth = button.getWidth() - leftIndent - rightIndent;

auto edge = 4;
auto offset = isButtonDown ? edge / 2 : 0;

if (textWidth > 0)
    g.drawFittedText (button.getButtonText(),
        leftIndent + offset,
        yIndent + offset,
        textWidth,
        button.getHeight() - yIndent * 2 - edge,
        juce::Justification::centred,
        2);
}
```

Build and run this and it should look similar to the following screenshot.

![](/_images/tutorial_look_and_feel_customisation_screenshot5.png "Buttons with shadows (Button 1 is shown clicked)")

> [!TIP]
>The completed code for this section can be found in the `LookAndFeelCustomisationTutorial_03.h` file of the demo project for this tutorial.

<!-- -->

> [!NOTE]
> Exercise: Add some changes to the drawing of the button to respond to the mouse pointer being over the button. For example you could adjust the background colour slightly, change the shadow colour, or subtly change the rectangle sizes or positions.

# Summary

In this tutorial we have introduced the concept of customising the look-and-feel of JUCE components using the [LookAndFeel](https://docs.juce.com/master/classLookAndFeel.html "LookAndFeel objects define the appearance of all the JUCE widgets, and subclasses can be used to appl...") class. In particular you should now be able to:

- Customise colours in the default look-and-feel.
- Create a new look-and-feel class.
- Customise slider and button drawing code.
- Find the look-and-feel methods for other components so you can customise any JUCE component.

# See also

- [Tutorial: Colours in JUCE](/tutorials/tutorial_colours/)
- [Tutorial: The Slider class](/tutorials/tutorial_slider_values/)
- [Tutorial: Listeners and Broadcasters](/tutorials/tutorial_listeners_and_broadcasters/)
- [Tutorial: Parent and child components](/tutorials/tutorial_component_parents_children/)
- [Tutorial: Animating geometry](/tutorials/tutorial_animation/)
- [Tutorial: Build an OpenGL application](/tutorials/tutorial_open_gl_application/)
