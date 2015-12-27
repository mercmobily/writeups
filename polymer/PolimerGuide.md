# Polymer: the definite (un) guide

This sums up Polymer's developers guide. To "know" Polymer you should know everything here 100%, and all of the API calls by heart.

## Overview

A basic element looks like this:

    <dom-module id="element-name">

      <template>
        <style>
          /**** CSS rules for your element ****/
        </style>

        <!-- local DOM for your element -->

        <div>{{greeting}}</div> <!-- data bindings in local DOM -->
      </template>

      <script>
        // element registration
        Polymer({
          is: "element-name",

          // add properties and methods on the element's prototype

          properties: {
            // declare properties for the element's public API
            greeting: {
              type: String,
              value: "Hello!"
            }
          }
        });
      </script>

    </dom-module>


## Registration and lifecycle

This is an overview of what happens when you register an element and what a Polymer element's lifecycle is like.

### Naked web components

#### Creating new elemenbts

Without polymer, you can register a web component with:

    var XFooPrototype = Object.create(HTMLElement.prototype);

    XFooPrototype.createdCallback = function() {
      this.textContent = "I'm an x-foo!";
    };

    var XFoo = document.registerElement('x-foo', {
      prototype: XFooPrototype
    });

This will return a _constructor_ function, with _prototype_ set to an object which is then chained to HTMLElement's prototype.

To create them (the next three forms are equivalent):

    <x-foo></x-foo>

    var xFoo = new XFoo();
    document.body.appendChild(xFoo);

    var xFoo = document.createElement( 'x-foo')
    document.body.appendChild(xFoo);


#### Extending native elements

Extending an element is a matter of using registerElement with `prototype` set to the native element's prototype, and `extends` set to the native element's name:

    var XFooButtonPrototype = Object.create(HTMLButtonElement.prototype);

    XFooButtonPrototype.createdCallback = function() {
      this.textContent = "I'm an x-foo button!";
    };

    var XFooButton = document.registerElement('x-foo-button', {
      prototype: XFooButtonPrototype,
      extends: 'button'
    });

To create them:

    <button is="x-foo-button"></button>

    var xFooButton = new XFooButton();
    document.body.appendChild(xFoo);

    var xFooButton = document.createElement('button', 'x-foo-button');
    document.body.appendChild(xFooButton);


### A basic element with Polymer

To register an element, simply call the Polymer function. For example:

    // register an element
    MyElement = Polymer({

      is: 'my-element',

      // See below for lifecycle callbacks
      created: function() {
        this.textContent = 'My element!';
        console.log("AH!");
      }

    });

From this point on, you can create an element with:

    <my-element></my-element>

    var myElement = new MyElement();
    document.body.appendChild(myElement);

    var myElement = document.createElement( 'my-element')
    document.body.appendChild(myElement);

`MyElement` will "inhert" from Polymer (that is, Polymer will be next in its prototype chain).

**Only** when using the `new MyElement()` syntax, the element's method factoryImpl() is called passing it the parameters.

### Extending an existing element with Polymer

To extend an existing element, use `extends`:

    MyInput = Polymer({
      is: 'my-input',

      extends: 'input',

      created: function() {
        this.style.border = '1px solid red';
      }
    });

Polymer will make sure that the resulting element has `HTMLInputElement` in the proto chain, and that document.registerElement is called with `extends: 'input'` and  `prototype: Object.create( HTMLInputElement.prototype )` as options.

Three ways of creating one:

    <input is="my-input">

    var myInput = new MyInput();
    document.body.appendChild(myInput);

    var myElement = document.createElement( 'input', 'my-input')
    document.body.appendChild(myElement);

### Class-like constructors

You can declare an element, but _not_ register it, using `Polymer.Class`:

    var MyElement = Polymer.Class({

      is: 'my-element',

      // See below for lifecycle callbacks
      created: function() {
        this.textContent = 'My element!';
      }
    });

To register the element:

    document.registerElement('my-element', MyElement);

Note that the second parameter of `registerElement()` takes an object with the `prototype` attibute set -- which is exactly the case here, since `MyElement()` is a constructor function.

To create elements you can:

    var el1 = new MyElement();

In order to use `document.createElement()`, you need to register it first:

    document.registerElement('my-element', MyElement);
    var el2 = document.createElement('my-element');


### Lifecycle callbacks

* `created()` (called by `createdCallback()`)
* `attached()` (called by `attachedCallback()`)
* `detached()` (called by  `detachedCallback()`)
* `attributeChanged()` (called by  `attributeChangedCallback()`)
* `ready()`. Called when the element's template has been stampled and all elements **inside the element’s local DOM** have been configured (with values bound from parents, deserialized attributes, or else default values) and had their `ready()` method called. Useful when it’s necessary to manipulate an element’s local DOM once element is constructed.

Note: you mustn't redefine a Polymer component's `createdCallback()`, `attachedCallback()`, `detachedCalback()`, `attributeChangedCallback()` or things will break.

### Initialization order and timing {#initialization-order}

WATCH: https://github.com/Polymer/docs/issues/1456

The element's basic initialization order for a given element is:

- `created` callback  
- local DOM initialized (This means that **local DOM** children are created, their property values are set as specified in the template, and `ready()` has been called on them)
- `ready` callback
- `factoryImpl` callback]
- `attached` callback

Note that while the life cycle callbacks listed above will be called in the described order for any given element, the  **initialization timing between elements may vary** depending on whether or not the
browser includes native support for web components.

#### Initialization timing for light DOM children

As far as initialization timing of light DOM children there are no guarantees at all, although broadly speaking you would expect them to be `ready` in document order, which means they'd be _usually_ initialized after their parents. Of course, the user can add light children at any time.

For example if you have something like this:

    <avatar-list>
      <my-photo class="photo" src="one.jpg">First photo</my-photo>
      <my-photo class="photo" sec="two.jpg">Second photo</my-photo>
    </avatar-list>

And `avatar-list` has a `<content>` element with selector on `.photo`, `<avatar-list>` is _likely_ to be `ready()` before the various `<my-photo>` elements are `ready()`.

#### Initialization timing for local DOM children

Local DOM children are created, their property values are set as specified in the template, and `ready` is called on them before their parent's `ready` is.

There are two caveats:

* `dom-repeat` and `dom-if` create DOM asynchronously based on the property values set on them (e.g. `if`, `items`, etc.). Note that `dom-repeat` isn't a "special" statement: it's a normal custom element. When you set an items value on it, it responds by asynchronously creating instances of its template contents. When you use it inside an element's template, the parent element instantiates a `dom-repeat` element, sets its items property, and calls ready on it -- at that point, the `dom-repeat` starts its own (async) work, while the parent element goes on getting `ready`.

* Polymer guarantees that local DOM children have their `ready` callback called  before their parent's; however, it cannot guarantee the same thing for the `attached` callback. This is one place where native and polyfill behavior is different.

#### Initialization timing for siblings

There are no guarantees with regard to initialization timing between sibling elements.

This means that siblings may become `ready` in any order.

For accessing sibling elements when an element initializes, you can call `async` from inside
the `attached` callback:

    attached: function() {
       this.async(function() {
          // access sibling or parent elements here
       });
    }

### Static attributes on host

If a custom element needs HTML attributes set on it at create-time, the attributes may be declared in a `hostAttributes` property on the prototype, where keys are the attribute names and values are the values to be assigned. Values should typically be provided as strings, as HTML attributes can only be strings;
however, the standard `serialize` method is used to convert values to strings, so `true` will serialize to an empty attribute, and `false` will result in no attribute set, and so forth (see [Attribute serialization](properties.html#attribute-serialization) for more details).

Example:

    <script>

      Polymer({

        is: 'x-custom',

        hostAttributes: {
          'string-attribute': 'Value',
          'boolean-attribute': true
          tabindex: 0
        }

      });

    </script>

Results in:

    <x-custom string-attribute="Value" boolean-attribute tabindex="0"></x-custom>

`hostAttribtes` is basically a set of _default_ attributes for the created element; if an element is created declaratively, the declared values will take precedence.

**Note:** The `class` attribute can't be configured using `hostAttributes`.


## Declared properties

You can have declared properties by setting the `properties` property to your element.

**Setting the HTML attribute for the element will set the corresponding property in the element.** The property will be cast appropriatedly depending on the `type` of the property.

When mapping attribute names to property names:

* Attribute names are converted to lowercase property names. For example, the attribute `firstName` maps to firstname.

* Attribute names with dashes are converted to `camelCase` property names by capitalizing the character following each dash, then removing the dashes. For example, the attribute `first-name` maps to `firstName`.

**NOTE**: Deserialization occurs both at create time, and at runtime (for example, when the attribute is changed using setAttribute). However, it is encouraged that attributes only be used for configuring properties in static markup, and instead that properties are set directly for changes at runtime.

### How to declare properties

In its most basic usage, `properties` can have:

* `type`	Type: constructor (one of `Boolean`, `Date`, `Number`, `String`, `Array` or `Object`)

* `value`	Type: `boolean`, `number`, `string` or `function`. Default value for the property. If `value` is a function, the function is invoked and the return value is used as the default value of the property. If the default value should be an array or object unique to the instance, create the array or object inside a function.

Alternatively, it might have just the property name/constructor fields, which will act as `type` and `value`. E.g.:

    Polymer( {

      properties: {

        is: 'my-element',

        // Explicitly setting type and value
        count: {
          type: Number,
          value: 10
        }

        user: String, // name/constructor pair

        mainName: String,
      },
    })

Since HTML attributes are mapped to properties, a user can configure declared properties from markup.

So, when typing:

    <my-element count="10">

The `count` property of the element will be set to (numeral) 10. Note that casting happens thanks to the defined type (`Number`) in the `properties` object.

**HTML attributes are constantly reflected on the element's declared properties**.So, doing:

    myElement.setAttribute('count', 20 );

Will also set `myElement.count`.

Dash attributes are converted to camelCase ones:

    myElement.setAttribute('main-name', 'Tony' );

Will set the property `mainName`.


### Observers

YOU ARE HERE





TO DOCUMENT:

* `reflectToAttribute`	Type: `boolean` Set to `true` to cause the corresponding attribute to be set on the host node when the property value changes. If the property value is Boolean, the attribute is created as a standard HTML boolean attribute (set if true, not set if false). For other property types, the attribute value is a string representation of the property value.

* `readOnly`	Type: `boolean` If `true`, the property can't be set directly by assignment or data binding.

* `notify`	Type: `boolean` If `true`, the property is available for two-way data binding. In addition, an event, `propertyName-changed` is fired whenever the property changes.

* `computed`	Type: `String`
The value is interpreted as a method name and argument list. The method is invoked to calculate the value whenever any of the argument values changes.

* `observer`	Type: `String` The value is interpreted as a method name to be invoked when the property value changes. Property change handlers must be registered explicitly.





## Local DOM

### How Shady DOM works in Polymer

With Polymer, you have local DOM (hidden from the world), light DOM (that is, what's in the custom element) and the composite DOM (the end result, what's actually rendered).

In a perfect world, the browser implements everything so that an element's local DOM, as well as its composite DOM, are hidden from the outside. However, waiting for 2020, Polymer uses _shady DOM_ -- an implementation of these calls, as long as you wrap elements with `Polymer.dom()` before you use DOM-manipulating functions.

#### Local DOM

Local DOM is "hidden", and it's normally copied from the template:

<link rel="import" href="bower_components/polymer/polymer.html">

    <dom-module id="test-component">
      <template>
        <div id="local-1">Local DOM starts here</div>
        <content id="content-1" select="#light-1"></content>
        <div>This is from the local dom again</div>
        <content id="content-2" select="#light-2"></content>
        <div id="local-2">Local DOM ends here</div>
      </template>

      <script>
        Polymer({
          is: 'test-component',
          ready: function(){
            console.log("this is: ", this );
            console.log("this.root is:", this.root )
            THIS = this;
          }
        })
      </script>

    </dom-module>

The local DOM here is whatever is in the `<template>` tags.
The rational is that the "local" DOM is actually hidden away from the rest of the DOM.

Programmatically, the local DOM is in `this.root`.

#### Light DOM

Light DOM is whatever is in the actual element when it's being used.

<test-component>
    <p id="light-1">One</p>
    <p id="light-2">Two</p>
    <p id="light-3">Three</p>
</test-component>

Note that not all of the elements might end up getting rendered (or "composed"), since the local DOM might not pick them (for example `<p id="light-3">Three</p>` will be 100% ignored and therefore invisible/unrendered).

Programmatically, the light DOM is in a fragment.

#### Composed DOM and access elements

The composed DOM is the result of the local DOM with bits of the light DOM inserted when appropriate, depending on the <content> tags.

Programmatically, the light dom is in `this` (the element itself) which will effectively be the result of local DOM + light DOM.

Since adding to the element itself (`this`) would only change the composed result, and not the local DOM nor the light DOM, when using Shady Dom you need to wrap any element before using it:

Programmatically, the composed DOM is in the document's flow (so that it gets rendered). Changes to the light DOM or the local DOM will imply re-rendering of the composed DOM.

### API

To enable Polymer's API on nodes:

* `Polymer.dom( this ).METHOD` -- affecting the light DOM
* `Polymer.dom( this.root ).METHOD` -- affecting the local DOM

Changes to the light DOM and the local DOM will affect the composed DOM. Don't forget:

* `this` represents the element itself, which is the composed DOM.

Also note that:

 * `this.$.someId` will reference an element in the local DOM with id `someId`. ONLY with elements in the `template` when it was created, nothing programmatic.
 * `this.$$.someSelector` will return the first node in the local DOM that matches `selector`.

#### Query selector

* `Polymer.dom(parent).querySelector(selector)` -- returns one element (first one matching)
* `Polymer.dom(parent).querySelectorAll(selector)` -- returns several elements

#### Adding and removing children

* `Polymer.dom(parent).appendChild(node)`
* `Polymer.dom(parent).insertBefore(node, beforeNode)`
* `Polymer.dom(parent).removeChild(node)`
* `Polymer.dom.flush()`

#### Parent and child API

* `Polymer.dom(parent).childNodes`
* `Polymer.dom(parent).children`
* `Polymer.dom(node).parentNode`
* `Polymer.dom(node).firstChild`
* `Polymer.dom(node).lastChild`
* `Polymer.dom(node).firstElementChild`
* `Polymer.dom(node).lastElementChild`
* `Polymer.dom(node).previousSibling`
* `Polymer.dom(node).nextSibling`
* `Polymer.dom(node).textContent`
* `Polymer.dom(node).innerHTML`

#### Node mutation API

* `Polymer.dom(node).setAttribute(attribute, value)`
* `Polymer.dom(node).removeAttribute(attribute)`
* `Polymer.dom(node).classList`

### Extended DOM API

Shady DOM has extra functions to explore the DOM.`

#### Distrbuted children

Here are the functions that deal with figuring out what modes in the light DOM were distributed through a `<content>` tag and vice versa. Note that the last one is the most useful one.

* `Polymer.dom(contentElement).getDistributedNodes()` -- given a `<content>` element in the local DOM, it will return the nodes (from the light DOM) that were distributed. Useless because 1) It doesn't look up a `content` tag 2) It will returns ALL nodes (including spaces etc.)

* `Polymer.dom(node).getDestinationInsertionPoints()` -- given an element in the local DOM, it will return the `<content>` element in the local DOM that distributed it. Useless because it's really hardly ever used.

* `this.getContentChildNodes(contentSelector)` -- Given a selector for a `<content>` element in the local DOM, it will return the child nodes (including space node etc.!)

* `this.getContentChildren(contentSelector)` -- Given a selector for a `<content>` element in the local DOM, it will return the children elements. **The most useful function for distributed children!**

#### Effective children

**"An element's _local DOM_ might have `<content>` tags; this is especially the case when the element is used inside another element's template (composition).".**

Effective children are the set of an element’s light DOM children, with any insertion points replaced by their distributed children.

* `Polymer.dom(element).getEffectiveChildNodes()` -- returns the effective child nodes in the light DOM.
* `this.getEffectiveChildNodes()` -- returns array of child nodes
* `this.getEffectiveChildren()` -- returns array of ONLY child elements
* `this.queryEffectiveChildren(selector)` -- returns first matching element
* `this.queryAllEffectiveChildren(selector)` -- returns array of all matching elements

Here is a very practical example:

    <dom-module id="element-counter">
      <template>
        <p>This is the element counter!</p>
        <content id='c'></content>
      </template>
      <script>
        Polymer({
          is: 'element-counter',
          attached: function(){
            console.log("Children:", Polymer.dom( this ).children );
            console.log("Effective children:", this.getEffectiveChildren() );
            console.log("That's it!");
          }
        })
      </script>
    </dom-template>

    <dom-module id="complex-widget">
      <template>
        <p>This is a complex widget that includes the element-counter</p>

        <element-counter>
          <content></content>
        </element-counter>

      </template>
      <script>
        Polymer({
          is: 'complex-widget',
        })
      </script>
    </dom-template>

Note that using `getContentChildren()` would have achieved the same result:

    // ...childCount
    attached: function(){
      this.childCount = this.getContentChildren('#c').length;
    }

Since it will return all of the nodes in the `<content>` tags with id `c`.

There are some cases where they are not equivalent and you cannot use `getContentChildren()`:

* the element has no local DOM.
* the element uses no `<content>` tag. For example, the element might not include its light DOM children using a `<content>` tag, and do something programmatic with them instead. In this case, you cannot use `getContentChildren()` since you don't have a `<content>` tag
* the element has one or more `<content>` tags with `select=` attributes. In this case, `getContentChildren()` will need to be used on each `content` tag and won't return the full list of children (which `getEffectiveChildren()`, on the countrary, will return).

### Observe added and removed children

Use the DOM API’s observeNodes method to track when children are added and removed from your element:

     this.anObserver = Polymer.dom(this.$.contentNode).observeNodes(function(info) {

      // Here, `this` is the element itself
      var addedElements = info.addedNodes.filter(function(node) {
        return (node.nodeType === Node.ELEMENT_NODE)
      });

      this.processNewNodes(addedElements);
      this.processRemovedNodes(info.removedNodes);
    });


    Polymer.dom(node).unobserveNodes(this.anObserver);

The observeNodes method behaves slightly differently depending on the node being observed:

* If the node being observed is a content node, the callback is called when the content node’s distributed children change.
  * If `<content>` is used without a selector, then anything added to the light DOM will trigger
  * If `<content>` is used with `select=".class"`, then only elements added to the light DOM with the right `.class` (satisfying the match) will trigger
* For any other node, the callback is called when the node’s effective children change.
* The observer function will be called the first time with *all* of the elements contained in the observed one.


## Styling

## Events

## Data binding

## Behaviours

## Utility function

## Global settings

## Browser compatibility
