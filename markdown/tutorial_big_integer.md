# Tutorial: The BigInteger class

This tutorial introduces the [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") class, which is for handling arbitrarily large integers. [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") objects are often used in cryptography applications, when large bit masks are needed, and anywhere else where really large integers are needed.

**LEVEL:** Beginner<br/>
**PLATFORMS:** Windows, macOS, Linux, iOS, Android<br/>
**CLASSES:** [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class."), [TextEditor](https://docs.juce.com/master/classTextEditor.html "An editable text box."), [MemoryBlock](https://docs.juce.com/master/classMemoryBlock.html "A class to hold a resizable block of raw data.")

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/BigIntegerTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/BigIntegerTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project sets up a simple text console where we can display the results of various calculations. This allows us to demonstrate some of the operations that can be performed by the [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") class in this tutorial.

In its default configuration, the demo project displays a series of 100 integers starting with the value 11. Each new value is the old value multiplied by 11.

![](/_images/tutorial_big_integer_screenshot1.png "The test console displaying a series of big integers")

As you can see, the [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") class can indeed represent very large integers! A signed 32-bit integer (for example, the `int` type on most compilers) can represent numbers up to 2,147,483,647 and a 64-bit integer (the `int64` type in JUCE) can represent up to \"just\" 9,223,372,036,854,775,807!

# Test bed code

Throughout this tutorial we will modify parts of the demo project to demonstrate different features of the [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") class. All of the code under discussion is in the `MainComponent` class. In fact, all of the changes will be to the `runExample()` function.

```cpp
void runExample()
{
    logMessage ("------------------------- START --------------------------");

    int base = 10;
    juce::BigInteger bigInt = 11;

    for (auto iteration = 0; iteration < 100; ++iteration)
    {
        logMessage (bigInt.toString (base));

        bigInt *= 11;
    }

    logMessage ("----------------------- FINISHED -------------------------");
}
```

Here we can see the [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") object initialised to 11 then it is multiplied by 11 for each iteration of the `for()` loop. The `logMessage()` function simply posts the string that is passed to our [TextEditor](https://docs.juce.com/master/classTextEditor.html "An editable text box.") object.

Now that we have introduced our test bed code we can start trying out some more operations on [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") objects.

> [!NOTE]
> Exercise: Try out different start values and multipliers to generate different geometric sequences.

# Binary patterns

The [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") class supports binary operations, too. In fact, the [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") class is often used as a bit mask (for example, the [AudioIODevice](https://docs.juce.com/master/classAudioIODevice.html "Base class for an audio device with synchronised input and output channels.") class uses [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") objects to represent enabled input and output channels --- see [Tutorial: The AudioDeviceManager class](/tutorials/tutorial_audio_device_manager/)). Here is a simple example where we display the [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") values in binary (using base-2) starting with a value of 3 (`11` in binary) and bit-shift to the left by one place for each iteration:

```cpp
void runExample()
{
    logMessage ("------------------------- START --------------------------");

    int base = 2;
    juce::BigInteger bigInt = 3;

    for (auto iteration = 0; iteration < 100; ++iteration)
    {
        logMessage (bigInt.toString (base));

        bigInt = bigInt << 1;
    }

    logMessage ("----------------------- FINISHED -------------------------");
}
```

Here you will see that we end up with a binary value containing two 1s and many trailing zeros:

```
...
11000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
110000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
1100000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
11000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
110000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
1100000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
11000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
----------------------- FINISHED -------------------------
```

> [!NOTE]
> Exercise: Try a different starting bit pattern. Or start with a very large value and gradually bit-shift it to the right instead.

We can also set and test bits individually. For example, this sets the first thirty even-numbered bits and displays the result in binary, decimal, hexadecimal and octal:

```cpp
void runExample()
{
    logMessage ("------------------------- START --------------------------");

    juce::BigInteger bigInt;

    for (auto bit = 0; bit < 60; bit += 2)
        bigInt.setBit (bit);

    logMessage (juce::String ("binary: ") + bigInt.toString (2));
    logMessage (juce::String ("decimal: ") + bigInt.toString (10));
    logMessage (juce::String ("hex: ") + bigInt.toString (16));
    logMessage (juce::String ("octal: ") + bigInt.toString (8));

    logMessage ("----------------------- FINISHED -------------------------");
}
```

The result should be:

```
------------------------- START --------------------------
binary: 10101010101010101010101010101010101010101010101010101010101
decimal: 384307168202282325
hex: 555555555555555
octal: 25252525252525252525
----------------------- FINISHED -------------------------
```

> [!NOTE]
> Exercise: Try setting different patterns of bits using the code above as a starting point.

We can test bits in a [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") object using the subscript operator as if a [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") object is an array of `bool` values. This example uses our original code to generate a geometric sequence and tests bit 3 for each value produced:

```cpp
void runExample()
{
    logMessage ("------------------------- START --------------------------");

    int base = 10;
    juce::BigInteger bigInt = 11;

    for (auto iteration = 0; iteration < 100; ++iteration)
    {
        bool isBit3set = bigInt[3];
        logMessage (bigInt.toString (base) + " : " + (isBit3set ? "Bit 3 is set" : "Bit 3 is NOT set"));

        bigInt *= 11;
    }

    logMessage ("----------------------- FINISHED -------------------------");
}
```

This makes code much more readable than using the standard bit-wise operators in C++. Since it is so simple to set and test bits using the [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") class, it is useful even if the integers are small!

# Arbitrary data

[BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") objects can also be converted to, and from, arbitrary data via [MemoryBlock](https://docs.juce.com/master/classMemoryBlock.html "A class to hold a resizable block of raw data.") objects. The following example converts a string to a [MemoryBlock](https://docs.juce.com/master/classMemoryBlock.html "A class to hold a resizable block of raw data.") object (via a [MemoryOutputStream](https://docs.juce.com/master/classMemoryOutputStream.html "Writes data to an internal memory buffer, which grows as required.") object), then to a [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class."), and finally back to a [String](https://docs.juce.com/master/classString.html "The JUCE String class!") object via a [MemoryBlock](https://docs.juce.com/master/classMemoryBlock.html "A class to hold a resizable block of raw data.") object:

```cpp
void runExample()
{
    logMessage ("------------------------- START --------------------------");

    juce::String originalText ("BigInteger objects are really useful for cryptography");
    logMessage ("Original text: ");
    logMessage (originalText);
    logMessage (newLine);

    juce::MemoryOutputStream originalData;
    originalData << originalText;

    juce::BigInteger originalInteger;
    originalInteger.loadFromMemoryBlock (originalData.getMemoryBlock());

    logMessage ("Original text as a BigInteger: ");
    logMessage (originalInteger.toString (10));
    logMessage (newLine);

    juce::MemoryBlock convertedData (originalInteger.toMemoryBlock());
    juce::String convertedString (convertedData.toString());

    logMessage ("BigInteger converted back to a string: ");
    logMessage (convertedString);

    logMessage ("----------------------- FINISHED -------------------------");
}
```

As the message in the applications says, [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") objects are really useful for cryptography. The [RSAKey](https://docs.juce.com/master/classRSAKey.html "RSA public/private key-pair encryption class.") class applies its cryptographic algorithm to [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") objects in order to encrypt and decrypt messages.

> [!NOTE]
> Exercise: Experiment with different strings in the code above. What happens when you use longer or shorter strings, for example?

# Summary

This tutorial has introduced the [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") class. After following this tutorial you should be able to:

- Use [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") objects to store integers and apply arithmetic operations to them just like regular integers.
- Convert a [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") object to a string for display in binary, decimal, hexadecimal and octal.
- Test and set the individual bits of a [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") object.
- Convert strings and other arbitrary data to and from [BigInteger](https://docs.juce.com/master/classBigInteger.html "An arbitrarily large integer class.") objects.

# See also

- [Tutorial: The Label class](/tutorials/tutorial_label/)
