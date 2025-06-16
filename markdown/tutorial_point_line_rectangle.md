# Tutorial: The Point, Line, and Rectangle classes

Use the [Point](https://docs.juce.com/master/classPoint.html "A pair of (x, y) coordinates."), [Line](https://docs.juce.com/master/classLine.html "Represents a line."), and [Rectangle](https://docs.juce.com/master/classRectangle.html "Manages a rectangle and allows geometric operations to be performed on it.") classes to simplify your geometry calculations.

**LEVEL:** Beginner<br/>
**PLATFORMS:** Windows, macOS, Linux, iOS, Android<br/>
**CLASSES:** [Point](https://docs.juce.com/master/classPoint.html "A pair of (x, y) coordinates."), [Line](https://docs.juce.com/master/classLine.html "Represents a line."), [Rectangle](https://docs.juce.com/master/classRectangle.html "Manages a rectangle and allows geometric operations to be performed on it."), [Path](https://docs.juce.com/master/classPath.html "A path is a sequence of lines and curves that may either form a closed shape or be open-ended."), [Random](https://docs.juce.com/master/classRandom.html "A random number generator."), [Range](https://docs.juce.com/master/classRange.html "A general-purpose range object, that simply represents any linear range with a start and end point.")

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/PointLineRectangleTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/PointLineRectangleTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project performs some simple drawing operations. In its default state it draws a grey background with a small orange square in the top-left corner:

![](/_images/tutorial_point_line_rectangle_screenshot1.png "Simple drawing of a square")

We are going to look at different ways of drawing lines and rectangles, and how the [Point](https://docs.juce.com/master/classPoint.html "A pair of (x, y) coordinates."), [Line](https://docs.juce.com/master/classLine.html "Represents a line."), and [Rectangle](https://docs.juce.com/master/classRectangle.html "Manages a rectangle and allows geometric operations to be performed on it.") classes can simplify the way you think about drawing operations (and component positions) in JUCE.

# Rectangle basics

A great deal of graphics drawing and component layout code needs to deal with rectangles. This tutorial starts with the simple drawing of a filled square. We are now going to explore how the [Point](https://docs.juce.com/master/classPoint.html "A pair of (x, y) coordinates."), [Line](https://docs.juce.com/master/classLine.html "Represents a line."), and [Rectangle](https://docs.juce.com/master/classRectangle.html "Manages a rectangle and allows geometric operations to be performed on it.") classes can help us with drawing operations. We can also apply these techniques to the positioning of child components (in your Component::resized() function). Our starter code in the `paint()` function is this:

```cpp
void paint (juce::Graphics& g) override
{
    g.fillAll (juce::Colours::darkgrey);
    g.setColour (juce::Colours::orange);

    g.fillRect (10, 10, 40, 40);
}
```

Here we specify the bounds of the rectangle directly to the [Graphics::fillRect()](https://docs.juce.com/master/classGraphics.html#a747de9976729f72d9f6188104e6b2215 "Fills a rectangle with the current colour or brush.") function as separate integers:

```cpp
g.fillRect (10, // x
    10, // y
    40, // width
    40); // height
```

While this is straightforward, if we specify the bounds as a [Rectangle](https://docs.juce.com/master/classRectangle.html "Manages a rectangle and allows geometric operations to be performed on it.") object, it becomes much easier to perform manipulations of the rectangle.

## Using the Rectangle and Point classes

It is easy to replace these separate coordinates, width, and height values with a [Rectangle](https://docs.juce.com/master/classRectangle.html "Manages a rectangle and allows geometric operations to be performed on it.") object, like so [1]:

```cpp
void paint (juce::Graphics& g) override
{
    g.fillAll (juce::Colours::darkgrey);

    juce::Rectangle<int> area (10, 10, 40, 40); // [1]

    g.setColour (juce::Colours::orange);
    g.fillRect (area);
}
```

While this is a very similar way of drawing the square, we can now very easily move the square around the component bounds, keeping its original size. The [Rectangle](https://docs.juce.com/master/classRectangle.html "Manages a rectangle and allows geometric operations to be performed on it.") class is a template class, as you can see above. In this example we use an `int` template parameter. In JUCE drawing code we will commonly use either [Rectangle\<int\>](https://docs.juce.com/master/classRectangle.html) or [Rectangle\<float\>](https://docs.juce.com/master/classRectangle.html) objects. The [Point](https://docs.juce.com/master/classPoint.html "A pair of (x, y) coordinates.") and [Line](https://docs.juce.com/master/classLine.html "Represents a line.") classes are also template classes, and again we commonly use either `int` or `float` template parameters.

There are other ways to create rectangles. For example, rather than specifying width and height, we might have two points that we want to use as corners of the rectangle. We can use the [Point](https://docs.juce.com/master/classPoint.html "A pair of (x, y) coordinates.") class and a different [Rectangle](https://docs.juce.com/master/classRectangle.html "Manages a rectangle and allows geometric operations to be performed on it.") constructor:

```cpp
void paint (juce::Graphics& g) override
{
    g.fillAll (juce::Colours::darkgrey);
    g.setColour (juce::Colours::orange);
    juce::Rectangle<int> area (juce::Point<int> (10, 10),
        juce::Point<int> (50, 50));

    g.fillRect (area);
}
```

One of the great things about this technique is that we can specify any two points. These points don\'t need to represent the top-left and bottom-right corners of the rectangle. Therefore, an equivalent would be:

```cpp
juce::Rectangle<int> area (juce::Point<int> (10, 50),
    juce::Point<int> (50, 10));
```

## Using the Point and Path classes

In fact, using a [Path](https://docs.juce.com/master/classPath.html "A path is a sequence of lines and curves that may either form a closed shape or be open-ended.") object we can specify a rectangle with four points defining each corner:

```cpp
void paint (juce::Graphics& g) override
{
    g.fillAll (juce::Colours::darkgrey);
    g.setColour (juce::Colours::orange);

    juce::Path path;
    path.startNewSubPath (juce::Point<float> (10, 10));
    path.lineTo (juce::Point<float> (50, 10));
    path.lineTo (juce::Point<float> (50, 50));
    path.lineTo (juce::Point<float> (10, 50));
    path.closeSubPath();

    g.fillPath (path);
}
```

> [!TIP]
>In this case we need to use the [Point\<float\>](https://docs.juce.com/master/classPoint.html) class, since that is what the [Path](https://docs.juce.com/master/classPath.html "A path is a sequence of lines and curves that may either form a closed shape or be open-ended.") class requires us to use. Although we can use the Point\<int\>::toFloat() and Point\<float\>::toInt() functions to convert between the two types of point.

<!-- -->

> [!NOTE]
> Exercise: Try changing the points in the path to create other quadrilateral shapes.

We can also add a rectangle directly to a path:

```cpp
void paint (juce::Graphics& g) override
{
    g.fillAll (juce::Colours::darkgrey);
    g.setColour (juce::Colours::orange);

    juce::Path path;
    juce::Rectangle<float> area (10, 10, 40, 40);
    path.addRectangle (area);

    g.fillPath (path);
}
```

> [!TIP]
>We could have used a [Rectangle\<int\>](https://docs.juce.com/master/classRectangle.html) object in this case, as the [Path](https://docs.juce.com/master/classPath.html "A path is a sequence of lines and curves that may either form a closed shape or be open-ended.") class will convert this to the floating-point version on our behalf.

## Testing points within rectangles

Another useful feature of a [Rectangle](https://docs.juce.com/master/classRectangle.html "Manages a rectangle and allows geometric operations to be performed on it.") object is that it can test whether it contains a specified point. To test this out, we are going to get our component to repaint itself each time we click the mouse. To do this, add the following function:

```cpp
void mouseDown (const juce::MouseEvent&) override
{
    repaint();
}
```

Now let's write our `paint()` function such that it randomly generates a rectangle and a point. Then it will draw the rectangle and then a smaller rectangle located around the point. This smaller rectangle will be drawn in a different colour depending upon whether the randomly-generated point is within the larger rectangle or not. Our `paint()` function should start as before, but we\'re going to generate some random values so let's cache a reference of the system [Random](https://docs.juce.com/master/classRandom.html "A random number generator.") object too.

```cpp
void paint (juce::Graphics& g) override
{
    g.fillAll (juce::Colours::darkgrey);
    g.setColour (juce::Colours::orange);

    auto& random = juce::Random::getSystemRandom();
```

Now we\'ll create our random rectangle. Add the following code:

```cpp
juce::Range<int> rectRange (20, getWidth() / 2);
juce::Rectangle<int> rectArea (random.nextInt (rectRange),
    random.nextInt (rectRange),
    random.nextInt (rectRange),
    random.nextInt (rectRange));

g.drawRect (rectArea, 2);
```

We limit the range of random values for each of the rectangle elements using a [Range](https://docs.juce.com/master/classRange.html "A general-purpose range object, that simply represents any linear range with a start and end point.") object. We also draw (rather than fill) the rectangle with a thickness of 2 points. To draw the smaller rectangle we are going to need another [Rectangle](https://docs.juce.com/master/classRectangle.html "Manages a rectangle and allows geometric operations to be performed on it.") object. If we use only two arguments to the [Rectangle](https://docs.juce.com/master/classRectangle.html "Manages a rectangle and allows geometric operations to be performed on it.") constructor, it creates a [Rectangle](https://docs.juce.com/master/classRectangle.html "Manages a rectangle and allows geometric operations to be performed on it.") object with the specified width and height (in that order) at a position of zero (0, 0). Add this line:

```cpp
juce::Rectangle<int> pointArea (10, 10);
```

Now let's randomly-generate a point and position the centre of the `pointArea` rectangle at that point. Add the following code:

```cpp
juce::Point<int> point (random.nextInt (juce::Range<int> (0, getWidth())),
    random.nextInt (juce::Range<int> (0, getHeight())));
pointArea.setCentre (point);
```

This demonstrates another useful feature of the [Rectangle](https://docs.juce.com/master/classRectangle.html "Manages a rectangle and allows geometric operations to be performed on it.") class as we can position its centre if we wish. This is sometimes preferable to considering a rectangle's position to be its top-left corner. Now we can use the [Rectangle::contains()](https://docs.juce.com/master/classRectangle.html#a48d94ffec18b4b4755b316c3b223c645 "Returns true if this coordinate is inside the rectangle.") function to determine whether the `point` object is within the bounds of the `rectArea` object. Add this code:

```cpp
g.setColour (rectArea.contains (point) ? juce::Colours::limegreen
                                       : juce::Colours::cornflowerblue);

g.fillRect (pointArea);
```

Run the application, it should look something like the following screenshot. Remember to click on the component in order to cause it to redraw:

![](/_images/tutorial_point_line_rectangle_screenshot4.png "Testing points within rectangles")

> [!TIP]
>The code for this example can be found in the `PointLineRectangleTutorial_02.h` file of the demo project.

# Dealing with lines

Drawing and dealing with lines is similarly straightforward. The following code draws a diagonal line:

```cpp
void paint (juce::Graphics& g) override
{
    g.fillAll (juce::Colours::darkgrey);
    g.setColour (juce::Colours::orange);

    juce::Line<float> line (juce::Point<float> (10, 10),
        juce::Point<float> (50, 50));

    g.drawLine (line, 2.0f);
}
```

## Line intersections

The [Line](https://docs.juce.com/master/classLine.html "Represents a line.") class can also perform line intersection tests. To test this out we are going to generate several randomly-generated lines, then we are going to draw a circle at the points where any of these lines intersect with any of the other lines. First let's set up our `paint()` function with a background and get ready for generating some random numbers:

```cpp
void paint (juce::Graphics& g) override
{
    g.fillAll (juce::Colours::darkgrey);
    g.setColour (juce::Colours::orange);

    auto& random = juce::Random::getSystemRandom();
```

Now let's generate the random lines, not only drawing them, but also storing them in an array. Add the following code:

```cpp
juce::Range<int> lineRange (0, getWidth());
juce::Array<juce::Line<float>> lines;
auto numLines = 10;

for (auto i = 0; i < numLines; ++i)
{
    juce::Line<float> line ((float) random.nextInt (lineRange),
        (float) random.nextInt (lineRange),
        (float) random.nextInt (lineRange),
        (float) random.nextInt (lineRange));

    lines.add (line);
    g.drawLine (line, 2.0f);
}
```

Then we\'ll change our colour and prepare a square which we will use as the bounds within which to paint our circle. Add this:

```cpp
g.setColour (juce::Colours::palegreen);
juce::Rectangle<float> pointArea (8, 8);
```

Finally, we\'ll iterate over the array of lines checking for intersections with any of the other lines using the [Line::intersects()](https://docs.juce.com/master/classLine.html#ab65eb5685ecd0cfa39bee23c3d4f36d7 "Finds the intersection between two lines.") function. We then move the centre of the `pointArea` rectangle to this point and paint a circle. To do this, add the following code:

```cpp
for (auto lineI : lines)
{
    for (auto lineJ : lines)
    {
        if (lines.indexOf (lineI) != lines.indexOf (lineJ))
        {
            juce::Point<float> intersection;

            if (lineI.intersects (lineJ, intersection)) // [2]
            {
                pointArea.setCentre (intersection);
                g.fillEllipse (pointArea);
            }
        }
    }
}
}
```

The piece of code that checks for the intersection [2], calls the [Line::intersects()](https://docs.juce.com/master/classLine.html#ab65eb5685ecd0cfa39bee23c3d4f36d7 "Finds the intersection between two lines.") function. This not only returns `true` if the lines actually intersect, but also returns the point at which they intersect in the `intersection` argument.

Run the application now. If you left in the code to repaint in response to mouse clicks then you can generate new sets of lines by clicking on the component.

![](/_images/tutorial_point_line_rectangle_screenshot5.png "Line intersections")

> [!TIP]
>The code for this example can be found in the `PointLineRectangleTutorial_03.h` file of the demo project.

If we didn\'t check the `bool` returned by the [Line::intersects()](https://docs.juce.com/master/classLine.html#ab65eb5685ecd0cfa39bee23c3d4f36d7 "Finds the intersection between two lines.") function, or we used the [Line::getIntersection()](https://docs.juce.com/master/classLine.html#a7d4ae6edb918339e3dad61a7bae179d0 "Finds the intersection between two lines.") instead, then points would be drawn in places where the lines _would_ intersect should they be extended in each direction to an infinite length. For example, look at the following code:

```cpp
//..
if (lines.indexOf (lineI) != lines.indexOf (lineJ))
{
    juce::Point<float> intersection;
    pointArea.setCentre (lineI.getIntersection (lineJ));
    g.fillEllipse (pointArea);
}
//..
```

This would generate something like the following screenshot:

![](/_images/tutorial_point_line_rectangle_screenshot6.png "Line intersections beyond the endpoints")

> [!NOTE]
> Exercise: The final `for()` loop in the code for this example is simple, but not ideal. The problem is that it checks each line against every other line _twice_. Rewrite this code such that it only checks each pair of lines once. You should be able to remove the `if (lines.indexOf (lineI) != lines.indexOf (lineJ))` statement as part of this rewrite.

# Manipulating rectangles

Now let's look at some more manipulations of rectangles that we can perform. Before we start, let's add a simple function to generate a random colour (see [Tutorial: The Random class](/tutorials/tutorial_random/)) as we\'re going to be doing this quite a number of times in the following examples:

```cpp
static juce::Colour getRandomColour()
{
    auto& random = juce::Random::getSystemRandom();

    return juce::Colour ((juce::uint8) random.nextInt (256),
        (juce::uint8) random.nextInt (256),
        (juce::uint8) random.nextInt (256));
}
```

Now, let's extend the code for drawing the rectangle by using the [Rectangle](https://docs.juce.com/master/classRectangle.html "Manages a rectangle and allows geometric operations to be performed on it.") class, by drawing ten squares in a diagonal pattern, and using a randomly-generated colour:

```cpp
void paint (juce::Graphics& g) override
{
    g.fillAll (juce::Colours::darkgrey);

    juce::Rectangle<int> area (10, 10, 40, 40);

    auto numSquares = 10;

    for (auto i = 0; i < numSquares; ++i)
    {
        g.setColour (getRandomColour());
        g.fillRect (area);

        area.translate (30, 30); // [3]
    }
}
```

The [Rectangle::translate()](https://docs.juce.com/master/classRectangle.html#aa13dfd466078c2710fdea6ff2f7b263b "Moves the rectangle's position by adding amount to its x and y coordinates.") function [3] moves the given rectangle by the horizontal and vertical offsets provided. The result should look something like the following screenshot:

![](/_images/tutorial_point_line_rectangle_screenshot2.png "Diagonal pattern of squares")

Here is an extension to that code that resizes the rectangle before the next drawing operation. In addition to this, the translation is performed such that it is equal to the width and height of the rectangle.

```cpp
void paint (juce::Graphics& g) override
{
    g.fillAll (juce::Colours::darkgrey);

    juce::Rectangle<int> area (10, 10, 40, 40);

    auto& random = juce::Random::getSystemRandom();
    auto numSquares = 10;

    for (auto i = 0; i < numSquares; ++i)
    {
        g.setColour (getRandomColour());
        g.fillRect (area);

        area.translate (area.getWidth(), area.getHeight()); // [4]
        area.setSize (random.nextInt (juce::Range<int> (20, 40)), // width
            random.nextInt (juce::Range<int> (20, 40))); // height
    }
}
```

This causes the rectangles to be \"joined\" at their corners as shown in the following screenshot:

![](/_images/tutorial_point_line_rectangle_screenshot3.png "Rectangles joined at the corners")

As an alternative to using the [Rectangle::translate()](https://docs.juce.com/master/classRectangle.html#aa13dfd466078c2710fdea6ff2f7b263b "Moves the rectangle's position by adding amount to its x and y coordinates.") function, we could instead use addition to \"add\" points to rectangle in order to translate them. This means we could replace [4] above with the following line:

```cpp
area += juce::Point<int> (area.getWidth(), area.getHeight());
```

## Rectangle intersections

If we have rectangles that overlap, we can determine the intersecting area using the [Rectangle::getIntersection()](https://docs.juce.com/master/classRectangle.html#a50e80bae322913e844bc628a99fa3166 "Returns the region that is the overlap between this and another rectangle.") function. In the following example we calculate the intersecting rectangle between the current and next rectangles in the series. To make this clear, we draw outlines for each rectangle in the series and highlight the intersecting areas by drawing them as filled rectangles.

```cpp
void paint (juce::Graphics& g) override
{
    g.fillAll (juce::Colours::darkgrey);

    juce::Rectangle<int> area (10, 10, 40, 40);

    auto& random = juce::Random::getSystemRandom();
    juce::Range<int> rectRandomRange (20, 40);
    auto numSquares = 10;

    for (auto i = 0; i < numSquares; ++i)
    {
        auto nextArea = area + juce::Point<int> (random.nextInt (rectRandomRange), // [5]
                            random.nextInt (rectRandomRange));

        g.setColour (getRandomColour());
        g.fillRect (area.getIntersection (nextArea)); // [6]

        g.setColour (getRandomColour());
        g.drawRect (area, 2); // [7]

        area = nextArea;
    }
}
```

Notice that we use the `+` operator to offset the rectangle [5]. We also fill the intersecting areas first [6] before drawing the rectangle outlines [7]. This causes the last intersecting area to be drawn without its final corresponding rectangle.

![](/_images/tutorial_point_line_rectangle_screenshot7.png "Rectangle intersections")

> [!TIP]
>The code for this final example can be found in the `PointLineRectangleTutorial_04.h` file of the demo project.

# Summary

In this tutorial we have introduced the template classes [Point](https://docs.juce.com/master/classPoint.html "A pair of (x, y) coordinates."), [Line](https://docs.juce.com/master/classLine.html "Represents a line."), and [Rectangle](https://docs.juce.com/master/classRectangle.html "Manages a rectangle and allows geometric operations to be performed on it."). In particular, we have covered the following techniques:

- Creating and manipulating [Rectangle](https://docs.juce.com/master/classRectangle.html "Manages a rectangle and allows geometric operations to be performed on it.") objects and using them in drawing code.
- Creating [Rectangle](https://docs.juce.com/master/classRectangle.html "Manages a rectangle and allows geometric operations to be performed on it.") objects from corner points or by intersections of two rectangles.
- Creating [Path](https://docs.juce.com/master/classPath.html "A path is a sequence of lines and curves that may either form a closed shape or be open-ended.") objects containing rectangles.
- Creating [Line](https://docs.juce.com/master/classLine.html "Represents a line.") objects, drawing lines, and finding points where lines intersect.

# See also

- [Tutorial: Parent and child components](/tutorials/tutorial_component_parents_children/)
- [Tutorial: The Graphics class](/tutorials/tutorial_graphics_class/)
- [Tutorial: The Random class](/tutorials/tutorial_random/)
- [Tutorial: Advanced GUI layout techniques](/tutorials/tutorial_rectangle_advanced/)
- [Tutorial: Responsive GUI layouts using FlexBox and Grid](/tutorials/tutorial_flex_box_grid/)
