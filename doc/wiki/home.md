## jsPlumb

jsPlumb provides a means for a developer to visually connect elements on their web pages. It uses SVG in modern browsers, and VML on IE 8 and below. 

The latest version is 1.7.7. From 1.7.0, jsPlumb has no longer supported YUI or Mootools - if you want to use some external library to manage dragging, then jQuery is your only option.  But it is recommended that you use vanilla jsPlumb wherever possible: it's faster than the jQuery version, it offers more features (multiple element dragging, multiple scopes), and since the [event handling](https://github.com/jsplumb/mottle) and [drag/drop](https://github.com/jsplumb/katavorio) code are also projects managed by jsPlumb, you're more likely to get a feature request for requirements related to those sorts of things to happen. Plus, it is highly likely that support for the jQuery flavour will be dropped at some point. There is no date yet planned for this, but it seems likely.

This will be the final version (apart from other bugfix releases in 1.7.x) to support IE8. The next major release will be 2.0.0, and this will work only in modern browsers that support SVG. 

## Imports and Setup

It is a good idea to read this entire page.  There are a few things you need to know about integrating jsPlumb with your UI.

  1. [Browser Compatibility](#browser)
  - [Setup](#setup)
  - [Doctype](#doctype)
  - [Required Imports](#imports)
  - [Initializing jsPlumb](#initializing)
  - [Multiple jsPlumb Instances](#multiple)
  - [Method Arguments](#arguments)
  - [Element Ids](#ids)
  - [Z-Index Considerations](#zindex)
  - [Where does jsPlumb add elements?](#container)
  - [Dragging](#dragging)
  - [Performance](#performance)

<a name="browser"></a>
### Browser Compatibility
jsPlumb runs on everything from IE6 up.  There are some caveats, though, because of various browser/library bugs:
                
  - jQuery 1.6.x and 1.7.x have a bug in their SVG implementation for IE9 that causes hover events to not get fired.
  - jQuery 2.0 does not support IE 6,7 or 8.
  - Safari 5.1 has an SVG bug that prevents mouse events from being passed through the transparent area of an SVG element (Safari 6.x does not seem to have the same problem)                

<a name="setup"></a>
### Setup

<a name="doctype"></a>
### Doctype
###### text/html
If you're serving your pages with content type `text/html`, as most people are, then you should use this doctype:

```html
<!doctype html>
```
    
This gives you Standards mode in all browsers and access to HTML5.

###### application/xhtml+xml
If you're serving `application/xhtml+xml` then you need to include the VML namespace in the `html` element:

```xml
<?xml version="1.0" encoding="UTF-8"?>
  <html xmlns='http://www.w3.org/1999/xhtml' xmlns:v="urn:schemas-microsoft-com:vml">
```
    
Don't forget that in IE<9, XHTML is not supported.  

<a name="imports"></a>
### Required Imports

##### Vanilla
No required imports. 

```html
<script src="PATH_TO/dom.jsPlumb-1.7.7-min.js "></script>
```

###### Touch events
Touch events are supported out of the box.

##### jQuery
- jQuery 1.3.x or higher. jsPlumb has been tested with everything up to 2.x. Remember that jQuery 2.x does not support IE<9.
- jQuery UI 1.7.x or higher (if you wish to support drag and drop). Always remember to check that the versions of jQuery and jQuery UI you are using are compatible.
  
```html
<script src="//ajax.googleapis.com/ajax/libs/jquery/1.9.0/jquery.min.js"></script>
<script src="//ajax.googleapis.com/ajax/libs/jqueryui/1.9.2/jquery-ui.min.js"></script>
<script src="PATH_TO/jquery.jsPlumb-1.7.7-min.js "></script>
```

###### Touch Events
There is a neat plugin you can use for this - [jQuery UI Touch Punch](http://touchpunch.furf.com/).  You need to include it in your page after both jQuery and jQueryUI. It does not, however, work with the `makeSource` functionality in jsPlumb. Better to use vanilla jsPlumb!
			
<a name="initializing"></a>
### Initializing jsPlumb

You should not start making calls to jsPlumb until the DOM has been initialized - perhaps no surprises there.  To handle this, you should bind to the `ready` event on jsPlumb (or the instance of jsPlumb you are working with):			

```javascript
jsPlumb.bind("ready", function() {
  ...  			
  // your jsPlumb related init code goes here
  ...  
});
```


There's a helper method that can save you a few precious characters:

```javascript
jsPlumb.ready(function() {
    ...			
    // your jsPlumb related init code goes here
    ...
});
```

If you bind to the `ready` event after jsPlumb has already been initialized, your callback will be executed immediately.

<a name="multiple"></a>
### Multiple jsPlumb instances
jsPlumb is registered on the browser's window by default, providing one static instance for the whole page to use.  You can also instantiate independent instances of jsPlumb, using the `getInstance` method, for example:

```javascript
var firstInstance = jsPlumb.getInstance();
```

The variable `firstInstance` can now be treated exactly as you would treat the `jsPlumb` variable - you can set defaults, call the `connect` method, whatever:

```javascript
firstInstance.importDefaults({
  Connector : [ "Bezier", { curviness: 150 } ],
  Anchors : [ "TopCenter", "BottomCenter" ]
});
  
firstInstance.connect({
  source:"element1", 
  target:"element2", 
  scope:"someScope" 
});
```

`getInstance` optionally takes an object that provides the defaults:

```javascript
var secondInstance = jsPlumb.getInstance({
  PaintStyle:{ 
    lineWidth:6, 
    strokeStyle:"#567567", 
    outlineColor:"black", 
    outlineWidth:1 
  },
  Connector:[ "Bezier", { curviness: 30 } ],
  Endpoint:[ "Dot", { radius:5 } ],
  EndpointStyle : { fillStyle: "#567567"  },
  Anchor : [ 0.5, 0.5, 1, 1 ]
});
    
secondInstance.connect({ 
  source:"element4", 
  target:"element3", 
  scope:"someScope"   
});
```

It is recommended to use separate instances of jsPlumb wherever possible.

<a name="ids"></a>
### Element Ids
jsPlumb uses the `id` attribute of any element with which it interacts. If `id` is not set, jsPlumb will create an id for the element. It is recommended that you set appropriate ids for the elements in your UI yourself.

#### Changing Element Id
Because of the fact that jsPlumb uses element ids, you need to tell jsPlumb if an element id changes. There are two methods to help you do this:

- `jsPlumb.setId(el, newId)` Use this if you want jsPlumb to take care of changing the id in the DOM. It will do so, and then update its references accordingly.
                     
- `jsPlumb.setIdChanged(oldId, newId)`  Use this if you have already changed the element's ID, and you just want jsPlumb to update its references.

<a name="arguments"></a>
### Method Arguments
Almost every method in jsPlumb that operates on elements supports multiple formats for specifying the element(s) on which to operate.

#### Selectors/NodeLists
In jQuery there is the concept of a "selector" - a list of elements conforming to some CSS path spec, for instance `$(".myClass")`. Both the jQuery flavour, and also vanilla, jsPlumb support these selectors as arguments for elements (vanilla jsPlumb can do this because jQuery's selector object is _list-like_, ie. it has a `length` property).

When using vanilla jsPlumb, you can use a `NodeList` - it is, effectively, the native equivalent of the selectors we just discussed.  There are several ways of getting a `NodeList` from the DOM; perhaps the most useful (supported in IE8+) is `document.querySelectorAll("some selector")`.

You will see **selector** referred to in many places in the jsPlumb documentation. Just think of it as a list of elements that match some CSS spec.

#### Element Ids
Passing a single string as argument will cause jsPlumb to treat that string as an element id.

#### Elements
You can pass DOM elements as arguments.  This will probably not surprise you.

#### Arrays
You can also pass an array of any or all of the types we just listed. The contents of the array can be mixed - they do not have to be all one type.


<a name="zindex"></a>
### Z-index Considerations
You need to pay attention to the z-indices of the various parts of your UI when using jsPlumb, in particular to ensure that the elements that jsPlumb adds to the DOM do not overlay other parts of the interface. 

jsPlumb adds an element to the DOM for each Endpoint, Connector and Overlay. So for a Connection having visible Endpoints at each end and a label in the middle, jsPlumb adds four elements to the DOM.  The actual elements it adds depend on the renderer in use (SVG/VML).			

To help you organise z-indices correctly, jsPlumb adds a CSS class to each type of element it adds. They are as follows:		
		
<table style="color:black;font-size:90%;width:100%;">
  <tr><td><strong>Component</strong></td><td><strong>Class</strong></td></tr>
  <tr><td>Endpoint</td><td>_jsPlumb_endpoint</td></tr>
  <tr><td>Connector</td><td>_jsPlumb_connector</td></tr>
  <tr><td>Overlay</td><td>_jsPlumb_overlay</td></tr>
</table>            
                            
In addition, whenever the mouse is hovering over an Endpoint or Connection, that component is assigned the class `_jsPlumb_hover`. For more information about styling jsPlumb with CSS, see [this page](styling-via-css). 

<a name="container"></a>
### Where does jsPlumb add elements?  	
It's important to understand where in the DOM jsPlumb will add any elements it creates. If you want a TL;DR version, then it boils down to this:

- It is strongly recommended that you set a Container before you begin plumbing.
- A Container is some element that jsPlumb will use as the parent for all of the artefacts it adds to the UI. Typically, you would make this the parent element of the nodes you are connecting.
- If you do not specify a Container, jsPlumb will infer that the Container should be the `offsetParent` of the first element on which you call `addEndpoint`, `makeSource` or `makeTarget`, or the `offsetParent` of the source element in the first `connect` call - whichever happens first.

Let's discuss this in more detail.
		
Early versions of jsPlumb added everything to the **body** element. This has the advantage of being the most flexible arrangement in terms of supporting which elements can be connected, but in certain use cases produced unexpected results. 

Consider the arrangement where you have some connected elements in a tab: you would expect jsPlumb to add elements inside the tab, so that when the user switches tabs and the current one is hidden, all the jsPlumb stuff is hidden too.  But when the elements are on the body, this does not happen!   

It is also quite common for jsPlumb to be used in a page in which the diagram is contained within some element that should show scrollbars when its content overflows. Appending elements to the document body prevents this from occurring automatically.

#### Container

You can - and **should** - instruct jsPlumb to use some element as the parent of everything jsPlumb adds to the UI through usage of the `setContainer` method in jsPlumb, or by providing the Container in the parameters to a `jsPlumb.getInstance` call.

**Important** This is a change from versions of jsPlumb prior to 1.6.2. In versions prior to 1.6.2 you could assign the Container directly via the `jsPlumb.Defaults.Container` property. You can still do this - we're in Javascript here of course, it's a free-for-all - but it will be ignored.

**Also Important** If you happen to be using jsPlumb's `draggable` method to make other parts of your UI draggable (ie. not just things you're plumbing together), be careful not to call `draggable` on the element that is acting as the `Container` for the current instance, or you will see some odd situations occur when dragging.  It is suggested that the best thing to do if you wish to use jsPlumb to enable dragging on non-plumbed elements is to create a new instance:

```javascript
var nonPlumbing = jsPlumb.getInstance();
nonPlumbing.draggable("some element");
```

##### Some examples
		
- Set a Container to use as the default Container, using a jQuery selector, and then add an Endpoint.  The related UI artefact will be a child of the document body:

```javascript
jsPlumb.setContainer($("body"));
...
jsPlumb.addEndpoint(someDiv, { endpoint options });
```

- Set a container to use as the default container, using Plain Old DOM:

```javascript
jsPlumb.setContainer(document.getElementById("foo"));
...
jsPlumb.addEndpoint(someDiv, { endpoint options });
```
		
- Set a container to use as the default container, using an element id, and then connect two elements.  The elements created in this example will be children of the element with id "containerId":

```javascript
jsPlumb.setContainer("containerId");
...
jsPlumb.connect({ source:someDiv, target:someOtherDiv });
```

- Get a new instance of jsPlumb and specify the container to use in the constructor params:

```javascript
var j = jsPlumb.getInstance({
  Container:"foo"
});
...
jsPlumb.addEndpoint(someDiv, { endpoint options });
```

##### Container CSS
The container you choose should have `position:relative` set on it, because it is the origin of that element that jsPlumb will use to compute the placement of the artefacts it adds to the DOM, and jsPlumb uses absolute positioning.

<a name="dragging"></a>
### Element Dragging
A common feature of interfaces using jsPlumb is that the elements are draggable.  You should use `jsPlumbInstance.draggable` to configure this:

```javascript
myInstanceOfJsPlumb.draggable("elementId");
```

...because if you don't, jsPlumb won't know that an element has been dragged, and it won't repaint the element.

Dragging is covered in detail [on this page](dragging).

<a name="performance"></a>
### Performance
The speed at which jsPlumb executes, and the practical limit to the number of manageable connections, is greatly affected by the browser upon which it is being run.  At the time of writing, it will probably not surprise you to learn that jsPlumb runs fastest in Chrome, followed by Safari/Firefox, and then IE browsers in descending version number order.

#### Suspending Drawing

Every `connect` or `addEndpoint` call in jsPlumb ordinarily causes a repaint of the associated element, which for many cases is what you want.  But if you are performing some kind of "bulk" operation - like loading data on page load perhaps - it is recommended that you suspend drawing before doing so:

```javascript
jsPlumb.setSuspendDrawing(true);
...
- load up all your data here -
...
jsPlumb.setSuspendDrawing(false, true);
```

Notice the second argument in the last call to `setSuspendDrawing`: it instructs jsPlumb to immediately perform a full repaint (by calling, internally, `repaintEverything`).

I said above it is recommended that you do this.  It really is.  It makes an enormous difference to the load time when you're dealing with a lot of Connections on slower browsers.

###### batch
This function abstracts out the pattern of suspending drawing, doing something, and then re-enabling drawing:

```javascript
jsPlumb.batch(fn, [doNotRepaintAfterwards]);
```

Example usage:

```javascript
jsPlumb.batch(function() {
  // import here
  for (var i = 0, j = connections.length; i < j; i++) {
      jsPlumb.connect(connections[i]);
  }
});
```

Here, we've assumed that `connections` is an array of objects that would be suitable to pass to the `connect` method, for example:

```javascript
{ source:"someElement", target:"someOtherElement" }
```
    
By default, this will run a repaint at the end. But you can suppress that, should you want to:

```javascript
jsPlumb.batch(function() {
  // import here
}, true);
```
    
*Note* this method used to be called `doWhileSuspended` and was renamed in version 1.7.3.

#### Anchor Types
Continuous anchors are the type that require the most maths, because they have to calculate their position every time a paint cycle occurs. 

Dynamic and Perimeter anchors are the next slowest (with Perimeter being slower than most Dynamic Anchors as they are actually Dynamic Anchors that have, by default, 60 locations to choose from).  

Static anchors like `Top`, `Bottom` etc are the fastest.

