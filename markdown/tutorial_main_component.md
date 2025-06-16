# Tutorial: The main component

This tutorial shows how to add graphical content to the application window by creating a main content component. This is important for displaying content to the user within a window.

**LEVEL:** Beginner<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**CLASSES:** [DocumentWindow](https://docs.juce.com/master/classDocumentWindow.html "A resizable window with a title bar and maximise, minimise and close buttons."), [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects."), [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.")

# Getting started

Launch the Projucer and create a new GUI application project with the name **MainComponentTutorial** . In the **Files to Auto-Generate:** field make sure you select **Create a Main.cpp file only** .

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

Modify the `MainComponentTutorialApplication` class to include the following `MainWindow` class as follows:

```cpp
//==============================================================================
class MainComponentTutorialApplication : public juce::JUCEApplication
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
            setUsingNativeTitleBar (true);
            centreWithSize (300, 200);
            setVisible (true);
        }

        void closeButtonPressed() override
        {
            JUCEApplication::getInstance()->systemRequestedQuit();
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

# Introduction

In the last tutorial ([Tutorial: The application window](/tutorials/tutorial_main_window/)), we covered the main window, which serves as the frame in which the application's graphical interface lives. In this tutorial, we will create a _main content component_, which is the object that shows the _content_ of the app's interface. The main content component is an essential object for every JUCE app.

If you create a new GUI application with [The Projucer](https://juce.com/projucer) , it will automatically generate a main content component for you. However, a good way to familiarise yourself with the concept and to understand how JUCE apps are structured is to create such a main content component yourself. This is what we will do in this tutorial.

Open the tutorial project in your IDE. We take off at the same point we arrived in the last tutorial: with an empty application window. In the `Main.cpp` file, we have a `MainWindow` class. We already learned how to use it in the last tutorial ([Tutorial: The application window](/tutorials/tutorial_main_window/)). Now, we will fill this window with content!

However, before we can do that, let's first explore the concept of a _component_ a little bit further.

# The Component class

The most important base class for all JUCE graphical interfaces is the [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") class. In JUCE, practically all visible elements of the GUI, be it buttons, sliders, or text fields, are _components_, deriving from this class. The way to write such an app in JUCE is to create a _main component_, which is owned by the main application window and is the window's _content_. All other components will then be _children_ of this main component (see [Tutorial: Parent and child components](/tutorials/tutorial_component_parents_children/)). The [DocumentWindow](https://docs.juce.com/master/classDocumentWindow.html "A resizable window with a title bar and maximise, minimise and close buttons.") class, from which our `MainWindow` derives, contains the necessary functionality to make sure the main window shows its content correctly (including the main component and its children).

> [!TIP]
>Remember: all graphical elements in JUCE derive from the [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") class. To build a GUI, different components are arranged in a nested hierarchy with *parent* and *child* components. The top-most component is called the *main content component*. See [Tutorial: Parent and child components](/tutorials/tutorial_component_parents_children/) for more detail.

# Adding a main component class

## Creating new source files

Now, let's create our main component class. For this, we need to create new files where the source code for this class will go. Go back to [The Projucer](https://juce.com/projucer) and open the tutorial project there. At the left, make sure that the **Files** browser is open. Then, right-click on the **Source** group (that's the file group where new C++ source files should always go) and select **Add new Component class (split between CPP & header)..** . [The Projucer](https://juce.com/projucer) will ask you how to name the new [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") subclass. In the dialog, enter **MainComponent** and click on **Create Files** . You will see that [The Projucer](https://juce.com/projucer) has created two new files: `MainComponent.cpp` and `MainComponent.h` . Now, save the project and open it again in your [IDE](https://en.wikipedia.org/wiki/Integrated_development_environment) . You should now see the new files there as well. [The Projucer](https://juce.com/projucer) has automatically created some code for the new component class, which we will examine in the next section.

![](/_images/tutorial_main_component_screenshot1.png "Adding a new component class in The Projucer.")

> [!TIP]
>Remember: If you create new classes, they should go into their own files, with the file names matching the class names. Always use [The Projucer](https://juce.com/projucer) to create new files; never do so from your [IDE](https://en.wikipedia.org/wiki/Integrated_development_environment) ([The Projucer](https://juce.com/projucer) would overwrite such changes the next time you save your project).

## A new component class

As you can see, [The Projucer](https://juce.com/projucer) automatically derived the new class from the [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") class and added the following class declaration:

```cpp
class MainComponent : public juce::Component
{
public:
    MainComponent();
    ~MainComponent();

    void paint (juce::Graphics&) override;
    void resized() override;

private:
    JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR (MainComponent)
};
```

The [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") base class has two important virtual member functions that should be overridden in any class deriving from it. [The Projucer](https://juce.com/projucer) already created these two overrides for you:

- Component::paint(): this member function determines how your component should be drawn on the screen, and should be implemented for every component class.
- Component::resized(): this member function determines what should happen to your component when it is resized, and should be implemented for every component class (except if you are sure that this component will never be resizable in you app).

> [!TIP]
>Always add the keyword `override` to all functions in a class that should override a function from a base class. This prevents unexpected errors in your app, and is part of the [JUCE coding standards](http://www.juce.com/learn/coding-standards) .

## Implementing the paint function

The `paint()` function determines how the [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") object is rendered on the screen. This is where we add a custom appearance to our `MainComponent` class.

[The Projucer](https://juce.com/projucer) has automatically added some demo code. Let's put some code of our own into the `paint()` function.

```cpp
void MainComponent::paint (juce::Graphics& g)
{
    g.fillAll (juce::Colours::lightblue);

    g.setColour (juce::Colours::darkblue);
    g.setFont (20.0f);
    g.drawText ("Hello, World!", getLocalBounds(), juce::Justification::centred, true);
}
```

We don\'t have to go into the details of this code too much. We will learn more about the functions used here (and more) in the next tutorial: [Tutorial: The Graphics class](/tutorials/tutorial_graphics_class/). For now, you can probably guess that this demo code fills the component with a light blue background, and then renders the text **Hello, World!** in a blue font in the centre of the component. The point here is that all the code that determines how the `MainComponent` object should look goes right here inside the `paint()` function.

## Make the main component visible

Now, compile and run the code. You will find that instead of the blue background and text, you still only see an empty application window. Why is that?

Well, we did not tell the `MainWindow` object that it should now show some content. First of all we need to include the header so that the `MainWindow` class knows about the `MainComponent` class. Add the folowing include at the top of the `Main.cpp` file, underneath the already existing include:

```cpp
#include "MainComponent.h"
```

The next step is to create a `MainComponent` object and to add it as the _content_ of the main window. We can do that by calling the [DocumentWindow::setContentOwned()](https://docs.juce.com/master/classResizableWindow.html#a97940f07b6014bac018cbe9330abc769 "Changes the current content component.") function.

> [!TIP]
>\"Owned\" means that the `MainWindow` object is now responsible for the lifetime of `MainComponent` object and will destroy it automatically when its own destructor is called.

Add the following line to the constructor of the `MainWindow` class:

```cpp
setContentOwned (new MainComponent(), true);
```

such that the `MainWindow` constructor looks like this:

```cpp
MainWindow (juce::String name) : DocumentWindow (name,
                                     juce::Colours::lightgrey,
                                     DocumentWindow::allButtons)
{
    setUsingNativeTitleBar (true);
    setContentOwned (new MainComponent(), true);
    centreWithSize (getWidth(), getHeight());
    setVisible (true);
}
```

Note that we changed one other detail: the arguments to the Component::centreWithSize() function have changed as well. We now don\'t explicitly set the size of the `MainWindow` object anymore, but tell it to figure out its size based on its content:

```cpp
centreWithSize (getWidth(), getHeight());
```

However, for this to work, the `MainComponent` object needs to have its size set before the `centreWithSize()` function call happens. If this isn\'t done then the main window won\'t know what window size is appropriate (this will trigger an assertion failure if you run it). The next section explains how to accomplish this.

## Setting the component's size

In principle, there are two ways to set the size of a [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") object. Either you set the size in the constructor of the component itself, or you set the size in the constructor of its parent component. For the main component, we usually set the size in the component itself. Add the following line to the constructor of our `MainComponent` class:

```cpp
setSize (400, 300);
```

(Of course, you can choose another size if you want.)

> [!WARNING]
> Remember: always set the size of your components. Omitting this step is a very common source of bugs in JUCE.

This is the reason why the calls to functions Component::getWidth() and Component::getHeight() in the `MainWindow` class can figure out the size of the window so that the main component is displayed with the correct size. The `MainComponent` object's size gets set in its own constructor, before the `MainWindow` object is positioned and sized.

Now all necessary pieces are in place. If you compile and run the app now, you should see the main component drawn correctly into the application window:

![](/_images/tutorial_main_component_screenshot2.png "The new MainComponent object is rendered on the screen.")

> [!NOTE]
> Exercise: Find out what the second argument of the `setContentOwned()` function means (which we set to `true` here) and how it behaves if you change it. Hint: check out the documentation for the [ResizableWindow::setContentOwned()](https://docs.juce.com/master/classResizableWindow.html#a97940f07b6014bac018cbe9330abc769 "Changes the current content component.") function.

# Implement a resize function

Now that we have covered the `paint()` function, let's move on and see how we can make the `MainComponent` class react to being resized.

First of all, we need to tell the main window that it should be resizable. Please refer to [Tutorial: The application window](/tutorials/tutorial_main_window/) if you don\'t remember how to do that.

Now, compile and run the app, and resize the window using the mouse. You will see that the `MainComponent` object resizes itself to fit the size of the main window --- all the necessary code to do that is already implemented for you in the [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") base class.

But what if you want some custom work to be done every time the component gets resized? Maybe it has child components which need to be laid out differently depending on the main component's size. In our simple demo app, let's change the text inside the main component such that it shows the current size of the component.

Anything that needs to be done or updated when we need to resize our component goes into our component's `resized()` function. Currently, the function is empty. Let's add our functionality here.

> [!WARNING]
> The Component::resized() function is called automatically whenever anything happens that leads to a size change of the component. Never call this function yourself!

The text that is displayed inside the component is currently given as a literal string --- \"Hello, World!\" --- inside the `paint()` function. Let's change that by introducing a new member variable to the `MainComponent` class. It is good practice to always give your variables descriptive, intention-revealing names. This makes the code easier to read and understand, and reduces the amount of additional code comments. We want our new variable to represent the current size of the main component as a string, so let's call it `currentSizeAsString` .

Member variables are always declared in the private section of a class:

```cpp
class MainComponent : public juce::Component
{
    // ...

private:
    juce::String currentSizeAsString;
    // ...
};
```

Now let's implement the desired behaviour for the `currentSizeAsString` object. There are two things to do:

- The contents of the `currentSizeAsString` object should be rendered on screen.
- The `currentSizeAsString` object should update itself whenever the main component's size changes.

How we achieve the first part should be fairly simple: inside the `paint()` function, when the `g.drawText()` function is called, simply replace the literal string there with the `currentSizeAsString` object:

```cpp
void MainComponent::paint (juce::Graphics& g)
{
    //...
    g.drawText (currentSizeAsString, getLocalBounds(), juce::Justification::centred, true);
}
```

The second part is more interesting. We already know that every time we resize that the `resized()` function is called. So let's update the value of the `currentSizeAsString` object there:

```cpp
void MainComponent::resized()
{
    currentSizeAsString = juce::String (getWidth()) + " x " + juce::String (getHeight());
}
```

Component::getWidth() and Component::getHeight() are convenient functions that let you query the current size of the component. We also need to convert these integers to [String](https://docs.juce.com/master/classString.html "The JUCE String class!") objects. (You can find out more about how to work with the [String](https://docs.juce.com/master/classString.html "The JUCE String class!") class in a future tutorial.)

If you compile and run the app now, you will see that it always displays its current size:

![](/_images/tutorial_main_component_screenshot3.png "The finished demo app.")

We can make two interesting observations here. First, the display is automatically updated --- the `paint()` function is called automatically after the `resized()` function is called. Second, the size is already shown correctly when the app starts up, even before you first resize the window yourself. Remember that the `resized()` function is always called whenever _anything_ changes the component's size. This includes the first time the component's size is set and the component is painted after the app launched.

> [!NOTE]
> Exercise: Modify the `MainComponent::resize()` function in such a way that on every resize, the `MainComponent` object also changes its background colour.

You can download a finished version of the tutorial project here: [PIP](https://docs.juce.com/tutorials/PIPs/MainComponentTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/MainComponentTutorial.zip) and compare it to your own.

# Summary

This tutorial explained the concept of a main component, how to add one to your app, and how to implement the `paint()` and `resized()` functions. After reading this tutorial, you should be familiar with the following important things:

- Every JUCE application window has a main component. It is the parent of all other components that make up your app's GUI.
- Every component, including the main component, has two important functions that you need to override: `paint()` and `resized()` .
- In the `paint()` function, you should add the code that will render the component on the screen.
- You should implement a `resized()` function if you need special behaviour for your component, in order for it to react to size changes.
- The `paint()` and `resized()` functions are _callback_ functions that are called automatically when needed.
- You must not forget to set the size of the main component and to add it to the main window to make it visible.

# See also

- [Tutorial: The application window](/tutorials/tutorial_main_window/)
- [Tutorial: The Graphics class](/tutorials/tutorial_graphics_class/)
- [Tutorial: Parent and child components](/tutorials/tutorial_component_parents_children/)
