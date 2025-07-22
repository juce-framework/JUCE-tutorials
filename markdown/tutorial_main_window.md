---
title: The application window
---
# Tutorial: The application window

This tutorial demonstrates how to create a minimal app with an application window, and how to customise the size and appearance of that window. This is critical for any JUCE GUI application.

**LEVEL:** Beginner<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**CLASSES:** [JUCEApplication](https://docs.juce.com/master/classJUCEApplication.html "An instance of this class is used to specify initialisation and shutdown code for the application."), [DocumentWindow](https://docs.juce.com/master/classDocumentWindow.html "A resizable window with a title bar and maximise, minimise and close buttons."), [ResizableWindow](https://docs.juce.com/master/classResizableWindow.html "A base class for top-level windows that can be dragged around and resized.")

# Getting started

Launch the Projucer and create a new GUI application project with the name **MainWindowTutorial** . In the **Files to Auto-Generate:** field make sure you select **Create a Main.cpp file only** .

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

Modify the `MainWindowTutorialApplication` class to include the following `MainWindow` class as follows:

```cpp
//==============================================================================
class MainWindowTutorialApplication : public juce::JUCEApplication
{
public:
    //...

    //==============================================================================
    class MainWindow : public juce::DocumentWindow
    {
    public:
        MainWindow (juce::String name) : DocumentWindow (name,
                                             juce::Colours::lightgrey,
                                             DocumentWindow::allButtons)
        {
            centreWithSize (300, 200);
            setVisible (true);
        }

        void closeButtonPressed() override
        {
            juce::JUCEApplication::getInstance()->systemRequestedQuit();
        }

    private:
        JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR (MainWindow)
    };

private:
    std::unique_ptr<MainWindow> mainWindow;
};
```

Add the following line to the `initialise()` function:

```cpp
void initialise (const juce::String& commandLine) override
{
    mainWindow.reset (new MainWindow (getApplicationName()));
}
```

Finally, add the following line to the `shutdown()` function:

```cpp
void shutdown() override
{
    mainWindow = nullptr;
}
```

# The demo project

The demo project is a very minimal JUCE app that has only one source file: `Main.cpp` . The only thing it does is to create and display an empty application window. This window should open when you run the app:

![](/_images/tutorial_main_window_screenshot1.png "The main app window.")

You can drag this window around, maximise and minimise it, and close it to quit the JUCE app.

The C++ code in this app is just about the minimum amount of code required for a working app. The application class itself (derived from the [JUCEApplication](https://docs.juce.com/master/classJUCEApplication.html "An instance of this class is used to specify initialisation and shutdown code for the application.") class) will be covered in a later tutorial. For now, all the app is really doing is the single line of code in the app's `MainWindowTutorialApplication::initialise()` function:

```cpp
void initialise (const juce::String& commandLine) override
{
    mainWindow.reset (new MainWindow (getApplicationName()));
}
```

This creates a new instance of the `MainWindow` class, which leads to the application window being shown.

> [!TIP]
>Any code that you put into the application's `initialise()` function will be executed as soon as the app starts.

# The main window implementation

## The MainWindow class

Now let's take a look at the implementation of this `MainWindow` class. It derives from the JUCE [DocumentWindow](https://docs.juce.com/master/classDocumentWindow.html "A resizable window with a title bar and maximise, minimise and close buttons.") class, a resizable window with a title bar, and maximise, minimise, and close buttons. These are all of the elements you would expect from an application window.

> [!TIP]
>Generally, this is how you implement the different components of your application: you create a new class that inherits from an appropriate JUCE base class offering the functionality you require. In this new class, you can then add your own custom functionality on top of that.

The `MainWindow` constructor is defined as follows:

```cpp
MainWindow (juce::String name) : DocumentWindow (name,
                                     juce::Colours::lightgrey,
                                     DocumentWindow::allButtons)
{
    centreWithSize (300, 200);
    setVisible (true);
}
```

First, the base class ([DocumentWindow](https://docs.juce.com/master/classDocumentWindow.html "A resizable window with a title bar and maximise, minimise and close buttons.")) is initialised, which requires three parameters:

- The name of the window that will be displayed in the title bar.
- The background colour of the window.
- Which buttons should be shown in the corner of the title bar.

Looking at the code in `Main.cpp` , you may notice that the first parameter is actually passed to the `MainWindow` constructor by the app's `initialise()` function. This is simply the name of the app (and part of the information auto-generated by [The Projucer](https://juce.com/projucer) when a project is created, along with the app's version number and and a few other things).

> [!NOTE]
> Exercise: Try to change the string that is displayed in the app's title bar.

## Setting the background colour

The next two arguments, the background colour and the buttons, are set directly in the `MainWindow` constructor. Most common colours have predefined constants in JUCE and can be directly used like in this example. So you could replace the [Colours::lightgrey](namespaceColours.html#a47f250277502112af4e9ac55a26d51fc) constant with, for example, constants [Colours::red](namespaceColours.html#adeee57dc4a6f202204ad3a3d7e0ec243) or [Colours::blue](namespaceColours.html#a712056f9defb3e4e19f670d2175290c8) to quickly change the background colour of the whole app window. Of course, it is also possible to define your own custom colours. See [Tutorial: Colours in JUCE](/tutorials/tutorial_colours/) for a more detailed introduction to colours in JUCE.

## Using bitmask arguments for the buttons

The third argument passed to the [DocumentWindow](https://docs.juce.com/master/classDocumentWindow.html "A resizable window with a title bar and maximise, minimise and close buttons.") base class constructor is the [DocumentWindow::allButtons](https://docs.juce.com/master/classDocumentWindow.html#a104d68bc145157c6ce79169a36fc3c31af4a9b49869cebf7e8cecaf7f6dd93420 "A combination of all the buttons above.") constant. It indicates that all three default buttons (maximise, minimise, and close) should be shown in the title bar. You can change this property by using the following other constants, defined in the `enum` [DocumentWindow::TitleBarButtons](https://docs.juce.com/master/classDocumentWindow.html#a104d68bc145157c6ce79169a36fc3c31 "The set of available button-types that can be put on the title bar."):

- [DocumentWindow::minimiseButton](https://docs.juce.com/master/classDocumentWindow.html#a104d68bc145157c6ce79169a36fc3c31a4200920dc08e7ecfe1c8b90444d99098)
- [DocumentWindow::maximiseButton](https://docs.juce.com/master/classDocumentWindow.html#a104d68bc145157c6ce79169a36fc3c31a84fe4bbe25739fe241f2e6128e7a51c9)
- [DocumentWindow::closeButton](https://docs.juce.com/master/classDocumentWindow.html#a104d68bc145157c6ce79169a36fc3c31a80e87a70beeecc78adc89517fa1df978)

Using the C++ operator `|` (the bitwise OR operator), you can define any other combination of the three possible buttons. For example, to show only the close and minimise buttons, you could type:

```cpp
MainWindow (juce::String name) : DocumentWindow (name,
                                     juce::Colours::lightgrey,
                                     DocumentWindow::closeButton | DocumentWindow::minimiseButton)
{
    centreWithSize (300, 200);
    setVisible (true);
}
```

This approach to pass a combination of flags to a function is called a _bitmask_ and is found in many other places in JUCE.

## Setting the window size

When the `MainWindow` object is created, the constructor's body is executed. The first statement calls the Component::centreWithSize() function to set the window's position and size. The second statement, a call to the Component::setVisible() function, is always required for the window to be shown after it has been created.

Using the Component::centreWithSize() function, you can specify the initial width and height of the window in pixels. The function will set the size to the one specified, and then centre the component with respect to its parent component. The [DocumentWindow](https://docs.juce.com/master/classDocumentWindow.html "A resizable window with a title bar and maximise, minimise and close buttons.") class derives from the [TopLevelWindow](https://docs.juce.com/master/classTopLevelWindow.html "A base class for top-level windows.") class, which does not have a parent component, so it will be centered with respect to the whole screen.

You may wish to specify another position for the window. The function Component::setBounds() offers more options in this case. To try this out, change the line containing the call to the `centreWithSize()` function to this:

```cpp
setBounds (50, 50, 800, 600);
```

This will set the size to 800 x 600 pixels and position the window closer to the top left corner of the screen (50 pixels away from the screen edge).

> [!NOTE]
> Exercise: Use the Component::setBoundsRelative() function to set a position and size relative to the screen size, rather than in absolute pixels.

<!-- -->

> [!TIP]
>You can learn more about sizes, bounds and positions in [Tutorial: Parent and child components](/tutorials/tutorial_component_parents_children/).

# Other customisation options

## Making the window resizable

The JUCE [DocumentWindow](https://docs.juce.com/master/classDocumentWindow.html "A resizable window with a title bar and maximise, minimise and close buttons.") class itself inherits from the [ResizableWindow](https://docs.juce.com/master/classResizableWindow.html "A base class for top-level windows that can be dragged around and resized.") class. This base class adds the functionality to make the window resizable by the user. This can be enabled with the function [ResizableWindow::setResizable()](https://docs.juce.com/master/classResizableWindow.html#a538709f0111ad32b05d858a511bcd5be "Make the window resizable or fixed.").

Add the following line to the `MainWindow` constructor:

```cpp
setResizable (true, true);
```

Take a look at the documentation for the [ResizableWindow::setResizable()](https://docs.juce.com/master/classResizableWindow.html#a538709f0111ad32b05d858a511bcd5be "Make the window resizable or fixed.") function to look up what its two boolean arguments mean. The first one determines whether the window should be resizable at all. The second one lets you add a handle to the bottom right corner to resize the window. If this argument is `false` , the window will instead be resizable by dragging with the mouse at any of the window's edges.

![](/_images/tutorial_main_window_screenshot2.png "A resizable window with a draggable corner resizer.")

## Using a native title bar

So far, we have been using the JUCE look-and-feel for the app's main window. Its main advantage is that will look and behave identically on every desktop operating system (Windows, Mac OS X, and Linux).

However, the JUCE [DocumentWindow](https://docs.juce.com/master/classDocumentWindow.html "A resizable window with a title bar and maximise, minimise and close buttons.") class allows you to let the window use the typical native window apperance for the operating system you run it on. This is achieved using the function [DocumentWindow::setUsingNativeTitleBar()](https://docs.juce.com/master/classTopLevelWindow.html#a749fbd5e688ed8c9af3d0d99b21e18c8 "Sets whether an OS-native title bar will be used, or a JUCE one."). Add the following line to the `MainWindow` constructor:

```cpp
setUsingNativeTitleBar (true);
```

Now the application's window will look like this:

![](/_images/tutorial_main_window_screenshot3.png "Using a native title bar on Ubuntu Linux, Windows, and Mac OS X, respectively.")

Please note that this will adjust not only the appearance, but also the behaviour of the window to match that of a native operating system window. For example, now the `useBottomRightCornerResizer` argument to the [ResizableWindow::setResizable()](https://docs.juce.com/master/classResizableWindow.html#a538709f0111ad32b05d858a511bcd5be "Make the window resizable or fixed.") function will not change the resizing behaviour of the window anymore. Instead, for resizing with the mouse, the window will adopt whatever behaviour is the native look-and-feel of the operating system you are running.

For most apps, the native look-and-feel is a better choice for the main app window, because this is the more familiar look-and-feel that a user will typically expect from a native app.

## More customisation options

> [!NOTE]
> Exercise: Have a look at the documentation for the [DocumentWindow](https://docs.juce.com/master/classDocumentWindow.html "A resizable window with a title bar and maximise, minimise and close buttons.") and [ResizableWindow](https://docs.juce.com/master/classResizableWindow.html "A base class for top-level windows that can be dragged around and resized.") classes. Experiment with other functions not covered here that let you customise other aspects of the app's window.

# Summary

In this tutorial we have covered how to set the behaviour of a basic JUCE app's main window, such as: adjust its size, make it resizable, set its title and appearance, and define what buttons should appear in its title bar. We have also described how to switch to a native title bar to make the window feel more like a native window on the operating system it is running on.

# Notes

- Later you may want to build more complicated apps, consisting of several different windows. You can use the same techniques to control the behaviour of each one.

# See also

- [Tutorial: The main component](/tutorials/tutorial_main_component/)
- [Tutorial: Colours in JUCE](/tutorials/tutorial_colours/)
- [Tutorial: Parent and child components](/tutorials/tutorial_component_parents_children/)
