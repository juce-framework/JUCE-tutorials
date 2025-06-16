# Tutorial: Using an UndoManager with a ValueTree

Implement undo/redo actions in your applications. Easily restore previous intermediate states with [UndoableAction](https://docs.juce.com/master/classUndoableAction.html "Used by the UndoManager class to store an action which can be done and undone.") objects and learn how to group undoable actions into transactions.

**LEVEL:** Intermediate<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**CLASSES:** [UndoManager](https://docs.juce.com/master/classUndoManager.html "Manages a list of undo/redo commands."), [UndoableAction](https://docs.juce.com/master/classUndoableAction.html "Used by the UndoManager class to store an action which can be done and undone."), [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ..."), [TreeView](https://docs.juce.com/master/classTreeView.html "A tree-view component."), [TreeViewItem](https://docs.juce.com/master/classTreeViewItem.html "An item in a TreeView.")

# Getting started

This tutorial assumes basic understanding of [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") objects as explained in [Tutorial: The ValueTree class](/tutorials/tutorial_value_tree/). If you haven\'t done so already, you should read that tutorial first.

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/UndoManagerValueTreeTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/UndoManagerValueTreeTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project illustrates the use of the [UndoManager](https://docs.juce.com/master/classUndoManager.html "Manages a list of undo/redo commands.") class in conjunction with [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") objects to show how easily past history can be restored. It presents the [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") data as a tree structure using the [TreeView](https://docs.juce.com/master/classTreeView.html "A tree-view component.") and [TreeViewItem](https://docs.juce.com/master/classTreeViewItem.html "An item in a TreeView.") classes. If you build and run the project, you should see something like this:

![](/_images/tutorial_undo_manager_value_tree_screenshot1.png "The demo project application window")

At the moment, we can drag and drop [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") nodes and change the hierarchy of the data structure. We can also expand and collapse children but we cannot undo and redo our changes. Let's try to implement that functionality using the [UndoManager](https://docs.juce.com/master/classUndoManager.html "Manages a list of undo/redo commands.") class.

> [!TIP]
>The code presented here is broadly similar to the **ValueTreesDemo** from the JUCE Demo.

# Adding Undo/Redo buttons

Let's first add two [TextButton](https://docs.juce.com/master/classTextButton.html "A button that uses the standard lozenge-shaped background with a line of text on it.") objects to our user interface to allow undo and redo functionality. You should be familiar with lambda functions for this section and if you need help with these steps, you can refer to the [Tutorial: Listeners and Broadcasters](/tutorials/tutorial_listeners_and_broadcasters/) tutorial.

In the `MainContentComponent` class, declare [TextButton](https://docs.juce.com/master/classTextButton.html "A button that uses the standard lozenge-shaped background with a line of text on it.") variables for each button [1]:

```cpp
juce::TreeView tree;
juce::TextButton undoButton, redoButton; // [1]
std::unique_ptr<ValueTreeItem> rootItem;
```

In the constructor member initialisation list, set the text for the [TextButton](https://docs.juce.com/master/classTextButton.html "A button that uses the standard lozenge-shaped background with a line of text on it.") objects [2]:

```cpp
MainContentComponent()
    : undoButton ("Undo"),
      redoButton ("Redo") // [2]
{
```

Finally, make the buttons visible [3] and prepare the lambda functions to be assigned to the [Button::onClick](https://docs.juce.com/master/classButton.html#a30b76ab312dc7f66e67596ae20540ec2 "You can assign a lambda to this callback object to have it called when the button is clicked.") helper objects [4]:

```cpp
//...
addAndMakeVisible (undoButton);
addAndMakeVisible (redoButton); // [3]
undoButton.onClick = [this] {};
redoButton.onClick = [this] {}; // [4]

setSize (600, 400);
```

We can then set the bounds for the buttons in the `resized()` method:

```cpp
void resized() override
{
    // This is called when the MainContentComponent is resized.
    // If you add any child components, this is where you should
    // update their positions.

    auto r = getLocalBounds();

    auto buttons = r.removeFromBottom (20);
    undoButton.setBounds (buttons.removeFromLeft (100));
    redoButton.setBounds (buttons.removeFromLeft (100));

    tree.setBounds (r);
}
```

# Passing the UndoManager instance as an argument

Since the [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") class handles undo/redo behaviour automatically, we need only pass the [UndoManager](https://docs.juce.com/master/classUndoManager.html "Manages a list of undo/redo commands.") instance as a parameter to register [UndoableAction](https://docs.juce.com/master/classUndoableAction.html "Used by the UndoManager class to store an action which can be done and undone.") objects. To implement this, first declare an instance of the [UndoManager](https://docs.juce.com/master/classUndoManager.html "Manages a list of undo/redo commands.") class [1]:

```cpp
juce::UndoManager undoManager; // [1]
```

Then assign the functions to be called when the buttons are clicked in order to handle the corresponding undo/redo behaviour. In the lambda functions, respectively call [`UndoManager::undo()`](https://docs.juce.com/master/classUndoManager.html#a39f45c284e8d0df1a0d378e676246931 "Tries to roll-back the last transaction.") and [`UndoManager::redo()`](https://docs.juce.com/master/classUndoManager.html#aaea507a3b9eaea3360c0e393edf69ccb "Tries to redo the last transaction that was undone.") as follows:

```cpp
addAndMakeVisible (undoButton);
addAndMakeVisible (redoButton); // [3]
undoButton.onClick = [this] { undoManager.undo(); };
redoButton.onClick = [this] { undoManager.redo(); }; // [4]

setSize (600, 400);
```

In the `ValueTreeItem` class, we also keep a reference to the [UndoManager](https://docs.juce.com/master/classUndoManager.html "Manages a list of undo/redo commands.") instance [2]:

```cpp
private:
    juce::ValueTree tree;
    juce::UndoManager& undoManager; // [2]
```

In the member initialisation list of the class constructor, assign the [UndoManager](https://docs.juce.com/master/classUndoManager.html "Manages a list of undo/redo commands.") reference [3]:

```cpp
ValueTreeItem (const juce::ValueTree& v, juce::UndoManager& um)
    : tree (v), undoManager (um) // [3]
{
```

Whenever a sub-item of ValueTreeItem is created recursively, we need to pass the [UndoManager](https://docs.juce.com/master/classUndoManager.html "Manages a list of undo/redo commands.") instance [4]:

```cpp
void refreshSubItems()
{
    clearSubItems();

    for (auto i = 0; i < tree.getNumChildren(); ++i)
        addSubItem (new ValueTreeItem (tree.getChild (i), undoManager)); // [4]
}
```

We can now instantiate the root ValueTreeItem by passing the [UndoManager](https://docs.juce.com/master/classUndoManager.html "Manages a list of undo/redo commands.") instance [5] in the `MainContentComponent` class:

```cpp
tree.setDefaultOpenness (true);
tree.setMultiSelectEnabled (true);
rootItem.reset (new ValueTreeItem (createRootValueTree(), undoManager)); // [5]
tree.setRootItem (rootItem.get());
```

Now there are three different methods that need to be updated to register the changes we perform on the [TreeView](https://docs.juce.com/master/classTreeView.html "A tree-view component.").

```cpp
void itemDropped (const juce::DragAndDropTarget::SourceDetails&, int insertIndex) override
{
    juce::OwnedArray<juce::ValueTree> selectedTrees;
    getSelectedTreeViewItems (*getOwnerView(), selectedTrees);

    moveItems (*getOwnerView(), selectedTrees, tree, insertIndex, undoManager); // [1]
}

static void moveItems (juce::TreeView& treeView, const juce::OwnedArray<juce::ValueTree>& items, juce::ValueTree newParent, int insertIndex, juce::UndoManager& undoManager)
{
```

```cpp
v.getParent().removeChild (v, &undoManager); // [2]
newParent.addChild (v, insertIndex, &undoManager); // [3]
```

- [1]: Whenever an item is drag and dropped, pass the undoManager to the static function that handles the move.
- [2]: We need to remove the child from the previous parent and register that action to the undoManager.
- [3]: We can then add the child to a new parent and register the new action to the undoManager.

By passing the `undoManager` reference to the [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") functions `addChild()` and `removeChild()` , we let the [UndoManager](https://docs.juce.com/master/classUndoManager.html "Manages a list of undo/redo commands.") perform the [UndoableAction](https://docs.juce.com/master/classUndoableAction.html "Used by the UndoManager class to store an action which can be done and undone.") for us by calling the `perform()` function under the hood. We will cover [UndoableAction](https://docs.juce.com/master/classUndoableAction.html "Used by the UndoManager class to store an action which can be done and undone.") objects in a future tutorial.

> [!NOTE]
> Exercise: Display descriptions of stored undo and redo actions in [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") components next to their respective [TextButton](https://docs.juce.com/master/classTextButton.html "A button that uses the standard lozenge-shaped background with a line of text on it.") objects using the `getUndoDescription()` and `getRedoDescription()` functions respectively.

# Handle events as transactions

Another useful feature of the [UndoManager](https://docs.juce.com/master/classUndoManager.html "Manages a list of undo/redo commands.") is its ability to group several actions together as a single undo/redo transaction. By calling the `beginNewTransaction()` function on the `undoManager` instance, all the calls to the `perform()` function of the [UndoManager](https://docs.juce.com/master/classUndoManager.html "Manages a list of undo/redo commands.") are grouped together until the next `beginNewTransaction()` call.

As an example, let's create a [Timer](https://docs.juce.com/master/classTimer.html "Makes repeated callbacks to a virtual method at a specified time interval.") to call the `beginNewTransaction()` function periodically and store groups of actions together as transactions. In the `MainContentComponent` , inherit from the [Timer](https://docs.juce.com/master/classTimer.html "Makes repeated callbacks to a virtual method at a specified time interval.") class to receive timer callbacks [1]:

```cpp
class MainContentComponent : public juce::Component,
                             public juce::DragAndDropContainer,
                             private juce::Timer // [1]
{
public:
```

Declare the callback function in the corresponding header file [2]:

```cpp
void timerCallback() override // [2]
{
    undoManager.beginNewTransaction(); // [4]
}
```

Start the timer in the constructor with the desired interval in milliseconds betwen transaction calls [3]:

```cpp
startTimer (500); // [3]
}
```

Finally, we can call the `beginNewTransaction()` function on the [UndoManager](https://docs.juce.com/master/classUndoManager.html "Manages a list of undo/redo commands.") in the timer callback [4]:

```cpp
void timerCallback() override // [2]
{
    undoManager.beginNewTransaction(); // [4]
}
```

> [!NOTE]
> Exercise: Implement transactions every five undo/redo actions instead of using a timer to separate groups of actions.

<!-- -->

> [!TIP]
>The source code for this modified version of the code can be found in the `UndoManagerTutorial_02.h` file of the demo project.

# Summary

By completing this tutorial, you have learnt how to restore previous states of your application. In particular, we have:

- Stored [UndoableAction](https://docs.juce.com/master/classUndoableAction.html "Used by the UndoManager class to store an action which can be done and undone.") objects into [UndoManager](https://docs.juce.com/master/classUndoManager.html "Manages a list of undo/redo commands.") objects.
- Passed an [UndoManager](https://docs.juce.com/master/classUndoManager.html "Manages a list of undo/redo commands.") instance as a parameter to [ValueTree](https://docs.juce.com/master/classValueTree.html "A powerful tree structure that can be used to hold free-form data, and which can handle its own undo ...") access functions.
- Handled groups of undo/redo actions as transactions.

# See also

- [Tutorial: The ValueTree class](/tutorials/tutorial_value_tree/)
- [Tutorial: Saving and loading your plug-in state](/tutorials/tutorial_audio_processor_value_tree_state/)
- [Tutorial: Listeners and Broadcasters](/tutorials/tutorial_listeners_and_broadcasters/)
