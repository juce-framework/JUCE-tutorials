---
title: The Graphics class
---
# Tutorial: The Graphics class

This tutorial shows how to use a [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") object to draw text, lines, and geometric shapes. This is fundamental to performing drawing in JUCE.

**LEVEL:** Beginner<br/>
**PLATFORMS:** Windows, macOS, Linux, iOS, Android<br/>
**CLASSES:** [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects."), [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image."), [Font](https://docs.juce.com/master/classFont.html "Represents a particular font, including its size, style, etc."), [Colours](namespaceColours.html "Contains a set of predefined named colours (mostly standard HTML colours)"), [Path](https://docs.juce.com/master/classPath.html "A path is a sequence of lines and curves that may either form a closed shape or be open-ended.")

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/GraphicsTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/GraphicsTutorial.zip)

# The demo project

The demo project for this tutorial contains a main application window and a main component. You should be familiar with these from the last tutorials: [Tutorial: The application window](/tutorials/tutorial_main_window/) and [Tutorial: The main component](/tutorials/tutorial_main_component/).

You already know that the appearance of the main component (or, as a matter of fact, any other Component!) is determined by the implementation of its `paint()` function. The demo project here takes off where the last tutorial ([Tutorial: The main component](/tutorials/tutorial_main_component/)) concluded. The implementation of the `paint()` function initially looks as follows:

```cpp
void paint (juce::Graphics& g)
{
    g.fillAll (juce::Colours::lightblue);

    g.setColour (juce::Colours::darkblue);
    g.setFont (14.0f);
    g.drawText ("Hello, World!", getLocalBounds(), juce::Justification::centred, true);
}
```

If you compile and run the app now, you should see that the window now has a light blue background colour, and the text **Hello, World!** is drawn on top of it, in the centre of the window.

In the following, we will add some code that draws some more graphics into the `MainComponent` object, using the [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") class. This is a very powerful class, and we will be using it a lot in future tutorials to implement the custom visual appearance of different JUCE components.

# The Graphics class

Let's have another look at the paint function. Remember that the `paint()` function is a callback called by the operating system when it is time to render your [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") on screen --- you should never call this function yourself.

Notice that as the argument to this callback, a reference to a [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") instance is passed in. This [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") object is provided by the underlying framework. It is the graphics context that you can use to render any graphical elements: text, lines, shapes, colours, gradients and much more. We will explore some of these in this tutorial.

> [!WARNING]
> The [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") class is usually only used in the `paint()` callback. Normally, you should never use it anywhere else unless when drawing onto an image.

# Rendering text

## Setting the font

Let's first continue with text. The line

```cpp
g.setFont (20.0f);
```

sets the font size to 20 pixels for the subsequent line (which draws the text **Hello, World!** using that font). But what if we want to not only change the size of the font, but also use another typeface and bold or italic letters? And how do we change the position of the text?

There is actually another version of the [Graphics::setFont()](https://docs.juce.com/master/classGraphics.html#a1fbdb321975d90c45243027a61ac2be9 "Changes the font to use for subsequent text-drawing functions.") function that takes a [Font](https://docs.juce.com/master/classFont.html "Represents a particular font, including its size, style, etc.") object instead of just a `float` that determines the size. You can create a new [Font](https://docs.juce.com/master/classFont.html "Represents a particular font, including its size, style, etc.") object like this:

```cpp
juce::Font mainComponentFont ("Times New Roman", 20.0f, juce::Font::italic);
```

Because we are using this font for our main component, we choose the descriptive variable name `mainComponentFont` .

The first argument of the [Font](https://docs.juce.com/master/classFont.html "Represents a particular font, including its size, style, etc.") constructor determines the typeface, the second is the font size, and the third is the font style. Here, we chose italic for the style. The font styles are actually flags that can be used as a bitmask (see [Tutorial: The application window](/tutorials/tutorial_main_window/)), so you can combine them for example like this:

```cpp
juce::Font mainComponentFont ("Times New Roman", 20.0f, juce::Font::bold | juce::Font::italic);
```

If you compile and run the app again, you should see that the font has changed.

> [!WARNING]
> Using a typeface for a font that is not actually installed on the computer is a very common reason of fonts not working properly in a JUCE app.

Instead of creating a named [Font](https://docs.juce.com/master/classFont.html "Represents a particular font, including its size, style, etc.") object and then setting it with the [Graphics::setFont()](https://docs.juce.com/master/classGraphics.html#a1fbdb321975d90c45243027a61ac2be9 "Changes the font to use for subsequent text-drawing functions.") function on the next line, you could also do both in one statement:

```cpp
g.setFont (juce::Font ("Times New Roman", 20.0f, juce::Font::italic));
```

although your code will usually be better readable and maintainable with separating the statements and using a named variable. (With modern compilers, introducing such an additional variable will have no impact on performance.)

## Setting the position

Now we change the position of the text. In the process, we will learn how positioning is handled in JUCE.

The easy way would be to simply change the alignment of the text with respect to the whole component, for example by changing the [Justification::centred](https://docs.juce.com/master/classJustification.html#a1f8c07756c56fe8f31ed44964e51bfbcaf9d9adde20dce1aa38ff9a69be2b4384 "Indicates that the item should be centred vertically and horizontally.") value to another one of the possible values, for example the [Justification::topLeft](https://docs.juce.com/master/classJustification.html#a1f8c07756c56fe8f31ed44964e51bfbcaf2d7cc575db9d4d9a1305407625b7afd "Indicates that the item should be placed in the top-left corner.") value. (You can also check out the other possible [Justification::Flags](https://docs.juce.com/master/classJustification.html#a1f8c07756c56fe8f31ed44964e51bfbc "Flag values that can be combined and used in the constructor.") values.) However, another very powerful approach is to explicitly define the size and position. There is another version of the [Graphics::drawText()](https://docs.juce.com/master/classGraphics.html#aff45fc088e19374d3984e44a5c7d7639 "Draws a line of text within a specified rectangle.") function using this approach. Change the line starting with the `g.drawText()` call to the following:

```cpp
g.drawText ("Hello, World!", 20, 40, 200, 40, juce::Justification::centred, true);
```

This tells the [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") object to render the text into an area that is 200 pixels wide, 40 pixels high, and located 20 pixels to the right and 40 pixels to the bottom from the top-left corner of the main component.

The app should now look like this:

![](/_images/tutorial_graphics_class_screenshot1.png "Customising the text font and position.")

> [!TIP]
>Remember: coordinates in JUCE are always measured from the top-left corner of the current component, which is the point `(0, 0)` . They can be given as `int` or `float` numbers. When used to specify the position of a graphical element or child component, it will be positioned such that its top left corner will appear at the given position.

Finally, the last argument of the [Graphics::drawText()](https://docs.juce.com/master/classGraphics.html#aff45fc088e19374d3984e44a5c7d7639 "Draws a line of text within a specified rectangle.") function is a `bool` flag which determines whether an ellipsis (...) should be shown if the text does not fit within the given width, or whether the text should be simply chopped off.

> [!NOTE]
> Exercise: Change the width of the text field from 200 to smaller values and note how the ellipsis flag works.

<!-- -->

> [!TIP]
>The [Graphics::drawText()](https://docs.juce.com/master/classGraphics.html#aff45fc088e19374d3984e44a5c7d7639 "Draws a line of text within a specified rectangle.") function is good for rendering single-line text. For multi-line text, it offers other functions such as [Graphics::drawMultiLineText()](https://docs.juce.com/master/classGraphics.html#ae6ae0dc5e9e1956af5998c18e0955a56 "Draws text across multiple lines.") and [Graphics::drawFittedText()](https://docs.juce.com/master/classGraphics.html#a41c5a930dfc9b8cdd8c8a464f7e11b46 "Tries to draw a text string inside a given space.").

# Rendering geometrical shapes

In this section, we continue with drawing some geometrical shapes using the [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") class.

## Drawing lines

Add the following lines to the bottom of the `paint()` function:

```cpp
g.setColour (juce::Colours::green);
g.drawLine (10, 300, 590, 300, 5);
```

This will draw a green horizontal line 5 pixels wide across the window, starting from `(10, 300)` and ending at `(590, 300)` . Note that every time you want to draw a geometric shape in another colour than the one used last time, you have to call the [Graphics::setColour()](https://docs.juce.com/master/classGraphics.html#a9a944a0006b7277fda473f0b1b4f028f "Changes the current drawing colour.") function before you draw.

You can of course also draw diagonal lines by specifying other coordinates. In fact, JUCE also supports subpixel coordinates (you can use `float` values for the positions). If the position falls between physical screen pixels, JUCE will apply anti-aliasing for the drawing.

> [!NOTE]
> Exercise: Explore other types of lines. Can you figure out how to draw dashed lines, or arrows? Hint: have a look at the [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") class documentation.

## Drawing rectangles

Drawing rectangles using the [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") object is quite straightforward. Add the following line to the `paint()` function body:

```cpp
g.setColour (juce::Colours::sandybrown);
g.drawRect (300, 120, 200, 170);
```

This will render a brown rectangle, 200 pixels wide, 170 pixels high, positioned with its top-left corner at the position `(300, 120)` .

An optional fifth argument lets you specify the line thickness:

```cpp
g.setColour (juce::Colours::sandybrown);
g.drawRect (300, 120, 200, 170, 3);
```

If you want a filled rectangle, use the function [Graphics::fillRect()](https://docs.juce.com/master/classGraphics.html#a747de9976729f72d9f6188104e6b2215 "Fills a rectangle with the current colour or brush.") instead:

```cpp
g.setColour (juce::Colours::sandybrown);
g.fillRect (300, 120, 200, 170);
```

Instead of giving the position, width, and height separately, there is a more convenient class to represent a rectangle: the [Rectangle](https://docs.juce.com/master/classRectangle.html "Manages a rectangle and allows geometric operations to be performed on it.") class. There is also a version of the [Graphics::drawRect()](https://docs.juce.com/master/classGraphics.html#a42dc569175294c1e20e8177031a761af "Draws a rectangular outline, using the current colour or brush.") function that takes such a [Rectangle](https://docs.juce.com/master/classRectangle.html "Manages a rectangle and allows geometric operations to be performed on it.") instance to specify the position of the rectangle:

```cpp
juce::Rectangle<int> house (300, 120, 200, 170);
g.setColour (juce::Colours::sandybrown);
g.fillRect (house);
```

This very convenient [Rectangle](https://docs.juce.com/master/classRectangle.html "Manages a rectangle and allows geometric operations to be performed on it.") class will be explored in a future tutorial.

> [!NOTE]
> Exercise: Find out how to draw a _rounded_ rectangle. Next, also try to draw a _filled_ rounded rectangle.

You don\'t have to fill the rectangle with a solid colour. You can also use a colour gradient or one of several other patterns. Let's imagine that the brown rectangle represents a house. We can add a brick-like texture by filling it with a checkered pattern. Use the following code to draw the rectangle:

```cpp
juce::Rectangle<float> house (300, 120, 200, 170);
g.fillCheckerBoard (house, 30, 10, juce::Colours::sandybrown, juce::Colours::saddlebrown);
```

If you compile and run the application now, it should look as follows:

![](/_images/tutorial_graphics_class_screenshot2.png "Adding a line and a rectangle filled with a checkered pattern.")

## Drawing circles

Let's see how the [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") class draws circles and ellipses. Have a look at the functions [Graphics::drawEllipse()](https://docs.juce.com/master/classGraphics.html#ae4cce288a14b690ce0eb3dd514559887 "Draws an elliptical stroke using the current colour or brush.") and [Graphics::fillEllipse()](https://docs.juce.com/master/classGraphics.html#ae965bf1c5fadee50c665b6508eb3e8f4 "Fills an ellipse with the current colour or brush."). They work just like the [Graphics::drawRect()](https://docs.juce.com/master/classGraphics.html#a42dc569175294c1e20e8177031a761af "Draws a rectangular outline, using the current colour or brush.") and [Graphics::fillRect()](https://docs.juce.com/master/classGraphics.html#a747de9976729f72d9f6188104e6b2215 "Fills a rectangle with the current colour or brush.") functions.

Let's add a sun to our little landscape. The following code will draw a circle 60 pixels across in the upper-right region of the window:

```cpp
g.setColour (juce::Colours::yellow);
g.drawEllipse (530, 10, 60, 60, 3);
```

Note that the position given `(530, 10)` does _not_ place the centre of the circle at that position. Instead, as with all other graphical elements, the object will be placed such that the top-left corner of its enclosing rectangle will be located at the given position.

You can also explicitly use the bounds of the component to calculate the position, for example:

```cpp
g.setColour (juce::Colours::yellow);
g.drawEllipse (getWidth() - 70, 10, 60, 60, 3);
```

> [!NOTE]
> Exercise: Write a wrapper function around the [Graphics::drawEllipse()](https://docs.juce.com/master/classGraphics.html#ae4cce288a14b690ce0eb3dd514559887 "Draws an elliptical stroke using the current colour or brush.") function for drawing circles more conveniently. The function should take the coordinates of a point and a radius, and then draw a circle with the centre at this point and the given radius.

## Drawing other polygons

Finally, let's add a roof to our house. This will be a red triangle.

You will find out that there is no function called `drawTriangle()` or `drawPolygon()` in the [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") class. For this, we have to take a more generic approach.

Check out the [Path](https://docs.juce.com/master/classPath.html "A path is a sequence of lines and curves that may either form a closed shape or be open-ended.") class. It essentially handles any sets of connected points. In this case, we need a triangle that comprises three points. For example, we could use the three points `(300, 110)` , `(500, 110)` , `(400, 70)` so that the roof triangle sits on top of the house rectangle.

That's how our red roof looks like in JUCE code:

```cpp
g.setColour (juce::Colours::red);

Path roof;
roof.addTriangle (300, 110, 500, 110, 400, 70);
g.fillPath (roof);
```

Here is how the finished demo app should look if you compile and run it now:

![](/_images/tutorial_graphics_class_screenshot3.png "The finished demo app.")

> [!TIP]
>The [Path](https://docs.juce.com/master/classPath.html "A path is a sequence of lines and curves that may either form a closed shape or be open-ended.") class is capable of many other things and will be explored in more depth in a future tutorial.

# Summary

In this tutorial, we showed how to use the [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") object inside the `paint()` callback of a [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") class to draw inside a component. You should now be familiar with:

- Rendering and formatting text.
- Drawing lines.
- Drawing geometric shapes such as rectangles, circles, and polygons.

You should now also know that a [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") object is a drawing context provided by the underlying framework and should not be used outside the `paint()` callback.

# Notes

The [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") class is capable of more graphical rendering functionality than discussed in this tutorial. Notably, you can use it to draw images (from image files) on the screen. There is also much more you can do using the [Path](https://docs.juce.com/master/classPath.html "A path is a sequence of lines and curves that may either form a closed shape or be open-ended.") class. Some other features of the [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") class include colour gradients, transparency layers, and transforms. Some of these will be covered in future tutorials.

# See also

- [Tutorial: Colours in JUCE](/tutorials/tutorial_colours/)
- [Tutorial: Parent and child components](/tutorials/tutorial_component_parents_children/)
- [Tutorial: The Point, Line, and Rectangle classes](/tutorials/tutorial_point_line_rectangle/)
- [Tutorial: Advanced GUI layout techniques](/tutorials/tutorial_rectangle_advanced/)
- [Tutorial: Build an OpenGL application](/tutorials/tutorial_open_gl_application/)
