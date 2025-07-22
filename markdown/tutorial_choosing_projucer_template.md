---
title: Projucer Part 3: Choosing the right Projucer template for your application
---
# Tutorial: Projucer Part 3: Choosing the right Projucer template for your application

Choosing the right template for your application can be a tricky task at first and inheriting from the right class from the start can give a tremendous boost in productivity. In this tutorial we explore the different template projects that the Projucer offers and the main classes to watch for when conceiving your applications.

**LEVEL:** Beginner<br/>
**PLATFORMS:** Windows, macOS, Linux, iOS, Android<br/>
**CLASSES:** [JUCEApplication](https://docs.juce.com/master/classJUCEApplication.html "An instance of this class is used to specify initialisation and shutdown code for the application."), [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects."), [AudioAppComponent](https://docs.juce.com/master/classAudioAppComponent.html "A base class for writing audio apps that stream from the audio i/o devices."), [AnimatedAppComponent](https://docs.juce.com/master/classAnimatedAppComponent.html "A base class for writing simple one-page graphical apps."), [OpenGLAppComponent](https://docs.juce.com/master/classOpenGLAppComponent.html "A base class for writing simple one-page graphical apps."), [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins."), [AudioProcessorEditor](https://docs.juce.com/master/classAudioProcessorEditor.html "Base class for the component that acts as the GUI for an AudioProcessor.")

# Getting started

Please make sure you are familiar with the Projucer and have a basic understanding of the structure of a Projucer project.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The right template for the right project

Depending on the project you are developing, you may want to choose a different template project from the Projucer wizard. The following graph provides a visual questionnaire to help you choose the best template for your project:

![](/_images/dot_inline_dotgraph_1.svg "")

You can navigate to the corresponding project type right away or read along for an explanation of how a JUCE project is structured.

# The GUI projects

All GUI project templates will provide you with source files to get you started in your development. Since the application will display a graphical user interface, the template provides you with a Main.cpp file in which a class derived from [JUCEApplication](https://docs.juce.com/master/classJUCEApplication.html "An instance of this class is used to specify initialisation and shutdown code for the application.") will have been created.

The [JUCEApplication](https://docs.juce.com/master/classJUCEApplication.html "An instance of this class is used to specify initialisation and shutdown code for the application.") class is an abstract base class that provides startup and shutdown functionalities for your applications. The Main.cpp file also creates the application window in which your GUI will reside.

## The GUI Application

The GUI Application project is the most generic of all GUI projects and in addition to the Main.cpp file, it also creates a MainComponent class derived from the [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") class.

The [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") class is a base class for all user-interface objects in JUCE. All your GUI elements should be defined and placed in the MainComponent class.

![](/_images/dot_inline_dotgraph_2.svg "")

When a GUI Application project is created you will find the following files in the source folder:

- Main.cpp: Derived from the [JUCEApplication](https://docs.juce.com/master/classJUCEApplication.html "An instance of this class is used to specify initialisation and shutdown code for the application.") class, it provides initialisation code for your application and window.
- MainComponent.h: Header file for the MainComponent derived from the [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") class.
- MainComponent.cpp: Implementation file for the MainComponent derived from the [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") class.

Use this project type when you know you will need a GUI in your application but are unsure or need flexibility as to the features of your application.

> [!TIP]
>For more details on the GUI Application project, please refer to the tutorial [Tutorial: The application window](/tutorials/tutorial_main_window/) for more information.

## The Audio Application

The Audio Application is similar to the GUI Application but instead of the MainComponent deriving from the [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") class, it derives from the [AudioAppComponent](https://docs.juce.com/master/classAudioAppComponent.html "A base class for writing audio apps that stream from the audio i/o devices.") class.

The [AudioAppComponent](https://docs.juce.com/master/classAudioAppComponent.html "A base class for writing audio apps that stream from the audio i/o devices.") class is an abstract base class that combines the functionalities of a [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") with an [AudioSource](https://docs.juce.com/master/classAudioSource.html "Base class for objects that can produce a continuous stream of audio.") to provide a convenient starting point to handle audio input/output.

![](/_images/dot_inline_dotgraph_3.svg "")

When an Audio Application project is created you will find the following files in the source folder:

- Main.cpp: Derived from the [JUCEApplication](https://docs.juce.com/master/classJUCEApplication.html "An instance of this class is used to specify initialisation and shutdown code for the application.") class, it provides initialisation code for your application and window.
- MainComponent.cpp: Header and implementation file for the MainComponent derived from the [AudioAppComponent](https://docs.juce.com/master/classAudioAppComponent.html "A base class for writing audio apps that stream from the audio i/o devices.") class.

Use this project type when you want audio input/output functionalities with a GUI in your application.

> [!TIP]
>For more details on the Audio Application project, please refer to the tutorial [Tutorial: Build a white noise generator](/tutorials/tutorial_simple_synth_noise/) for more information.

## The Animated Application

The Animated Application is similar to the GUI Application but instead of the MainComponent deriving from the [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") class, it derives from the [AnimatedAppComponent](https://docs.juce.com/master/classAnimatedAppComponent.html "A base class for writing simple one-page graphical apps.") class.

The [AnimatedAppComponent](https://docs.juce.com/master/classAnimatedAppComponent.html "A base class for writing simple one-page graphical apps.") class is an abstract base class that combines the functionalities of a [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") with a [Timer](https://docs.juce.com/master/classTimer.html "Makes repeated callbacks to a virtual method at a specified time interval.") to provide a convenient starting point to display animations.

![](/_images/dot_inline_dotgraph_4.svg "")

When an Animated Application project is created you will find the following files in the source folder:

- Main.cpp: Derived from the [JUCEApplication](https://docs.juce.com/master/classJUCEApplication.html "An instance of this class is used to specify initialisation and shutdown code for the application.") class, it provides initialisation code for your application and window.
- MainComponent.cpp: Header and implementation file for the MainComponent derived from the [AnimatedAppComponent](https://docs.juce.com/master/classAnimatedAppComponent.html "A base class for writing simple one-page graphical apps.") class.

Use this project type when you want to create simple animations in the GUI of your application.

> [!TIP]
>For more details on the Animated Application project, please refer to the tutorial [Tutorial: Animating geometry](/tutorials/tutorial_animation/) for more information.

## The OpenGL Application

The OpenGL Application is similar to the GUI Application but instead of the MainComponent deriving from the [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") class, it derives from the [OpenGLAppComponent](https://docs.juce.com/master/classOpenGLAppComponent.html "A base class for writing simple one-page graphical apps.") class.

The [OpenGLAppComponent](https://docs.juce.com/master/classOpenGLAppComponent.html "A base class for writing simple one-page graphical apps.") class is an abstract base class that combines the functionalities of a [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") with an [OpenGLRenderer](https://docs.juce.com/master/classOpenGLRenderer.html "A base class that should be implemented by classes which want to render openGL on a background thread...") to provide a convenient starting point to render complex graphics.

![](/_images/dot_inline_dotgraph_5.svg "")

When an OpenGL Application project is created you will find the following files in the source folder:

- Main.cpp: Derived from the [JUCEApplication](https://docs.juce.com/master/classJUCEApplication.html "An instance of this class is used to specify initialisation and shutdown code for the application.") class, it provides initialisation code for your application and window.
- MainComponent.cpp: Header and implementation file for the MainComponent derived from the [OpenGLAppComponent](https://docs.juce.com/master/classOpenGLAppComponent.html "A base class for writing simple one-page graphical apps.") class.

Use this project type when you want to render complex graphical elements using OpenGL in the GUI of your application.

> [!TIP]
>For more details on the OpenGL Application project, please refer to the tutorial [Tutorial: Build an OpenGL application](/tutorials/tutorial_open_gl_application/) for more information.

# The Command Line project

## The Console Application

When a Console Application project is created you will find the following file in the source folder:

- Main.cpp: Provides the C-style main() function for your application to run with corresponding command line arguments.

When the binary is executed in the command line, the following function will be called:

```cpp
int main (int argc, char* argv[])
{
    // ..your code goes here!

    return 0;
}
```

This project type will not create any classes in the Main.cpp file nor inherit any base classes but will create a .jucer file and a JUCE library code folder along with any exporter you may have chosen when creating the project.

Use this project type when you do not need a GUI in your application and want to run the application in a command line console.

# The Plugin project

## The Audio Plugin

The Audio Plugin project has an all around different project structure compared to other template projects offered by the Projucer and creates an [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.") and an [AudioProcessorEditor](https://docs.juce.com/master/classAudioProcessorEditor.html "Base class for the component that acts as the GUI for an AudioProcessor.").

The [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.") class is an abstract base class that handles the audio processing of your audio plugin. It represents an instance of a loaded plugin and is wrapped by the different plugin formats such as VST, AU, RTAS and AAX.

The [AudioProcessorEditor](https://docs.juce.com/master/classAudioProcessorEditor.html "Base class for the component that acts as the GUI for an AudioProcessor.") class is a base class derived from the [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") class and holds the GUI functionalities of your audio plugin.

::: dotgraph
**This browser is not able to show SVG: try Firefox, Chrome, Safari, or Opera instead.**
:::

When an Audio Plugin project is created you will find the following files in the source folder:

- PluginProcessor.h: Header file for the PluginProcessor derived from the [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.") class.
- PluginProcessor.cpp: Implementation file for the PluginProcessor derived from the [AudioProcessor](https://docs.juce.com/master/classAudioProcessor.html "Base class for audio processing classes or plugins.") class.
- PluginEditor.h: Header file for the PluginEditor derived from the [AudioProcessorEditor](https://docs.juce.com/master/classAudioProcessorEditor.html "Base class for the component that acts as the GUI for an AudioProcessor.") class.
- PluginEditor.cpp: Implementation file for the PluginEditor derived from the [AudioProcessorEditor](https://docs.juce.com/master/classAudioProcessorEditor.html "Base class for the component that acts as the GUI for an AudioProcessor.") class.

Use this project type when you want to create a plugin that can be hosted in a DAW or as a standalone application with a GUI.

> [!TIP]
>For more details on the Audio Plugin project, please refer to the tutorial [Tutorial: Create a basic Audio/MIDI plugin, Part 1: Setting up](/tutorials/tutorial_create_projucer_basic_plugin/) for more information.

# The Library projects

## The Static/Dynamic Library

In case your interest lies in creating a library to use in other projects, the Projucer offers an option to create static and dynamic libraries.

This project type will not create any source files but will create a .jucer file and a JUCE library code folder along with any exporter you may have chosen when creating the project.

You are then free to add files as you wish with the convenience of having access to the JUCE modules at any time.

# Summary

In this tutorial, we have learnt the different types of Projucer template projects. In particular, we have:

- Learnt how each template calls for a specific type of project.
- Explored the different JUCE classes that facilitate those different projects.
- Understood the basic mechanics of the JUCE framework.

# See also

- [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/)
- [Tutorial: Projucer Part 2: Manage your Projucer projects](/tutorials/tutorial_manage_projucer_project/)
- [Tutorial: Create a basic Audio/MIDI plugin, Part 1: Setting up](/tutorials/tutorial_create_projucer_basic_plugin/)
- [Tutorial: Create a basic Audio/MIDI plugin, Part 2: Coding your plug-in](/tutorials/tutorial_code_basic_plugin/)
- [Tutorial: The application window](/tutorials/tutorial_main_window/)
- [Tutorial: Build a white noise generator](/tutorials/tutorial_simple_synth_noise/)
- [Tutorial: Animating geometry](/tutorials/tutorial_animation/)
- [Tutorial: Build an OpenGL application](/tutorials/tutorial_open_gl_application/)
