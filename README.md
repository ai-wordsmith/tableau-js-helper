# Tableau Helper
This library helps to abstract some of the Tableau JS API functionality in
the hopes of making it easier to work with embedded Tableau visualizations.

## Requirements
None

## Installation
Presently, this is written as a standalone JS lib so you'll need to clone
the repo or download the file to include in a project.

## Guide
### Instantiation
Instantiation of the Tableau Helper object is performed by passing a Tableau
Viz object:

```javascript
let viz = new tableau.Viz(containerDiv, url, options);
let tableauHelper = new TableauHelper(viz);
```

### Working with a Viz Worksheet
Tableau visualizations are broken into worksheets that correspond to the
worksheets used to build the visualization in Tableau Desktop. Most of the
functionality in this library happens at the worksheet level. The following
are a few common examples.

**List All Worksheets**
```javascript
console.log(tableauHelper.sheets);
```

**Get Selections**

In Tableau, selections are referred to as Marks. Assuming we have a
worksheet named "My Tableau Worksheet" we can get any selected marks
using the following:
```javascript
tableauHelper.getSheet('My Tableau Worksheet').getMarks().then((marks) => {
    console.log(marks);
})
```

**Get Data**

The fundamental concept when integrating Tableau and Wordsmith is the
ability to programmatically grab data from a Tableau visualization. In
Tableau there are two variants of data for any worksheet: full data and
summary data (*NOTE: full data is also referred to as underlying data in
the Tableau JS API*).

Whether you are getting summary or full data, you'll need to pass two
arguments: options and useFormattedValue.

Arguments

*options*: the options argument is used to specify how the Tableau JS API
should pull data. For convenience, this lib includes a default set of
options in the constant `defaultOptions`. This set of default options is
defined as:

```javascript
const defaultOptions = {
    maxRows: 0,
    ignoreAliases: false,
    ignoreSelection: false,
    includeAllColumns: true
}
```

*useFormattedValue*: the Tableau JS API provides values back in a raw and
formatted state (for instance, if you are dealing with a float and the
dashboard user has specified 2 digits, the value returned might be 
"10.5329" whereas the formatted value would be "10.53"). By default, this
library will use the value. For columns where you want the formatted value,
add them to this array.

Examples

Get Summary Data for our worksheet:
```javascript
let options = defaultOptions;
let useFormattedValue = [];
tableauHelper.getSheet('My Tableau Worksheet').getSummaryData(options, useFormattedValue).then((data) => {
    console.log(data);
})
```

Get Full/Underlying Data for our worksheet:
```javascript
let options = defaultOptions;
let useFormattedValue = [];
tableauHelper.getSheet('My Tableau Worksheet').getFullData(options, useFormattedValue).then((data) => {
    console.log(data);
})
```

### Debouncing

For some reason, on certain events, the Tableau Javascript API event listeners
will fire more than one time. In rare cases, they will fire 50+ times on a
single event. This creates an issue as we attempt to generate narrative when
an event listener fires. Included in this library is a debouncing function
that prevents multiple calls of the same function within a given span of time.
The following is an example of how you might use a debounce when listening for
a filter change:

Ignore multiple function calls within 500 milliseconds of each other
```javascript
let viz = new tableau.Viz(containerDiv, url, options);
viz.addEventListener(tableau.TableauEventName.FILTER_CHANGE, debounce((filterEvent) => {
    // code to execute when a filter is changed
}, 500))
```

## Full Documentation

### Constants

| Name | Return Type | Description |
|------|-------------|-------------|
| defaultOptions | Object | Provides a set of default options to be used with `getSummaryData()` and `getFullData()` methods. |

### Functions

| Name | Return Type | Description |
|------|-------------|-------------|
| debounce(function: function, wait: integer, immediate: boolean) | Function | Returns a `function` that, as long as it continues to be invoked, will not be triggered. The function will be called after it stops being called for `wait` milliseconds. Optionally, `immediate` can be passed to trigger the function on the leading edge, instead of the trailing.

### TableauHelper

#### Properties

| Name | Return Type | Description |
|------|-------------|-------------|
| viz | TableauViz | The Tableau Viz object passed to instantiate `TableauHelper`. |
| sheets | String[] | Names of worksheet objects. |

#### Methods

| Name | Return Type | Description |
|------|-------------|-------------|
| getSheet(sheetName: String) | Sheet | Gets the Sheet object idenified by `sheetName`. |
| getParams() | Promise<Param[]> | Returns parameters for the workbook. |

### Sheet

#### Properties

| Name | Return Type | Description |
|------|-------------|-------------|
| sheetObj | TableauSheet | The Sheet object as defined by the Tableau Javascript API. |
| name | String | The name of the worksheet. |
| type | String | The type of the worksheet. |

#### Methods

| Name | Return Type | Description |
|------|-------------|-------------|
| getMarks() | Promise<TableauMarkPair[]> | Fetches an array of any currently selected marks. |
| setMarks(fieldMap, updateType) | None | Sets marks on the worksheet. The `fieldMap` allows selection based on an object (see notes below) while the updateType corresponds to the SelectionUpdateType enum provided by the Tableau Javascript API. |
| clearMarks() | None | Clears any currently selected marks on the worksheet. |
| getFilters() | Promise<TableauFilter[]> | Gets the current selected value of any filter applicable to the worksheet. |
| getSummaryData(options: Object, useFormattedValue: Array) | Promise<DataObject> | Gets the summary-level data for the worksheet. The returned `DataObject` is a Javascript object, see notes below for more details. The `options` argument is used to set various options relevant to the Tableau JS API. See the [Constants](#Constants) section for additional detail. For a given datapoint, Tableau returns a `value` and `formattedValue`. By default, this library uses the `value` but if you wish to override this, pass an array of column names using the `useFormattedValue` argument. |
| getFullData(options: Object, useFormattedValue: Array) | Promise<DataObject> | Gets the full underlying data for the worksheet. See `getSummaryData()` documentation for details on method arguments. |

#### Notes

*fieldMap*

The `setMarks()` method takes a `fieldMap` argument that allows you to specify which fields and values to select. This argument should be formatted as an object using the following general form:

```javascript
{
    "Field1": value,
    "Field2": [1, 2, 3]
}
```

*DataObject*

The `getSummaryData()` and `getFullData()` methods return a custom `DataObject`. This `DataObject` consists of columns and data. The columns are `Column` objects (see the [Column](#Column) section for more detail). The data object is a multi-dimensional array where each array element represents a row of data from the dataset. The `DataObject` takes the general form:

```javascript
{
    columns: [
        {
            $impl: {
                $fieldName: "My First Field",
                $dataType: "string",
                $isReferenced: false,
                $index: 0
            }
        },
        {
            $impl: {
                $fieldName: "My Second Field",
                $dataType: "integer",
                $isReferenced: true,
                $index: 1
            }
        }
    ],
    data: [
        ["Foo", 1],
        ["Bar", 2]
    ]
}
```

### Column

The column object is a simple abstraction of the object returned by the Tableau Javascript API's `getColumns()` method.

#### Properties

| Name | Return Type | Description |
|------|-------------|-------------|
| name | String | The name of the column. |
| dataType | String | The data type, as specified by the Tableau Javascript API, for the column. |
| index | Integer | The index where values from this column appear in the dataset. |
| isReferenced | Boolean | Denotes whether data from this column is referenced in the visualization. |

### Param

The param object is an abstraction of the parameter object in the Tableau Javascript API. Params within Tableau can take many forms so this is not a comprehensive abstraction but rather provides an interface to the most common properties.

#### Properties

| Name | Return Type | Description |
|------|-------------|-------------|
| paramObj | TableauParameter | The original parameter object returned by the Tableau Javascript API. |
| dataType | String | The data type, as specified by the Tableau Javascript API, for the parameter. |
| name | String | The name of the parameter |
| value | String | The raw value of the parameter |
| formattedValue | String | The formatted value of the parameter based upon formatting options set in Tableau Desktop. |