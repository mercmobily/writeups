
// Node: commonJS
// Dojo: AMD (with requireJS)



// Do something when DOM is here
require(['dojo/domReady!'], function(){
});

// A super-simple module: easy.js
define(['dojo/dom'], function(dom){
  return {  something:function(){ /* ...  */ } };
});
/* ...*/
require(['easy.js'], function(easy){
  easy.something();
});


/* ************************************** */
/* DECLARE                                */
/* ************************************** */


// Named classes. NOTE: USE NAMED CLASSES
// ONLY IF WILL BE USED BY DOJO PARSER
declare("mynamespace.MyClass", null, {
    // Custom properties and methods here
});

// Anonymous class
var MyClass = declare(null, {
    // Custom properties and methods here
});

// Anonymous class inheriting. NOTE: first param is not string,
// so it's not the class name
var MySubClass = declare(MyClass, {
});

// Anonymous class inheriting multiple classes
// MyMultiSubClass now has all of the properties and methods from:
// MySubClass (prototype), MyOtherClass, and MyMixinClass (mixins left to right))
var MyMultiSubClass = declare([ MySubClass, MyOtherClass, MyMixinClass ],{
});

// With declare.safeMixin
var MyClass = declare(null, {
  name: 'default name',

  constructor: function(args){
    declare.safeMixin(this,args);
  }
});
// ...
myclass = new MyClass({ name => 'name' });


/* ************************************** */
/* TEMPLATES                              */
/* ************************************** */

require(["dojo/_base/declare", "dijit/_WidgetBase", "dijit/_Templated", "dojo/text!./templates/TonysWidget.html"], 
  function(declare, _WidgetBase, _Templated, template) {

    declare('tonysWidget', [_WidgetBase, _Templated], {
      templateString: template
    });

  }


/* Template:

<div class="${baseClass}" data-dojo-attach-point="focusNode" data-dojo-attach-event="ondijitclick:_onClick" role="menuitem" tabindex="-1">
    <span data-dojo-attach-point="containerNode"></span>
</div>
*/


//templateString    //  a string representing the HTML of the template
//templatePath      //  a URL pointing at a file to be used as a template
//widgetsInTemplate //  a Boolean indicating whether or not child widgets are defined in the template
                    //  NOTE: you _have_ to mixin with _WidgetsInTemplateMixin if you want this to work




/* ************************************** */
/* DOM FUNCTIONS */
// BIBLE: http://www.w3schools.com/dom/default.asp
/* ************************************** */

// Get a node by id
require(["dojo/dom"], function(dom){
  node = dom.byId('button');
  node.innerHTML = "This is the inner HTML";
});

// Dojo attributes
require(["dojo/dom-attr", "dojo/domReady!"], function(domAttr){
  domAttr.set(node, 'type', 'something');
  domAttr.set(node, {type: 'something'} );
  domAttr.set(node, {className: 'something'} ); // Same as "class"
  domAttr.set(node, {innerHTML: 'something'} ); // Will change HTML
  domAttr.set(node, {style: {background:'red', /* ... */ } } ); // Will make string for styles  
});

// Add a class
require(['dojo/dom-class', "dojo/domReady!"], function(domClass){
  domClass.add(node,'red');
});

// Toggle a class
require(['dojo/dom-class', "dojo/domReady!"], function(domClass){
  domClass.toggle(node,'red', true); // Add
  domClass.toggle(node,'red', false); // Take out
});



// Create an element, place it
require(["dojo/dom", "dojo/dom-construct", "dojo/domReady!"], 
  function(dom, domConstruct) {    
    // Create an element
    element = domConstruct.create('h1', { innerHTML:'The text', className: 'the-class'} );
    // Create an element, place it after "parent". Values: 'before', 'after', 'replace', 'only', 'first'
    element = domConstruct.create('h1', { innerHTML:'The text', className: 'the-class'}, parent, 'after' );   
    // Place a widget somewhere in the DOM. "where" can be '<html fragment>', 'some_id' or node
    // Position can be before after replace  only first last (default is "last")

    domConstruct.place(nodeOrId, node, "after");
  }
);

// Destroy and delete elements
require(["dojo/dom", "dojo/dom-construct", "dojo/domReady!"], 
  function(dom, domConstruct) { 
    // Destroy Element
    domConstruct.destroy(element);
    // Empty an element
    domConstruct.empty(element);
  }
);

/* ************************************** */
/* QUERIES */
/* ************************************** */

// Get a node list/query
// HINT: Generates 'query'
// http://dojotoolkit.org/reference-guide/dojo/query.html#standard-css3-selectors
require(["dojo/query", "dojo/domReady!"], function(query){
  
  // Get all elements with id #list
  nodelist = query('#list');
  // Get all elements with class #list
  nodelist = query('.list');    
  // Get all elements of class 'item' in elements of id 'list'
  nodelist = query('#list .item ');      
  // Get all elements of class 'item' within the element 'list'
  nodelist = query('.item',list);      
  //Get all <a> tags of class 'odd'
  nodelist = query('a.odd',list);
  //Retrieve an array of any a element that has an li as its ancestor.
  nodelist = query("li a");
  //Retrieve an array of any a element that has an li as its direct ancestor.
  nodelist = query("li > a");

  // Cycle through the nodes
  list.forEach( function(node,index,nodelist){
    /* ... */
  });

  list.on('click',function(){
    alert("clicked!");
    
  });
  
});

// NodeList-dom: style, addClass, removeClass, toggleClass, place
require(["dojo/query", "dojo/NodeList-dom", "dojo/domReady!"], function(query){ 
  nodelist = query('#list');

  nodelist.empty(); // Remove children
  nodelist.addClass('newClass');
  nodelist.removeClass('oldClass');
  nodelist.addClass('newClass').removeClass('oldClass'); // Chaining
  nodelist.replaceClass('newClass', 'oldClass');
  nodelist.toggleClass('binaryClass');
  nodelist.style({color:'red'});
});

// NodeList-fx: All the fx methods available for queries!
// NOTE: They return an Animate object, rather than a Nodelist, so that you can .play().
// To return a Nodelist, set "auto:true"  
require(["dojo/query", "dojo/NodeList-fx", "dojo/domReady!"], function(query) {
  query("#button").on("click", function(){
      query("li.fresh").slideTo( { left: 200 }) . play();
  });
});
  
// NodeList-data: Add a "data" method
require(["dojo/query", "dojo/NodeList-data", "dojo/domReady!"], function(query, NodeList) {
  var a=10;
  var b=20;
  
  // Assign the data "a" to the key "key"
  query('#button').data('keyOne', a);
  // You can chain calls as usual
  query('#button').data('keyOne', a).data('keyTwo', b);
  // The previous two calls, the fast way
  query('#button').data({ keyOne:a, keyTwo:b});
  
  // Assign something to _all_ elements of list
  query('.list').data('key',a); // EACH element will have a as data

  // Get the value from an ID. Returns: ALWAYS an array
  result = query('#button').data('keyOne')[0];
  
  //Returns an array as big as the number of elemenbts in the previous list
  result = query('.list').data('key'); // This could have several elements 
  
});

// NodeList-data: Going through the DOM
// http://livedocs.dojotoolkit.org/dojo/NodeList-traverse
require(["dojo/query", "dojo/NodeList-traverse", "dojo/NodeList-dom", "dojo/domReady!"], function(query) {
  query("li.yum")             // get LI elements with the class 'yum'
      .addClass("highlight")  // add a 'highlight' class to those LI elements
      .closest(".fruitList")  // find the closest parent elements of those LIs with the class 'fruitList'
      .prev()                 // get the previous sibling (headings in this case) of each of those fruitList elements
      .addClass("happy")      // add a 'happy' class to those headings
      .style({backgroundPosition: "left", paddingLeft: "20px"}); // add some style properties to those headings
});

// NodeList-manipulation: Manilupating elements
// http://livedocs.dojotoolkit.org/dojo/NodeList-manipulate
require(["dojo/query", "dojo/NodeList-manipulate", "dojo/domReady!"], function(query) {
  query(".yum")  // get elements with the class 'yum'
      .clone()        // create a new NodeList containing cloned copies of each element
      .prepend('<span class="emoticon happy"></span>') // inject a span inside each of the cloned elements
      .appendTo("#likes"); // insert the clones into the element with id 'likes'

  query(".yuck")
      .clone()
      .append('<span class="emoticon sad"></span>')
      .appendTo("#dontLikes");
});

/* ************************************** */
/* EVENTS */
/* ************************************** */


/* ************************* */
/* Events basics: Evented    */
/* ************************* */

require(["dojo/_base/declare", "dojo/Evented", "dojo/on"], function(declare, Evented, on) {

  // Define the class
  var eventedClass = declare([Evented], {

    // This doesn't strictly _have to_ be here
    onAction: function(action){
      console.log("Object's onAction() method called with parameter " + action );
    }
  });

  // Create the class
  evented = new eventedClass();

  // Add an observer to the "onAction" method
  evented.on('Action', function(arg){
    console.log("This is likely used by a different object, parameter: " + arg);
  });

  on(evented, 'Action', function(arg){
    console.log("Added with Dojo's on(), parameter: " + arg);
  });

  evented.emit('Action','[Passed parameter]');

});

/* ************************* */
/* Events basics: topic      */
/* ************************* */

require(["dojo/topic"], function(topic) {

  // Built on top of Evented. Once you require "dojo/topic", you have topic.publish and topic.subscribe
  // which emits events for the singleton topic. You can then subscribe to them: 
  topic.publish("something/happened", { something: "other" } ); 
  topic.subscribe("something/happened", function(params){ 
    console.log("Hey, something happened!");
  } ); 


/* ************************* */
/* The REAL story: dojo/on   */
/* ************************* */


/* dojo/on on DOM nodes 
 *
 * It will simply connect the event with the right name with "on" at the beginning. You can EMIT too!
 * WILL USE NATIVE DOM CALLS.
 * Will return an object with a .remove() call to remove (using DOM call)
*/

require(["dojo/dom", "dojo/on", "dojo/dom-construct" ], function(dom, on, domConstruct) {

  // A simple div where the "click" DOM event is listened to
  aDiv = domConstruct.create('div',{ innerHTML: "aDiv" });
  domConstruct.place(aDiv, win.body() );
  on( aDiv, 'click', function(e){
    console.log("aDiv Clicked");
  } );

  // Another div which actually _generates_ a DOM event with on.emit()
  anotherDiv = domConstruct.create('div',{ innerHTML: "anotherDiv" });
  domConstruct.place(anotherDiv, win.body() );
  on( anotherDiv, 'click', function(e){ 
    console.log("anotherDiv Clicked"); 

     on.emit(aDiv,'click', {
       cancelable: true,
       bubbles: false,
       screenX: 33,
       screenY: 44
     });
  });

});


/* dojo/on on Widgets
 *
 * Widgets implement their own .on() method, which actually just uses
 * Dojo's aspects to do the work
 * 
 * The widget uses a _map() function, which means it will only ever
 * work for specific event names
 *
*/

require(["dojo/dom", "dojo/on", "dijit/form/Button" ], function(dom, on, Button) {


  // Creates a button, place it in the <body>
  aButton = new Button({
    label: 'A button',
    onSomething: function(){
      console.log("onSomething was called");
    },

  } );
  aButton.placeAt(win.body(), 'last');

  // This will work as the widget itself will call "onClick" when it's click, through a DOM connection
  // often-times placed straight into the template
  on(aButton, 'click', function(){
    console.log("aButton button Clicked!");
  });

  // This will NOT work as the widget's "on" (which is called since it's defined, and "on" delegates when it's there)
  // will only ever work with _map which is: 
  // blur: "onBlur" change: "onChange" click: "onClick" close: "onClose"  dblclick: "onDblClick" focus: "onFocus" 
  // hide: "onHide" keydown: "onKeyDown" keypress: "onKeyPress" keyup: "onKeyUp" mousedown: "onMouseDown" 
  // mouseenter: "onMouseEnter" mouseleave: "onMouseLeave" mousemove: "onMouseMove" mouseout: "onMouseOut" 
  // mouseover: "onMouseOver" mouseup: "onMouseUp" show: "onShow"
  on(aButton, 'onSomething', function(){
    console.log("This will (not) ALSO happen when calling onSomething");
  });

  // Call the method, see if it works
  aButton.onSomething();


/* dojo/on on Widgets -- passing a FUNCTION rather than an event
 *
 * A typical example is mouse.enter. In this case, on() does *NOTHING*, 
 * and simply delegates the event assigning to the passsed function.
 * (Which will most likely call on() but passing the right string
 * according to the browser)
*/


require(["dojo/dom", "dojo/on", "dijit/form/Button" ], function(dom, on, Button) {
  aButton = new Button({
    label: 'NEW',
    onSomething: function(){
      console.log("onSomething was called");
    },

  } );
  aButton.placeAt(win.body(), 'last');

  // This will basically delegate the whole thing to mouse.enter, which is a function
  // that takes the target node and the listener, and will set "on" according to the browser
  // (This is a gross simplification but it is more or less what happens)
  on(aButton, mouse.enter, function(){
    console.log("NEW button Clicked!");
  });



/* dojo/on -- other notes
 *
*/


// Add an event to a button. NOTE: handler has handler.remove() to remove handlers later!
require(['dojo/on', 'dojo/dom', 'dojo/domReady!'], function(on, dom){
  // Add the event to the button
  button1 = dom.byId('button1');
  handler = on(button1, 'click', function(event){
    alert('Clicked!');
  });
});
 

// Add an event to list of nodeList. NOTE: returns an array of handlers. PLUS, the array returned has
// a handy handlers.remove() method to remove the lot!
require(['dojo/on', 'dojo/query', 'dojo/domReady!'], function(on, query){  
  // Add the event to the nodelist
  handlers = query('#button1').on('click', function(event){
    alert('Clicked!');
  });
});

// Add an event to a button. Mouse events: mouse.enter, mouse.leave
require(['dojo/on', 'dojo/dom', 'dojo/mouse', 'dojo/domReady!'], function(on, dom, mouse){
  // Add the event to the button
  button1 = dom.byId('button1');
  handler = on(button1, mouse.enter, function(event){
    alert('Clicked!');
  });
});

// The second parameter of "on" can be a search!
// (Note: if the event name includes the search, 'dojo/query' NEEDS to be included
require(['dojo/dom', "dojo/on", "dojo/query", "dojo/domReady!"], function(on){
  var buttons = dom.byId("buttons");       // <div id='buttons'>
  on(buttons, ".button:click", function(){ // <button class="button">
    alert("Clicked " + this.id);           // "this" will be the clicked element
  });
         
});


/* ************************************** */
/* EFFECTS */
/* ************************************** */

/* - dojo/_base/fx:  base effects methods found previously in Dojo base: animateProperty, anim, fadeIn, fadeOut
 * - dojo/fx: more advanced effects: chain, combine, wipeIn, wipeOut and slideTo
   Parameters: node, delay, duration, easing (a function), rate, repeat
   Animation: object with play(), pause(), stop(), status(), and gotoPercent()
*/

// fadeIn, fadeOut
function fades(){
  

require(['dojo/_base/fx','dojo/dom', 'domReady!'], function(fx, dom){
  button1 = dom.byId('button1');
  button2 = dom.byId('button2');
  button3 = dom.byId('button3');
  
  var button1 = dom.byId('button1');
  var button2 = dom.byId('button2');
  var button3 = dom.byId('button3');
  var animation; 
  
  on(button1,'click', function(){
    animation = effect = fx.fadeOut( { duration:2000, node: button2} ).play();   
  });
  
  on(button3, 'click', function(){
    animation = fx.fadeIn(  { duration:2000, node: button2} ).play();    
  });
  
});

}

//wipeIn, wipeOut. NOTE: object will _disappear_, contents shifted

  require(['dojo/on','dojo/fx', 'dojo/dom','dojo/domReady!'],function(on, fx, dom){
  
  var button1 = dom.byId('button1');
  var button2 = dom.byId('button2'); // Give "wipe" class for unknown reasons
  var button3 = dom.byId('button3');
  var animation;
  
  on(button1,'click', function(){
    animation = fx.wipeOut( { duration:500, node: button2 } ).play();   
  });
  
  on(button3, 'click', function(){
    animation = fx.wipeIn(  { duration:500, node: button2} ).play();    
  });  
});


// Slide
  
require(['dojo/on','dojo/fx', 'dojo/dom','dojo/domReady!'],function(on, fx, dom){
  
  var button1 = dom.byId('button1');
  var animation;
  
  on(button1,'click', function(){
    animation = fx.slideTo( { node: button1, left: "0", top: "100"  } ).play();   
  });
   
});

// NOTE ABOUT ANIMATIONS: on onEnd, make sure you kill the object so that it doesn't hang around:

...
  this.animation = fx.animateProperty({ 
    node: this.domNode,
    properties: {
      backgroundColor: { end: this.mouseBaseColor}
    },
    onEnd:lang.hitch(this, function(){
      console.log(this);
        this.animation = null;
      }),
    }
  ).play();
  ...


// Animation events.
// AND ALSO:
//   beforeBegin (sync) -- Callback before playing the animation.
//   onBegin (async) -- Callback  immediately after starting the animation.
//   onEnd (sync) -- Callback when the animation ends.
//   onPlay (sync) -- callback when the animation is played.
//   onAnimate -- callback  fired for every step of the animation, passing value from a dojo._Line
// NOTE: FIXME apparently also settable with on(button1, 'End', function(){}) but ONLY 'End' seems to work

  require(['dojo/on','dojo/fx', 'dojo/dom','dojo/domReady!'],function(on, fx, dom){
      
    var button1 = dom.byId('button1');
    var animation;
      
    on(button1,'click', function(){
      animation = fx.slideTo( 
        { node: button1, 
          left: "0", 
          top: "100",
          beforeBegin:function(){ alert("Before Begin");},
          onEnd:function(){ alert("At the end");},
        } ).play();
        on(animation,'End', function(){ alert("At the end, with event handler"); } ); // Only this one works
    });
       
  });


// Chaining and combining. Just chains a list of animations
  require(["dojo/_base/fx", "dojo/fx", "dojo/on", "dojo/dom", "dojo/domReady!"], function(baseFx, fx, on, dom) {
 
   fx.chain([
      baseFx.fadeIn({ node: slideTarget }),
      fx.slideTo({ node: slideTarget, left: "200", top: "200" }),
      baseFx.fadeOut({ node: slideTarget })
     ]).play();
    
   fx.combine([
     baseFx.fadeIn({ node: slideTarget }),
     fx.slideTo({ node: slideTarget, left: "200", top: "200" })
    ]).play(); 
});



/* ************************************** */
/* ANIMATIONS */
/* TODO: After CSS                        */
/* ************************************** */

/* ************************************** */
/* HITCH AND PARTIAL */
/* ************************************** */

// 
require(["dojo/query", "dojo/_base/lang", "dojo/domReady!"], function(query,lang) {
 
        var myObject = {
            foo: "bar",
            myHandler: function(evt, extra){
                //  this is very contrived but will do.
                alert("The value of 'foo' is " + this.foo + "And extra is " + extra);
            }
        };
 
        //  Hitch to switch the context to "My object"
        query("h1").forEach(function(node){
            node.onclick = lang.hitch(myObject, "myHandler", 'extra'); // NOTE: extra will be static
        });
        
        // Partial to bound arguments to a callback
        query("h1").forEach(function(node){
            node.onclick = lang.partial(myObject.myHandler, 'extra'); // NOTE: extra will be static
        });
        
});

/* ************************************** */
/* ANIMATIONS */
/* TODO: later    
 * http://dojotoolkit.org/documentation/tutorials/1.7/using_behavior/                        */
/* ************************************** */




/* ************************************** */
/* AJAX                                   */
/* ************************************** */

require(["dojo/_base/xhr", "dojo/dom", "dojo/domReady!"],
    function(xhr, dom) {
         
        // Using xhr.get, as very little information is being sent
        xhr.post({
         
            handleAs: 'json',
            timeout: 5000,
            content: { param1: 'one', 
                       param2: 'two' },
       
            // Wouldn't do this as well as "content"
            //form: dom.byId("contactForm"),
                       
            // The URL of the request
            url: "url_to_fetch",
            
            // The success callback with result from server
            load: function(newContent) {
                dom.byId("contentNode").innerHTML = newContent.name;
            },
            // The error handler
            error: function() {
                // Do nothing -- keep old content there
            }
        });
         
});


/* ************************************** */
/* WIDGETS                                */
/* ************************************** */

// PARSER. IT JUST CREATES OBJECTS WITH "new" ACCORDING TO data-dojo-type
//
// test.html:
// <span data-dojo-type="CustomWidget" param1="hello">Contents</span>
// <script src="http://ajax.googleapis.com/ajax/libs/dojo/1.7.1/dojo/dojo.js"
//        data-dojo-config="async: true">
//</script>


// VERY VERY minimal way. Just to show that all there is, for the parser, is a function call
require(["dojo/parser"], function(parser) {

  window.CustomWidget = function(params, srcNodeRef){
    console.log(params);
    console.log(srcNodeRef);

    // change srcNodeRef as you like with something really fancy (TODO: do it! Need book)
  }
  
  parser.parse();
});


// DECLARATIVE WAY actually using Dojo's _declare
// NOTE: _setValueAttr and _getValueAttr
// Node:
//  * domNode needs to be set in buildRendering(). After buildRendering, this.domNode replaces srcNode
// *  once you have your widget, you can do widget.placeAt([domString|widget],position) to place them in the DOM!

require(['dojo/dom-construct', 'dojo/parser','dijit/_WidgetBase','dojo/_base/declare','dojo/dom','dojo/on','dojo/query','dojo/_base/xhr','dijit/Dialog','dijit/registry', 'dojo/domReady!'],
function(domConstruct, parser, _WidgetBase, declare, dom, on, query, xhr, Dialog, registry){

    // Proper widget.
    declare('CustomWidget', _WidgetBase, {
      constructor: function(params, srcNodeRef){
        this.customValue = 0;
        console.log(params);
        console.log(srcNodeRef);
      },

      id: 'my_id',

      // This defines domNode.
      // REMEMBER: Whatever is set in domNode will REPLACE srcNodeRef. NEAT!
      buildRendering: function(){

        // create the DOM for this widget
        this.domNode = domConstruct.create("button", {innerHTML: "push me"});
        console.log("Called buildRendering");
      },


      // Custom fields, with getters
      _setCustomValueAttr:function(value){
        this._set("customValue",value); // <--- avoid this.customValue so that "watch" works
      }, 
      _getCustomValueAttr:function(){
        this.customValue;
      },

               
    });
   

  registry.byId('my_id'); // This will return the created widget
  parser.parse();
});



// Widget's lifecycle (common to _WidgetBase)
// 
// --------------------------------------
// * constructor (common to all prototypes, called when instantiated)
// * postscript (common to all prototypes built using declare)
//   * create
//     * postMixInProperties
//     * buildRendering
//         (Anything set as domNode will replace the current HTML)
//     * postCreate (!!!!!) 
//         (AFTER all properties of a widget are defined, and the document fragment with widget is created
//          BEFORE the fragment itself is added to the main document)
// * startup (called by parser)
//     (AFTER any DOM fragments have been actually added to the document
//     To be called by hand for widgets created programmatically
//
// Hint: remember to call this.inherited(arguments) when redifining methods
// ---------------------------------------
// * destroyRecursive
//   * destroyDescendants
//   * destroy
//     * uninitialize
//     * destroyRendering
// ----------------------------------------
// Pre-defined properties:
//
// * id        : a unique string identifying the widget
// * lang      : a rarely-used string that can override the default Dojo locale
// * dir       : useful for bi-directional support
// * class     : the HTML class attribute for the widget's domNode
// * style     : the HTML style attribute for the widget's domNode
// * title     : most commonly, the HTML title attribute for native tooltips
// * baseClass : the root CSS class of the widget
// * srcNodeRef: the original DOM node that existed before it was "widgetified".
//               In some cases, (templated widgets), it's killed by postCreate()
//
//
//
/*
 NOTE ABOUT SETTERS:
    _setXXXAttr can also be a string/hash/array mapping from a widget
     attribute XXX to the widget's DOMNodes:

      - DOM node attribute
       _setFocusAttr: {node: "focusNode", type: "attribute"}
       _setFocusAttr: "focusNode"  (shorthand)
       _setFocusAttr: ""   (shorthand, maps to this.domNode)
      Maps this.focus to this.focusNode.focus, or (last) this.domNode.focus
  
      - DOM node innerHTML
       _setTitleAttr: { node: "titleNode", type: "innerHTML" }
      Maps this.title to this.titleNode.innerHTML
  
      - DOM node innerText
       _setTitleAttr: { node: "titleNode", type: "innerText" }
      Maps this.title to this.titleNode.innerText
  
     - DOM node CSS class
       _setMyClassAttr: { node: "domNode", type: "class" }
      Maps this.myClass to this.domNode.className

ALSO NOTE (IMPORTANT!):
  * Anything passed to the 
FIXME: (thing about setters getting called, and things needing to be in the prototype)



*/


/* ************************************** */
/* TEMPLATES
/* ************************************** */


require(['dojo/parser','dijit/_WidgetBase','dijit/_TemplatedMixin', 'dojo/_base/declare','dojo/domReady!'],
function(parser, _WidgetBase, _TemplatedMixin, declare ){

  // NOTE: Most of the time you'd have:
  // require( ... , "dojo/text!./templates/SomeWidget.html" ...);


  // Let's assume in the HTML:
  // span(data-dojo-type="TemplateWidget", data-dojo-props='baseClass:"base-class", description:"Initial description"', extra='me') Existing text
  // NOTE: data-dojo-props is to keep HTML5 happy

  declare("TemplateWidget", [_WidgetBase, _TemplatedMixin], {

    // The template
    // Note:
    //  - ${baseclass} will be this.baseClass
    //  - data-dojo-attach-point="someNode" will create this->someNode pointing to that tag's DOM
    templateString: '<div class="${baseClass}" data-dojo-attach-point="focusNode" data-dojo-attach-event="onclick:_onClick">'
                  + '  <span data-dojo-attach-point="containerNode"></span>'
                  + '  Dynamic description: <span data-dojo-attach-point="descriptionNode"></span>'
                  + ' </div>',

    // Needs to be "true" if the template itself contains widgets which will then be instanced
    widgetsInTemplate: false,

    baseClass: 'defaultBaseClass',

    constructor:function(params){
      this.inherited(arguments);
      console.log(params);


      a = this; // Very useful for the console

      // Simple, unchanging value. Could come from the template
      // itself, or could be from a parameter passed to new TemplateWidget({'baseClass':'base-class')
      this.baseClass = params.baseClass; 
    },

 
    // Setter for "description", which maps the attach point descriptionNode's innerHTML
    // to the attribute this->description. This means that set('description','something')
    // will change what's in the template
    _setDescriptionAttr: { node: 'descriptionNode', 'type' : 'innerHTML'},

    postCreate: function(){
      // Set 'description' if not set by template
      this.get('description') ? false : this.set('description',"Default description");
    },

    _onClick:function(){
      alert("It was clicked!");
    }

  });

  parser.parse();
 
});


/* ************************************** */
/* LAYOUT, CONTAINERS, PANES, ETC
/* ************************************** */


/* ********************************************************* 

   http://dojotoolkit.org/reference-guide/1.8/quickstart/layoutWidgetHierarchies.html
   http://livedocs.dojotoolkit.org/dijit/layout/BorderContainer
   http://livedocs.dojotoolkit.org/dijit/layout/ContentPane

   * RULE #1: ContentPanes and Containers NEED TO HAVE HEIGHT DEFINED VIA CSS. Always
   * RULE #2: Layout widgets and children need a resize() method. 
              EXCEPT ContentPane children: do not need a resize() method unless they do JS sizing).

   * EXCEPTION #1: A content pane that contains a single layout widget (tabContainer) will resize it
     to its height. So, widgets contained in the layout widget won't need a height

   *********************************************************
*/

/* 

CONTENT:
- ContentPane

CONTAINERS:
- BorderContainer
- TabContainer
- StackContainer
- AccordionContainer

layout: headline

--------------------------------
|           top                |
|------------------------------|
|    l |                |    t |
| l  e |                | r  r |
| e  a |                | i  a |
| f  d |    center      | g  i |
| t  i |                | h  l |
|    n |                | t  i |
|    g |                |    n |
|      |                |    g |
|------------------------------|
|          bottom              |
--------------------------------

layout: sidebar

--------------------------------
|      |    top         |      |
|      |-----------------      |
|    l |                |    t |
| l  e |                | r  r |
| e  a |                | i  a |
| f  d |    center      | g  i |
| t  i |                | h  l |
|    n |                | t  i |
|    g |                |    n |
|      |                |    g |
|      ------------------      |
|      |   bottom       |      |
--------------------------------


/*
BOILERPLATE OF A WIDGET APP
---------------------------

main.css
--------
html, body {
    height: 100%;
    margin: 0;
    padding: 0;
    overflow: hidden;
}

#appContainer {
    height: 100%;
}

index.jade
----------
!!!
head#head
  title#Title
  script(type='text/javascript', src='application/dojo/dojo.js',data-dojo-config="async:true,isDebug:true, cacheBust:true")
  link(type='text/css', rel='stylesheet', href="application/app/main.css")
  script(type='text/javascript', src='/application/app/main.js')
  link(type='text/css', rel='stylesheet', href='/application/dijit/themes/claro/claro.css', media='screen')
body.claro
    #appContainer


main.js
-------*/
require(["app/widgets/AppContainer" ], function( AppContainer){
  // Create the "application" object, and places them in the right spot.
  app = new AppContainer( {design:'headline'} , 'appContainer');
  app.startup();
});
/*


LAYOUT WIDGETS
--------------
*/

require([
  "dijit/layout/BorderContainer",
  "dijit/layout/StackContainer",
  "dijit/layout/StackController",
  "dijit/layout/TabContainer",
  "dijit/layout/ContentPane",
  "dojo/parser",
  "dojo/domReady!",
   ], function(
     BorderContainer,
     StackContainer,
     StackController,
     TabContainer,
     ContentPane,
     parser
 ){

  app = new BorderContainer({
    design: "headline",
  },'appContainer'); // DO NOT set id: if second parameter is given!


  tabcontainer = new TabContainer({
    region: "center",
    id: "contentTabs",
    tabPosition: "left-h", // "top", "bottom", "left-h", "right-h"
    "class": "centerTabsClass",
  });
  app.addChild(tabcontainer);

  tab1 = new ContentPane({
    content: "First tab",
    title: "Group 1"
  });
  tabcontainer.addChild(tab1);

  tab2 = new ContentPane({
    content: "Second tab",
    title: "Group 2"
  });
  tabcontainer.addChild(tab2);

  app.startup();
});
/*
 #appContainer(data-dojo-type='dijit.layout.BorderContainer', data-dojo-props='design: "headline"')
    #tabs(data-dojo-type='dijit.layout.TabContainer', data-dojo-props='region:"center", tabPosition:"top"')
      #tab1(data-dojo-type='dijit.layout.ContentPane',data-dojo-props='title: "First tab"') Tab 1
      #tab2(data-dojo-type='dijit.layout.ContentPane',data-dojo-props='title: "Second tab"') Tab 2
*/




/* ***************************************************** */
/*                  DATA STORES
/* ***************************************************** */



/* ************************* */
/*  BASIC: MEMORY STORES
/* ************************* */

require(["dojo/store/Memory"],
  function(Memory){

    // Creates a list of people
    var employees = [
      {name:"Jim", department:"accounting"},
      {name:"Bill", department:"engineering"},
      {name:"Mike", department:"sales"},
      {name:"John", department:"sales"}
    ];

    // Creates the store
    var employeeStore = new Memory({data:employees, idProperty: "name"});

    // Querying #1
    employeeStore.query({department:"sales"}).forEach(function(employee){
      // this is called for each employee in the sales department
      console.log(employee.name);
    });
     
    // Querying #2
    employeeStore.query({department:"sales"}, {
      sort:[
        {attribute:"department", descending: false}
      ], 
      start: 0,
      count: 10
    }).forEach(function(employee){
      // this is called for each employee in the sales department
      console.log(employee.name);
    });


    // Using get() or put() -- in RESTFUL Api's this will result in HTTP call and jim will be a "promise"
    jim = employeeStore.get('Jim');
    jim.department = 'sales';

    // Using put() -- in RESTFUL Api's these will result in HTTP calls and it will return a promise
    employeeStore.put('Jim'); 
  }
);


/* ************************* */
/*  OBSERVABLE QUERIES
/* ************************* */

require(["dojo/store/Memory", 'dojo/store/Observable'],
  function(Memory, Observable){

    // Create the data store
    var employees = [
      {name:'merc', department:'sales'},
      {name:'chiara', department:'sales'},
      {name:'keith', department:'tech'},
      {name:'brock', department:'tech'},
      {name:'jim', department:'tech'},
    ];
    employeeStore = new Memory({data:employees, idProperty: "name"});

    // Make it observable
    employeeStore = Observable(employeeStore);

    // Run a query, and observe whatever was returned by it!
    list = employeeStore.query({department:'sales'});
    list.observe(function(item, removedIndex, insertedIndex){
      console.log("Added " + item + " with " + removedIndex + " and " + insertedIndex);
    });

    // Jim is in "sales": the query observer will be called
    jim = employeeStore.get('jim');
    jim.department = 'sales';
    employeeStore.put(jim);
  }
);


/* ************************* */
/*  OBSERVABLE ITEMS
/* ************************* */

require(["dojo/store/Memory", 'dojo/store/Observable', 'dojo/Stateful'],
  function(Memory, Observable, Stateful){

    // Create the data store
    var employees = [
      {name:'merc', department:'sales'},
      {name:'chiara', department:'sales'},
      {name:'keith', department:'tech'},
      {name:'brock', department:'tech'},
      {name:'jim', department:'tech'},
    ];
    employeeStore = new Memory({data:employees, idProperty: "name"});

    // Jim is in "sales": the query observer will be called
    jim = employeeStore.get('jim');

    // Make item observable
    jim = Stateful(jim);

    // Set the watching function
    jim.watch( function(name, oldValue, newValue) {
      console.log("Edited " + name + ", oldValue was " + oldValue + ", newValue is " + newValue);
    });

    // Set to something
    jim.set('department','sales');

    // This will trigger the function
    employeeStore.put(jim);
  }
);



/* ************************* */
/*  VALIDATION IN STORES
/* ************************* */

var oldPut = inventoryStore.put;
inventoryStore.put = function(object, options){
  if(object.quantity < 0){
      throw new Error("quantity must not be negative");
  }
  // now call the original
  oldPut.call(this, object, options);
};





/* ****************************** */
/*  USING MEMORY STORES AS CACHE
/* ****************************** */

require(["dojo/store/JsonRest", "dojo/store/Memory", "dojo/store/Cache", "dojo/store/Observable"],
        function(JsonRest, Memory, Cache, Observable){

  // Defune the JsonRest store
  masterInventoryStore = new JsonRest({
    target: "/Inventory/",
    idProperty: "name"
  });

  // Create a memory store with the same idProperty
  cacheInventoryStore = new Memory({ idProperty: "name" });

  // Create a cached store -- neat!
  inventoryStore = Cache(masterStore, cacheStore);

});


// NOTE: SKIPPED "TRANSACTIONAL" AND "HIERARCHY" FROM
// http://dojotoolkit.org/documentation/tutorials/1.7/data_modeling/



