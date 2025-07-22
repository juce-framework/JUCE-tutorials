---
title: The TableListBox class
---
# Tutorial: The TableListBox class

Incorporate tables into your JUCE user interfaces. Display data loaded from an XML file and customise the format of your table.

**LEVEL:** Intermediate<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**CLASSES:** [TableListBox](https://docs.juce.com/master/classTableListBox.html "A table of cells, using a TableHeaderComponent as its header."), [TableListBoxModel](https://docs.juce.com/master/classTableListBoxModel.html "One of these is used by a TableListBox as the data model for the table's contents."), [ListBox](https://docs.juce.com/master/classListBox.html "A list of items that can be scrolled vertically."), [ListBoxModel](https://docs.juce.com/master/classListBoxModel.html "A subclass of this is used to drive a ListBox."), [TableHeaderComponent](https://docs.juce.com/master/classTableHeaderComponent.html "A component that displays a strip of column headings for a table, and allows these to be resized,..."), [XmlDocument](https://docs.juce.com/master/classXmlDocument.html "Parses a text-based XML document and creates an XmlElement object from it."), [XmlElement](https://docs.juce.com/master/classXmlElement.html "Used to build a tree of elements representing an XML document.")

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/TableListBoxTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/TableListBoxTutorial.zip) . Unzip the project and open the first header file in the Projucer.

> [!WARNING]
> If using the PIP version of this project, please make sure to copy the `Resources` folder into the generated Projucer project.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project displays a table loaded from an XML file containing information about JUCE modules. The table can be sorted according to the column of your choice, certain entries can be edited and columns can be hidden.

![](/_images/tutorial_table_list_box_screenshot1.png "The app window")

# The ListBox class

The base [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") class that allows us to display tables in JUCE is called the [ListBox](https://docs.juce.com/master/classListBox.html "A list of items that can be scrolled vertically.") class. The [ListBox](https://docs.juce.com/master/classListBox.html "A list of items that can be scrolled vertically.") behaviour is managed by a [ListBoxModel](https://docs.juce.com/master/classListBoxModel.html "A subclass of this is used to drive a ListBox.") class which describes the data model to display. This is useful to display a list of items in a scrollable viewport but in order to turn this into a proper table with headers describing the columns, we can use respectively the [TableListBox](https://docs.juce.com/master/classTableListBox.html "A table of cells, using a TableHeaderComponent as its header.") and [TableListBoxModel](https://docs.juce.com/master/classTableListBoxModel.html "One of these is used by a TableListBox as the data model for the table's contents.") classes. These classes encapsulate the same behaviour as their counterparts but instead incorporate the functionalities of a [TableHeaderComponent](https://docs.juce.com/master/classTableHeaderComponent.html "A component that displays a strip of column headings for a table, and allows these to be resized,...") to display column headers.

When implementing a [TableListBox](https://docs.juce.com/master/classTableListBox.html "A table of cells, using a TableHeaderComponent as its header.") and inheriting the [TableListBoxModel](https://docs.juce.com/master/classTableListBoxModel.html "One of these is used by a TableListBox as the data model for the table's contents."), there are several functions to override namely:

- `getNumRows()` : Needs to return the current number of rows in the table.
- `paintRowBackground()` : Using the [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") context provided, we must draw the background of the row specified by the row number.
- `paintCell()` : Using the [Graphics](https://docs.juce.com/master/classGraphics.html "A graphics context, used for drawing a component or image.") context provided, we must draw the cell specified by the row and column number.
- `refreshComponentForCell()` : We can optionally override this method to create and update custom components in the table.
- `getColumnAutoSizeWidth()` : When using auto size for the width of the columns, this method can optionally specify how the column resizes itself.
- `sortOrderChanged()` : If using a custom sorting order, we can optionally specify how the column gets reordered.

There are other functions that you can optionally override for additional functionality but in this tutorial we will implement the ones presented here.

# Reading Data from XML

Let's start by loading the data that we want to present in the table from an XML document.

In the `Resources` folder of the project, you can find sample data to use with this tutorial in a file called `TableData.xml` that contains information in the following format:

```cpp
<TABLE_DATA>
<HEADERS>
<COLUMN columnId="1" name="ID" width="50"/>
//...
</HEADERS>
<DATA>
<ITEM ID="01" Module="juce_module" Name="JUCE example classes" Version="5.2.0" License="ISC" Groups="2" Dependencies="1" Description="..." Select="0"/>
//...
</DATA>
</TABLE_DATA>
```

Here we encapsulate the whole file with the `TABLE_DATA` tag and the table headers and the actual data is separated using the `HEADERS` and `DATA` tags respectively. The columns are defined with individual `COLUMN` tags and the rows are defined with `ITEM` tags using attributes for the content in each column of that entry.

The implementation of the code in this project is tailored for this file structure but you can modify the XML tags as you wish.

In the `TableTutorialComponent` class, we define a pointer to temporarily store the content of the file in a single [XmlElement](https://docs.juce.com/master/classXmlElement.html "Used to build a tree of elements representing an XML document.") member variable and also define two additional XmlElements for the column content and the row content as shown here:

```cpp
class TableTutorialComponent : public juce::Component,
                               public juce::TableListBoxModel
{
```

```cpp
std::unique_ptr<juce::XmlElement> tutorialData;
juce::XmlElement* columnList = nullptr;
juce::XmlElement* dataList = nullptr;
```

In the class constructor, we launch a [`FileChooser`](https://docs.juce.com/master/classFileChooser.html "Creates a dialog box to choose a file or directory to load or save.") to select the data file to display. When the [`FileChooser`](https://docs.juce.com/master/classFileChooser.html "Creates a dialog box to choose a file or directory to load or save.") completes, it will call the `callback` lambda function, to load the file content into our [`XmlElement`](https://docs.juce.com/master/classXmlElement.html "Used to build a tree of elements representing an XML document.") objects [1]. With our data loaded from the XML file, we can iterate through the column headers using a for() loop and [`XmlElement::getChildIterator`](https://docs.juce.com/master/classXmlElement.html#ac620244d67b05fb572a37bc5f01f6d0d "Allows iterating the children of an XmlElement using range-for syntax.") to assign the table headers using the `addColumn()` function [2] like so:

```cpp
TableTutorialComponent()
{
    const auto callback = [this] (const juce::FileChooser& chooser) {
loadData (chooser.getResult()); // [1]
```

```cpp
if (columnList != nullptr)
{
    for (auto* columnXml : columnList->getChildIterator())
    {
        table.getHeader().addColumn (columnXml->getStringAttribute ("name"), // [2]
            columnXml->getIntAttribute ("columnId"),
            columnXml->getIntAttribute ("width"),
            50,
            400,
            juce::TableHeaderComponent::defaultFlags);
    }
}
```

This function specifies the name, width and ID of the column along with property flags that define sortability and resizability of columns.

In the `loadData()` helper function called in the constructor, we first find the `Resources` directory and the XML file to load and parse this [File](https://docs.juce.com/master/classFile.html "Represents a local file or directory.") object using the `parse()` function [3]. We can then retrieve the rows and columns from the temporary [XmlElement](https://docs.juce.com/master/classXmlElement.html "Used to build a tree of elements representing an XML document.") by traversing the XML structure and finding the corresponding tags with the `getChildByName()` functions [4]. This is a good place to set the number of rows from the data [XmlElement](https://docs.juce.com/master/classXmlElement.html "Used to build a tree of elements representing an XML document.") by calling `getNumChildElements()` on it [5] as follows:

```cpp
void loadData (juce::File tableFile)
{
    if (tableFile == juce::File() || !tableFile.exists())
        return;

    tutorialData = juce::XmlDocument::parse (tableFile); // [3]

    dataList = tutorialData->getChildByName ("DATA");
    columnList = tutorialData->getChildByName ("HEADERS"); // [4]

    numRows = dataList->getNumChildElements(); // [5]
}
```

> [!WARNING]
> Make sure to select the \"TableData.xml\" file provided in the resources for this tutorial.

Let's also define a helper function called `getAttributeNameForColumnId()` that returns the name of the column based on its ID which will become handy later on:

```cpp
juce::String getAttributeNameForColumnId (const int columnId) const
{
    for (auto* columnXml : columnList->getChildIterator())
    {
        if (columnXml->getIntAttribute ("columnId") == columnId)
            return columnXml->getStringAttribute ("name");
    }

    return {};
}
```

Here we iterate through the child XML elements and find the matching column ID in order to return its name attribute.

# Custom Cell Components

A [TableListBox](https://docs.juce.com/master/classTableListBox.html "A table of cells, using a TableHeaderComponent as its header.") can hold custom components in its cells other than just text. In the following sections we explore how to incorporate a [ToggleButton](https://docs.juce.com/master/classToggleButton.html "A button that can be toggled on/off.") into one of the columns and an editable [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") that listens to user input.

## Editable Labels

In the `EditableTextCustomComponent` class, we first inherit from the [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked.") class in order to set it as editable when the user double-clicks on it:

```cpp
class EditableTextCustomComponent : public juce::Label
{
public:
    EditableTextCustomComponent (TableTutorialComponent& td)
        : owner (td)
    {
        setEditable (false, true, false);
    }
```

```cpp
private:
    TableTutorialComponent & owner;
    int row, columnId;
    juce::Colour textColour;
};
```

Here we also keep track in which row and column this object is displayed and a reference to the actual table.

When the user interacts with the [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked."), we have to extend the usual `mouseDown()` functionalities to account for multiple selections in the table. This is achieved by calling `selectRowsBasedOnModifierKeys()` on the table and passing the modifier keys as an argument. Notice here that we still need to call the base class function to keep the original behaviour as well:

```cpp
void mouseDown (const juce::MouseEvent& event) override
{
    owner.table.selectRowsBasedOnModifierKeys (row, event.mods, false);

    Label::mouseDown (event);
}
```

When the user edits the text in the [Label](https://docs.juce.com/master/classLabel.html "A component that displays a text string, and can optionally become a text editor when clicked."), we receive a callback from the `textWasEdited()` function and we call the helper function `setText()` defined later in the `TableTutorialComponent` class to save the changes in the corresponding [XmlElement](https://docs.juce.com/master/classXmlElement.html "Used to build a tree of elements representing an XML document.") object like so:

```cpp
void textWasEdited() override
{
    owner.setText (columnId, row, getText());
}
```

The following function is called by the [`TableListBoxModel`](https://docs.juce.com/master/classTableListBoxModel.html "One of these is used by a TableListBox as the data model for the table's contents.") when creating or updating an `EditableTextCustomComponent` object and sets the row, column and the displayed text from the [XmlElement](https://docs.juce.com/master/classXmlElement.html "Used to build a tree of elements representing an XML document.") using the `getText()` helper function defined after:

```cpp
void setRowAndColumn (const int newRow, const int newColumn)
{
    row = newRow;
    columnId = newColumn;
    setText (owner.getText (columnId, row), juce::dontSendNotification);
}
```

The `getText()` and `setText()` helper functions are defined as follows:

```cpp
juce::String getText (const int columnNumber, const int rowNumber) const
{
    return dataList->getChildElement (rowNumber)->getStringAttribute (getAttributeNameForColumnId (columnNumber));
}
```

Here we find the text from the row and column number in the child element contained within the [XmlElement](https://docs.juce.com/master/classXmlElement.html "Used to build a tree of elements representing an XML document.").

```cpp
void setText (const int columnNumber, const int rowNumber, const juce::String& newText)
{
    const auto& columnName = table.getHeader().getColumnName (columnNumber);
    dataList->getChildElement (rowNumber)->setAttribute (columnName, newText);
}
```

Here we store the text in the child element contained within the [XmlElement](https://docs.juce.com/master/classXmlElement.html "Used to build a tree of elements representing an XML document.") from the row and column number.

## Selectable Button

In the `SelectionColumnCustomComponent` class, we first inherit from the [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") class in order to set a [ToggleButton](https://docs.juce.com/master/classToggleButton.html "A button that can be toggled on/off.") as a child [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") to it and assign the callback function to be called when the user interacts with it:

```cpp
class SelectionColumnCustomComponent : public Component
{
public:
    SelectionColumnCustomComponent (TableTutorialComponent& td)
        : owner (td)
    {
        addAndMakeVisible (toggleButton);

        toggleButton.onClick = [this] { owner.setSelection (row, (int) toggleButton.getToggleState()); };
    }
```

```cpp
private:
    TableTutorialComponent & owner;
    juce::ToggleButton toggleButton;
    int row, columnId;
};
```

Here we also keep track in which row and column this object is displayed and a reference to the actual table. The lambda function calls the `setSelection()` helper function defined later in the `TableTutorialComponent` that sets the toggle state of the button.

```cpp
void resized() override
{
    toggleButton.setBoundsInset (juce::BorderSize<int> (2));
}
```

The `resized()` function sets the bounds of the [ToggleButton](https://docs.juce.com/master/classToggleButton.html "A button that can be toggled on/off.") object.

The following function is called by the [`TableListBoxModel`](https://docs.juce.com/master/classTableListBoxModel.html "One of these is used by a TableListBox as the data model for the table's contents.") when creating or updating a `SelectionColumnCustomComponent` object and sets the row, column and the toggle state from the [XmlElement](https://docs.juce.com/master/classXmlElement.html "Used to build a tree of elements representing an XML document.") using the `getSelection()` helper function defined after:

```cpp
void setRowAndColumn (int newRow, int newColumn)
{
    row = newRow;
    columnId = newColumn;
    toggleButton.setToggleState ((bool) owner.getSelection (row), juce::dontSendNotification);
}
```

The `getSelection()` and `setSelection()` helper functions are defined as follows:

```cpp
int getSelection (const int rowNumber) const
{
    return dataList->getChildElement (rowNumber)->getIntAttribute ("Select");
}
```

Here we find the toggle state from the row and column number in the child element contained within the [XmlElement](https://docs.juce.com/master/classXmlElement.html "Used to build a tree of elements representing an XML document.").

```cpp
void setSelection (const int rowNumber, const int newSelection)
{
    dataList->getChildElement (rowNumber)->setAttribute ("Select", newSelection);
}
```

Here we store the toggle state in the child element contained within the [XmlElement](https://docs.juce.com/master/classXmlElement.html "Used to build a tree of elements representing an XML document.") from the row and column number.

> [!NOTE]
> Exercise: Create additional custom cell components that incorporate [ComboBox](https://docs.juce.com/master/classComboBox.html "A component that lets the user choose from a drop-down list of choices."), [TextButton](https://docs.juce.com/master/classTextButton.html "A button that uses the standard lozenge-shaped background with a line of text on it.") or even [Slider](https://docs.juce.com/master/classSlider.html "A slider control for changing a value.") components.

# Sorting the Data

In order for the table to sort elements based on a chosen column, we have to define a comparator class that can be passed as a template class to be used by the `sortChildElements()` function of the [XmlElement](https://docs.juce.com/master/classXmlElement.html "Used to build a tree of elements representing an XML document.") object.

We name this class `TutorialDataSorter` that keeps track of the name of the [XmlElement](https://docs.juce.com/master/classXmlElement.html "Used to build a tree of elements representing an XML document.") attribute to sort and the ascending or descending direction of the sorting algorithm as follows:

```cpp
class TutorialDataSorter
{
public:
    TutorialDataSorter (const juce::String& attributeToSortBy, bool forwards)
        : attributeToSort (attributeToSortBy),
          direction (forwards ? 1 : -1)
    {
    }
```

```cpp
private:
    juce::String attributeToSort;
    int direction;
};
```

For the `sortChildElements()` function to recognise the `TutorialDataSorter` as a valid comparator, we have to declare a function named `compareElements()` that takes two [XmlElement](https://docs.juce.com/master/classXmlElement.html "Used to build a tree of elements representing an XML document.") objects and returns the order as an int.

The function is required to return:

- a negative value if the first comes before the second.
- a value of 0 if the two objects are equivalent.
- a positive value if the second comes before the first.

```cpp
int compareElements (juce::XmlElement* first, juce::XmlElement* second) const
{
    auto result = first->getStringAttribute (attributeToSort)
                      .compareNatural (second->getStringAttribute (attributeToSort)); // [1]

    if (result == 0)
        result = first->getStringAttribute ("ID")
                     .compareNatural (second->getStringAttribute ("ID")); // [2]

    return direction * result; // [3]
}
```

Therefore, in the above function we compare the string attributes of both XmlElements by using the `compareNatural()` method of the [String](https://docs.juce.com/master/classString.html "The JUCE String class!") class which returns an int with the same set of rules [1]. If the two strings are equivalent for the attribute in question then we compare the ID column of these two elements [2]. Finally we need to invert the result if the direction of sorting is reversed [3].

# Setting the Model

Let's assemble all the pieces together by implementing the [TableListBoxModel](https://docs.juce.com/master/classTableListBoxModel.html "One of these is used by a TableListBox as the data model for the table's contents.").

We first start by inheriting from the [TableListBoxModel](https://docs.juce.com/master/classTableListBoxModel.html "One of these is used by a TableListBox as the data model for the table's contents.") class in the `TableTutorialComponent` class as shown here:

```cpp
class TableTutorialComponent : public juce::Component,
                               public juce::TableListBoxModel
{
```

```cpp
private:
    juce::TableListBox table { {}, this };
    juce::Font font { 14.0f };

    std::unique_ptr<juce::XmlElement> tutorialData;
    juce::XmlElement* columnList = nullptr;
    juce::XmlElement* dataList = nullptr;
    int numRows = 0;
```

Here we define a [TableListBox](https://docs.juce.com/master/classTableListBox.html "A table of cells, using a TableHeaderComponent as its header.") member variable and set this class as its [TableListBoxModel](https://docs.juce.com/master/classTableListBoxModel.html "One of these is used by a TableListBox as the data model for the table's contents.") which means that our model class actually holds the table itself in this scenario. We also keep track of the number of rows in the table as required by the model.

In the class constructor we add the [TableListBox](https://docs.juce.com/master/classTableListBox.html "A table of cells, using a TableHeaderComponent as its header.") as a child [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") [1]. We can also specify properties for the appearance of the table such as the outline colour and its thickness [2].

```cpp
addAndMakeVisible (table); // [1]

table.setColour (juce::ListBox::outlineColourId, juce::Colours::grey); // [2]
table.setOutlineThickness (1);
```

```cpp
table.getHeader().setSortColumnId (1, true); // [3]

table.setMultipleSelectionEnabled (true); // [4]
```

The sorting column and the column visibility is set by calling corresponding functions on the [TableHeaderComponent](https://docs.juce.com/master/classTableHeaderComponent.html "A component that displays a strip of column headings for a table, and allows these to be resized,...") of the table [3] and we also allow multiple selections on the table [4].

The first function to override is the `getNumRows()` function that returns the member variable holding the number of rows. This function is necessary for the model to update the table properly:

```cpp
int getNumRows() override
{
    return numRows;
}
```

The `paintRowBackground()` function is implemented by first finding an alternate colour that complements the default background colour for the table:

```cpp
void paintRowBackground (juce::Graphics& g, int rowNumber, int /*width*/, int /*height*/, bool rowIsSelected) override
{
    auto alternateColour = getLookAndFeel().findColour (juce::ListBox::backgroundColourId).interpolatedWith (getLookAndFeel().findColour (juce::ListBox::textColourId), 0.03f);
    if (rowIsSelected)
        g.fillAll (juce::Colours::lightblue);
    else if (rowNumber % 2)
        g.fillAll (alternateColour);
}
```

The row is then filled in light blue if the row is selected, otherwise we paint every other row with this alternate colour.

In order to fill the cells with content we override the `paintCell()` function as follows:

```cpp
void paintCell (juce::Graphics& g, int rowNumber, int columnId, int width, int height, bool rowIsSelected) override
{
    g.setColour (rowIsSelected ? juce::Colours::darkblue : getLookAndFeel().findColour (juce::ListBox::textColourId)); // [5]
    g.setFont (font);

    if (auto* rowElement = dataList->getChildElement (rowNumber))
    {
        auto text = rowElement->getStringAttribute (getAttributeNameForColumnId (columnId));

        g.drawText (text, 2, 0, width - 4, height, juce::Justification::centredLeft, true); // [6]
    }

    g.setColour (getLookAndFeel().findColour (juce::ListBox::backgroundColourId));
    g.fillRect (width - 1, 0, 1, height); // [7]
}
```

- [5]: First we select an appropriate colour for the text depending on whether the row is selected or not and set the font size.
- [6]: If the child row element exists in the [XmlElement](https://docs.juce.com/master/classXmlElement.html "Used to build a tree of elements representing an XML document.") object, we retrieve the right column from the row and fill the cell with corresponding text from the [XmlElement](https://docs.juce.com/master/classXmlElement.html "Used to build a tree of elements representing an XML document.").
- [7]: Finally, we draw the separation line on the right side of the cell with the default background colour.

```cpp
void sortOrderChanged (int newSortColumnId, bool isForwards) override
{
    if (newSortColumnId != 0)
    {
        TutorialDataSorter sorter (getAttributeNameForColumnId (newSortColumnId), isForwards);
        dataList->sortChildElements (sorter);

        table.updateContent();
    }
}
```

When a sort order change is requested by the user, the `sortOrderChanged()` callback function is called and if the sort column is valid, we instantiate a `TutorialDataSorter` object with the correct attribute and direction. We can then pass that object to the `sortChildElements()` function of the [XmlElement](https://docs.juce.com/master/classXmlElement.html "Used to build a tree of elements representing an XML document.") and force a table refresh by calling `updateContent()` on the table.

The `refreshComponentForCell()` function is where the custom cell components can be instantiated and updated as follows:

```cpp
Component* refreshComponentForCell (int rowNumber, int columnId, bool /*isRowSelected*/, Component* existingComponentToUpdate) override
{
    if (columnId == 9) // [8]
    {
        auto* selectionBox = static_cast<SelectionColumnCustomComponent*> (existingComponentToUpdate);

        if (selectionBox == nullptr)
            selectionBox = new SelectionColumnCustomComponent (*this);

        selectionBox->setRowAndColumn (rowNumber, columnId);
        return selectionBox;
    }

    if (columnId == 8) // [9]
    {
        auto* textLabel = static_cast<EditableTextCustomComponent*> (existingComponentToUpdate);

        if (textLabel == nullptr)
            textLabel = new EditableTextCustomComponent (*this);

        textLabel->setRowAndColumn (rowNumber, columnId);
        return textLabel;
    }

    jassert (existingComponentToUpdate == nullptr);
    return nullptr; // [10]
}
```

- [8]: If the function is called on the correct \"Select\" column for the selection cell, then we check whether the `SelectionColumnCustomComponent` already exists in the cell. If not we instantiate a new instance and update its content by calling the `setRowAndColumn()` function on it and return the [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.").
- [9]: If the function is called on the correct \"Description\" column for the text editor cell, then we check whether the `EditableTextCustomComponent` already exists in the cell. If not we instantiate a new instance and update its content by calling the `setRowAndColumn()` function on it and return the [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.").
- [10]: Otherwise, it means that the function was called on a regular column and the custom [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") object for that cell should be non-existant.

Finally, the [TableListBox](https://docs.juce.com/master/classTableListBox.html "A table of cells, using a TableHeaderComponent as its header.") offers a nice feature where the columns can be resized according to an automatic behaviour defined hereafter in the `getColumnAutoSizeWidth()` function:

```cpp
int getColumnAutoSizeWidth (int columnId) override
{
    if (columnId == 9)
        return 50;

    int widest = 32;

    for (auto i = getNumRows(); --i >= 0;)
    {
        if (auto* rowElement = dataList->getChildElement (i))
        {
            auto text = rowElement->getStringAttribute (getAttributeNameForColumnId (columnId));

            widest = juce::jmax (widest, font.getStringWidth (text));
        }
    }

    return widest + 8;
}
```

Here we decide to inspect all the elements in a certain column and retrieve the widest text in a cell. We then return the width with some added padding or a fixed width if the column happens to be the \"Select\" column with a custom [ToggleButton](https://docs.juce.com/master/classToggleButton.html "A button that can be toggled on/off.").

> [!NOTE]
> Exercise: Modify the content of the XML document by adding additional columns and/or data and changing the implementation accordingly.

# Summary

In this tutorial, we have learnt how to display information in a table. In particular, we have:

- Loaded data into a table using XML.
- Incorporated custom components into our table cells.
- Sorted the table data according to a custom sorting behaviour.

# See also

- [Tutorial: The Label class](/tutorials/tutorial_label/)
- [Tutorial: The ComboBox class](/tutorials/tutorial_combo_box/)
- [Tutorial: The Slider class](/tutorials/tutorial_slider_values/)
- [Tutorial: Radio buttons and checkboxes](/tutorials/tutorial_radio_buttons_checkboxes/)
