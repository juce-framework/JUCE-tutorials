---
title: Animating geometry
---
# Tutorial: Animating geometry

Create simple animations in your JUCE applications. Bring static geometry shapes to life using the [AnimatedAppComponent](https://docs.juce.com/master/classAnimatedAppComponent.html "A base class for writing simple one-page graphical apps.") class.

**LEVEL:** Beginner<br/>
**PLATFORMS:** Windows, macOS, Linux, iOS, Android<br/>
**CLASSES:** [AnimatedAppComponent](https://docs.juce.com/master/classAnimatedAppComponent.html "A base class for writing simple one-page graphical apps."), [Path](https://docs.juce.com/master/classPath.html "A path is a sequence of lines and curves that may either form a closed shape or be open-ended."), [Point](https://docs.juce.com/master/classPoint.html "A pair of (x, y) coordinates.")

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/AnimationTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/AnimationTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

When completed, the demo project will display a continuous and smooth animation of a fish on the screen made of multiple [Path](https://docs.juce.com/master/classPath.html "A path is a sequence of lines and curves that may either form a closed shape or be open-ended.") and [Point](https://docs.juce.com/master/classPoint.html "A pair of (x, y) coordinates.") objects.

![](/_images/tutorial_animation_screenshot1.png "The demo project window")

> [!TIP]
>The code presented here is broadly similar to the **AnimationAppExample** from the JUCE Examples.

# The AnimatedAppComponent class

When creating a simple animated JUCE application, a useful class to inherit from is the [AnimatedAppComponent](https://docs.juce.com/master/classAnimatedAppComponent.html "A base class for writing simple one-page graphical apps.") class. Just as the [AudioAppComponent](https://docs.juce.com/master/classAudioAppComponent.html "A base class for writing audio apps that stream from the audio i/o devices.") or [OpenGLAppComponent](https://docs.juce.com/master/classOpenGLAppComponent.html "A base class for writing simple one-page graphical apps.") classes are useful for audio applications and OpenGL applications respectively, the [AnimatedAppComponent](https://docs.juce.com/master/classAnimatedAppComponent.html "A base class for writing simple one-page graphical apps.") offers functions that are beneficial to animation making namely:

- setFramesPerSecond(): This function allows us to set the FPS at the very start of our application for our animation to run as smoothly as possible. It also starts the repaint process at the given frequency when called.
- update(): This function gets called at the rate set by the setFramesPerSecond() function and this is where step-by-step advancements of animation parameters are performed.
- getFrameCounter(): Returns the total number of calls to the update() function since the start of the animation at the FPS rate defined. This is useful in periodic mathematical functions to compute animations.
- getMillisecondsSinceLastUpdate(): Another useful function that returns the time taken since the last update() function call in milliseconds in order to create animations that are accurately timed regardless of the frame rate.

By using these functions along with the paint() function of the parent [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") class, we can start creating simple animations.

# Animating a Circle

As we can see in the `MainContentComponent` class, the MainContentComponent inherits from the [AnimatedAppComponent](https://docs.juce.com/master/classAnimatedAppComponent.html "A base class for writing simple one-page graphical apps.").

The first step to creating an animation is to set the frame rate of our animation. We do this in the MainContentComponent constructor like so [1] by calling the setFramesPerSecond() function:

```cpp
MainContentComponent()
{
    setSize (800, 600);
    setFramesPerSecond (60); // [1]
}
```

Here we set the FPS to 60 and this will internally call a timer at a frequency of 60Hz for our animation to be updated 60 times per second. This is roughly equivalent to most screen's refresh rate and will result in a smooth animation.

Let's start by animating a simple circle in a circular motion. In the paint() function, first set the colour in which we want to draw the circle by calling the setColour() function of the [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") class [2]. Next define the radius of the circular path that the shape will follow [3] as shown here:

```cpp
void paint (juce::Graphics& g) override
{
    //...

    g.setColour (getLookAndFeel().findColour (Slider::thumbColourId)); // [2]

    int radius = 150; // [3]

    juce::Point<float> p (getWidth() / 2.0f + 1.0f * radius,
        getHeight() / 2.0f + 1.0f * radius); // [4]

    g.fillEllipse (p.x, p.y, 30.0f, 30.0f); // [5]
}
```

Then create a [Point](https://docs.juce.com/master/classPoint.html "A pair of (x, y) coordinates.") that represents the position of the center of our shape at the given time frame [4]. Here, in order to create a circular motion first find the center of the screen by dividing the width and height of the screen by two. Then offset the x and y coordinates of the shape by adding the radius value to both of them.

Lastly, paint the actual circle by using the fillEllipse() function by providing the previously defined [Point](https://docs.juce.com/master/classPoint.html "A pair of (x, y) coordinates.") coordinates and a diameter of 30 as arguments.

Can you guess what happens to the circle if we run our application now? That's right, the circle gets painted in a static manner in the bottom right corner of the screen because as the coordinate system starts in the top left corner, the circle is only pushed by the radius value to the opposite direction as shown here:

![](/_images/tutorial_animation_screenshot2.png "The static circle")

Let's modify our declaration of our [Point](https://docs.juce.com/master/classPoint.html "A pair of (x, y) coordinates.") to create the actual motion.

```cpp
void paint (juce::Graphics& g) override
{
    // (Our component is opaque, so we must completely fill the background with a solid colour)
    g.fillAll (getLookAndFeel().findColour (juce::ResizableWindow::backgroundColourId));

    g.setColour (getLookAndFeel().findColour (juce::Slider::thumbColourId));

    int radius = 150;

    juce::Point<float> p ((float) getWidth() / 2.0f + 1.0f * (float) radius * std::sin ((float) getFrameCounter() * 0.04f),
        (float) getHeight() / 2.0f + 1.0f * (float) radius * std::cos ((float) getFrameCounter() * 0.04f));

    g.fillEllipse (p.x, p.y, 30.0f, 30.0f);
}
```

Here we use the getFrameCounter() function to retrieve the counter on the number of frames since the start of our animation and use its value to compute a value between `-1 .. 1` using the sine and cosine functions for the width and height respectively. The scalar multiplication of 0.04 on the frame counter controls the speed at which the periodic functions will alternate to create circular motion.

If we run the application now, we should see the circular motion appearing.

> [!TIP]
>The source code for this modified version of the code can be found in the `AnimationTutorial_02.h` file of the demo project.

# Animating a Path

Instead of a circle, let's animate a line along a circular path next.

Using the same code base as the previous section we are going to create multiple [Point](https://docs.juce.com/master/classPoint.html "A pair of (x, y) coordinates.") objects along which a [Path](https://docs.juce.com/master/classPath.html "A path is a sequence of lines and curves that may either form a closed shape or be open-ended.") will be created instead of just a single animated [Point](https://docs.juce.com/master/classPoint.html "A pair of (x, y) coordinates."). Modify the paint() function as follows:

```cpp
void paint (juce::Graphics& g) override
{
    // (Our component is opaque, so we must completely fill the background with a solid colour)
    g.fillAll (getLookAndFeel().findColour (juce::ResizableWindow::backgroundColourId));

    g.setColour (getLookAndFeel().findColour (juce::Slider::thumbColourId));

    auto numberOfDots = 15; // [1]

    juce::Path spinePath; // [2]

    for (auto i = 0; i < numberOfDots; ++i) // [3]
    {
        int radius = 150;

        juce::Point<float> p ((float) getWidth() / 2.0f + 1.0f * (float) radius * std::sin ((float) getFrameCounter() * 0.04f + (float) i * 0.12f),
            (float) getHeight() / 2.0f + 1.0f * (float) radius * std::cos ((float) getFrameCounter() * 0.04f + (float) i * 0.12f));

        if (i == 0)
            spinePath.startNewSubPath (p); // if this is the first point, start a new path..
        else
            spinePath.lineTo (p); // ...otherwise add the next point
    }

    // draw an outline around the path that we have created
    g.strokePath (spinePath, juce::PathStrokeType (4.0f)); // [4]
}
```

- [1]: First define the number of dots to create along the path.
- [2]: Next create a [Path](https://docs.juce.com/master/classPath.html "A path is a sequence of lines and curves that may either form a closed shape or be open-ended.") object to draw the line that will connect the dots.
- [3]: Now for every dot, create the same [Point](https://docs.juce.com/master/classPoint.html "A pair of (x, y) coordinates.") object as before but this time we are going to offset the animation for every subsequent iteration in the loop. Notice the addition of an offset of 0.12 between every dot in the circular motion. If the iteration is the first one, we create a new sub path by calling the startNewSubPath() function on the [Path](https://docs.juce.com/master/classPath.html "A path is a sequence of lines and curves that may either form a closed shape or be open-ended.") object, otherwise we connect the current dot to the previously defined dots on the [Path](https://docs.juce.com/master/classPath.html "A path is a sequence of lines and curves that may either form a closed shape or be open-ended.").
- [4]: Finally, draw an outline around the created [Path](https://docs.juce.com/master/classPath.html "A path is a sequence of lines and curves that may either form a closed shape or be open-ended.") along the [Point](https://docs.juce.com/master/classPoint.html "A pair of (x, y) coordinates.") objects.

By running the application, we can see a line drawn in a circular way.

![](/_images/tutorial_animation_screenshot3.png "The circular path")

> [!NOTE]
> Exercise: Try changing the FPS of the application and notice how the smoothness of the animation varies. What happens at the standard 24FPS rate of film animations?

<!-- -->

> [!TIP]
>The source code for this modified version of the code can be found in the `AnimationTutorial_03.h` file of the demo project.

# Animating a Fish

Let's try something a little more interesting by animating a fish from the [Path](https://docs.juce.com/master/classPath.html "A path is a sequence of lines and curves that may either form a closed shape or be open-ended.") and [Point](https://docs.juce.com/master/classPoint.html "A pair of (x, y) coordinates.") objects we have created so far.

In order to show the actual points drawn along the circular path and create the body of the fish, let's add a line in our for loop that draws the circles by using the fillEllipse() function and by specifying an increasing width and height for each dot along the line [1] like so:

```cpp
void paint (juce::Graphics& g) override
{
    //...
    g.setColour (getLookAndFeel().findColour (juce::Slider::thumbColourId));

    auto fishLength = 15;

    juce::Path spinePath;

    for (auto i = 0; i < fishLength; ++i)
    {
        int radius = 150;

        juce::Point<float> p (getWidth() / 2.0f + 1.0f * radius * std::sin (getFrameCounter() * 0.04f + i * 0.12f),
            getHeight() / 2.0f + 1.0f * radius * std::cos (getFrameCounter() * 0.04f + i * 0.12f));

        // draw the circles along the fish
        g.fillEllipse (p.x - i, p.y - i, 2.0f + 2.0f * i, 2.0f + 2.0f * i); // [1]

        if (i == 0)
            spinePath.startNewSubPath (p); // if this is the first point, start a new path..
        else
            spinePath.lineTo (p); // ...otherwise add the next point
    }

    // draw an outline around the path that we have created
    g.strokePath (spinePath, juce::PathStrokeType (4.0f));
}
```

Now by running the application we should see something that starts to look like a fish but does not behave like one yet.

![](/_images/tutorial_animation_screenshot4.png "The body of the fish")

So let's change its animation a bit to mimic the motion of a fish.

```cpp
void paint (juce::Graphics& g) override
{
    //...

    for (auto i = 0; i < fishLength; ++i)
    {
        auto radius = 100 + 10 * std::sin (getFrameCounter() * 0.1f + i * 0.5f); // [2]

        juce::Point<float> p (getWidth() / 2.0f + 1.0f * radius * std::sin (getFrameCounter() * 0.04f + i * 0.12f),
            getHeight() / 2.0f + 1.0f * radius * std::cos (getFrameCounter() * 0.04f + i * 0.12f));

        //...
    }
    //...
}
```

Here we apply modulation to the radius of the circle using a sine function and the frame counter by using the same getFrameCounter() function along with a scalar and a slightly different offset for every dot along the line [2]. This should provide us with a snake like motion if we run the application.

The motion looks convincing but seems a little monotonous as it repeats itself fairly quickly with no surprise and still on its initial circular trajectory.

```cpp
void paint (juce::Graphics& g) override
{
    //...

    for (auto i = 0; i < fishLength; ++i)
    {
        auto radius = 100 + 10 * std::sin (getFrameCounter() * 0.1f + i * 0.5f);

        juce::Point<float> p (getWidth() / 2.0f + 1.5f * radius * std::sin (getFrameCounter() * 0.02f + i * 0.12f),
            getHeight() / 2.0f + 1.0f * radius * std::cos (getFrameCounter() * 0.04f + i * 0.12f)); // [3]

        //...
    }
    //...
}
```

If we offset the rate of our sine and cosine functions in the [Point](https://docs.juce.com/master/classPoint.html "A pair of (x, y) coordinates.") creation and provide a different ratio for the width and height of the radius [3], we can get much more convincing results.

Run the application one last time to notice the improvement in the randomness of the motion.

> [!NOTE]
> Exercise: Modify the fish length or create your own custom animated shape using similar methods.

<!-- -->

> [!TIP]
>The source code for this modified version of the code can be found in the `AnimationTutorial_04.h` file of the demo project.

# Summary

In this tutorial, we have learnt how to animate geometry shapes in a JUCE application. In particular, we have:

- Explored the mechanics of the [AnimatedAppComponent](https://docs.juce.com/master/classAnimatedAppComponent.html "A base class for writing simple one-page graphical apps.") class.
- Painted shapes using the [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") class.
- Animated the shapes on a frame-by-frame basis.

# See also

- [Tutorial: The Graphics class](/tutorials/tutorial_graphics_class/)
- [Tutorial: The Point, Line, and Rectangle classes](/tutorials/tutorial_point_line_rectangle/)
- [Tutorial: Advanced GUI layout techniques](/tutorials/tutorial_rectangle_advanced/)
- [Tutorial: Build an OpenGL application](/tutorials/tutorial_open_gl_application/)
