# Tutorial: Parent and child components

This tutorial introduces the hierarchical nature of the [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") class whereby one component can contain one or more nested child components. This is key to laying out user interfaces in JUCE.

**LEVEL:** Beginner<br/>
**PLATFORMS:** Windows, macOS, Linux, iOS, Android<br/>
**CLASSES:** [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects."), [Path](https://docs.juce.com/master/classPath.html "A path is a sequence of lines and curves that may either form a closed shape or be open-ended."), [Colours](namespaceColours.html "Contains a set of predefined named colours (mostly standard HTML colours)")

# Getting started

> [!TIP]
>This tutorial leads on from [Tutorial: The Graphics class](/tutorials/tutorial_graphics_class/), which you should have read and understood first.

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/ComponentParentsChildrenTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/ComponentParentsChildrenTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project displays a scene containing a simple drawing of a house, as shown in the following screenshot:

![](/_images/tutorial_component_parents_children_screenshot1.png "A scene comprising separate elements.")

Does that look familiar? It's rather similar to the end result of [Tutorial: The Graphics class](/tutorials/tutorial_graphics_class/)! The difference here is that each of the parts is drawn into a separate [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") object using separate `paint()` functions. As we will see, these are grouped logically. For example, the wall of the house and its roof are grouped into a single \"house\" object.

Let's explore how this is put together and why it is a good idea to structure your components in this way.

# The Component class hierarchy

Most user interfaces comprise a number of elements, such as pieces of text, buttons, sliders, menus, and so on. For example, the following screenshot shows the [AudioDeviceSelectorComponent](https://docs.juce.com/master/classAudioDeviceSelectorComponent.html "A component containing controls to let the user change the audio settings of an AudioDeviceManager ob...") class (which is for controlling audio hardware settings, see [Tutorial: The AudioDeviceManager class](/tutorials/tutorial_audio_device_manager/) for more information). This contains a button, some labels, some menus (combo boxes), some radio buttons, and an audio level indicator.

![](/_images/tutorial_component_parents_children_screenshot2.png "A user interface for controlling audio hardware settings.")

Some individual user interface elements may also group together other user interface elements to form more useful controls. For example, the JUCE [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value.") class can contain not only the slider itself but also a text box that shows the slider's current value. This is shown in the following screenshot:

![](/_images/tutorial_component_parents_children_screenshot3.png "The JUCE Slider class.")

In each of these cases, by separating the individual elements into separate parts of hierarchy, it is much easier to design the layout of the interface (and respond to user interaction). Some components may use their `paint()` function to draw themselves. Other components may simply contain other components. Some components may contain other components _and_ perform some drawing. The design choices are quite flexible.

# The MainContentComponent class

In this tutorial the `MainContentComponent` class contains an instance of another component class as a member. This is the `SceneComponent` class, which draws the actual scene. Look at the `MainContentComponent` class within the project. A SceneComponent object is added as a private member:

```cpp
private:
    SceneComponent scene;
    JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR (MainContentComponent)
};
```

## Adding child components

Within the `MainContentComponent` constructor, this `SceneComponent` object is added as a _child_ component and the `MainContentComponent` object becomes its _parent_.

> [!WARNING]
> A child component must have one parent at any one time. You can remove a child component from its parent and then add it to another parent if you wish.

In order for the child component to be displayed, it also needs to be made visible. These two steps can be done separately, but it is a common idiom in JUCE to perform both of these actions in one single step using the Component::addAndMakeVisible() function:

```cpp
MainContentComponent()
{
    addAndMakeVisible (scene);
    setSize (600, 400);
}
```

## Setting child component bounds

While our `MainContentComponent` class sets its own size during construction, many component objects initially have a zero size. The call to the Component::setSize() function will in turn trigger a call to our `MainContentComponent::resized()` function. This is a good place to set the size and position of any child components:

```cpp
void resized() override
{
    scene.setBounds (0, 0, getWidth(), getHeight());
}
```

The important point here is that the coordinates in the call to the `SceneComponent::setBounds()` function are relative to its parent component (in this case our `MainContentComponent` object). What this means is that the top-left corner of the parent component is point `(0, 0)` and the child component will be positioned such that its top-left corner will be relative to this point. In fact our `SceneComponent` object fills the entire content of our `MainContentComponent` object. An alternative way to write this is to use the Component::getLocalBounds() function. This returns a [Rectangle](https://docs.juce.com/master/classRectangle.html "Manages a rectangle and allows geometric operations to be performed on it.") object representing the bounds of the component that calls it. This results in a rectangle with a position `(0, 0)` and a size that its width and height. This [Rectangle](https://docs.juce.com/master/classRectangle.html "Manages a rectangle and allows geometric operations to be performed on it.") object can then be passed to the `SceneComponent::setBounds()` function. The alternative code is shown in the following code snippet:

```cpp
void resized() override
{
    scene.setBounds (getLocalBounds());
}
```

The next section of this tutorial reflects the structure of this `SceneComponent` object.

> [!TIP]
>Child components may be positioned to exceed the bounds of the parent component but everything outside the parent component's bounds will not be drawn. If you can\'t see your component, make sure that the bounds have been set properly (for example, in the parent component's `resized()` function).

# The scene

The `SceneComponent` class does some of its own drawing and contains two child components (representing the floor and the house). The `SceneComponent` declaration is as follows:

```cpp
class SceneComponent : public juce::Component
{
    // ...
private:
    FloorComponent floor;
    HouseComponent house;

    //==============================================================================
    JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR (SceneComponent)
};
```

The `FloorComponent` and `HouseComponent` objects are added and made visible in the constructor:

```cpp
SceneComponent()
{
    addAndMakeVisible (floor);
    addAndMakeVisible (house);
}
```

To draw the sky, we fill the entire component with light blue in the `SceneComponent::paint()` function.

```cpp
void paint (juce::Graphics& g) override
{
    g.fillAll (juce::Colours::lightblue);
}
```

The floor and the house have their bounds positioned within the `SceneComponent::resized()` function:

```cpp
void resized() override
{
    floor.setBounds (10, 297, 580, 5);
    house.setBounds (300, 70, 200, 220);
}
```

> [!TIP]
>A component draws itself first, then child components are drawn on top of this. You can override the Component::paintOverChildren() function if you need to draw on top of child components. Child components are drawn in the order that they were added to the parent component. This can be adjusted later by using the functions Component::toFront(), Component::toBack(), Component::toBehind(), or Component::setAlwaysOnTop()

Let's look at how the floor and the house are drawn within the respective classes.

## The floor

The floor is drawn as a green horizontal line (as in [Tutorial: The Graphics class](/tutorials/tutorial_graphics_class/)) five pixels thick, centred vertically within the component, and spanning its full width. Here is the `FloorComponent::paint()` function (from the `FloorComponent` class):

```cpp
void paint (juce::Graphics& g) override
{
    g.setColour (juce::Colours::green);
    g.drawLine (0.0f, (float) getHeight() / 2.0f, (float) getWidth(), (float) getHeight() / 2.0f, 5.0f);
}
```

## The house

The house itself doesn\'t perform any drawing of its own (it does not have a `paint()` function) but comprises two other components (representing the wall and roof of the house) in the `HouseComponent` class:

```cpp
class HouseComponent : public juce::Component
{
    // ...
private:
    WallComponent wall;
    RoofComponent roof;

    //==============================================================================
    JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR (HouseComponent)
};
```

The `WallComponent` and `RoofComponent` objects are added and made visible in the constructor:

```cpp
HouseComponent()
{
    addAndMakeVisible (wall);
    addAndMakeVisible (roof);
}
```

These are positioned proportionally in the `HouseComponent::resized()` function:

```cpp
void resized() override
{
    auto separation = juce::jlimit (2, 10, getHeight() / 20); // [1]

    roof.setBounds (0, 0, getWidth(), (int) (getHeight() * 0.2) - separation / 2); // [2]
    wall.setBounds (0, (int) (getHeight() * 0.20) + separation / 2, getWidth(), (int) (getHeight() * 0.80) - separation); // [3]
}
```

- [1]: First we calculate the separation between the roof and wall. Let's make this <sup>1</sup>⁄<sub>20</sub> of the height of the house but make it no smaller than 2 pixels --- using the [jlimit()](group__juce__core-maths.html#gacf7cd562ab136d15d3bf5c3be047cf96) function. This so that there is always a gap between the roof and the wall even when the height is small. When the height is large then the gap remains proportional to the height.
- [2]: The roof is set to be the full width of the house and <sup>1</sup>/<sub>5</sub> of the height of the house. This is adjusted to account for the separation.
- [3]: The wall is positioned under the roof occupying <sup>4</sup>/<sub>5</sub> of the height of the house. This is also adjusted to account for the separation.

### The wall

The `WallComponent` class is simple. It just fills itself with a checkerboard pattern in the `WallComponent::paint()` function (from the `WallComponent` class):

```cpp
void paint (juce::Graphics& g) override
{
    g.fillCheckerBoard (getLocalBounds().toFloat(), 30, 10, juce::Colours::sandybrown, juce::Colours::saddlebrown);
}
```

### The roof

The `RoofComponent` class draws a triangle using a [Path](https://docs.juce.com/master/classPath.html "A path is a sequence of lines and curves that may either form a closed shape or be open-ended.") object in the `RoofComponent::paint()` function:

```cpp
void paint (juce::Graphics& g) override
{
    g.setColour (juce::Colours::red);

    juce::Path roof;
    roof.addTriangle (0.0f, (float) getHeight(), (float) getWidth(), (float) getHeight(), (float) getWidth() / 2.0f, 0.0f);
    g.fillPath (roof);
}
```

If we call the width of the `RoofComponent` object `w` and the height `h` then the three points that make up the triangle are: `(0, h)` , `(w, h)` , (<sup>w</sup>/<sub>2</sub>.

> [!NOTE]
> Exercise: Try changing the positions of objects within the scene.

# Adding a sun

Let's add a sun to our scene. A number of empty functions are provided for you in the `SunComponent` class, to which we will add some code in a moment.

First, we need to make some changes to the `SceneComponent` class. Add an instance of the `SunComponent` class to the private section [4]:

```cpp
private:
    FloorComponent floor;
    HouseComponent house;
    SunComponent sun; // [4]

    JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR (SceneComponent)
};
```

Then we need to add the sun and make it visible [5]:

```cpp
SceneComponent()
{
    addAndMakeVisible (floor);
    addAndMakeVisible (house);
    addAndMakeVisible (sun); // [5]
}
```

And position the sun in the top-right corner [6]:

```cpp
void resized() override
{
    floor.setBounds (10, 297, 580, 5);
    house.setBounds (300, 70, 200, 220);
    sun.setBounds (530, 10, 60, 60); // [6]
}
```

We need to add the drawing code to the `SunComponent::paint()` function (in the `SunComponent` class):

```cpp
void paint (juce::Graphics& g) override
{
    g.setColour (juce::Colours::yellow);

    auto lineThickness = 3.0f;
    g.drawEllipse (lineThickness * 0.5f,
        lineThickness * 0.5f,
        (float) getWidth() - lineThickness * 2,
        (float) getHeight() - lineThickness * 2,
        lineThickness);
}
```

Notice that we need to position the ellipse slightly within the bounds of the component. This should be dependent on the line thickness. This is because it is the centre of the line that sits exactly on the coordinates specified. For example, if we draw a line on an edge of the component, then half of the thickness of the line will sit outside the bounds of the component. Look as the following screenshot to see what could happen if we didn\'t adjust the position and size of the ellipse slightly.

![](/_images/tutorial_component_parents_children_screenshot4.png "Drawing is clipped to the bounds of the component.")

> [!TIP]
>Another way around this is to use the Component::setPaintingIsUnclipped() function to allow the component to draw beyond its bounds. Make sure you read the [API](https://juce.com/doc/classes) documentation, as there are some caveats in the use of this function.

Our final scene should look like this:

![](/_images/tutorial_component_parents_children_screenshot5.png "Our final scene.")

> [!TIP]
>The changed code for this step can be found in the `ComponentParentsChildrenTutorial_02.h` file of the demo project.

<!-- -->

> [!NOTE]
> Exercise: Using the code above, the sun is always drawn as an ellipse. Since we specify the `SunComponent` object to be a square, we don\'t notice this potential problem. Fix the `SunComponent` class such that it always draws a circle within its bounds --- rather than an ellipse --- even if its width and height aren\'t the same.

# Reusing components

One of the main benefits of the coordinate system used by the [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") class, is that drawing is always performed relative to the top-left of the component. Another benefit of encapsulating drawing into a separate class is that it can be reused easily.

For example, we can easily add another house [7] to the `SceneComponent` class:

```cpp
private:
    FloorComponent floor;
    HouseComponent house;
    HouseComponent smallHouse; // [7]
    SunComponent sun;

    JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR (SceneComponent)
};
```

Then add it and make it visible [8] in the `SceneComponent` constructor:

```cpp
SceneComponent()
{
    addAndMakeVisible (floor);
    addAndMakeVisible (house);
    addAndMakeVisible (smallHouse); // [8]
    addAndMakeVisible (sun);
}
```

And position it [9] in the `SceneComponent::resized()` function:

```cpp
void resized() override
{
    floor.setBounds (10, 297, 580, 5);
    house.setBounds (300, 70, 200, 220);
    smallHouse.setBounds (50, 50, 50, 50); // [9]
    sun.setBounds (530, 10, 60, 60);
}
```

With the small house added, the scene should look like this:

![](/_images/tutorial_component_parents_children_screenshot6.png "Our final scene with an additional small house.")

> [!TIP]
>The changed code for this final step can be found in the `ComponentParentsChildrenTutorial_03.h` file of the demo project.

<!-- -->

> [!NOTE]
> Exercise: Create a new class `StreetComponent` that contains a number of `HouseComponent` objects in a row to form a street, and add it to the project. Modify the `SceneComponent` class such that it contains some streets and individual houses.

# Summary

In this tutorial we have introduced the hierarchical parent and child system that is used by the [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") class. In particular, we have learned:

- How to use the Component::addAndMakeVisible() function to add child components to other components.
- How to position and size child components relative to their parent components.
- That components can do their own drawing, contain child component that perform drawing, or both.
- That components perform their drawing first, then child components are painted in the order that they were added to the parent.
- Drawing is normally clipped to the component's bounds.

# See also

- [Tutorial: The Graphics class](/tutorials/tutorial_graphics_class/)
- [Tutorial: Customise the look and feel of your app](/tutorials/tutorial_look_and_feel_customisation/)
- [Tutorial: The Point, Line, and Rectangle classes](/tutorials/tutorial_point_line_rectangle/)
- [Tutorial: Advanced GUI layout techniques](/tutorials/tutorial_rectangle_advanced/)
- [Tutorial: Build an OpenGL application](/tutorials/tutorial_open_gl_application/)
