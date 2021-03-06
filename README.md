<p align="center"><img width="460" src="https://raw.githubusercontent.com/fdebef/Gridact/master/img/GridAct.png"></p>

# Gridact #
## Your Excellent React Grid Component ##

Gridact is React component for displaying large datasets in table supporting

- filtering
- sorting
- paging
    - custom page lengths with initial set
- custom row classes based on row data
- individual columns definitions
    - column name
    - sortable, editable,
    - hidden, width, table head class
    - custom cell rendering (based on cell and row data)
    - custom cell class rendering based on row data
- inline editing with server side processing
    - edit cells
    - add and remove rows
    - custom allowed values and characters
    - error messages

I have done my best to optimize rendering, so the work is for user very good with rendering displayed table
with 2000 cells (100 rows with 20 columns) in ~200 ms or 10000 cells in ~800 ms. (Change page to page render time.) 
Render speed is not very much affected with total size of data, but with data displayed. For user experience
I recommend not to allow page longer then 100 rows with 20 column table.  

Gridact component is result of my experience working with Microsoft Excel as an user and
[jQuery DataTables](https://datatables.net/) as developer. Datatables taught me to love data 
in JavaScript and that's a big thanks to _Allan Jardine_. This motivated me to create Gridact.

**This is my first public React Component, I'll be really glad for any feedback and I'll do my best 
to satisfy any of your requests. Thanks.**

# Installation #
Nothing but easy.<br>
`npm i gridact`<br>
Prerequsities:<br>
```
react
react-dom
mdi-react
bootstrap
```
These modules must be installed in parent component. Bootstrap must be imported in whichever parent component.

# Basic configuration #
The basic configuration is pretty simple. Just import Gridact module and pass data and some configuration:
```
import Gridact from 'gridact'
import React from 'react';

const App = () => {
  const [data, setData] = useState();  
  useEffect(() => {
    yourApi().then(yourData => {
    setData(yourData)}
  )}, [])

  <Gridact    
    data={data}    
    colDefs={tableColsDef}
    wrapperDivClass={['table-responsive', 'col-xl-12']}
    tableClasses={['table', 'table-sm', 'table-striped', 'text-nowrap', 'table-bordered']}
    pagingOptions={[5, 10, 20, 50, 100]} 
    fnRowClass={(row) => {
      if (row.name_original) {
        if (row.name_original.includes('X')) return 'font-weight-bold text-danger';
        return undefined;
      }
      return undefined;
    }
    }
    serverSideEdit={editData => YourApi('yourUri', editData)
    }
    primaryKey="line_id"
    showFilter
    addRemove
    pageSelector
    pagingSelector
    searchPlaceHolder="Search..."
    onEnterMoveDown={false}
  />
      
```

## Gridact props ##
### data ###
Type: _Array of Objects_ **\<mandatory\>**<br>
```
[{col1: val, col2: val,...}, {col1: val, col2: val,...},...] 
```

### wrapperDivClass ###
Type: _Array, String_ _\<option>_<br>
Props is joined to final classNames `classNames=wrapperDivClass.join(' ')`<br>
Final table is wrapped in div - main reason was enabling responsiveness in bootstraps, what requires table's outer div.
For table responsiveness, use at least 'table-responsive col'

### tableClasses ###
Type: _Array | String_ _\<option>_<br>
Props is joined to final classNames `classNames=wrapperDivClass.join(' ')`<br>
Best use with boostrap's table classes. E.g. ```['table', 'table-sm', 'table-striped', 'text-nowrap', 'table-bordered']```

### pagingOptions ###
Type: _Array_ _\<option>_<br>
Page length options. Will always be sorted numerically, first value will be initials.<br> 
Example: `[20,10,50,100]` - paging options will be `[10,20,50,100]` - initial set will be `[20]`.
If undefined, default paging is `[10, 20, 50]`.

### fnRowClass ###
Type: _String | Array | function_ _\<option>_<br>
String or Array of classNames<br>
If function is provided, it is passed row data. Must return String or Array.
Used e.g. for coloring whole row based on row data.

### primaryKey ###
Type: String **\<mandatory\>**<br>
Key from colDefs, where data primary key is stored. Is necessary for proper function of component.
primaryKey must be included in colDefs, but you can hide it (hidden property)

### serverSideEdit ###
AcceptedValues: _function_ **\<mandatory\> if editable columns** <br>
Function is provided with Object according to selected operation.<br> 

#### Objects passed to function are based on operation  ###
**edit**<br>
Cell edit<br>
Passed argument: `{operation: 'edit', data: {primary_key: val, column_name: new_value}}`<br>
Required response: Promise resolved with `{data: updatedRow}` <br>
If you want to validate data server-side, you can response with `{error: errorMsg}`, where `errorMsg` is string,
which will be displayd in small overlay next to edited cell. <br>
If `error` is in response, data will not be updated in table, previous value will be restored.   

**new**<br>
Add new row<br>
Passed argument: `{operation: 'new'}`<br>
Required response: Promise resolved with `{data: newRow}` In newRow, primary_key must not be `null`.  

**delete**<br>
Delete row, where is focused cell.<br>
Passed argument: `{operation: 'delete', data: {primary_key: val}}`<br>
Required response: `{data: 1}`<br>
If value of `data` key is < 1, row will not be deleted in table.

### showFilter, addRemove, pageSelector, pagingSelector ###
Type: _Boolean \<option>_<br>
Shows/hide filter (search field), add remove row buttons, page selector (what page is displayed), 
paging selector (page length) 

### searchPlaceHolder ###
Type: _String_ _\<option>_<br>
Default: "Search..."

### onEnterMoveDown  ###
Type: Boolean<br>
If `true`, on `enter` cursor moves down. otherwise moves right.

## colDefs ## 
*Columns definitions*<br>
Type: _Object_ **\<mandatory>**<br>

colDefs is object, where keys are column codes, values are column definitions itsels in Object.

```
{line_id: {
    cellRender: v => v,
    name: 'ID',
    editable: false,
    // filterEditValue: function | regexp of chars to filter out (newValue.prototype.replace(regexp, '')
    filterEditValue: (newValue, curCellValue, row) => (newValue.replace(/[{]|[}]|[<]|[>]|[\\]|[/]/g, '')),
    // filterEditValue: function which returns true to allow or false for disable char
    // regexp of chars allowed (newValue.prototype.match(regexp))
    filterEditChar: (char, cellValue, row) => char.match(/[a-zA-ZščřžýáíéóúůďťňĎŇŤŠČŘŽÝÁÍÉÚŮ0-9 ]/g),
    sortable: true,
    filterable: true,
    resizable: true,
    width: 80,
    hidden: false
    }
  },
...}
```
### Column definition properties ###
#### cellRender ####
Type: function | string _\<option>_<br>
If undefined, the cell is rendered with value provided in data.
If string is provided, the string is returned.
Function is passed `cell_value` and `row_data`. This means, you can render the cell based on other values in row.<br>
*Example:*
`cellRender: (cellValue, rowData) => if (rowData.volume) > 10 return 'TOO HIGH' else return cellValue` 
  
#### name ###
Type: string **\<mandatory\>**<br>
Name of column in table header
  
### width ###
Type: integer _\<option>_<br>
Set width of current column. Default `<td>` css style is `overflow: hidden`.

### editable, sortable ###
Type: Boolean
Column cells will be editable and sortable.
  
### hidden ###
Type: Boolean _\<option>_<br>
Column will be hidden. Usefull when you want to calculate cell value from multiple columns or style cell based on data
from other columns but the other columns you do not want to show.
  
_As the inline edit is build upon contentEditable, I paid a lot of attention to sanitize input text to avoid XSS. 
There two filter options for values users can write to cell._  
 
### filterEditChar ###
Type: _RegExp | function_ _\<optional\>_<br>
This option is applied directly onKeyPress event - it does not let user write the characters you specify.
If RegExp is provided, it is passed to `character_typed.match()`.
Function is passed three arguments: `(character_typed, cell_value_before_edit, row_data)` Must return `true` or `false`.
True = character is allowed, false = disallowed.
*Default value* is
`(char, CellValue, rowData) => return char.match(/[a-zA-ZščřžýáíéóúůďťňĎŇŤŠČŘŽÝÁÍÉÚŮ0-9 ]/);`
Allowed are only alphanumeric characters included Czech ones.
  
### filterEditValue ###
Type: _RegExp | function_ _\<optional\>_<br>
Because above mentioned option does not block pasting from clipboard, and I didn't want to forbid it generally, 
we have to check the final value user sends to your serverSideEdit function and which will be displayed in table.
As this is potential XSS risc, pay close attention to this function.
Function is passed three arguments: `(new_cell_value, cell_value_before_edit, row_data)` Must return sanitized string.
*Default value* is
`(newValue, cellValue, row) => (newValue.replace(/[{]|[}]|[<]|[>]|[\\]|[/]/g, ''))`<br>
If RegExp is provided, it is passed to `newValue.replace(RegExp, '')`

## Styling
You can find basic stylesheet in src on GitHub, you can simply overwrite it on your own.
For best results I do recommend using Bootstrap ^4.3 and styling table, rows and cells with bootstrap classes.
Currently it is not possible to change icons, surely will be in future version. **mdi-react** must be installed
to icons working properly.
 
# Further development
I'll be glad if you leave me a message. What you miss, what you want, what you like.
What can be added in the future:
  - customizable icons
  - custom placement of Search, add / remove buttons and paging options.
  - better page selector 
  
 
