# excel4node

An OOXML (xlsx) generator that supports formatting options.

This is a fork of [natergj/excel4node](https://github.com/natergj/excel4node) with lots of cleanup (and tests).


## Installation

    npm install excel4node


## Usage Example

```javascript
var xl = require('excel4node');

var wb = new xl.WorkBook();
var ws = wb.WorkSheet('My Worksheet');

var myCell = ws.Cell(1, 1);
myCell.String('Test Value');

wb.write('MyExcel.xlsx');
```


## Sample

A sample.js script is provided in the code. Running this will output a sample excel workbook named Excel.xlsx

    node sample.js
    open Excel.xlsx


## Workbook

A Workbook represents an Excel document.

```javascript
var xl = require('excel4node');
var wb2 = new xl.WorkBook({      // optional params object
    jszip: {
        compression: 'DEFLATE'   // change the zip compression method
    },
    fileSharing: {               // equates to "password to modify option"
        password: 'Password',    // This does not encrypt the workbook,
        userName: 'John Doe'     // and users can still open the workbook as read-only.
    },
    allowInterrupt: false        // do not asynchrously forEach loops
});
```

`allowInterrupt` uses an asynchronous forEach loop within code as to not block other operations if reports are being generated on the same thread as other processes that should take higher priority.


## Worksheet

A Worksheet represents a tab within an excel document.

```javascript
var ws = wb.WorkSheet('My Worksheet', {
    margins: {                         // page margins
        left: .75,
        right: .75,
        top: 1.0,
        bottom: 1.0,
        footer: .5,
        header: .5
    },
    printOptions: {                    // page print options
        centerHorizontal: true,
        centerVertical: false
    },
    view: {                            // page zoom
        zoom: 100
    },
    outline: {
        summaryBelow: true
    },
    fitToPage: {
        fitToHeight: 100,
        orientation: 'landscape',
    },
    sheetProtection: {                 // same as "Protect Sheet" in Review tab of Excel
        autoFilter: false,
        deleteColumns: false,
        deleteRow : false,
        formatCells: false,
        formatColumns: false,
        formatRows: false,
        insertColumns: false,
        insertHyperlinks: false,
        insertRows: false,
        objects: false,
        password: 'Password',
        pivotTables: false,
        scenarios: false,
        sheet: true,
        sort: false
    }
});
```

The `sheetProtection` options are the same as the "Protect Sheet" functions in the Review tab of Excel to prevent certain user editing.  Setting a value to true means that that particular function is protected and the user will not be able to do that thing. All options are false by default except for 'sheet' which defaults to true if the sheetProtection attribute is set in the worksheet options, but false if it is not.


### Worksheet Validations

Optionally, you can set validations for a WorkSheet.

```javascript
ws.setValidation({
    type: 'list',
    allowBlank: 1,
    showInputMessage: 1,
    showErrorMessage: 1,
    sqref: 'X2:X10',
    formulas: [
        'value1,value2'
    ]
});

ws.setValidation({
    type: 'list',
    allowBlank: 1,
    sqref: 'B2:B10',
    formulas: [
        '=sheet2!$A$1:$A$2'
    ]
});
```

--------------------------------------------------------------------------------

## Rows & Columns

Set dimensions of rows or columns:

```javascript
ws.Row(1).Height(30);
ws.Column(1).Width(100);
```


Freeze rows and columns:

```javascript
ws.Column(3).Freeze();   // freeze the first two columns (everything prior to the specified column)
ws.Column(3).Freeze(8);  // freeze the first two columns and scroll to the 8th column
ws.Row(3).Freeze();      // freeze the first two rows (everything prior to the specified row)
ws.Row(3).Freeze(8);     // freeze the first two rows and scroll to the 8th row.
```

See also "Series with frozen Row" tab in sample output workbook.


Set a row to be a filter row:

```javascript
ws.Row(1).Filter();    // no arguments passed will add filter to any populated columns
ws.Row(1).Filter(1,8); // optional start and end columns
```

See also "Departmental Spending Report" tab in sample output workbook.


Hide a specific oow or column:

```javascript
ws.Row(2).Hide();
ws.Column(2).Hide();
```


Set groupings on rows and optionally collapse them:

```
ws.Row(rowNum).Group(level,isCollapsed)
ws.Row(1).Group(1,true)
```

See also "Groupings Summary Top" and "Groupings Summary Bottom" in sample output.


--------------------------------------------------------------------------------

## Cells

Represents a cell within a worksheet.

Cell can take 6 data types: `String`, `Number`, `Formula`, `Date`, `Link`, and `Bool`.

Cell takes two arguments: row, col.

Add a cell to a WorkSheet with some data:

```javascript
ws.Cell(1, 1).String('My String');
ws.Cell(2, 1).Number(5);
ws.Cell(2, 2).Number(10);
ws.Cell(2, 3).Formula('A2+B2');
ws.Cell(2, 4).Formula('A2/C2');
ws.Cell(2, 5).Date(new Date());
ws.Cell(2, 6).Link('http://google.com');
ws.Cell(2, 6).Link('http://google.com', 'Link name');
ws.Cell(2, 7).Bool(true);
```


## Styles

Style objects can be applied to cells:

```javascript
var myStyle = wb.Style();
myStyle.Font.Bold();
myStyle.Font.Italics();
myStyle.Font.Underline();
myStyle.Font.Family('Times New Roman');
myStyle.Font.Color('FF0000');
myStyle.Font.Size(16);
myStyle.Font.Alignment.Vertical('top');
myStyle.Font.Alignment.Horizontal('left');
myStyle.Font.Alignment.Rotation('90');
myStyle.Font.WrapText(true);

var myStyle2 = wb.Style();
myStyle2.Font.Size(14);
myStyle2.Number.Format('$#,##0.00;($#,##0.00);-');

var myStyle3 = wb.Style();
myStyle3.Font.Size(14);
myStyle3.Number.Format('##%');
myStyle3.Fill.Pattern('solid');
mystyle3.Fill.Color('CCCCCC');
myStyle3.Border({
    top: {
        style:'thin',
        color:'CCCCCC'
    },
    bottom: {
        style:'thick'
    },
    left: {
        style:'thin'
    },
    right: {
        style:'thin'
    }
});

ws.Cell(1, 1).Style(myStyle);
ws.Cell(1, 2).String('My 2nd String').Style(myStyle);
ws.Cell(2, 1).Style(myStyle2);
ws.Cell(2, 2).Style(myStyle2);
ws.Cell(2, 3).Style(myStyle2);
ws.Cell(2, 4).Style(myStyle3);
```

Available styles:

- `Font.Bold()` bolds text
- `Font.Italics()` italicizes text
- `Font.Underline()` underlines text
- `Font.Family('Arial')` name of font family
- `Font.Color('DDEEFF')` hex rgb font color
- `Font.Size(12)` font size in Pts
- `Font.WrapText()` set text wrapping
- `Font.Alignment.Vertical('top')` options are `top`, `center`, `bottom`
- `Font.Alignment.Horizontal('left')` options are `left`, `center`, `right`
- `Font.Alignment.Rotation(15)` degrees to rotate
- `Number.Format('style')` number style string
- `Fill.Color('DDEEFF')` background color in rgb
- `Fill.Pattern('solid')` pattern style `solid`, `lightUp`, etc.
- `Border({...})` border styles (see below)

Border Styles:

Takes one argument: object defining border. Each ordinal (top, right, etc) are only required if you want to define a border. If omitted, no border will be added to that side.  Style is required if oridinal is defined. If color is omitted, it will default to black.

 ```javascript
myStyle3.Border({
    top: {
        style: 'thin',
        color: 'CCCCCC'
    },
    right: {
        style: 'thin',
        color: 'CCCCCC'
    },
    bottom: {
        style: 'thin',
        color: 'CCCCCC'
    },
    left: {
        style: 'thin',
        color: 'CCCCCC'
    },
    diagonal: {
        style: 'thin',
        color: 'CCCCCC'
    }
});
```


Apply formatting directly to a cell (similar syntax to creating styles):

```javascript
ws.Cell(1, 1).Format.Font.Color('FF0000');
ws.Cell(1, 1).Format.Fill.Pattern('solid');
ws.Cell(1, 1).Format.Fill.Color('AEAEAE');
```

Merge cells and apply styles or mormats to ranges:

`ws.Cell(row1, col1, row2, col2, merge)`

```javascript
ws.Cell(1, 1, 2, 5, true).String('Merged Cells');
ws.Cell(3, 1, 4, 5).String('Each Cell in Range Contains this String');
ws.Cell(3, 1, 4, 5).Style(myStyle);
ws.Cell(1, 1, 2, 5).Format.Font.Family('Arial');
```

--------------------------------------------------------------------------------

## Images

Images can be inserted into a worksheet.

`img.Position(row, col, [rowOffset], [colOffset])`

```javascript
var imgPath = './my-image.jpg'; // relative path from node script
var img1 = ws.Image(imgPath);
img1.Position(1,1);

Set image position directly:

```javascript
var img2 = ws.Image(imgPath2).Position(
    3,       // row to anchor top left corner of image
    3,       // col to anchor top left corner of image
    1000000, // offset from top of row in EMUs
    2000000  // offset from left of col in EMUs
);
```

Currently images should be saved at a resolution of 96dpi.

--------------------------------------------------------------------------------

## Writing Output

Write the Workbook to local file synchronously or
Write the Workbook to local file asynchrously or
Send file via node response

```javascript
wb.write('My Excel File.xlsx'); // write synchronously

wb.write('My Excel File.xlsx', function (err) {
    // done writing
});

wb.write('My Excel File.xlsx', res); // write to http response
```


## Notes

- [MS-XSLX spec (pdf)](http://download.microsoft.com/download/D/3/3/D334A189-E51B-47FF-B0E8-C0479AFB0E3C/[MS-XLSX].pdf)
