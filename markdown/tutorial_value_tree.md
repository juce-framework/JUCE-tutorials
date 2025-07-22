---
title: The ValueTree class
---
# Tutorial: The ValueTree class

Learn how to use the [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") class to manage data effectively in your applications.

**LEVEL:** Intermediate<br/>
**PLATFORMS:** Windows, macOS, Linux, iOS, Android<br/>
**CLASSES:** [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ..."), [var](https://docs.juce.com/master/classvar.html "A variant class, that can be used to hold a range of primitive values."), [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.")

# Introduction

The [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") class is JUCE's secret weapon. It has the power to massively streamline the internal complexity of your applications, turning the traditionally passive data model into an active participant in runtime operation. It takes care of some of the fiddlier aspects of development, such as interfacing data with the GUI, and providing automatic undo and redo of data modifications, as well as providing a universal container for your data. It is also inherently serialisable, making the export and import process very straightforward.

You may not know about them, or perhaps you may have simply not realised their potential. This guide is intended to cover everything you need to know in order to use them.

# Three essential classes

The [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") class is really an ensemble and, when working with them, you will always be interacting with at least three important classes. These are the [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") class itself, the [var](https://docs.juce.com/master/classvar.html "A variant class, that can be used to hold a range of primitive values.") class, and the [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") class. It is impossible to do anything meaningful with [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") objects without encountering these, so it is useful to appreciate how (and why) you should be using them. Therefore, before we discuss the actual practicalities of manipulating such data, you should familiarise yourself with the following:

## ValueTree class overview

The [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") class is the ultimate container class, capable of holding any kind of information. Whilst this may be the obvious role of the type, there are many other facets to it, taking it beyond mere structural duties.

### Data Storage

A [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") object is a flexible, multi-purpose data object. It has a basic _type_ name, and can hold an arbitrary set of named properties. It is like a kind of _universal struct_; you don't need to define what (or how much) data it should hold, you can just use it for whatever you like at runtime.

For illustrative purposes, one object (shown here as _pseudo-data_) might hold the following information:

```
Pet
Name = "Fluffmuff"
Animal = "Cat"
Size = 2.4
```

As the name suggests, a [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") object may also function as a node in a tree structure. Along with its named properties, it can contain any number of child nodes (and can in turn itself be added to a parent node).

```
Pet
Name = "Fluffmuff"
Animal = "Cat"
Size = 2.4
Accessories
Collar
Colour = "Pink"
Camera
HasFlash = false
Capacity = 32
```

These structures are very similar to those formed in the XML format; an individual [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") node is very much like an [XmlElement](https://docs.juce.com/master/classXmlElement.html "Used to build a tree of elements representing an XML document.") object, having a name, properties, and children. The big difference is that the properties are stored as actual typed data, rather than the text-for-everything representation used by the XML format . This means they can hold more complex types of data, access it more efficiently, and be generally better suited to use in a data model (and not just serialisation). In fact, it is possible to automatically generate XML text (or a special binary format) from any given [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") node, and later restore that structure.

### Reference counting

Each node is reference counted, so their lifetimes are easily managed. The data itself is actually stored in a hidden shared instance, for which the [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") class is just a light, reference-keeping wrapper interface.

You can pass them around quickly by value, and not have to use pointers directly; returning a [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") object from a function does not copy any data, only a reference, so it makes your interfaces both simple and safe.

You never have to worry about deleting them yourself; as soon as you are no longer using one anywhere, it will be automatically destroyed. This also means that the node data will never be accidentally deleted whilst you might be still using it, which is particularly handy for ensuring an asynchronous UI will never encounter dangling pointers.

### Simple interface

It has a simple, universal interface for manipulating its properties and children. By being generic, it can have a single interface to access the content, regardless of its type or organisation. Using the universal [var](https://docs.juce.com/master/classvar.html "A variant class, that can be used to hold a range of primitive values.") class as the common property type, it can take a variety of inputs. Using the [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") class to store and retrieve these properties by name, it presents an intuitive way to organise your data.

### Undo and redo

By having such a concise set of controls, it is able to provide built-in support for undo; all of the functions which modify content incorporate predefined undoable actions, so you need only supply an [UndoManager](https://docs.juce.com/master/classUndoManager.html "Manages a list of undo/redo commands.") object to provide such functionality in your application. In case you weren't already convinced, this is a very compelling reason to use [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") objects in a data model.

### Notifications

Another very powerful feature is the ability for ValueTrees to send notifications when their contents change; this offers huge practical simplifications, particularly in keeping the UI up to date. For example, a [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") object being used to display the contents of a node can simply refresh itself whenever it sees that the data has been edited---you need only implement it as a [ValueTree::Listener](https://docs.juce.com/master/classValueTree_1_1Listener.html "Listener class for events that happen to a ValueTree.") subclass.

### Summary

The [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") class is like some kind of miracle class for the application data model, effortlessly providing a wealth of functionality to simplify the way the insides of your application slot together.

## var class overview

The [var](https://docs.juce.com/master/classvar.html "A variant class, that can be used to hold a range of primitive values.") class is a universal _variant_ class, for holding data of various types. Its functionality makes it suitable for representing JSON data structures.

Traditionally, you would have to decide in advance what type of data any given variable in your code may store (for example, if you want to hold a whole number, you would choose an `int` , and that is all that variable is allowed to be used for). However, if you were to use a [var](https://docs.juce.com/master/classvar.html "A variant class, that can be used to hold a range of primitive values."), such a decision does not have to be made, as it is compatible with a variety of types.

It is like a generic chameleon variable, capable of representing basic numeric values (`int` or `double` ), text (a [String](https://docs.juce.com/master/classString.html "The JUCE String class!") object), and `bool` values, as well as a `void` state (because `0` or `false` can be conceptually different to _nothing_). They can also hold a pointer to any class derived from the [ReferenceCountedObject](https://docs.juce.com/master/classReferenceCountedObject.html "A base class which provides methods for reference-counting.") class (which could be comprised of any kind of complex data you can imagine). As if that weren't enough, a single [var](https://docs.juce.com/master/classvar.html "A variant class, that can be used to hold a range of primitive values.") object can also be used as an array of multiple [var](https://docs.juce.com/master/classvar.html "A variant class, that can be used to hold a range of primitive values.") objects!

This versatility makes it ideal for use in a generic container (such as the [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") class), allowing its interface the luxury of not having to care what type you're giving to it (or expecting from it), as long as it can be held in a var. There is implicit casting (and overloaded assignment operators) for the basic types, making interaction in code simple. You can even automatically return a string representation of its current value (including non-text types). The only place these luxuries don't all apply is when a [var](https://docs.juce.com/master/classvar.html "A variant class, that can be used to hold a range of primitive values.") object holds a [ReferenceCountedObject](https://docs.juce.com/master/classReferenceCountedObject.html "A base class which provides methods for reference-counting.") object. As these are unknown types, you have to cast them yourself (you can do this safely with a `dynamic_cast` , as this returns `nullptr` if it is not of the correct type).

In a [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") object, all properties are held as [var](https://docs.juce.com/master/classvar.html "A variant class, that can be used to hold a range of primitive values.") objects. By choosing this multi-purpose class as a property type, it is possible to have a single set of functions to access them, regardless of what they may be. You don't need to use one function for `int` values and another for text; all such functions can be unified by dealing with [var](https://docs.juce.com/master/classvar.html "A variant class, that can be used to hold a range of primitive values.") objects.

## Identifier class overview

This class is intended as a human-readable _key_, used to identify data.

Essentially, an [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") object is a string. You can assign one from a [String](https://docs.juce.com/master/classString.html "The JUCE String class!") object, and you can also retrieve their contents as a [String](https://docs.juce.com/master/classString.html "The JUCE String class!") object. In the context of the [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") class, it is used both to specify a type name for the node, and also to uniquely label each of its properties.

### Why not just use the String class?

There are two main reasons for using a specialised class instead of the general purpose [String](https://docs.juce.com/master/classString.html "The JUCE String class!") class.

- It can force a limited character set: Firstly, this class enforces a limitation on the characters which can make up a valid key; it only allows alphanumeric characters and special characters `_-:#$%` . This might sound a bit rubbish, but it makes it possible to ensure compatibility with other systems with the same limitation (for example, the XML format and scripts).
- It can be optimised for purpose: The second (but most important) reason stems from how we want to use them. They are to act as a key to identify a single item from a list of any size, and so the most common operation to be performed with them is a comparison. However, comparisons of [String](https://docs.juce.com/master/classString.html "The JUCE String class!") objects can be fairly slow. We actually have to check the text, and only when we find the first different character can we say for sure that two [String](https://docs.juce.com/master/classString.html "The JUCE String class!") objects are not the same. For strings that are mostly the same, we may end up checking most of the letters (and strings which are a match will have had all of their characters tested). For a single comparison, that might be acceptable, but when you want to compare one string against a list (as we would when using it as a key), the whole business can take a long time.

By using a special class, we can ensure that it is optimised to allow for quick comparisons. Because of this optimisation (and the limited character set), they are not intended to be used for general text handling, but they are perfectly suited to using string-like data as a key.

### How can they be faster to compare?

It is useful to know about the optimisation of the [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") class. For one thing, it will reassure you that the code is not secretly checking all the characters in each test. More importantly, however, as with many optimisations, there is a cost involved, and it helps to know where that is so you can avoid it being an issue.

It is perfectly possible to use the [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") without knowing this stuff, but you will have a better understanding of where the trade-offs occur if you do.

> [!TIP]
>The details of this optimisation may well change (this is Secondary Knowledge of a class), but it is unlikely that the implications for your implementation will.

### A special case in String comparisons

To know that two [String](https://docs.juce.com/master/classString.html "The JUCE String class!") objects are the same, what you actually have to do is prove that they are not different. As it happens, the [String](https://docs.juce.com/master/classString.html "The JUCE String class!") class holds reference counted text, so there is a chance that two [String](https://docs.juce.com/master/classString.html "The JUCE String class!") objects actually point to the same data; if that is the case, we can spot it straight away (as they will both hold the same address).

These special cases are much quicker, but the [String](https://docs.juce.com/master/classString.html "The JUCE String class!") class can only take advantage of them if they happen. When the addresses are not the same, it does not prove that their content is not equivalent, and so we must still check the characters.

### Exploiting the special case

The [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") class exploits this behaviour, and makes it so that we can actually guarantee that all equivalent [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") objects will always be pointing to the same piece of data. Because of the way they are optimised, a character-for-character match of different [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") objects will never exist for different memory addresses. This means that the _special case_ of identical strings is now the only way to be equivalent, and thus proving that they are different is as simple as spotting that they hold different addresses.

This is not magic, however, as the cost is simply moved elsewhere.

In order to enforce this behaviour, all [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") objects share a single, concealed, global pool of unique strings. The strings held by every [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") object used at runtime are stored in this pool. When assigning an [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") object from a [String](https://docs.juce.com/master/classString.html "The JUCE String class!") object, the pool is checked to see if it already contains an equivalent copy. If it does, that will be used instead; if not, the new [String](https://docs.juce.com/master/classString.html "The JUCE String class!") object is added, ready to be found by the next [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") object to look for it.

### The cost

Thus, instantiating an [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") object from a [String](https://docs.juce.com/master/classString.html "The JUCE String class!") object is the most expensive part of their operation. Once one exists, if you always copy from it, you will never have to pay that price for it again. Sometimes it is unavoidable (specifically if you are acquiring an [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") object from input, or data held in a normal [String](https://docs.juce.com/master/classString.html "The JUCE String class!") object), but this is far less regular an occurrence than comparing them. Every time you assign one from a bare [String](https://docs.juce.com/master/classString.html "The JUCE String class!") object (either from data, or as a literal in code), a pool check is required to enforce the optimisation. If you instead assign from another existing [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") object, it is guaranteed that such a check has already been done.

### How to minimise costs

A good strategy is to initialise some easily-accessible [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") instances at startup. From your code, you can then use these instead of literal strings, and you will never incur any further lookup penalties from them. You could put them in a globally accessible namespace, use file static instances, or even static class members to help organise them.

# Basic ValueTree class usage

In addition to the [var](https://docs.juce.com/master/classvar.html "A variant class, that can be used to hold a range of primitive values.") and [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") classes, there are actually many more that you will see used in conjunction with [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") objects, as we shall see later. These, however, are required for most of the fundamental operations in the [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") class.

Once you\'ve learned about the three essential classes, you should be in a good position to proceed to pick up the basics.

This is a basic guide introducing all of the essential core functions of the [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") class. It covers all of the functionality you need to be able to get started using them in your code.

## Creating a ValueTree object

You can easily use [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") objects by simply creating an object on the stack (or as a member of a class).

## Invalid ValueTree objects

A [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") object is, by default, **invalid**. That is, a [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") object initialised with the _default constructor_ will have no data at all. Unless you explicitly initialise it with an [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") (or an existing, valid, [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") object), it will hold nothing; it is an empty shell.

```cpp
juce::ValueTree myNode; // this object is invalid - it does not hold any node data
```

This is similar to a null pointer (in fact, internally, that's exactly what it is), but without any of the inherent dangers. Until it is assigned a valid node, it refers to nothing. The main difference here is that the interface functions can still be used, albeit largely without any effect (as there is nothing to be manipulated).

Whilst it is safe to \'accidentally\' call the access functions on an invalid node, it doesn\'t get you very far!

## How to check if a node is invalid

In your code you can call the member function [ValueTree::isValid()](https://docs.juce.com/master/classValueTree.html#a9c1506afbe6b840e4ab43fabf275845a "Returns true if this tree refers to some valid data.") to find out if the [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") is empty:

```cpp
if (myNode.isValid())
{
    // This will not be reached for an invalid node
}
```

## Valid ValueTree objects

The primary means of creating a valid [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") object is by initialising with a valid [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") object. This is used to _name_ the new node, and represents the node's **type**.

```cpp
static juce::Identifier myNodeType ("MyNode"); // pre-create an Identifier
juce::ValueTree myNode (myNodeType); // This is a valid node, of type "MyNode"
```

> [!TIP]
>You can use a string literal instead of explicitly providing an [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") object, but it is better practice to use an existing [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") instance where possible. (See [Identifier class overview](#tutorial_value_tree_indentifier_overview) for more information.)

By requiring that a valid node has a type, we have a mechanism of indicating what sort of data it should contain. Conversely, when faced with a [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") object (for example, as a parameter supplied to a function), you can check its type---using the [ValueTree::getType()](https://docs.juce.com/master/classValueTree.html#aaba6256213ee02c418e9ef3d320f0ff1 "Returns the type of this tree.") function---to determine what you might expect to find inside it.

```cpp
void foo (const juce::ValueTree& someNode)
{
    if (someNode.getType() == myNodeType)
    {
        // This would be hit for nodes created as “MyNode”
    }
}
```

> [!TIP]
>Once a node is created, you cannot change its type. This is intentional, as such information is not really a property of the object, it is the fundamental essence of the object.

## Sharing valid ValueTree objects

The third way in which a [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") can be initialised is via the copy constructor, by providing an existing [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") object.

```cpp
[1] : juce::ValueTree myNode (myNodeType); // creates a new node of type "MyNode"
[2] : juce::ValueTree sameNode (myNode); // This object points to the same data as myNode
```

This results in an object which refers to _the same node data_ as the existing object.

It is important to understand that nothing is copied here except for a reference; both objects are attached to **the same underlying instance** of the original node data. Changes made to either (if the shared node is valid) will be found upon inspection of the other.

You can achieve the same result (a sharing of node data) by using the assignment operator.

```cpp
[3] : juce::ValueTree otherNode (myNodeType); // This creates a second (new) "MyNode" node...
[4] : otherNode = sameNode; // ...but the object now points to the first instance
```

Here is a question:

What happens to the node instance created in line [3] when we replace it in line [4]?

By the end, all three variables point to the same initial instance. There are no longer any [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") objects referring to the second instance, so **it is destroyed**.

Whenever a [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") object is reassigned (or goes out of scope), the underlying data it was pointing to loses a reference. If that data is not held elsewhere, it is automatically destroyed. This may sound scary, as if you might be operating in a world where your data could self-destruct at any moment; in reality, it is very unlikely that you will inadvertently lose your nodes! It is a very robust system. Besides, if the data is important, you'll probably find that you have in fact already stored it somewhere; it is as intuitive as you could wish for.

> [!WARNING]
> This also means that you need to be careful in real-time code such as an audio callback. Reassigning a [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") object in an audio callback could inadvertently cause the `delete` operator if there are no longer references to the previously assigned data.

## A note about the UndoManager class

All of the access functions in the [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") class understand how to use [UndoManager](https://docs.juce.com/master/classUndoManager.html "Manages a list of undo/redo commands.") objects. They come ready-prepared with suitable [UndoableAction](https://docs.juce.com/master/classUndoableAction.html "Used by the UndoManager class to store an action which can be done and undone.") objects; all you have to do is provide a pointer to an [UndoManager](https://docs.juce.com/master/classUndoManager.html "Manages a list of undo/redo commands.") object with your call, and an operational, undoable record of the modification will be added to it automatically.

There's not a lot you need to know to manage this, but we cover it in more detail in [Tutorial: Using an UndoManager with a ValueTree](/tutorials/tutorial_undo_manager_value_tree/).

For now, we will make use of the functions without bothering to store a history, by providing a `nullptr` value. We will take it as read that any such modification functions must be given at least a null pointer to an [UndoManager](https://docs.juce.com/master/classUndoManager.html "Manages a list of undo/redo commands.") object when describing them; there will of course be `nullptr` values visible in the examples.

## Receiving notifications as a Listener

Registering as a listener to the [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") allows us to be notified synchronously when the data has changed. The pointer to the listener is held in the [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") instance and therefore it is usually best to take a copy of the [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") before registering like so:

```cpp
struct ExampleListener : public juce::ValueTree::Listener
{
    ExampleListener (juce::ValueTree v)
        : tree (v)
    {
        tree.addListener (this);
    }

    juce::ValueTree tree;
};
```

We can then implement the following callback functions depending on the behaviour we wish to be notified for:

- valueTreePropertyChanged(): Called when a property change occurs in the [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") or any of its children.
- valueTreeChildAdded(): Called when a child node is added to the [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") or any of its children.
- valueTreeChildRemoved(): Called when a child node is removed from the [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") or any of its children.
- valueTreeChildOrderChanged(): Called when a child node is reordered in the [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") or any of its children.
- valueTreeParentChanged(): Called when the [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") is added or removed from a parent node.

Callbacks are propagated upwards in the tree hierarchy so listeners to parent nodes will also receive property change callbacks from children and therefore types and property names should be checked.

> [!WARNING]
> Since callbacks are synchronous, for any time-critical applications you should use an [AsyncUpdater](https://docs.juce.com/master/classAsyncUpdater.html "Has a callback method that is triggered asynchronously.") to handle property changes.

# Basic property access

## Setting a property

The [ValueTree::setProperty()](https://docs.juce.com/master/classValueTree.html#a550e9c8780d4ee56d14d67de17f200e2 "Changes a named property of the tree.") function is one direct way to set a property on a valid node. You must provide an [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") object to specify the name of the property you wish to set, and a [var](https://docs.juce.com/master/classvar.html "A variant class, that can be used to hold a range of primitive values.") value for it to take:

```cpp
static juce::Identifier propertyName ("name");
myNode.setProperty (propertyName, "Fluffmuff", nullptr);
```

## Getting a property

There are two basic ways to retrieve a property. For each approach, you must provide an [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") to specify which named property you are requesting.

Using the [ValueTree::getProperty()](https://docs.juce.com/master/classValueTree.html#a97f6d4b2a0e28be73d349cba363ab7e2 "Returns the value of a named property.") function:

```cpp
juce::String name (myNode.getProperty (propertyName));
```

Or using the subscript operator (that is, the [ValueTree::operator\[](https://docs.juce.com/master/classValueTree.html#a6cf8b3f5eac2d14f1323f5fe0f228a71 "Returns the value of a named property.") function):

```cpp
name = myNode[propertyName];
```

Both of the above lines store the same result into the `name` variable. Remember that these properties can hold any supported [var](https://docs.juce.com/master/classvar.html "A variant class, that can be used to hold a range of primitive values.") type. Thus, the following code is also valid:

```cpp
static juce::Identifier propertySize ("size");
myNode.setProperty (propertySize, 2.4, nullptr);
double size = myNode[propertySize];
```

Here, a property is set in the same way using a `double` value, and is easily read back in as such. It is also possible to replace an existing property with a different type of value.

If all you want to do is store individual values in a [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") node (and nothing more complex), then you already know enough to get started!

## Finding out about properties

If you were using structs, you would only have the ability to set or get these member variables directly. With a [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") node, you can also (at runtime) find out what _variables_ there are.

The [ValueTree::getNumProperties()](https://docs.juce.com/master/classValueTree.html#a018730ee95c743705aa9132a37829b32 "Returns the total number of properties that the tree contains.") function tells you how many properties a node has:

```cpp
int numProperties = myNode.getNumProperties();
```

The [ValueTree::getPropertyName()](https://docs.juce.com/master/classValueTree.html#a6ea2ce53b4759a37d58e3f99d9702e36 "Returns the identifier of the property with a given index.") function returns an [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") object giving the name of the property at the specified position. By using this in combination with the [ValueTree::getNumProperties()](https://docs.juce.com/master/classValueTree.html#a018730ee95c743705aa9132a37829b32 "Returns the total number of properties that the tree contains.") function, it is possible to iterate over the properties without previously knowing what they are.

```cpp
for (int i = 0; i < numProperties; ++i)
{
    juce::Identifier name (myNode.getPropertyName (i));
    // …
}
```

The [ValueTree::hasProperty()](https://docs.juce.com/master/classValueTree.html#a0dbec0d665fb9d57b9fd7b3189ec0ee0 "Returns true if the tree contains a named property.") function simply tells you whether or not a particular named property has been set for a node:

```cpp
if (myNode.hasProperty (nameProperty))
{
    // property was found
}
```

These functions allow for what is called reflection, whereby your program is able to inspect the nature of your objects. When faced with an object to work with, your code doesn\'t necessarily need a definition (for example, class header) to be able to make use of it; you can check to see what members it has, and use them if they are appropriate.

# Basic child access

If you want to create more complex structures, you will want to start using these objects as nodes in a hierarchy.

## Adding a child

Whilst a node can have a bunch of properties attached to it, it can also contain a number of child nodes, much like an array.

If we have a node we wish to add as a child, we simply call the [ValueTree::addChild](https://docs.juce.com/master/classValueTree.html#a93d639299ef9dfedc651544e05f06693 "Adds a child to this tree.") () function on the node we wish to add it to. We naturally pass in the new child node, but we must also specify where it should go. If you don\'t need it to be inserted at any particular position, you can specify `-1` to simply place it on the end.

```cpp
juce::ValueTree childNode (myNodeType);
myNode.addChild (childNode, -1, nullptr);
```

Here, the data held in `childNode` node belongs to the data held in `myNode` .

Note that this _belonging_ is also an additional reference. If we reassign the `childNode` variable, the existing data will not be lost.

```cpp
childNode = juce::ValueTree (myNodeType);
myNode.addChild (childNode, -1, nullptr);
```

Whilst we may no longer have any [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") objects in our immediate scope pointing directly to that instance, it is kept alive within the original node.

## Getting a child

There are a number of ways in which you can retrieve a child node. Which you should choose for any given task depends upon how your chosen structures are organised.

With all of them, any requests which don\'t correspond to a child will return an invalid object.

The [ValueTree::getChild()](https://docs.juce.com/master/classValueTree.html#a3f89053219ec515f24997a115cfd74e5 "Returns one of this tree's sub-trees.") function returns the child currently sitting in the specified position within the node's internal list. If you\'re always adding them to the end, `0` would correspond to the first one you put in:

```cpp
childNode = myNode.getChild (0);
```

The [ValueTree::getChildWithName()](https://docs.juce.com/master/classValueTree.html#a4047208c32c4e024a1f2809d1c35be48 "Returns the first sub-tree with the specified type name.") function returns the first child with the specified name identifier as its node type:

```cpp
childNode = myNode.getChildWithName (myNodeType);
```

The [ValueTree::getChildWithProperty()](https://docs.juce.com/master/classValueTree.html#aa17ae8b3ef8149fe5b9cc8b37d4ebd7d "Looks for the first sub-tree that has the specified property value.") function returns the first child which has the named property set to the specified value:

```cpp
childNode = myNode.getChildWithProperty (nameProperty, "Fluffmuff");
```

The last two, along with any other such methods you require, can of course be written using the first one; it's easy to write your own functions to retrieve a child by some other criteria (for example, a combination of matching node type and a property value).

You can also retrieve the current owner of any given node using the [ValueTree::getParent()](https://docs.juce.com/master/classValueTree.html#ad22561c896d9bcdb763d27aacbb5815c "Returns the parent tree that contains this one.") function:

```cpp
ValueTree parent (childNode.getParent());
```

# Summary

This tutorial has given a detailed overview of the [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") class and related classes. In particular, you should:

- Know the importance of the [var](https://docs.juce.com/master/classvar.html "A variant class, that can be used to hold a range of primitive values.") and [Identifier](https://docs.juce.com/master/classIdentifier.html "Represents a string identifier, designed for accessing properties by name.") classes.
- How to create [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") objects and pass them around.
- How to add and access properties of [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") objects.
- How to add and access child nodes to a [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") object.

# See also

- [Tutorial: Using an UndoManager with a ValueTree](/tutorials/tutorial_undo_manager_value_tree/)
- [Tutorial: Saving and loading your plug-in state](/tutorials/tutorial_audio_processor_value_tree_state/)
