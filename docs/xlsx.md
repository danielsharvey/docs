`xlsx` recipe generates excel files from plain [Office Open XML SpreadsheetML File Format](http://msdn.microsoft.com/en-us/library/dd922181%28v=office.12%29.aspx). The source xml is assembled using templating engines and helpers provided itself. See the example in playground:

<iframe src='https://playground.jsreport.net/studio/workspace/rJftqRaQ/10?embed=1' width="100%" height="400" frameborder="0"></iframe>

##Examples

- [Add row](https://playground.jsreport.net/studio/workspace/r1vaurbw/3)
- [Using xlsxMerge to rename sheet](https://playground.jsreport.net/studio/workspace/BJa5OBWD/2)
- [Conditional formatting](https://playground.jsreport.net/studio/workspace/H1BHqBZw/9)
- [Merged cells](https://playground.jsreport.net/studio/workspace/rkX89bHD/2)
- [Add sheet](https://playground.jsreport.net/studio/workspace/SyL6aErP/2)
- [Pivot table tutorial](http://jsreport.net/learn/dynamic-excel-pivot-table)
- [Chart](https://playground.jsreport.net/studio/workspace/rJftqRaQ/10)

## General concept

Excel files are defined by several xml files zipped into one with xlsx extension. This recipe employs templating engine to directly modify these source files and produces excel files. 

The excel source xml follows up quite verbose and complicated [Office Open XML SpreadsheetML File Format](http://msdn.microsoft.com/en-us/library/dd922181%28v=office.12%29.aspx) specification and it would be difficult to create one from the scratch. That's why the recipe rather replaces or adds just particular pieces to particular places. 

The modification of source XML is used by several predefined helpers the recipe provides. There is for example helper `xlsxAdd` which adds piece of xml into particular file and its particular path. The following code is the most simple example using handlebars for creating excel with one row and cell.

```html
{{#xlsxAdd "xl/worksheets/sheet1.xml" "worksheet.sheetData[0].row"}}
    <row>
        <c t="inlineStr"><is><t>Hello world</t></is></c>
    </row>
{{/xlsxAdd}}

{{{xlsxPrint}}}
```

The parameter `xl/worksheets/sheet1.xml` is the physical path to the file representing the sheet. You can always verify the path by unzipping the xlsx file and observing its content.  The parameter `worksheet.sheetData[0].row` is then javascript based path into the particular xml. We use here javascript instead of XPath because we anyway keep in memory javascript representation of xml because it is more practical. Finally the value inside the `xlsxAdd` helper is xml which is added.

At the end of the template, there always must be called helper `xlsxPrint` which prints the internal json representation into xml which is by recipe zipped into xlsx.

Using other provided helper you should be able to manipulate the source xml into every desired shape. 

Unfortunately [Office Open XML SpreadsheetML File Format](http://msdn.microsoft.com/en-us/library/dd922181%28v=office.12%29.aspx) is not very well documented and quite hard to handle. When you are lost, it is always good idea to create a test xlsx file in excel, unzip it and analyze its content. Another approach we recommend is to create the whole excel the first, upload it into jsreport and then only replace its data, which is simple.

## Predefined helpers

### xlsxPrint
The must be the call to `xlsxPrint` at the end of every template.

### xlsxReplace
```html
{{#xlsxReplace filePath xmlPath}}...{{/xlsxReplace}}
```
Replace the whole xml in `filePath` and `xmlPath` with the xml produced by the block helper

### xlsxMerge
```html
{{#xlsxMerge filePath xmlPath}}...{{/xlsxMerge}}
```
Merges the xml in `filePath` at `xmlPath` with content generated by block helper.

### xlsxAdd
```html
{{#xlsxAdd filePath xmlPath}}...{{/xlsxAdd }}
```
Add xml provided by block helper into `filePath` and collection found at `xmlPath`. 


### xlsxRemove

```html
{{#xlsxRemove filePath  xmlPath}}...{{/xlsxRemove }}
```
Remove element from collection in `filePath` at `xmlPath`. 

### xlsxAddImage

```html
{{#xlsxAddImage "test" "sheet1.xml" 0 0 1 1}}
{#image myImage @encoding=base64}
{{/xlsxAddImage}}
```

Add an base64 encoded image provided by the block helper content into the sheet cell. Arguments are `imageName`, `sheet id`,  `column from`, `row from`, `column to`, `row to`

### custom
You can always write your custom helpers. The best is to get started by checking the [source of the standard ones](https://github.com/jsreport/jsreport-xlsx/blob/master/static/helpers.js)

## Preview in studio
The excel preview in the studio uses Microsoft Excel Online service which requires a public access to the displayed file. This is why the recipe by default uploads the excel file to our publicly running server. This is happening only during the preview and you should not be leaking any production data through here. However you can disable or change this behavior in the configuration.

```js
...
"xlsx": {
  "previewInExcelOnline": false,
  "publicUriForPreview": "http://mypublicSever"
}
```

The first options `previewInExcelOnline` will force the recipe to always return a file stream to the studio instead of the excel online preview.  The second option `publicUriForPreview` allows you to set url to the public server which should be used instead of jsreport.net. Such a server need to accept POST with the file stream, return a string id of the file which needs to be accessible afterwards with GET.

