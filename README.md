

# vue-advanced-datatable
Bootstrap advanced dynamic datatable for Vue
## Features
 - Sorting
 - Searching
 - Totals row
 - Show/Hide Columns
 - Table customization stored in localstorage
 - Activity Indicator
 - Conditional Formatting
 - Advanced filtering via modal (requires [bootstrap-vue](https://github.com/bootstrap-vue/bootstrap-vue))
 - Exportable (requires [SheetJS](https://github.com/sheetjs/sheetjs))

## Properties
| Name | Type | Default |  Description |
|--|--|--|--|
| data | Array | - | The data to display in the table |
| table-class | String | 'table table-hover table-vcenter border' | List of CSS classes to add to the table HTML element |
| thead-class | String | NULL | List of CSS classes to add to the thead HTML element |
| identifier | String | NULL | Unique table identifier when storing column config in localstorage |
| sticky-header | Boolean | false | Setting to true allows the table to be scrollable with the headings staying at the top |
| conditional-formatting | Array | [] | See below for conditional formatting schema |
| bordered | Boolean | false | Adds a border around the full datatable element |
| totals | Boolean | false | Shows the total of any number column at the bottom of the table |
| report-meta | Array | [] | SQL Server metadata for retrieved columns, see below section on format |


## Conditional Formatting
The `conditional-formatting` property requires an array of objects in the following format:
| Key | Description | Value |
|--|--|--|
| column | The key of the column you are targeting for styling | String |
| condition | Operator to use for comparison | `equal`, `not_equal`, `less`, `less_or_equal`, `greater`, `greater_or_equal`, `is_null`, `is_not_null` | 
| value | Value to compare against, compare with another column by wrapping the name in square brackets, e.g. [Column2] | String |
| transformation | The transformation to apply to the row/column | `text-colour`, `cell-colour`, `row-colour`, `bold`, `underline`, `italics`, `strikethrough` |
| colour | The colour to apply if using a colour transformation, otherwise ignored. | Any valid CSS colour |

## Column Metadata
This tool was made to display data retrieved from [tediousjs](https://github.com/tediousjs/tedious) so the 'report-metadata' property expects an array in the following format:

```json
[
    {
        "colName": "Column Name",
        "type": {
            "type": "tediousjs_type"
        }
    }
]
```
There are 4 internal field types: **String**, **Number**, **Boolean** & **Date**, below is a list of conversions from tediousjs to these internal field types:
```json
{
    "INT1": "Number",
    "INTN": "Number",
    "BIT": "Boolean",
    "INT2": "Number",
    "INT4": "Number",
    "DATETIM4": "Date",
    "FLT4": "Number",
    "FLTN": "Number",
    "MONEY": "Number",
    "DATETIME": "Date",
    "DATETIMN": "Date",
    "FLT8": "Number",
    "DECIMAL": "Number",
    "NUMERIC": "Number",
    "MONEY4": "Number",
    "INT8": "String",
    "IMAGE": "String",
    "TEXT": "String",
    "GUIDN": "String",
    "NTEXT": "String",
    "BIGVARBIN": "String",
    "BIGVARCHR": "String",
    "BIGBinary": "String",
    "BIGCHAR": "String",
    "NVARCHAR": "String",
    "NCHAR": "String",
    "XML": "String",
    "TIMEN": "Date",
    "DATEN": "Date",
    "DATETIME2N": "Date",
    "DATETIMEOFFSETN": "Date",
    "UDTTYPE": "String",
    "SSVARIANTTYPE": "String"
}
```
## License
```
Copyright 2021 Connor Howell

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

## To-Do

 - [ ] Add better field definition types, currently only works with results from node-tedious
 - [ ] Configurable aggregations on footer row
 - [ ] Configurable aggregations on footer row
 - [ ] Remove bootstrap-vue dependancy
 - [ ] Manually resize columns
 - [ ] Configurable theme, bootstrap & tailwind
 - [ ] Update for Vue 3 support
