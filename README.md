
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

## Conditional Formatting
The `conditional-formatting` property requires an array of objects in the following format:
| Key | Description | Value |
|--|--|--|
| column | The key of the column you are targeting for styling | String |
| condition | Operator to use for comparison | `equal`, `not_equal`, `less`, `less_or_equal`, `greater`, `greater_or_equal`, `is_null`, `is_not_null` | 
| value | Value to compare against, compare with another column by wrapping the name in square brackets, e.g. [Column2] | String |
| transformation | The transformation to apply to the row/column | `text-colour`, `cell-colour`, `row-colour`, `bold`, `underline`, `italics`, `strikethrough` |
| colour | The colour to apply if using a colour transformation, otherwise ignored. | Any valid CSS colour |


## To-Do

 - Add better field definition types, currently only works with results from node-tedious
 - Configurable aggregations on footer row
 - Remove bootstrap-vue dependancy
 - Manually resize columns
 - Configurable theme, bootstrap & tailwind
 - Update for Vue 3 support
