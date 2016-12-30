# Processing *.svg files in a node.js environment
This document shall summarise my findings on how to process *.svg files in a node.js environment.
Regarding the *.svg library, the focus will be on paper.js. Most of the findings, however, should be applicable to other libraries, too.

## General situation

There are two fundamental options for generating *.svgs on the server-side:
* *Option A*: Move the browser to the server (e.g. using PhantomJS)
* *Option B*: Do NOT move an actual browser to the server but actually draw in node.js (using JSDOM)

### Option A: Move the browser to the server (e.g. using PhantomJS)
* PhantomJS is a WebKit implementation that can be controlled with JavaScript.
* Roughly speaking, it is an actual — yet headless — browser, meaning that Web pages are never rendered.
* (This is based on: https://www.smashingmagazine.com/2014/05/love-generating-svg-javascript-move-to-server/)
* Advantages:
  * ...
* Disadvantages:
  * see some here: https://github.com/tmpvar/jsdom#jsdom-vs-phantomjs

### Option B: Do NOT move an actual browser to the server but actually draw in node.js (using JSDOM)
* This option appears to be the superior option (e.g. https://github.com/tmpvar/jsdom#jsdom-vs-phantomjs)
* *.svg files are XML files
* XML files make use of the DOM = Document Object Model
* The XML DOM is (http://www.w3schools.com/xml/dom_intro.asp) :
  * A standard object model for XML
  * A standard programming interface for XML
  * Platform- and language-independent

Now, the problem is that the JavaScript SVG libraries extensively use the DOM API to create an SVG document, append nodes to it and manipulate it in different ways. 
Node.js, however, - as opposed to the browser - lacks this kind of API.

The solution for this problem is JSDOM.
JSDOM (https://github.com/tmpvar/jsdom) is a JavaScript implementation of the DOM that can be used with Node.js.


## Technical requirements

* Node.js
* Paper.js (latest development build): http://paperjs.org/download/#development-builds
  * Paper.js depends on the Cairo Graphics library (https://www.cairographics.org/)

## Importing, processing and exporting an *.svg in paper.js
What we want to do is to import an *.svg file, modify it (e.g. by adding a rectangle) and then to export the modified *.svg again.

### General problems you will encounter when trying to import *.svgs

#### Asynchronous node.js
Node.js is asynchronous, that is the code is generally executed concurrently rather than consecutively. Because loading *.svgs takes relatively longer, you need to make sure that the *.svg is actually loaded and available before you start using it.

If you load the *.svg but do not process it in the callback function, then the processing will be excecuted while the *.svg has not been loaded yet.
The following code would allow for the _correct_ behaviour:

```javascript
project.importSVG('./svg-import/115784.svg', function(item) {
    
    // This is the callback function of the importSVG()
    
    // Do something to or with the imported *.svg
    // ...here...

    // Export the *.svg again
    var paper_svg = project.exportSVG({ asString: true });

    // Write the exported *.svg into a file on the system
    fs.writeFile(path.resolve("./svg-export/export.svg"), paper_svg, function (err) {
        if (err) throw err;
        console.log('*.svg exported and saved as /svg-export/export.svg!');
    });
});
```

#### Cascading of styles (problem when no fill is defined for the imported *.svg)

"It turns out that your SVG data doesn't define a fill but expects the default fill to cascade through. Unfortunately this doesn't seem to work on Node.js.
I believe it's because styles don't cascade for SVG elements in JSDOM yet. I'd recommend _explicitly_ setting styles in your SVG data for now."

* TODO: How do I set styles in the *.svg? Can I add the style using code?
* Add e.g. this in the SVG XML: fill="#000000"

#### Paper.js does not feature 'futures' yet to manage synchronous workflow

* (I still have to fully understand this...)

### Current related issues

* See my thread here: https://github.com/paperjs/paper.js/issues/1220#event-906925682
* Paper.js issue: SVGImport does not correctly cascade styles in Node.js
  * https://github.com/paperjs/paper.js/issues/1229
* JSDOM issue: Cascading of inline styles
  * https://github.com/tmpvar/jsdom/issues/1696

## Digital hugs
* Thanks to Jürg Lehni (paper.js) for providing the amazing paper.js library and assisting in solving issues. Please consider donating to his project here: http://paperjs.org/donation/

## Sources
* https://www.smashingmagazine.com/2014/05/love-generating-svg-javascript-move-to-server/
* https://github.com/tmpvar/jsdom#jsdom-vs-phantomjs
* https://github.com/paperjs/paper.js
