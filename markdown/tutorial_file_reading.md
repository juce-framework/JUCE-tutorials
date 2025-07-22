---
title: File reading
---
# Tutorial: File reading

Open and read data from text and binary files.

**LEVEL:** Beginner<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**CLASSES:** [File](https://docs.juce.com/master/classFile.html "Represents a local file or directory."), [FileInputStream](https://docs.juce.com/master/classFileInputStream.html "An input stream that reads from a local file."), [FilenameComponent](https://docs.juce.com/master/classFilenameComponent.html "Shows a filename as an editable text box, with a 'browse' button and a drop-down list for recently se..."), [TextEditor](https://docs.juce.com/master/classTextEditor.html "An editable text box."), [String](https://docs.juce.com/master/classString.html "The JUCE String class!"), [MemoryBlock](https://docs.juce.com/master/classMemoryBlock.html "A class to hold a resizable block of raw data.")

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/FileReadingTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/FileReadingTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project presents a simple window which allows you to select a file, using a [FilenameComponent](https://docs.juce.com/master/classFilenameComponent.html "Shows a filename as an editable text box, with a 'browse' button and a drop-down list for recently se...") object. This file is opened, read as a string, and displayed in a [TextEditor](https://docs.juce.com/master/classTextEditor.html "An editable text box.") component.

# Files in JUCE

This tutorial illustrates the basic techniques for reading files using JUCE. Files are one aspect of cross-platform development that need to be handled carefully because file systems on different operating systems sometimes work in quite different ways. As a developer using JUCE you are not immune to these problems, but JUCE makes the experience more consistent across different platforms and will often raise assertions in your debug builds if it can see that your code does something that may lead to problems.

## The File class

The [File](https://docs.juce.com/master/classFile.html "Represents a local file or directory.") class in JUCE represents an _absolute_ path to a file or a directory (whether it actually exists or not). The simplest way to create a [File](https://docs.juce.com/master/classFile.html "Represents a local file or directory.") object is to pass it a [String](https://docs.juce.com/master/classString.html "The JUCE String class!") that contains the absolute path. For example, on macOS, Linux or Android the following would be an absolute path:

```cpp
juce::File path ("/path/to/file.txt");
```

But the [File](https://docs.juce.com/master/classFile.html "Represents a local file or directory.") class enables ways of manipulating paths by requesting child files that are relative to a parent directory, obtaining the parent directory, and so on. For example, the code above could be rewritten using the [File::getChildFile()](https://docs.juce.com/master/classFile.html#a087bb8f0a60069234b13ab965fb5014a "Returns a file that represents a relative (or absolute) sub-path of the current one.") function like so:

```cpp
juce::File path (File ("/path").getChildFile ("to").getChildFile ("file.txt"));
```

In this example the code has become more verbose but in real-world scenarios it is common to need to access multiple files from the same directory. Therefore, it makes sense to store the parent directory in one [File](https://docs.juce.com/master/classFile.html "Represents a local file or directory.") object and request child files as and when they are needed.

On Windows, an equivalent absolute path might be:

```cpp
juce::File path ("C:\\path\\to\\file.txt");
```

Dealing with child and parent directories is the same on each platform. JUCE handles the platform differences (such as path separators).

## The FilenameComponent class

In this tutorial we are going to use a [FilenameComponent](https://docs.juce.com/master/classFilenameComponent.html "Shows a filename as an editable text box, with a 'browse' button and a drop-down list for recently se...") object to allow the user to select a file using a standard operating system window. We can attach a listener to the [FilenameComponent](https://docs.juce.com/master/classFilenameComponent.html "Shows a filename as an editable text box, with a 'browse' button and a drop-down list for recently se...") object (see [Tutorial: Listeners and Broadcasters](/tutorials/tutorial_listeners_and_broadcasters/)) and when the file changes we can obtain the currently chosen file as a [File](https://docs.juce.com/master/classFile.html "Represents a local file or directory.") object.

A [FilenameComponent](https://docs.juce.com/master/classFilenameComponent.html "Shows a filename as an editable text box, with a 'browse' button and a drop-down list for recently se...") object shows a text box that contains an absolute path. It also provides a button for the user to select a file from the operating system. There is also a drop-down menu that contains a list of recently used files. This is populated automatically during use, but these recently-used files can be added manually too (for example, hardcoded into your application or from a preferences file).

As we will see in a moment, the [FilenameComponent](https://docs.juce.com/master/classFilenameComponent.html "Shows a filename as an editable text box, with a 'browse' button and a drop-down list for recently se...") constructor has quite a few arguments---and it doesn\'t have a default constructor. When this is the case, it is often easier to store child components in std::unique*ptr objects. (Since this means that they don\'t have to be initialised in the class initialiser list in the constructor.) We also need a [TextEditor](https://docs.juce.com/master/classTextEditor.html "An editable text box.") component, which we will use to display the file's contents. The [TextEditor](https://docs.juce.com/master/classTextEditor.html "An editable text box.") class \_does* have a default constructor, but for consistency we store both component objects in std::unique_ptr objects:

```cpp
std::unique_ptr<juce::FilenameComponent> fileComp;
std::unique_ptr<juce::TextEditor> textContent;
```

In our `MainContentComponent` constructor we allocate a new [FilenameComponent](https://docs.juce.com/master/classFilenameComponent.html "Shows a filename as an editable text box, with a 'browse' button and a drop-down list for recently se...") object and initialise it with some settings suitable for opening files (since the [FilenameComponent](https://docs.juce.com/master/classFilenameComponent.html "Shows a filename as an editable text box, with a 'browse' button and a drop-down list for recently se...") class can also be used for choosing locations for saving files too). Within this constructor we can provide a list of file suffixes that we wish to be able to select (for example, `"*.txt;*.foo"` ). We can also enforce a suffix (which is more useful for saving files). In both of these arguments we pass an empty string, which means that no filtering will be performed. Other arguments are self-explanatory, defining other ways that we want the [FilenameComponent](https://docs.juce.com/master/classFilenameComponent.html "Shows a filename as an editable text box, with a 'browse' button and a drop-down list for recently se...") object to behave (see the comments in the code):

```cpp
MainContentComponent()
{
    fileComp.reset (new juce::FilenameComponent ("fileComp",
        {}, // current file
        false, // can edit file name,
        false, // is directory,
        false, // is for saving,
        {}, // browser wildcard suffix,
        {}, // enforced suffix,
        "Select file to open")); // text when nothing selected
    addAndMakeVisible (fileComp.get());
    fileComp->addListener (this);
```

In the [FilenameComponentListener](https://docs.juce.com/master/classFilenameComponentListener.html "Listens for events happening to a FilenameComponent.") callback we obtain the currently chosen file and pass it to our `readFile()` function:

```cpp
void filenameComponentChanged (juce::FilenameComponent* fileComponentThatHasChanged) override
{
    if (fileComponentThatHasChanged == fileComp.get())
        readFile (fileComp->getCurrentFile());
}
```

In each of the examples that follow, the `readFile()` function will read the chosen file in different ways. But we need somewhere to display the results, so we set up a [TextEditor](https://docs.juce.com/master/classTextEditor.html "An editable text box.") component in our `MainContentComponent` constructor too:

```cpp
textContent.reset (new juce::TextEditor());
addAndMakeVisible (textContent.get());
textContent->setMultiLine (true);
textContent->setReadOnly (true);
textContent->setCaretVisible (false);

setSize (600, 400);
}
```

## Reading a whole file into a string

While the [File](https://docs.juce.com/master/classFile.html "Represents a local file or directory.") class is designed primarily to store and manipulate paths to files, there are a few convenient functions for reading files in really simple ways. For example, the [File::loadFileAsString()](https://docs.juce.com/master/classFile.html#a07b95f6831ee4ed5b49241ce6af8539d "Reads a file into memory as a string.") function does exactly what it says: it reads a whole file into a [String](https://docs.juce.com/master/classString.html "The JUCE String class!") object. Of course, if the file selected isn\'t a text file then the result may be impossible to make sense of (although JUCE won\'t crash). This function can detect and read both UTF-8 and UTF-16 formats:

```cpp
void readFile (const juce::File& fileToRead)
{
    if (!fileToRead.existsAsFile()) // [1]
        return;

    auto fileText = fileToRead.loadFileAsString();

    textContent->setText (fileText);
}
```

Notice that we check to see if the file chosen actually exists [1]. Since we chose the file from the operating systems then this shouldn\'t fail, but it's good practice to make these kinds of checks when dealing with files. Run the app and load the `juce.txt` text file provided in the `Resources` directory of the demo project. The result will be as shown in the following screenshot:

![](/_images/tutorial_file_reading_screenshot1.png "Reading and displaying a text file")

There is an equivalent function---[File::loadFileAsData()](https://docs.juce.com/master/classFile.html#a889b8b0784b4f97c5bf93ffc24c3650d "Loads a file's contents into memory as a block of binary data.")---to read an entire file into a [MemoryBlock](https://docs.juce.com/master/classMemoryBlock.html "A class to hold a resizable block of raw data.") object.

## Reading a file line by line

For more control over the file reading process, you will need to use a [FileInputStream](https://docs.juce.com/master/classFileInputStream.html "An input stream that reads from a local file.") object. One way to do this is to construct a [FileInputStream](https://docs.juce.com/master/classFileInputStream.html "An input stream that reads from a local file.") object by passing its constructor the [File](https://docs.juce.com/master/classFile.html "Represents a local file or directory.") object that represents the file you want to read [2].

> [!TIP]
>The [FileInputStream](https://docs.juce.com/master/classFileInputStream.html "An input stream that reads from a local file.") class is a subclass of the [InputStream](https://docs.juce.com/master/classInputStream.html "The base class for streams that read data.") class, which is the base class for streams that read data.

Add the following code:

```cpp
void readFile (const juce::File& fileToRead)
{
    if (!fileToRead.existsAsFile())
        return; // file doesn't exist

    juce::FileInputStream inputStream (fileToRead); // [2]

    if (!inputStream.openedOk())
        return; // failed to open
```

We are going to read the file line by line, but we are also going to detect lines that start with the \"\*\" character. Then we\'ll format these lines in a different font, using these lines as headings for the other text. Add the following code:

```cpp
textContent->clear();

auto normalFont = textContent->getFont();
auto titleFont = normalFont.withHeight (normalFont.getHeight() * 1.5f).boldened();
juce::String asterix ("*");
```

The next part is to read data from the `inputStream` object, in a `while()` loop, until the stream is exhausted [3]. Add this:

```cpp
while (!inputStream.isExhausted()) // [3]
{
    auto line = inputStream.readNextLine();

    if (line.startsWith (asterix))
    {
        line = line.removeCharacters (asterix);
        textContent->setFont (titleFont);
    }
    else
    {
        textContent->setFont (normalFont);
    }

    // append the text to the textContent
    textContent->insertTextAtCaret (line + juce::newLine);
}
}
```

You can see that this:

- reads a line, using the [InputStream::readNextLine()](https://docs.juce.com/master/classInputStream.html#af16acc8f2fd769adb559a781ece8e903 "Reads a UTF-8 string from the stream, up to the next linefeed or carriage return.") function,
- checks to see if the line starts with a \"\*\",
- sets the font for the `textContent` object accordingly,
- removes any \"\*\" characters, if necessary, and
- appends line of text to the `textContent` object.

Loading the same `juce.txt` file should result in the something like the following screenshot:

![](/_images/tutorial_file_reading_screenshot2.png "Reading the file line by line")

> [!TIP]
>The code for the example in this subsection can be found in the `FileReadingTutorial_02.h` file of the demo project.

## Reading a file byte by byte

The [InputStream](https://docs.juce.com/master/classInputStream.html "The base class for streams that read data."), and therefore [FileInputStream](https://docs.juce.com/master/classFileInputStream.html "An input stream that reads from a local file."), classes also have functions for reading files in smaller pieces, right down to reading a byte at a time. To illustrate this, let's load a text file and display each word in a different colour. First let's add a function to generate random colours. Add this function which generates a random colour, but clips the brightness to a specified minimum (this is to ensure that the colour will be visible against the black background):

```cpp
static juce::Colour getRandomColour (float minBrightness)
{
    auto& random = juce::Random::getSystemRandom();
    juce::Colour colour ((juce::uint8) random.nextInt (256),
        (juce::uint8) random.nextInt (256),
        (juce::uint8) random.nextInt (256));

    return colour.getBrightness() >= minBrightness ? colour
                                                   : colour.withBrightness (minBrightness);
}
```

Now let's add a function that reads data from a [FileInputStream](https://docs.juce.com/master/classFileInputStream.html "An input stream that reads from a local file.") object until it reaches a space character. This will then return the text up to, and including, the space. It creates a small memory buffer, using the [MemoryBlock](https://docs.juce.com/master/classMemoryBlock.html "A class to hold a resizable block of raw data.") class, then uses the [InputStream::readByte()](https://docs.juce.com/master/classInputStream.html#a2e5944641712d84b6da2eee5d394326a "Reads a byte from the stream.") function to read bytes one at a time from the `inputStream` object:

```cpp
static juce::String readUpToNextSpace (juce::FileInputStream& inputStream)
{
    juce::MemoryBlock buffer (256);
    auto* data = static_cast<char*> (buffer.getData());
    size_t i = 0;

    while ((data[i] = inputStream.readByte()) != 0 && i < buffer.getSize())
        if (data[i++] == ' ')
            break;

    return juce::String::fromUTF8 (data, (int) i); // [4]
}
```

The [String::fromUTF8()](https://docs.juce.com/master/classString.html#aa0116dc51d7bdd363d14c72bba60060a "Creates a String from a UTF-8 encoded buffer.") [4] function attempts to convert the raw binary data to a [String](https://docs.juce.com/master/classString.html "The JUCE String class!") object.

Finally, in our `readFile()` function, we use our `readUpToNextSpace` function to read the words from the text file until the stream is exhausted. Add the following code:

```cpp
void readFile (const juce::File& fileToRead)
{
    if (!fileToRead.existsAsFile())
        return; // file doesn't exist

    juce::FileInputStream inputStream (fileToRead);

    if (!inputStream.openedOk())
        return; // failed to open

    textContent->clear();

    while (!inputStream.isExhausted())
    {
        auto nextWord = readUpToNextSpace (inputStream);
        textContent->setColour (juce::TextEditor::textColourId, getRandomColour (0.75f));
        textContent->insertTextAtCaret (nextWord);
    }
}
```

Running this code will result in something like the following screenshot.

![](/_images/tutorial_file_reading_screenshot3.png "Reading the file in smaller chunks")

> [!TIP]
>The code for the example in this subsection can be found in the `FileReadingTutorial_03.h` file of the demo project.

An alternative way of creating a [FileInputStream](https://docs.juce.com/master/classFileInputStream.html "An input stream that reads from a local file.") object is to use the [File::createInputStream()](https://docs.juce.com/master/classFile.html#a668db4b90ac7839c70f0a9ac22124c29 "Creates a stream to read from this file.") function. This function returns a [FileInputStream](https://docs.juce.com/master/classFileInputStream.html "An input stream that reads from a local file.") object on the heap, allocated using the `new` operator. This means that it is _very_ important that you deallocate the object when you are finished. Ideally, you should use a std::unique_ptr object for this. A slight difference here is that the [File::createInputStream()](https://docs.juce.com/master/classFile.html#a668db4b90ac7839c70f0a9ac22124c29 "Creates a stream to read from this file.") function will return a `nullptr` value if the file stream fails to open. The following code shows the typical pattern that you should use in this case:

```cpp
void readFile (const juce::File& fileToRead)
{
    if (!fileToRead.existsAsFile())
        return; // file doesn't exist

    if (std::unique_ptr<juce::FileInputStream> inputStream { fileToRead.createInputStream() })
    {
        textContent->clear();

        while (!inputStream->isExhausted())
        {
            auto nextWord = readUpToNextSpace (*inputStream);
            textContent->setColour (juce::TextEditor::textColourId, getRandomColour (0.75f));
            textContent->insertTextAtCaret (nextWord);
        }
    }
}
```

> [!NOTE]
> Exercise: You may notice that the colours of words at the ends of some lines are identical to the colour of the word at the start of the next line. This is because we are looking only for the space character as a delimiter. Modify the code to look for new line, carriage return, and tab characters too. (These are characters `'\\n'` , `'\\r'` , and `'\\t'` .)

# Reading binary data

In this tutorial we have looked at reading string data from a file. In addition to reading single bytes from a file, the [InputStream](https://docs.juce.com/master/classInputStream.html "The base class for streams that read data.") class also includes functions for reading other fundamental types. For example:

- [InputStream::readInt()](https://docs.juce.com/master/classInputStream.html#a59eb456ebfbe9d4c7fdfd4c14337e19a "Reads four bytes from the stream as a little-endian 32-bit value.") --- reads an `int` (32-bit) from the stream.
- [InputStream::readShort()](https://docs.juce.com/master/classInputStream.html#a0c4b0f9f1fa9515fea5a98d2ffe7ae02 "Reads two bytes from the stream as a little-endian 16-bit value.") --- reads a `short` (16-bit) integer from the stream.
- [InputStream::readFloat()](https://docs.juce.com/master/classInputStream.html#a46d7f191f1872bc27550db3fa0733f59 "Reads four bytes as a 32-bit floating point value.") --- reads a `float` (32-bit) from the stream.

These all read multi-byte values using the little endian byte order. To read them as big endian values there are alternative versions---for example, the [InputStream::readIntBigEndian()](https://docs.juce.com/master/classInputStream.html#a84ab1bcc547eee621c4c6c2502af808d "Reads four bytes from the stream as a big-endian 32-bit value.") function. You can also read a block of data from a stream using the [InputStream::read()](https://docs.juce.com/master/classInputStream.html#aa5350c414bad6b97ae3b463a3401c0d6 "Reads some data from the stream into a memory buffer.") or [InputStream::readIntoMemoryBlock()](https://docs.juce.com/master/classInputStream.html#a7708d25af96e8d8b937a4642dcf55a23 "Reads from the stream and appends the data to a MemoryBlock.") functions.

These are useful if you need to read existing files in binary format, or if you really need to have your data stored as binary. In most cases, it is probably preferable to use XML (using the [XmlDocument](https://docs.juce.com/master/classXmlDocument.html "Parses a text-based XML document and creates an XmlElement object from it.") and [XmlElement](https://docs.juce.com/master/classXmlElement.html "Used to build a tree of elements representing an XML document.") classes) or [JSON](https://docs.juce.com/master/classJSON.html "Contains static methods for converting JSON-formatted text to and from var objects.") (by storing data in var objects) for storing and reading your data.

# Summary

This tutorial has introduced simple file reading technique using JUCE through reading a text file in various ways. In particular you should be able to:

- Use the [FilenameComponent](https://docs.juce.com/master/classFilenameComponent.html "Shows a filename as an editable text box, with a 'browse' button and a drop-down list for recently se...") and the [FilenameComponentListener](https://docs.juce.com/master/classFilenameComponentListener.html "Listens for events happening to a FilenameComponent.") to store a file path and offer the user a means of choosing a file.
- Read the entire contents of a file into a [String](https://docs.juce.com/master/classString.html "The JUCE String class!") object.
- Read a file in smaller chunks using the appropriate [InputStream](https://docs.juce.com/master/classInputStream.html "The base class for streams that read data.") functions.

# See also

- [Tutorial: Colours in JUCE](/tutorials/tutorial_colours/)
- [Tutorial: The Label class](/tutorials/tutorial_label/)
- [Tutorial: Build an audio player](/tutorials/tutorial_playing_sound_files/)
