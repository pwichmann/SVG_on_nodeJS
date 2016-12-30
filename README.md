# Processing *.svg files in a node.js environment (here with focus on paper.js)

## General situation

The general problem with processing *.svg in node.js is that the DOM (_which_?) needs to be processed which is usually done by the browser. This leads to some awkward requirements when you try to do the same thing on the server-side.

## Importing, processing and exporting an *.svg in paper.js

### General problems you will encounter when trying to import *.svgs

#### Asynchronous node.js
If you load the *.svg but do not process it in the callback function, then the processing will be excecuted while the *.svg has not been loaded yet.
The following code would allow for the _correct_ behaviour:

```javascript
project.importSVG('./svg-import/115784.svg', function(item) {
    var paper_svg = project.exportSVG({ asString: true });

    // Export the file in the callback function. Not outside this function.
    fs.writeFile(path.resolve("./svg-export/export.svg"), paper_svg, function (err) {
        if (err) throw err;
        console.log('*.svg exported and saved as /svg-export/export.svg!');
    });
});
```

#### Cascading of styles

It turns out that your SVG data doesn't define a fill but expects the default fill to cascade through. Unfortunately this doesn't seem to work on Node.js.
I believe it's because styles don't cascade for SVG elements in JSDOM yet. I'd recommend _explicitly_ setting styles in your SVG data for now.

#### Paper.js does not feature 'futures' yet to manage synchronous workflow

### Current related issues

* See my thread here: https://github.com/paperjs/paper.js/issues/1220#event-906925682
* Paper.js issue: SVGImport does not correctly cascade styles in Node.js
  * https://github.com/paperjs/paper.js/issues/1229
* JSDOM issue: Cascading of inline styles
  * https://github.com/tmpvar/jsdom/issues/1696
