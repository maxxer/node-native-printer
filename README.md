# node-native-printer

A node module to natively print your files

## Requirements

* ### Windows:

   * **[edge](https://github.com/tjanczuk/edge)**, [electron-edge](https://github.com/kexplo/electron-edge) (depending on your environment) or any other forks of electorn-edge. Both of them need to be **installed before** this module. During installation it will be prompted to select which package do
      you want to use.<br>**BE CAREFULLY**: it will be listed all package in `node_modules` containing the word "edge"; be sure to make the right choice

* ### Unix (Mac and Linux)

   * **[node-gyp](https://github.com/nodejs/node-gyp)** to build native modules
   * **[cups](https://www.cups.org/)** (Linux only). You need libcups2-dev and libcups2. The first contains source code needed to build native modules, the second contains command-line executables needed in order to print. **It may also need libcups2 in production**. See [API](#api)

## Important notes

Due to important differences in enviroments and ecosystems between Microsoft and Unix-based systems, this module has been written in C++ for Unix and C# for Windows. This can result in different behaviours during the execution of the methods (such as different execution time, returning values, allowed options for printer, etc.).

***
## Installation
```
npm install node-native-printer --save
```

## Usage
First of all you need to require the module inside your app this way:
```javascript
const printer = require('node-native-printer');
```

Then you can set a printer to work on, calling `setPrinter(printerName)`. In this way you don't need to pass else the printer name on each method it's required.

## API

### `listPrinters()`
&nbsp;&nbsp;&nbsp;&nbsp;return an array with all installed printers

### `defaultPrinterName()`
&nbsp;&nbsp;&nbsp;&nbsp;return the default printer name

### `setPrinter(printer)`
&nbsp;&nbsp;&nbsp;&nbsp;set the printer to work on


### `getCurrentPrinter()`
&nbsp;&nbsp;&nbsp;&nbsp;get current printer you are working on

### `printerInfo(printer)`

&nbsp;&nbsp;&nbsp;&nbsp;return general info about current working printer such jobs and options: 

* **printer**: printer of which get informations
* **returning value**:
   * **Windows**: return only jobs in printer queue
   * **Unix**: return jobs in printer queue and CUPS options. Theese last depends on printer

### `printerOptions(printer)`
&nbsp;&nbsp;&nbsp;&nbsp;return printer-specific options:
   * **printer**: name of the printer to print on. If not specified it will print on previously setted printer. If printer is not set it will print on default printer.
   * **Returning value**:
      * **Windows**: return an object containing main options for printer:

      ```json 
      {
          "Collate": "array containing collation options",
          "Duplexing": "array containing collation options",
          "MaxCopy": "max number of copies you can send to printer",
          "SupportsColor": "boolean indicating whether a print can print with colors",
          "PaperSheets": "available paper formats supported from printer. If custom is present it can be submitted custom width and height",
          "Resolutions": "printer resolutions (i.e.: High, Medium)"
      }
      ```

      * **Unix**: return an object containing printer-specific options and from PPD file.

### `print(filename[, options, printer])`
   * **filename**: file to print
   * **options**: a JSON object containing options for printer:
      * **Windows**: default options:
         ```json
         {
             "collate": true,
             "color": true,
             "copies": 1,
             "duplex": "Default",
             "landscape": false,
             "paperSize": "",
             "fromPage": 0,
             "toPage": 0
         }
         ```

         List of supported extensions can be found [here](https://github.com/MatteoMeil/node-native-printer/blob/master/supported-extensions.md)

         **Notes:** duplex is case sensitive, so be careful to write correctly. `"paperSize"` refers to size of sheet to print on; if you want to print on a paper with custom dimensions, pass `"Custom.WidthxHeight"` where Width and Height are **integer** dimensions in hundredths of inch. `"fromPage": 0` means document will be printed from first page; `"toPage": 0` means document will be printed from `"fromPage"` to last page.

      * **Unix**: you can use [command-line options](https://www.cups.org/doc/options.html) in JSON-style and/or printer-specific options retrieved from `printerOptions()`; i.e.:

         ```json      
         {
             "landscape": true,
             "n": 2,
             "sides": "two-sided-long-edge"
         }
         ```

         For options that doesn't have a value (like `landscape` or `fit-to-page`) you can assign a boolean (see above)

         It will be generated and executed a command like `lp -d printerName /path/to/filename -o landscape -n 2 -o sides=two-sided-long-edge`
   * **Returning value (only for Unix)**: job id of work sent to printer

### `printText(text[, options, printer])`

Same as `print()` but you can pass directly a string of text on the first parameter.
