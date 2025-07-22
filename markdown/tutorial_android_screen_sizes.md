---
title: Managing Android screen sizes
---
# Tutorial: Managing Android screen sizes

Build your application for different screen sizes. There are many available screen sizes on Android, this tutorial examines some strategies to manage this.

**LEVEL:** Intermediate<br/>
**PLATFORMS:** Android , macOS, Windows
**CLASSES:** [Desktop](https://docs.juce.com/master/classDesktop.html "Describes and controls aspects of the computer's desktop."), [AffineTransform](https://docs.juce.com/master/classAffineTransform.html "Represents a 2D affine-transformation matrix."), [TabbedComponent](https://docs.juce.com/master/classTabbedComponent.html "A component with a TabbedButtonBar along one of its sides.")

# Getting started

This tutorial illustrates a handful of strategies for managing different screen sizes on the Android platform using JUCE. There are several demo projects to accompany this tutorial. Download links to these projects are provided in the relevant sections of the tutorial.

If you need help with this step in each of these sections, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo projects

The demo projects provided with this tutorial illustrate several different methods for managing different screen sizes on the Android platform using JUCE. Broadly, these methods are:

- Resizing child components within the main component.
- Resizing the main component using a transform.
- Designing different component layouts for different orientations.
- Designing different component layouts for different sizes.

# Android screen sizes

With devices that expect fullscreen operation in particular (such as mobile devices) it is a challenge to design user interfaces that are effective on all screen sizes and for various device orientations. This is a particular challenge on the Android platform where there are many possible screen sizes and resolutions. There are three main issues here:

- _Physical size_: The physical size of the screen measure in standard units of measure (the common measure being the distance across the diagonal of the screen, measured in inches).
- _Resolution_: The screen resolution measured in pixels.
- _Orientation_: The device orientation, whether landscape or portrait.

The relationship between the physical size and the resolution is important. It is especially important when considering high resolution screens where the physical pixels are smaller, and more densely packed, than standard resolution screens. The combination of a particular physical screen size and its resolution results in the screen's _dots-per-inch_ (DPI). This is related to the screen _pixel density_. This is how many physical pixels take up the space of a \"software\" pixel, in each dimension, as a pixel on a standard density screen.

For some applications the physical size will be most important. This may be, for example, where an application uses complex interaction comprising delicate finger movements. In this case the screen size and the size of the typical user's hand is important. In other applications the DPI of the screen is more important. For example, text will remain readable using smaller font sizes at higher DPIs. But there is a limit to how readable text will be, when measured using its physical size on the screen. You will sometimes need to take both the physical size and the resolution (and therefore the DPI) into account when designing your application.

By default, JUCE scales its coordinate system based on the pixel density of the screen. This means that shapes and text drawn on a high density screen should appear roughly the same physical size as they do on standard density screens. In JUCE, you can access information about a particular display via the [Desktop](https://docs.juce.com/master/classDesktop.html "Describes and controls aspects of the computer's desktop.") class. Here you can find the available displays and which one is marked as the \"main display\" (especially if there are multiple displays).

Unfortunately, the value that JUCE can access to obtain the display's DPI is only an approximation (since not all screen devices report this information properly). This means we can\'t measure the physical size of the user's screen accurately. But the information provided by the [Desktop](https://docs.juce.com/master/classDesktop.html "Describes and controls aspects of the computer's desktop.") class should be good enough as a guide for scaling your user interface depending on the needs of your application.

In each of the examples that follow we use a child component, called `ResizingComp` , that is managed and resized by the parent component (`MainContentComponent` ).

> [!TIP]
>If you test these projects on macOS or Windows you will be able to dynamically resize the width and height of the main window. While this works to some extent, it is not intended as a feature of the projects, except for testing purpose. The projects are designed to expect infrequent changes in size. For example, a one-time setting when the application launches, or when the user rotates the Android device.

# Resizing child components (simple resize)

Download the demo project for this section here: [PIP](https://docs.juce.com/tutorials/PIPs/AndroidScreenSizesTutorialSimpleResize.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/AndroidScreenSizesTutorialSimpleResize.zip) . Unzip the project and open the first header file in the Projucer.

In this example we use a simple interface containing a collection of sliders and buttons. Each of these child components is given a proportion of the screen's height (minus a small border around the edge). A similar approach could be taken in the horizontal dimension. For simplicity, the sliders and buttons just take up the whole width of the screen (again, minus a small border). With a screen in portrait orientation and a size in the region of a few hundred pixels this will look similar to the following screenshot:

![](/_images/tutorial_android_screen_sizes_screenshot1.png "Simple resizing of child components: Portrait")

In landscape orientation it will look something like the following screenshot:

![](/_images/tutorial_android_screen_sizes_screenshot2.png "Simple resizing of child components: Landscape")

## Arrays of components

To store the buttons and sliders in the `ResizingComp` class we use the [OwnedArray](https://docs.juce.com/master/classOwnedArray.html "An array designed for holding objects.") template class (which means that these child components will be deleted automatically in the `ResizingComp` destructor). First, in the `ResizingComp` constructor, we build an array of [Colour](https://docs.juce.com/master/classColour.html "Represents a colour, also including a transparency value.") objects. These are used to set the colours of the buttons, the slider thumbs, and the slider tracks:

<!-- wp:code -->

```cpp
ResizingComp()
{
    juce::Array<juce::Colour> colours { juce::Colour (0xffb3c3Da), juce::Colour (0xff5973b8), juce::Colour (0xffd65667), juce::Colour (0xffd99154), juce::Colour (0xffe5ad6c), juce::Colour (0xffecc664), juce::Colour (0xffefe369), juce::Colour (0xffdddB74) };
```

<!-- /wp:code -->

> [!TIP]
>These happen to be the colours from the JUCE logo!

We then use a `for()` loop to allocate and configure the multiple buttons:

```cpp
for (auto i = 0; i < 6; ++i)
{
    auto* button = buttons.add (new juce::TextButton (juce::String ("Button ") + juce::String (i + 1)));
    addAndMakeVisible (button);

    button->setColour (juce::TextButton::buttonColourId,
        colours.getUnchecked (i % colours.size()));
}
```

And the sliders are set up similarly (although we mix up the colour selection using the array of colours to keep it interesting).

```cpp
for (auto i = 0; i < 6; ++i)
{
    auto* slider = sliders.add (new juce::Slider());
    addAndMakeVisible (slider);

    slider->setColour (juce::Slider::thumbColourId,
        colours.getUnchecked ((buttons.size() + i) % colours.size()));
    slider->setColour (juce::Slider::backgroundColourId,
        colours.getUnchecked ((buttons.size() + i + 2) % colours.size()).withAlpha (0.4f));
    slider->setColour (juce::Slider::trackColourId,
        colours.getUnchecked ((buttons.size() + i + 2) % colours.size()));
    slider->setColour (juce::Slider::textBoxTextColourId, juce::Colours::black);
}
```

## Using a custom slider thumb size

In order to be more usable with a touchscreen interface the slider thumb has been customised so that it is usually larger than the standard size. To do this we have added a subclass of the [LookAndFeel_V4](https://docs.juce.com/master/classLookAndFeel__V4.html "The latest JUCE look-and-feel style, as introduced in 2017.") and overridden the [LookAndFeel::getSliderThumbRadius()](structSlider_1_1LookAndFeelMethods.html#a71905c89182698a67da8929077e768f8) function.

```cpp
class CustomLookAndFeel : public juce::LookAndFeel_V4
{
public:
    int getSliderThumbRadius (juce::Slider& slider) override
    {
        return juce::jmin (slider.getWidth(), slider.getHeight()) / 2;
    }
};
```

We add an instance of this class as a member of our `ResizingComp` class:

```cpp
juce::OwnedArray<juce::Button> buttons;
juce::OwnedArray<juce::Slider> sliders;
CustomLookAndFeel lf;
};
```

And at the end of our `ResizingComp` constructor we set this as our look-and-feel for this component and all of its children.

```cpp
setLookAndFeel (&lf);
```

And in our `ResizingComp` destructor we set this to nullptr.

```cpp
~ResizingComp() override
{
    setLookAndFeel (nullptr);
}
```

## Resizing the buttons and sliders

In the `ResizingComp::resized()` function we iterate over the arrays of buttons and sliders and set their bounds:

```cpp
void resized() override
{
    auto space = 8;
    auto widgetHeight = (getHeight() - space) / (buttons.size() + sliders.size()) - space;

    for (auto* button : buttons)
        button->setBounds (space, space + (widgetHeight + space) * buttons.indexOf (button), getWidth() - space - space, widgetHeight);

    for (auto* slider : sliders)
        slider->setBounds (space, space + (widgetHeight + space) * (sliders.indexOf (slider) + buttons.size()), getWidth() - space - space, widgetHeight);
}
```

Here we us a constant value (8) to separate the components. Then we calculate the \"widget height\" based on the available height and the number of \"widgets\" (buttons and sliders) that we have.

If the screen size is too small then the interface becomes unusable and unreadable as shown in the following screenshot:

![](/_images/tutorial_android_screen_sizes_screenshot3.png "Simple resizing when the screen is too small")

Having said this, it should look reasonable on most Android devices.

> [!NOTE]
> Exercise: Vary the number of sliders and buttons in the interface.

# Resizing the main component using a transform

Download the demo project for this section here: [PIP](https://docs.juce.com/tutorials/PIPs/AndroidScreenSizesTutorialTransform.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/AndroidScreenSizesTutorialTransform.zip) . Unzip the project and open the first header file in the Projucer.

This example uses an alternative to resizing the child components. Instead, the `ResizingComp` component is set to a nominal size (480 by 640 pixels) and then the `MainContentComponent` object applies an affine transform to scale this up or down to match the screen size. This is done while keeping the same aspect ratio (leaving whitespace to the side or above and below the sliders and buttons). The code for the `ResizingComp` class is the same as for the [simple resize](#tutorial_android_screen_sizes_resizing_child_components) example. But in the `MainContentComponent::resized()` function we set the size of the `ResizingComp` component then calculate the required transform:

```cpp
void resized() override
{
    auto contentWidth = 480;
    auto contentHeight = 640;

    auto scaleX = (float) getWidth() / static_cast<float> (contentWidth);
    auto scaleY = (float) getHeight() / static_cast<float> (contentHeight);
    auto scale = juce::jmin (scaleX, scaleY);

    resizingComp->setTransform (juce::AffineTransform::scale (scale, scale));
    resizingComp->centreWithSize (contentWidth, contentHeight);
}
```

This code calculates the ratio between our nominal size and the actual size of the screen in software pixels. It then chooses the smallest of these ratios in order to maintain the aspect ratio, while keeping all of the content onscreen. We then create an instance of the [AffineTransform](https://docs.juce.com/master/classAffineTransform.html "Represents a 2D affine-transformation matrix.") class using the [AffineTransform::scale()](https://docs.juce.com/master/classAffineTransform.html#a43665170b0ec3ef0d1afef4053e77a24 "Returns a new transform which is a re-scale about the origin.") function and centre the scale transform around the centre of the screen. The transform is applied to the component using the Component::setTransform() function. The result is quite different from the [simple resize](#tutorial_android_screen_sizes_resizing_child_components) method.

![](/_images/tutorial_android_screen_sizes_screenshot4.png "Scaling the whole UI using a transform showing both portrait and landscape orientations")

> [!TIP]
>Applying a transform to a component not only transforms the drawing of the user interface, it also transforms the position of touch (and mouse) activity.

# Designing different layouts for different orientations

Download the demo project for this section here: [PIP](https://docs.juce.com/tutorials/PIPs/AndroidScreenSizesTutorialLayoutsOrientation.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/AndroidScreenSizesTutorialLayoutsOrientation.zip) . Unzip the project and open the first header file in the Projucer.

This example looks at one method of displaying a different layout depending on the orientation of the screen (or device). The [Desktop::getCurrentOrientation()](https://docs.juce.com/master/classDesktop.html#a02ae21e312faf1715e8311f1f53e5e00 "In a tablet device which can be turned around, this returns the current orientation.") function provides means of accessing the device orientation. In fact, there are four possible orientations:

- Upright (portrait).
- Upside down (portrait rotated 180 degrees).
- Device rotated clockwise by 90 degrees (one version of landscape orientation).
- Device rotated anti-clockwise by 90 degrees (another version of landscape orientation).

For simplicity, in this tutorial, we are going to treat portrait orientation as a screen that is taller than it is wide (and landscape orientation as a screen that is wider than it is tall).

This example uses the same technique for scaling the user interface [using a transform](#tutorial_android_screen_sizes_resizing_main_component_transform) that we saw earlier. The differences are that this `ResizingComp` class uses a different layout depending on the orientation, and the `MainContentComponent` class has two nominal sizes (one for landscape orientation and one for portrait orientation). The orientation is determined in the `MainContentComponent::resized()` function:

```cpp
void resized() override
{
    auto isLandscape = getWidth() > getHeight();
    auto contentWidth = isLandscape ? 640 : 480;
    auto contentHeight = isLandscape ? 480 : 640;
```

Then, in the `ResizingComp::resized()` function we select from two resizing functions depending on the orientation:

```cpp
void resized() override
{
    if (getHeight() > getWidth())
        resizedPortrait();
    else
        resizedLandscape();
}
```

The `resizedPortrait()` and `resizedLandscape()` functions then use different arithmetic to layout the buttons and sliders.

When the landscape orientation is used, the buttons and sliders are shown in two columns, rather than one column. This is shown in the following screenshot:

![](/_images/tutorial_android_screen_sizes_screenshot5.png "Using a different layout in landscape orientation")

> [!NOTE]
> Exercise: Change the code to use the [Desktop::getCurrentOrientation()](https://docs.juce.com/master/classDesktop.html#a02ae21e312faf1715e8311f1f53e5e00 "In a tablet device which can be turned around, this returns the current orientation.") function to determine the screen orientation, rather than comparing the width and height of the screen.

# Designing different layouts for different screen sizes

Download the demo project for this section here: [PIP](https://docs.juce.com/tutorials/PIPs/AndroidScreenSizesTutorialLayoutsSize.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/AndroidScreenSizesTutorialLayoutsSize.zip) . Unzip the project and open the first header file in the Projucer.

This final example uses different layouts for different screen orientations and screen sizes. This kind of technique might be especially useful if you want to use a totally different layout for screen orientations or even make a universal application for Android phones and tablets. The method employed here is to use the [TabbedComponent](https://docs.juce.com/master/classTabbedComponent.html "A component with a TabbedButtonBar along one of its sides.") class to arrange pages of components if they won\'t fit onto a single page.

The responsibility of the `ResizingComp` class changes a little in this project compared to the three earlier projects. In particular, we don\'t add the buttons and sliders as direct child components. Notice the lack of calls to the Component::addAndMakeVisible() function in the following code for the constructor:

```cpp
ResizingComp()
{
    juce::Array<juce::Colour> colours { juce::Colour (0xffb3c3Da), juce::Colour (0xff5973b8), juce::Colour (0xffd65667), juce::Colour (0xffd99154), juce::Colour (0xffe5ad6c), juce::Colour (0xffecc664), juce::Colour (0xffefe369), juce::Colour (0xffdddB74) };

    for (auto i = 0; i < 6; ++i)
    {
        auto* button = buttons.add (new juce::TextButton (juce::String ("Button ") + juce::String (i + 1)));

        button->setColour (juce::TextButton::buttonColourId,
            colours.getUnchecked (i % colours.size()));
    }

    for (auto i = 0; i < 6; ++i)
    {
        auto* slider = sliders.add (new juce::Slider());

        slider->setColour (juce::Slider::thumbColourId,
            colours.getUnchecked ((buttons.size() + i) % colours.size()));
        slider->setColour (juce::Slider::backgroundColourId,
            colours.getUnchecked ((buttons.size() + i + 2) % colours.size()).withAlpha (0.4f));
        slider->setColour (juce::Slider::trackColourId,
            colours.getUnchecked ((buttons.size() + i + 2) % colours.size()));
        slider->setColour (juce::Slider::textBoxTextColourId, juce::Colours::black);
    }

    setLookAndFeel (&lf);
}
```

The `ResizingComp` class manages the lifetime of the buttons and sliders but in terms of component hierarchy they are added to one or more instances of another component class called `ComponentHolder` .

## Laying out the components

The `ComponentHolder` class holds an array of [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") pointers and lays the components out in one or two columns depending on the orientation. (The technique for achieving this layout was covered earlier when looking at different [screen orientations](#tutorial_android_screen_sizes_different_layouts_orientation).) There is a single function---(`ComponentHolder::addComp()` ---for adding a component that adds it to the internal array and calls Component::addAndMakeVisible():

```cpp
void addComp (Component* comp)
{
    comps.add (comp);
    addAndMakeVisible (comp);
}
```

The layout functions---`ResizingComp::resizedPortrait()` and `ResizingComp::resizedLandscape()` ---should look familiar. Although these need to be slightly different since we no longer have separate arrays of sliders and buttons:

```cpp
void resizedPortrait()
{
    auto space = 8;
    auto widgetHeight = (getHeight() - space) / comps.size() - space;

    for (auto* comp : comps)
        comp->setBounds (space, space + (widgetHeight + space) * comps.indexOf (comp), getWidth() - space - space, widgetHeight);
}

void resizedLandscape()
{
    auto space = 8;
    auto halfComps = comps.size() / 2;
    auto widgetHeight = (getHeight() - space) / halfComps - space;

    for (auto* comp : comps)
    {
        auto index = comps.indexOf (comp);

        if (index < halfComps)
        {
            comp->setBounds (space, space + (widgetHeight + space) * index, getWidth() / 2 - space - space, widgetHeight);
        }
        else
        {
            comp->setBounds (getWidth() / 2 + space, space + (widgetHeight + space) * (index - halfComps), getWidth() / 2 - space - space, widgetHeight);
        }
    }
}
```

## Choosing single or multiple pages

If the screen size is large, then only one of these `ComponentHolder` components is created and all of the buttons and sliders are added to it. If the screen size is small then the `ResizingComp` class uses a [TabbedComponent](https://docs.juce.com/master/classTabbedComponent.html "A component with a TabbedButtonBar along one of its sides.") object and adds two instances of the `ComponentHolder` class to form the tabs of the [TabbedComponent](https://docs.juce.com/master/classTabbedComponent.html "A component with a TabbedButtonBar along one of its sides."). This decision to use a single page or multiple pages is managed by the `ResizingComp` class in the `ResizingComp::resized()` function.

```cpp
void resized() override
{
    if (holder.get() != nullptr)
    {
        removeChildComponent (holder.get());
        holder.reset();
    }

    auto minimumDimension = juce::jmin (getWidth(), getHeight());

    if (minimumDimension >= 480)
        layoutSinglePage();
    else
        layoutTabs();
}
```

Here you can see that we\'re saying a \"large\" screen is one that has one of its dimensions greater than or equal to 480 software pixels. Of course, you can choose a different value for your applications. The `ResizingComp::layoutSinglePage()` function is straightforward:

```cpp
void layoutSinglePage()
{
    holder.reset (new ComponentHolder());

    for (auto* button : buttons)
        dynamic_cast<ComponentHolder*> (holder.get())->addComp (button);

    for (auto* slider : sliders)
        dynamic_cast<ComponentHolder*> (holder.get())->addComp (slider);

    addAndMakeVisible (holder.get());
    holder->setBounds (getLocalBounds());
}
```

Here you can see that we add all of the buttons and sliders to the `ComponentHolder` instance and add it as a child component of our `ResizingComp` object. The `ResizingComp::layoutTabs()` function is only a little more involved:

```cpp
void layoutTabs()
{
    auto orientation = getWidth() < getHeight() ? juce::TabbedButtonBar::TabsAtBottom
                                                : juce::TabbedButtonBar::TabsAtLeft;

    holder.reset (new juce::TabbedComponent (orientation)); // [1]
    addAndMakeVisible (holder.get()); // [2]

    auto* buttonTab = new ComponentHolder(); // [3]
    auto* sliderTab = new ComponentHolder();

    dynamic_cast<juce::TabbedComponent*> (holder.get())->addTab ("Buttons", juce::Colours::white, buttonTab, true); // [4]
    dynamic_cast<juce::TabbedComponent*> (holder.get())->addTab ("Sliders", juce::Colours::white, sliderTab, true);

    for (auto* button : buttons) // [5]
        buttonTab->addComp (button);

    for (auto* slider : sliders) // [6]
        sliderTab->addComp (slider);

    holder->setBounds (getLocalBounds()); // [7]
}
```

- [1]: We create the [TabbedComponent](https://docs.juce.com/master/classTabbedComponent.html "A component with a TabbedButtonBar along one of its sides.") object and store it in our `holder` member using the orientation of the screen to position the tab buttons.
- [2]: We add the [TabbedComponent](https://docs.juce.com/master/classTabbedComponent.html "A component with a TabbedButtonBar along one of its sides.") object as a child component.
- [3]: We create the `ComponentHolder` objects.
- [4]: We add these as tabs to the [TabbedComponent](https://docs.juce.com/master/classTabbedComponent.html "A component with a TabbedButtonBar along one of its sides.") object (the final `true` arguments tell the [TabbedComponent](https://docs.juce.com/master/classTabbedComponent.html "A component with a TabbedButtonBar along one of its sides.") objects that they can delete the `ComponentHolder` objects when they are no longer needed).
- [5]: We add the buttons to the \"Buttons\" tab.
- [6]: And the sliders to the \"Sliders\" tab.
- [7]: We set the size of the [TabbedComponent](https://docs.juce.com/master/classTabbedComponent.html "A component with a TabbedButtonBar along one of its sides.") object to fill the bounds of the `ResizingComp` object.

When the size of the screen is determined as \"small\" then the [TabbedComponent](https://docs.juce.com/master/classTabbedComponent.html "A component with a TabbedButtonBar along one of its sides.") is used as shown in the following screenshot.

![](/_images/tutorial_android_screen_sizes_screenshot6.png "The tabbed interface")

> [!NOTE]
> Exercise: Increase the number of sliders and buttons in the interface and then devise a means of distributing these controls over more than two tabs if necessary.

# Summary

In this tutorial we have examined various issues to do with Android device screen sizes and orientations. In particular we have:

- Shown the basic methods of scaling components based on the parent component's dimensions.
- Shown how to use the [AffineTransform](https://docs.juce.com/master/classAffineTransform.html "Represents a 2D affine-transformation matrix.") class to scale components.
- Shown how to display different content depending on device orientation.
- Shown how to use the [TabbedComponent](https://docs.juce.com/master/classTabbedComponent.html "A component with a TabbedButtonBar along one of its sides.") class to distribute a user interface across several pages.

# See also

- [Tutorial: Getting started with Android](/tutorials/tutorial_android_studio/)
- [Tutorial: Parent and child components](/tutorials/tutorial_component_parents_children/)
- [Tutorial: Responsive GUI layouts using FlexBox and Grid](/tutorials/tutorial_flex_box_grid/)
