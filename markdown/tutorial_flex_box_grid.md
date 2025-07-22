---
title: Responsive GUI layouts using FlexBox and Grid
---
# Tutorial: Responsive GUI layouts using FlexBox and Grid

Build responsive GUI layouts that work across different screen sizes and orientations using the [FlexBox](https://docs.juce.com/master/classFlexBox.html "Represents a FlexBox container, which contains and manages the layout of a set of FlexItem objects.") and [Grid](https://docs.juce.com/master/classGrid.html "Container that handles geometry for grid layouts (fixed columns and rows) using a set of declarative ...") classes.

**LEVEL:** Intermediate<br/>
**PLATFORMS:** Windows, macOS, Linux, iOS, Android<br/>
**CLASSES:** [FlexBox](https://docs.juce.com/master/classFlexBox.html "Represents a FlexBox container, which contains and manages the layout of a set of FlexItem objects."), [FlexItem](https://docs.juce.com/master/classFlexItem.html "Describes the properties of an item inside a FlexBox container."), [Grid](https://docs.juce.com/master/classGrid.html "Container that handles geometry for grid layouts (fixed columns and rows) using a set of declarative ..."), [GridItem](https://docs.juce.com/master/classGridItem.html "Defines an item in a Grid.")

# Getting started

This tutorial assumes understanding of simple layout techniques using the [Rectangle](https://docs.juce.com/master/classRectangle.html "Manages a rectangle and allows geometric operations to be performed on it.") class as explained in [Tutorial: Advanced GUI layout techniques](/tutorials/tutorial_rectangle_advanced/). If you haven\'t done so already, you should read that tutorial first.

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/FlexBoxGridTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/FlexBoxGridTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project demonstrates different responsive layout techniques using [FlexBox](https://docs.juce.com/master/classFlexBox.html "Represents a FlexBox container, which contains and manages the layout of a set of FlexItem objects.") and [Grid](https://docs.juce.com/master/classGrid.html "Container that handles geometry for grid layouts (fixed columns and rows) using a set of declarative ...") objects when dealing with variable screen sizes and resolutions. If we first run the project in its initial state, it should look something like this:

![](/_images/tutorial_flex_box_grid_screenshot1.png "The demo project application window")

Right now, the layout uses common non-responsive techniques to lay out the components on the screen and does not accomodate for orientation changes. We will make use of the [FlexBox](https://docs.juce.com/master/classFlexBox.html "Represents a FlexBox container, which contains and manages the layout of a set of FlexItem objects.") and [Grid](https://docs.juce.com/master/classGrid.html "Container that handles geometry for grid layouts (fixed columns and rows) using a set of declarative ...") items to eradicate these problems.

# The FlexBox and Grid layout systems

The [FlexBox](https://docs.juce.com/master/classFlexBox.html "Represents a FlexBox container, which contains and manages the layout of a set of FlexItem objects.") and [Grid](https://docs.juce.com/master/classGrid.html "Container that handles geometry for grid layouts (fixed columns and rows) using a set of declarative ...") classes are highly inspired by the responsive layout practices used in CSS web development. If you have designed a responsive website before, you should be familiar with the layout systems described in this section.

When using [FlexBox](https://docs.juce.com/master/classFlexBox.html "Represents a FlexBox container, which contains and manages the layout of a set of FlexItem objects."), we first need to define the direction of layout as horizontal or vertical and every subsequent computation will be executed on this basis. We call this direction the _main axis_ and its perpendicular counterpart the _cross axis_. Based on this information the following properties will affect the layout as follows:

- Justification affects the position of items along the main axis.
- Alignment affects the position of items along the cross axis.
- Wrapping is performed on overflow of items on the main axis by spilling on the cross axis.

The items inside the container are defined by the [FlexItem](https://docs.juce.com/master/classFlexItem.html "Describes the properties of an item inside a FlexBox container.") class and have 3 flexible properties that affect its dynamic resizing:

- Flex-grow defines the ability for an item to grow if necessary.
- Flex-shrink defines the ability for an item to shrink if necessary.
- Flex-basis defines the default size of an item before dynamic resizing.

As a two-dimensional layout system, [Grid](https://docs.juce.com/master/classGrid.html "Container that handles geometry for grid layouts (fixed columns and rows) using a set of declarative ...") works on both the _row_ axis and the _column_ axis. Similarly to [FlexBox](https://docs.juce.com/master/classFlexBox.html "Represents a FlexBox container, which contains and manages the layout of a set of FlexItem objects."), the following properties will affect the layout as follows:

- Justification affects the position of items along the row axis.
- Alignment affects the position of items along the column axis.
- Wrapping can be performed on overflow of items on rows or columns.

[GridItem](https://docs.juce.com/master/classGridItem.html "Defines an item in a Grid.") objects are contained within the [Grid](https://docs.juce.com/master/classGrid.html "Container that handles geometry for grid layouts (fixed columns and rows) using a set of declarative ...") and have useful properties that affect its size:

- Margin can provide gaps around specific items.
- [Span](https://docs.juce.com/master/classSpan.html "A non-owning view over contiguous objects stored in an Array or vector or other similar container.") can extend items to fill more than one grid cell.

Now that we know how specific properties can affect these layout systems we can start implementing those changes in our demo project.

# Using FlexItem and GridItem objects

Let's start by replacing the button layout in the `RightSidePanel::resized()` method using [FlexBox](https://docs.juce.com/master/classFlexBox.html "Represents a FlexBox container, which contains and manages the layout of a set of FlexItem objects."):

```cpp
void resized() override
{
    juce::FlexBox fb; // [1]
    fb.flexWrap = juce::FlexBox::Wrap::wrap; // [2]
    fb.justifyContent = juce::FlexBox::JustifyContent::center; // [3]
    fb.alignContent = juce::FlexBox::AlignContent::center; // [4]

    for (auto* b : buttons) // [5]
        fb.items.add (juce::FlexItem (*b).withMinWidth (50.0f).withMinHeight (50.0f));

    fb.performLayout (getLocalBounds()); // [6]
}
```

- [1]: We create a [FlexBox](https://docs.juce.com/master/classFlexBox.html "Represents a FlexBox container, which contains and manages the layout of a set of FlexItem objects.") object.
- [2]: We can specify whether we want our objects to wrap around in case of overflow.
- [3]: We justify the content to the center of the bounds.
- [4]: We specify the alignment of the content to the center.
- [5]: We iterate over the [TextButton](https://docs.juce.com/master/classTextButton.html "A button that uses the standard lozenge-shaped background with a line of text on it.") components and add them as [FlexItem](https://docs.juce.com/master/classFlexItem.html "Describes the properties of an item inside a FlexBox container.") objects to the items array of the [FlexBox](https://docs.juce.com/master/classFlexBox.html "Represents a FlexBox container, which contains and manages the layout of a set of FlexItem objects.") object. The [FlexItem](https://docs.juce.com/master/classFlexItem.html "Describes the properties of an item inside a FlexBox container.") can be constrained as in this case where we set the minimum width and height of the button. We can alternatively set its maximum width and height using the `withMaxWidth()` and `withMaxHeight()` methods respectively.
- [6]: We perform the layout logic on the [FlexItem](https://docs.juce.com/master/classFlexItem.html "Describes the properties of an item inside a FlexBox container.") objects by specifying the bounds to the `performLayout()` method.

As for the rotary slider layout on the left side panel, we adjust our `LeftSidePanel::resized()` method accordingly:

```cpp
void resized() override
{
    //==============================================================================
    juce::FlexBox knobBox;
    knobBox.flexWrap = juce::FlexBox::Wrap::wrap;
    knobBox.justifyContent = juce::FlexBox::JustifyContent::spaceBetween; // [1]

    for (auto* k : knobs)
        knobBox.items.add (juce::FlexItem (*k).withMinHeight (50.0f).withMinWidth (50.0f).withFlex (1)); // [2]

    //==============================================================================
    juce::FlexBox fb; // [3]
    fb.flexDirection = juce::FlexBox::Direction::column;

    fb.items.add (juce::FlexItem (knobBox).withFlex (2.5)); // [4]

    fb.performLayout (getLocalBounds());
}
```

- [1]: This time we specify that we want to have the items spaced out by specifying the `JustifyContent::spaceBetween` property.
- [2]: The knobs are added in the same way to the items array with an additional flex-grow value of `1` . The flex-grow factor determines the amount of space inside the container that the item should take up.
- [3]: Another [FlexBox](https://docs.juce.com/master/classFlexBox.html "Represents a FlexBox container, which contains and manages the layout of a set of FlexItem objects.") is created to act as a container for the previously created [FlexItem](https://docs.juce.com/master/classFlexItem.html "Describes the properties of an item inside a FlexBox container.") objects and the main axis of the flex layout is set with the `Direction::column` property.
- [4]: The previously defined [FlexBox](https://docs.juce.com/master/classFlexBox.html "Represents a FlexBox container, which contains and manages the layout of a set of FlexItem objects.") is added as a [FlexItem](https://docs.juce.com/master/classFlexItem.html "Describes the properties of an item inside a FlexBox container.") to the container [FlexBox](https://docs.juce.com/master/classFlexBox.html "Represents a FlexBox container, which contains and manages the layout of a set of FlexItem objects.") with a flex-grow factor of `2.5` .

Nesting [FlexBox](https://docs.juce.com/master/classFlexBox.html "Represents a FlexBox container, which contains and manages the layout of a set of FlexItem objects.") objects allows us to create intricate responsive layouts with ease by encapsulating smaller groups of Components together.

Lastly, we can deal with the main panel sliders by making them responsive to orientation changes in the `MainPanel::resized()` method:

```cpp
void resized() override
{
    auto isPortrait = getLocalBounds().getHeight() > getLocalBounds().getWidth(); // [1]

    juce::FlexBox fb;
    fb.flexDirection = isPortrait ? juce::FlexBox::Direction::column // [2]
                                  : juce::FlexBox::Direction::row;

    for (auto* s : sliders)
    {
        s->setSliderStyle (isPortrait ? juce::Slider::SliderStyle::LinearHorizontal // [3]
                                      : juce::Slider::SliderStyle::LinearVertical);

        fb.items.add (juce::FlexItem (*s).withFlex (0, 1, isPortrait ? (float) getHeight() / 5.0f // [4]
                                                                     : (float) getWidth() / 5.0f));
    }

    fb.performLayout (getLocalBounds());
}
```

- [1]: First, we determine whether our device is in portrait or landscape by checking the width and height.
- [2]: Next, we can decide on the main axis direction accordingly and set the appropriate property.
- [3]: Similarly, we set the appropriate slider style to match the device orientation.
- [4]: When adding the sliders to the items array, we provide a flex-basis by determining the proportion of each slider in the direction of flow.

The sliders will now accomodate to the device orientation and adjust direction accordingly.

Finally, we can change the overall layout system of our panels to use flex as well:

```cpp
void resized() override
{
    juce::FlexBox fb;

    juce::FlexItem left ((float) getWidth() / 4.0f, (float) getHeight(), leftPanel);
    juce::FlexItem right ((float) getWidth() / 4.0f, (float) getHeight(), rightPanel);
    juce::FlexItem main ((float) getWidth() / 2.0f, (float) getHeight(), mainPanel);

    fb.items.addArray ({ left, main, right });
    fb.performLayout (getLocalBounds());
}
```

If we run our newly-modified code to use flex, we should see something like this:

![](/_images/tutorial_flex_box_grid_screenshot2.png "The demo project application window using flex")

> [!TIP]
>The source code for this modified version of the code can be found in the `FlexBoxGridTutorial_02.h` file of the demo project.

Let's try to implement the last portion of code using the [Grid](https://docs.juce.com/master/classGrid.html "Container that handles geometry for grid layouts (fixed columns and rows) using a set of declarative ...") class instead. Here we create a [Grid](https://docs.juce.com/master/classGrid.html "Container that handles geometry for grid layouts (fixed columns and rows) using a set of declarative ...") object to perform our layout operations on, just like flex:

```cpp
void resized() override
{
    juce::Grid grid;

    using Track = juce::Grid::TrackInfo;
    using Fr = juce::Grid::Fr;

    grid.templateRows = { Track (Fr (1)) };
    grid.templateColumns = { Track (Fr (1)), Track (Fr (2)), Track (Fr (1)) };

    grid.items = { juce::GridItem (leftPanel), juce::GridItem (mainPanel), juce::GridItem (rightPanel) };

    grid.performLayout (getLocalBounds());
}
```

However instead of specifying the flex-grow, flex-shrink and flex-basis values on the individual [FlexItem](https://docs.juce.com/master/classFlexItem.html "Describes the properties of an item inside a FlexBox container.") objects, in this case we set the number of rows and columns on the [Grid](https://docs.juce.com/master/classGrid.html "Container that handles geometry for grid layouts (fixed columns and rows) using a set of declarative ...") object using TrackInfo objects. The constraints can be specified in fractions or pixels by using the `_fr` and `_px` suffixes respectively. In this example we define a grid with 1 row and 3 columns with the center column taking twice as much space as the others.

> [!WARNING]
> Pixels in JUCE are not equivalent to physical pixels. Internal calculations convert the pixel density depending on the screen DPI resolution.

<!-- -->

> [!TIP]
>The source code for this modified version of the code can be found in the `FlexBoxGridTutorial_03.h` file of the demo project.

# Pro and cons of FlexBox and Grid classes

There are many cases where either of these classes can be used to create responsive layouts. However there are certain scenarios where one is more suitable and sometimes even necessary to solve certain layout constraints.

Some of the advantages of the [FlexBox](https://docs.juce.com/master/classFlexBox.html "Represents a FlexBox container, which contains and manages the layout of a set of FlexItem objects.") class:

- Suitable for laying out components where a main axis is desired.
- Content wrapping, direction and alignment is easily specified.
- Can accomodate unaligned content over the cross axis.

Some of the advantages of the [Grid](https://docs.juce.com/master/classGrid.html "Container that handles geometry for grid layouts (fixed columns and rows) using a set of declarative ...") class:

- Suitable for 2D grid-type layouts where rows and columns are aligned.
- Ratios of components can be specified in fractions or pixels.
- Can accomodate spanning content over multiple rows or columns.

> [!NOTE]
> Exercise: Implement the previous [FlexBox](https://docs.juce.com/master/classFlexBox.html "Represents a FlexBox container, which contains and manages the layout of a set of FlexItem objects.") layouts using the [Grid](https://docs.juce.com/master/classGrid.html "Container that handles geometry for grid layouts (fixed columns and rows) using a set of declarative ...") class instead. Were there any inconvenient use cases where the [FlexBox](https://docs.juce.com/master/classFlexBox.html "Represents a FlexBox container, which contains and manages the layout of a set of FlexItem objects.") class was more suitable?

# Summary

In this tutorial, we have learnt how to design responsive layouts using the [FlexBox](https://docs.juce.com/master/classFlexBox.html "Represents a FlexBox container, which contains and manages the layout of a set of FlexItem objects.") and [Grid](https://docs.juce.com/master/classGrid.html "Container that handles geometry for grid layouts (fixed columns and rows) using a set of declarative ...") classes. In particular, we have:

- Learnt the layout logic for [FlexItem](https://docs.juce.com/master/classFlexItem.html "Describes the properties of an item inside a FlexBox container.") and [GridItem](https://docs.juce.com/master/classGridItem.html "Defines an item in a Grid.") objects.
- Handled orientation changes and adapted our interface accordingly.
- Discussed the pros and cons for each of these classes.

# See also

- [Tutorial: Parent and child components](/tutorials/tutorial_component_parents_children/)
- [Tutorial: The Graphics class](/tutorials/tutorial_graphics_class/)
- [Tutorial: The Point, Line, and Rectangle classes](/tutorials/tutorial_point_line_rectangle/)
- [Tutorial: Advanced GUI layout techniques](/tutorials/tutorial_rectangle_advanced/)
- [Tutorial: Managing Android screen sizes](/tutorials/tutorial_android_screen_sizes/)
