---
title: Projucer Part 1: Getting started with the Projucer
---
# Tutorial: Projucer Part 1: Getting started with the Projucer

This tutorial shows you how to install JUCE and how to create a new cross-platform JUCE project using the Projucer. You also learn how to export the project to an IDE such as Xcode or Visual Studio to develop, run and debug your JUCE application.

**LEVEL:** Beginner<br/>
**PLATFORMS:** Windows, macOS, Linux, iOS, Android

# Getting started

[Download JUCE](https://www.juce.com/get-juce). Unpack the JUCE folder and place it to some location on your computer. Your user home folder is a convenient place.

Go into the JUCE folder you just installed. Launch the Projucer, which is located there.

# The new project window

The first time you launch Projucer, you are presented with the new project window. (You can also launch this later by selecting **New Project...** from the Projucer's main menu.):

![](/_images/tutorial_new_projucer_project_screenshot1.png "Projucer - new project window.")

## Select your project type

On the left-hand side of the window, select which type of project you want to create.

For each project type, the Projucer will generate all project files and add the appropriate minimal code that sets everything up. In this way, after you have created your project, you can immediately get started with developing your actual functionality.

If you are exploring JUCE for the first time and don\'t know where to start, go for a GUI application --- this will set up all the basics for a working application for you, for both desktop and mobile platforms.

Below is an overview over all currently supported project types.

| Project type                        | Description                                                                                                                                                                                                                                                                                                                                                                           |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Application/Blank**               | This creates a blank JUCE application.                                                                                                                                                                                                                                                                                                                                                |
| **Application/GUI**                 | This creates a minimal JUCE application with an empty application window. You can start here and add more functionality, such as more GUI components, using the various classes that JUCE offers.                                                                                                                                                                                     |
| **Application/Audio**               | This creates a minimal JUCE application, like **Application/GUI**, but automatically adds all the setup code that you need to easily get audio input and output. You can use this for games, multimedia applications, and much more.                                                                                                                                                  |
| **Application/Console**             | JUCE is also very useful for developing command-line applications that do not have any GUI at all. Use this project type to create such an application.                                                                                                                                                                                                                               |
| **Application/Animated**            | This creates an application which draws an animated graphical display. You can start here to create an animated mobile app, for example.                                                                                                                                                                                                                                              |
| **Application/OpenGL**              | This creates a blank JUCE application, like **Application/GUI**, but adds support for OpenGL to draw features including 3D model import and GLSL shaders.                                                                                                                                                                                                                             |
| **Plug-In/Basic**                   | This creates a basic audio plug-in. All the code to support VST, AudioUnit and AAX plug-in formats, is added automatically. Depending on your setup, this project type may require some additional preparation steps to work correctly. See [Tutorial: Create a basic Audio/MIDI plugin, Part 1: Setting up](/tutorials/tutorial_create_projucer_basic_plugin/) for more information. |
| **Library/Static, Library/Dynamic** | This project type is useful to create re-usable software libraries that build on top of JUCE. The Projucer supports creating libraries for both static and dynamic linking.                                                                                                                                                                                                           |

There are also many example projects which can serve as an alternative starting point for your project, They are located in the subfolder `JUCE/examples` and can be browsed by clicking the **Open Example** tab.

> [!TIP]
>You are not limited by the project type you choose; for example, if you selected **Application/GUI** , you can always add animations and OpenGL later. However, you cannot easily convert an Application to a Library or an Audio Plug-In later.

## Create your new project

After you have selected the appropriate project type, you can fill out some additional project settings on the right-hand side of the window:

- **Project Name** - Here, you choose a name for your app.
- **Modules** - The JUCE framework code is organised into _modules_. Here, you can select which modules are included in your project and in the section below you can specify the location of the `modules` subfolder located inside the JUCE folder you installed earlier.
- **Exporters** - Here, you select which native IDEs you want to use to build and debug your app. This also defines the desktop and mobile platforms that your app will support. Don\'t worry, this is not a final choice --- with the Projucer, you can add additional platforms and IDEs later.

The Projucer currently has exporters for the following IDEs, build systems, and platforms:

| Target OS | Supported build systems |
|--------|--------|
| macOS | Xcode |
| Windows | Visual Studio, Code::Blocks |
| Linux | Makefile,  Code::Blocks |
| iOS| Xcode |
| Android | Android Studio | 

- **[File](https://docs.juce.com/master/classFile.html "Represents a local file or directory.") Creation Options** - Depending on the project type, this field may offer some options on what code to auto-generate. If you are new to JUCE, the default selection is usually the best way to go to auto-generate everything you need and get you going quickly.
- When you have reviewed all your settings, click the **Create Project...** button to generate the project at a specified location.

## Export the project and open in your native IDE

After you have created your project, you can launch the native IDE with your project directly from the Projucer. Use the button near the top:

![](/_images/tutorial_new_projucer_project_screenshot2.png "Projucer - save and open project in IDE")

Now that you have opened your IDE (Xcode, Visual Studio, and so on), you can compile and run your JUCE app, and get started with coding!

> [!TIP]
>You need to install the appropriate native IDE/build system on your computer before you can use it. For some platforms, you may need to install additional dependencies as well, for example you need to install the Android SDK to develop for Android.

All export targets (which are the native IDE projects) are generated when you create your Projucer project. They are also updated every time you save it in the Projucer. At any time, you can create new export targets to add support for more IDEs and platforms to your project. You can read about this and more features for managing your Projucer project in [Tutorial: Projucer Part 2: Manage your Projucer projects](/tutorials/tutorial_manage_projucer_project/).

# Open an existing project

To open up an existing Projucer project, you can either double-click on the `.jucer` file contained in the project folder or click on **Open Existing Project** from the wizard. (You can also navigate to **Open...** from the Projucer's main menu.)

## Opening PIP files

You may come across _Projucer Instant Project_ (PIP) files when following other JUCE tutorials. These are essentially header files with the usual `.h` extension that provide metadata to the Projucer in order to automatically create a project with the correct modules and exporters from a single file.

PIP files can be opened similarly by either selecting the file from the **Open...** dialog of the Projucer's main menu or a simple drag-and-drop onto the Projucer interface window.

# Summary

After reading this tutorial, you should be able to:

- Install JUCE.
- Create a new project using the Projucer.
- Export this project to your native IDE such as Xcode on OSX, or Visual Studio on Windows.
- Compile, run, and debug your app in the native IDE for the platform(s) you want to target.
- Open existing projects and PIP files in the Projucer.

# See also

- [Tutorial: Projucer Part 2: Manage your Projucer projects](/tutorials/tutorial_manage_projucer_project/)
- [Tutorial: Create a basic Audio/MIDI plugin, Part 1: Setting up](/tutorials/tutorial_create_projucer_basic_plugin/)
- [Tutorial: Projucer Part 3: Choosing the right Projucer template for your application](/tutorials/tutorial_choosing_projucer_template/)
