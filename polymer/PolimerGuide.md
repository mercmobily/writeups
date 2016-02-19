# Polymer: the definite (un) guide

This sums up Polymer's developers guide. To "know" Polymer you should know everything here 100%, and all of the API calls by heart.

**TODO: Talk about decorator patterns: https://github.com/Polymer/polymer/issues/1990**

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

**TODO: APPLY EXTRA WRITING: https://github.com/Polymer/docs/pull/1506  AND ALSO WATCH: https://github.com/Polymer/docs/issues/1456 TO CHECK**


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

If a custom element needs HTML attributes set on it at create-time, the attributes may be declared in a `hostAttributes` property on the prototype, where keys are the attribute names and values are the values to be assigned. Values should typically be provided as strings, as HTML attributes can only be strings; however, the standard `serialize` method is used to convert values to strings, so `true` will serialize to an empty attribute, and `false` will result in no attribute set, and so forth (see [Attribute serialization](properties.html#attribute-serialization) for more details).

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

**NOTE**: Deserialization occurs both at create time, and at runtime (for example, when the attribute is changed using setAttribute). However, it is encouraged that attributes only be used for configuring properties in static markup, and instead that properties are set directly for changes at runtime.

When mapping attribute names to property names:

* Attribute names are converted to lowercase property names. For example, the attribute `firstName` maps to firstname.

* Attribute names with dashes are converted to `camelCase` property names by capitalizing the character following each dash, then removing the dashes. For example, the attribute `first-name` maps to `firstName`.

* For a Boolean property to be configurable from markup, it must default to `false`. If it defaults to `true`, you cannot set it to `false` from markup, since the presence of the attribute, with or without a value, equates to `true`.

### How to declare properties

In its most basic usage, `properties` can have:

* `type`	Type: constructor (one of `Boolean`, `Date`, `Number`, `String`, `Array` or `Object`)

* `value`	Type: `boolean`, `number`, `string` or `function`. Default value for the property. If `value` is a function, the function is invoked and the return value is used as the default value of the property. If the default value should be an array or object unique to the instance, create the array or object inside a function.

Alternatively, it might have just the property name/constructor fields, which will act as `type` and `value`. E.g.:

    <script>
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
    </script>

Since HTML attributes are mapped to properties, a user can configure declared properties from markup.

So, when typing:

    <my-element count="10">

The `count` property of the element will be set to (numeral) 10. Note that casting happens thanks to the defined type (`Number`) in the `properties` object.

**HTML attributes are constantly reflected on the element's declared properties**. So, doing:

    myElement.setAttribute('count', 20 );

Will also set `myElement.count`.

Dash attributes are converted to camelCase ones:

    myElement.setAttribute('main-name', 'Tony' );

Will set the property `mainName`.


### Notifications

You can enable the firing of an event called `propertyName-changed` when a property changes by adding `notify: true` to the property's definition. This event is also used by two-way data binding.

For example:

    <script>
    Polymer( {

      properties: {

        is: 'my-element',

        // Explicitly setting type and value
        count: {
          type: Number,
          value: 10,
          notify: true
        }

        user: String, // name/constructor pair

        mainName: String,
      },
    })
    </script>

The event `count-changed` will be fired whenever `count` is changed.

### Read-only properties

When a property only “produces” data and never consumes data, this can be made explicit to avoid accidental changes from the host by setting the readOnly flag to true in the properties property definition. In order for the element to actually change the value of the property, it must use a private generated setter of the convention `_setProperty(value)`.

    <script>
      Polymer({

        properties: {
          response: {
            type: Object,
            readOnly: true,
            notify: true
          }
        },

        responseHandler: function(response) {
          this._setResponse(response);
        }

      });
    </script>


### Computed properties {#computed-properties}

Polymer supports virtual properties whose values are calculated from other properties.

To define a computed property, add it to the `properties` object with a `computed` key mapping to a computing function:

        fullName: {
          type: String,
          computed: 'computeFullName(first, last)'
        }


The function is provided as a string with dependent properties as arguments in parenthesis. The function will be called once for any change to the dependent properties.

The computing function is not invoked until **all** dependent properties are defined (`!== undefined`). So each dependent properties should have a default `value` defined in `properties` (or otherwise be initialized to a non-`undefined` value) to ensure the property is computed.

**Note:** The definition of a computing function looks like the definition of a [multi-property observer](#multi-property-observers), and the two act almost identically. The only difference is that the computed property function returns a value that's exposed as a virtual property.

    <dom-module id="x-custom">

      <template>
        My name is <span>{%raw%}{{fullName}}{%endraw%}</span>
      </template>

      <script>
        Polymer({

          is: 'x-custom',

          properties: {
            first: String,
            last: String,

            fullName: {
              type: String,
              // when `first` or `last` changes `computeFullName` is called once
              // and the value it returns is stored as `fullName`
              computed: 'computeFullName(first, last)'
            }
          },
          computeFullName: function(first, last) {
            return first + ' ' + last;
          }
        });
      </script>
    </dom-module>

Arguments to computing functions may be simple properties on the element, as well as any of the arguments types supported by `observers`, including `paths`, `paths with wildcards` and `paths to array splices`.

The arguments received by the computing function match those described in the sections referenced above.

### Reflect from property to attribute

Remember how **setting the HTML attribute for the element will set the corresponding property in the element.** This is true especially so that when defining elements declarative, e.g. `<my-element surname="Mobily">`, the `surname` property of the element is _also_ set to `Mobily` (and the property will be cast appropriatedly depending on the `type` of the property).

In specific cases, it may be useful to keep an HTML attribute value in sync with a property value. This may be achieved by setting `reflectToAttribute: true` on a property in the `properties` configuration object. This will cause any change to the property to be serialized out to an attribute of the same name.

    <script>
      Polymer({

        properties: {
         response: {
            type: Object,
            reflectToAttribute: true
         }
        },

        responseHandler: function(response) {
          this.response = 'loaded';
          // results in this.setAttribute('response', 'loaded');
        }

      });
    </script>

#### Attribute serialization

When reflecting a property to an attribute or binding a property to an attribute, the property value is serialized to the attribute.

By default, values are serialized according to value’s current type (regardless of the property’s type value):

* `String`. No serialization required.
* `Date` or `Number`. Serialized using  toString.
* `Boolean`. Results in a non-valued attribute to be either set (true) or removed (false).
* `Array` or Object`. Serialized using JSON.stringify.

To supply custom serialization for a custom element, override your element’s `serialize` method.

### Observers

#### Basic observers

Basic observers are achieved by adding an `observer` attribute to your property declaration, which represents the _name_ of the method used as the observer.

For example:

    <script>
    Polymer({

      is: 'my-observer',

      properties: {
        name: {
          type: String,
          observer: '_nameChanged'
        },
      },

      _nameChanged: function(newValue, oldValue) {
        console.log("The name has changed");
      },

    });
    </script>

Property change observation is achieved in Polymer by installing setters on the custom element prototype. So there is no need to use special setters to assign a value.

NOTE: You can use `observer` for arrays and objects as well; however, it will only detect changes to what array/object is references (e.g. `this.anArray = [ 1, 2, 3 ]` or `this.anObject = { a:10 }`) rather than modifications to the contents of the object or array.

#### Observing multuple properties

You can set the same observer for multiple properties by adding elements to the `observers` element of your Polymer object:

    <script>
    Polymer({

      is: 'my-observer',

      properties: {
        preload: Boolean,
        src: String,
        size: String
      },

      observers: [
        'updateImage(preload, src, size)'
      ],

      _updateImage: function(preload, src, size) {
        // ... do work using dependent values
      }
    });
    </script>

Polymer will parser the elements in `observers`, so that `this._updateImage()` will be called with the parameters `preload`, `src` and `size`. Note that:

* Observers are not invoked until all dependent properties are defined (`!== undefined`). So each dependent properties should have a default value defined in properties (or otherwise be initialized to a non-undefined value) to ensure the observer is called.
* Observers do not receive old values as arguments, only new values. Only single-property observers defined in the properties object receive both old and new values.

#### Observing deep changes made to objects

If you want to observe deep objects, you will need to use the same syntax as multiple observers, and specify an object path you want to observe; for example `_singleObserver( complex.s )` will be triggered when the attribute `s` of `this.complex` changes.

Just like multiple observers, you can specify more than one attribute to watch; also, just like multiple observers, when having more than one parameter the observer will only be triggered when all of the watched attrivutes are not `undefined`.

For example:

    <script>
    Polymer({
      is: 'my-observer',
      properties: {
        complex:  {
          type: Object,
          // Note: 'c' is missing on purpose
          value: function() { return { s:1000, a: 20, b:30 } },
        },
      },

      observers: [
        '_singleObserver( complex.s )',
        '_multipleObserver1( complex.a, complex.b )',
        '_multipleObserver2( complex.a, complex.b, complex.c )',
      ],

      created: function(){
        console.log("CREATED!");
        THIS = this;
      },

      _singleObserver: function( s ){
        console.log("(SINGLE) VALUES: ", s );
      },

      _multipleObserver1: function( a, b ){
        console.log("(MULTUPLE1) VALUES: ", a, b );
      },

      // ONLY INVOKED ONCE complex.c IS DEFINED
      _multipleObserver2: function( a, b, c ){
        console.log("(MULTUPLE2) VALUES: ", a, b, c );
      },

    })
    </script>

In order for Polymer to properly detect the sub-property change, the sub-property must be updated in one of the following two ways:

* Via a property binding.
* By calling `set`. Remember that you can pass a `path` to the `this.set()` method.

For example:

    this.set( 'complex.c', '1000');


#### Watching multuple deep properties

If you want to watch multiple sub-properties, you can use a wildcard when defining the observer's parameter. E.g. `_multipleObserver3( complex.d.* )` will make sure `_multipleObserver3` will be called whenever anything under `this.complex.d` changes:

    <script>
    Polymer({
      is: 'my-observer',
      properties: {
        complex:  {
          type: Object,
          value: function() { return { d: { da: { daa: 91, dab:92 } , db: 80 } } },
        },
      },

      observers: [
        '_multipleObserver3( complex.d.* )',
      ],

      created: function(){
        console.log("CREATED!");
        THIS = this;
      },

      _multipleObserver3: function( s ){
        console.log("(MULTUPLE3) VALUES: ", s );
      }

    })
    </script>

When you specify a path with a wildcard, the argument passed to your observer is a change record object with the following properties:

* `path`. Path to the property that changed. You need this since any one of the sub-property may have changed.
* `value`. New value of the path that changed.
* `base`. The object matching the non-wildcard portion of the path. This is especially useful if you want to then change the changed attribute's value using `this.set()`

#### Watching arrays

In Javascript an array can be mutated using `push`, `pop`, `shift`, `unshift`, and `splice` (these functions are generally referred to as "splices"). To observe mutations to arrays specify a path to an array followed by ``.splices` as an argument to the observer function.

For example:

    <script>
    Polymer({

      is: 'my-observer',

      properties: {
        users: {
          type: Array,
          value: function() {
            return [];
          }
        }
      },

      observers: [
        '_usersAddedOrRemoved(users.splices)'
      ],

      created: function(){
        console.log("CREATED!");
        THIS = this;
      },

      _usersAddedOrRemoved: function( changeRecord ) {
        console.log("(ARRAYOBSERVER) Here: ", changeRecord)
      },

      addUser: function() {
        this.push('users', {name: "Jack Aubrey"});
      }

    });
    </script>

The `changeRecord` parameter is tricky. To understand it fully, keep in mind the syntax of the array-changing function `splice()` in Javascript, which has the following parameters:

* `index`. The position where items will be added/remove
* `howmany`. How many items will be removed, from `index`
* `item1, ..., itemX`. The itesm to be added, from `index`

Keeping this in mind `changeRecord` will have:

* `indexSplices`. Lists the set of changes that occurred to the array, in terms of array indicies. Each `indexSplices` record contains the following properties:
  * `index`. Position where the splice started.
  * `removed`. Array of removed items.
  * `addedCount`. Number of new items inserted at `index`.
* `keySplices`. Lists the set of changes that occurred to the array in terms of “keys” used by Polymer for identifying array elements. Each `keySplices` record contains the following properties:
  * `added`. Array of added keys.
  * `removed`. Array of removed keys.

Note that I used `this.push()` to push into the array: Polymer has the equivalent of the splices functions in Javascript, with the bonus that the registered observer will actually get called.

**TODO: FIX THIS . This seems wrong, and I couldn't work out what is "right". Documentation seems totally wrong, and even the basic example doesn't work. Ticket open:  https://github.com/Polymer/polymer/issues/3239 (Arrays not working)**


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

The composed DOM is the result of the local DOM with bits of the light DOM inserted when appropriate, depending on the `<content>` tags.

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

* `this.getContentChildNodes(contentSelector)` -- Given a selector for a `<content>` element in the local DOM, it will return the child nodes (including spaces etc.!) Uselese because it returns _all_ nodes, including text nodes!

* `this.getContentChildren(contentSelector)` -- Given a selector for a `<content>` element in the local DOM, it will return the children elements. **The most useful function for distributed children!**

**TODO: Create code example for this to onclude ALL of these at once**

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

````javascript
    // ...childCount
    attached: function(){
      this.childCount = this.getContentChildren('#c').length;
    }
````

Since it will return all of the nodes in the `<content>` tag with id `c`.

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

The `observeNodes` method behaves slightly differently depending on the node being observed:

* If the node being observed is a content node, the callback is called when the content node’s distributed children change.
  * If `<content>` is used without a selector, then anything added to the light DOM will trigger
  * If `<content>` is used with `select=".class"`, then only elements added to the light DOM with the right `.class` (satisfying the match) will trigger
* For any other node, the callback is called when the node’s effective children change.
* The observer function will be called the first time with *all* of the elements contained in the observed one.

## Styling

Styling using CSS is a huge subject in its own right. One of the problems of CSS is that it's difficult to have proper _encapsulation_: styles from any loaded stylesheet will be applied to every matching selector within the page, according to their specificity.

While in this section I won't cover any of the _visual results_ of what CSS directives do to a page, I will cover CSS seen as a list of styles applied to elements in a web page according to the specified selectors.

The "box model", floats, etc. can be learned here:

* [http://learn.shayhowe.com/](Shay Howe's tutorials, the best in the west at the moment)
* [http://www.w3.org/community/webed/wiki/Main_Page#CSS](CSS Web Standards Curriculum])
* [https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Getting_started](MDN's tutorials)
* [http://matthewjamestaylor.com/blog/perfect-multi-column-liquid-layouts](Responsive layouts)
* [http://flatuicolors.com/](Paper colours)

In terms of CSS standard:

* [http://www.w3.org/TR/CSS2/](Specs of 2.1)
* [http://www.w3.org/TR/css3-roadmap/](Specs for 3 -- except, there is NO 3!)

### CSS Basics

In this section I will cover basic CSS usage in terms of syntax and grammar.
To apply CSS rules to elements, you have 4 ways:

#### How to import CSS

##### CSS via external stylesheet

    <link rel="stylesheet" type="text/css" href="mystyles.css" media="screen" />

##### CSS embedded in the page itself

````
    <style media="screen" type="text/css">
     ... CSS styling here ...
    </style>
````

##### CSS inline


You can add styling rules directly to the HTML itself.

````
    <h2 style="color:red;">This will come up red</h2>
````

##### Importing CSS from CSS

Finally, you can import a stylesheet directly from another stylesheet:

    @import "newstyles.css";

#### CSS Syntax

In all cases except inline CSS, for each CSS rule you define:

* A selector (or a list of selectors, comma separated)
* A list of properties that will be applied to that selector

For example, you may have:

    p, li {
      background: red;
    }

Here, `p` and `li` are the two selectors (which in this case mean "every `p` tag" and "every `li` tag") to which the property `background: red` will be applied.

What property is meanigful depends on the tag itself.

### CSS Cascading order

The order in which directives are applied is really important. From [the cascading order specs for 2.1](http://www.w3.org/TR/CSS21/cascade.html#cascading-order):

-----------
To find the value for an element/property combination, user agents must apply the following sorting order:

* Find all declarations that apply to the element and property in question, for the target media type. Declarations apply if the associated selector matches the element in question and the target medium matches the media list on all @media rules containing the declaration and on all links on the path through which the style sheet was reached.
* Sort according to importance (normal or important) and origin (author, user, or user agent). In ascending order of precedence:
  * user agent declarations
  * user normal declarations
  * author normal declarations
  * author important declarations
  * user important declarations
* Sort rules with the same importance and origin by specificity of selector: more specific selectors will override more general ones. Pseudo-elements and pseudo-classes are counted as normal elements and classes, respectively.
* Finally, sort by order specified: if two declarations have the same weight, origin and specificity, the latter specified wins. Declarations in imported style sheets are considered to be before any declarations in the style sheet itself.

User agent declarations are the "browser defaults". Author declarations are the ones in the stylesheets. User declarations are the ones added by the user (for example, visually-impaired users might enlarge all writing). A declaration can be marked as important by adding `!important` ad the end of the line (for example `background: red !important;`).
---------------

The specifications talk about "specificity": that means how "precise" a selector is. I will talk about this more very shortly.

### CSS Selectors

Here are [selectors from the official documentation](http://www.w3.org/TR/CSS21/selector.html) (note that I changed the order a little, compared to the specs):

````
    *	`*` -- Matches any element.	Universal selector
    *	`E` -- 	Matches any E element (i.e., an element of type E).	Type selectors
    *	`DIV.warning` -- Language specific. (In HTML, the same as DIV[class~="warning"].)	Class selectors
    *	`E#myid`	Matches any E element with ID equal to "myid".	ID selectors
    *	`E F` -- 	Matches any F element that is a descendant of an E element.	Descendant selectors
    *	`E > F` -- 	Matches any F element that is a child of an element E.	Child selectors
    *	`E + F` -- Matches any F element immediately preceded by a sibling element E.	Adjacent selectors
    *	`E:first-child` -- 	Matches element E when E is the first child of its parent.	The :first-child pseudo-class
    *	`E:link E:visited` -- 	Matches element E if E is the source anchor of a hyperlink of which the target is not yet visited (`:link`) or already visited (`:visited`).	The link pseudo-classes
    *	`E:active` `E:hover` `E:focus` -- Matches E during certain user actions.	The dynamic pseudo-classes
    *	`E:lang(c)` -- Matches element of type E if it is in (human) language c (the document language specifies how language is determined).	The :lang() pseudo-class
    *	`E[foo]` -- Matches any E element with the "foo" attribute set (whatever the value).	Attribute selectors
    *	`E[foo="warning"]` -- Matches any E element whose "foo" attribute value is exactly equal to "warning".	Attribute selectors
    *	`E[foo~="warning"]` -- Matches any E element whose "foo" attribute value is a list of space-separated values, one of which is exactly equal to "warning".	Attribute selectors
    *	`E[lang|="en"]` -- Matches any E element whose "lang" attribute has a hyphen-separated list of values beginning (from the left) with "en".	Attribute selectors
````

Note that class selectors can also be `div.class1.class2` (meaning "has both classes").

For example, here:

````
    html
    .content p  { color: red; }
````

Means "an element `p`, contained by an element of class  `content`.
A few selectors:

* `#container > .box`
* `article > p`
* `a[href="http://google.com/"]`

Please note that CSS3 adds more powerful selectors.

### CSS selectors' specificity

A selector's specificity is calculated as follows:

------------------------
* count 1 if the declaration is from is a 'style' attribute rather than a rule with a selector, 0 otherwise (= a) (In HTML, values of an element's "style" attribute are style sheet rules. These rules have no selectors, so a=1, b=0, c=0, and d=0.)
* count the number of ID attributes in the selector (= b)
* count the number of other attributes and [pseudo-classes](https://developer.mozilla.org/en-US/docs/Web/CSS/pseudo-classes) in the selector (= c)
* count the number of element names and [pseudo-elements](https://developer.mozilla.org/en/docs/Web/CSS/Pseudo-elements) in the selector (= d)
The specificity is based only on the form of the selector. In particular, a selector of the form "[id=p33]" is counted as an attribute selector (a=0, b=0, c=1, d=0), even if the id attribute is defined as an "ID" in the source document's DTD.

Concatenating the four numbers a-b-c-d (in a number system with a large base) gives the specificity.

Some examples:

````
     *             {}  /* a=0 b=0 c=0 d=0 -> specificity = 0,0,0,0 */
     li            {}  /* a=0 b=0 c=0 d=1 -> specificity = 0,0,0,1 */
     li:first-line {}  /* a=0 b=0 c=0 d=2 -> specificity = 0,0,0,2 */
     ul li         {}  /* a=0 b=0 c=0 d=2 -> specificity = 0,0,0,2 */
     ul ol+li      {}  /* a=0 b=0 c=0 d=3 -> specificity = 0,0,0,3 */
     h1 + *[rel=up]{}  /* a=0 b=0 c=1 d=1 -> specificity = 0,0,1,1 */
     ul ol li.red  {}  /* a=0 b=0 c=1 d=3 -> specificity = 0,0,1,3 */
     li.red.level  {}  /* a=0 b=0 c=2 d=1 -> specificity = 0,0,2,1 */
     #x34y         {}  /* a=0 b=1 c=0 d=0 -> specificity = 0,1,0,0 */
     style=""          /* a=1 b=0 c=0 d=0 -> specificity = 1,0,0,0 */
    <HEAD>
    <STYLE type="text/css">
      #x97z { color: red }
    </STYLE>
    </HEAD>
    <BODY>
    <P ID=x97z style="color: green">
    </BODY>
````

In the above example, the color of the P element would be green. The declaration in the "style" attribute will override the one in the STYLE element because of cascading rule 3, since it has a higher specificity.
---------------
In general, and easy way to remember this is simple:

1) inline styles always win
2) The presence of an ID turns the race into a "race of who has the most IDs". The rest only applies if it's a tie
3) The presence of an attribute or a pseudo-class turns the race into a race of "who has the most attributes OR [pseudo-classes](https://developer.mozilla.org/en-US/docs/Web/CSS/pseudo-classes)". The rest only applies if it's a tie
4) If none of the above is there, or all of the above is a tie, whoever has the highest number of elements or [pseudo-elements](https://developer.mozilla.org/en/docs/Web/CSS/Pseudo-elements) wins.

### Styles

Styling in Polymer is done using CSS. Two extra options are available: `custom-style` style elements, and shared styles.

#### Using `<style is="custom-style">`

Instead of using `<style>` as it is, in Polymer you should use  `<style is="custom-style">`.
**`custom-style` should only be used for defining document styles, outside of a custom element’s local DOM.**
Using it will make sure that rules will not affect the elements' local DOM. You will also be able to use the `:root` selector.

##### Pseudo-CSS files with `custom-style`

`custom-style` can be used to create pseudo-css files -- that is, html file that can be included and that only define a stylesheet.

E.g. the file `app-theme.html` might contain:

    <link rel="import" href="../bower_components/polymer/polymer.html">
    <link rel="import" href="../bower_components/paper-styles/typography.html">

    <style is="custom-style">

      :root {
        --primary-text-color: #212121;
        --primary-background-color: #ffffff;
      }

      h1 {
        @apply(--paper-font-display2)
      }
    </style>


Then, `index.html` might load it:

    <!doctype html>
    <html lang="en">
    <head>
    ...
    <link rel="import" href="app-theme.html">
    ...
    </head>
    ...

##### Inline CSS with `<style is="custom-style">`

You can also add in-line CSS in your file:

    <!doctype html>
    <html lang="en">
    <head>
    ...
    <link rel="import" href="app-theme.html">
    <style is="custom-style">
      :root {
        --primary-text-color: #212121;
      }
    </style>
    ...
    </head>
    ...

#### Shared CSS rules with `<style include="shared-styles" [is="custom-style"] >`

You can create an html file that contains styles, but they are _not_ applied straight away.
You do that by wrapping the styles in a module:

    <!-- shared-styles.html -->
    <dom-module id="shared-styles">
      <template>
        <style>
          .red { color: red; }
        </style>
      </template>
    </dom-module>

The id attribute specifies the name you’ll use to reference your shared styles. Style module names use the same namespace as elements, so your style modules must have unique names.

Note that in this case importing the file _won't_ apply the styles. However, it will obviously load the module. To activate the styles, you will need to use the `include` attribute to the `<style>` element.

The `include` attribute will work within an element (just using `<style>`):

    <!-- import the module  -->
    <link rel="import" href="../shared-styles/shared-styles.html">
    <dom-module id="x-foo">
      <template>
        <!-- include the style module by name -->
        <style include="shared-styles"></style>
        <style>:host { display: block; }</style>
        Hi
      </template>
      <script>Polymer({is: 'x-foo'});</script>
    </dom-module>

And defining local styles (with higher priority):

    <!-- import the module  -->
    <link rel="import" href="../shared-styles/shared-styles.html">
    <dom-module id="x-foo">
      <template>
        <!-- include the style module by name AND define local styles-->
        <style include="shared-styles">
          :host {
            display: block;
          }
        </style>
        Hi
      </template>
      <script>Polymer({is: 'x-foo'});</script>
    </dom-module>

You can use  `include` also in a file, outside an element:

    <!-- import the shared styles  -->
    <link rel="import" href="../shared-styles/shared-styles.html">
    <!-- include the shared styles -->
    <style is="custom-style" include="shared-styles"></style>

And defining local styles (with higher priority):

    <!-- import the shared styles  -->
    <link rel="import" href="../shared-styles/shared-styles.html">
    <!-- include the shared styles -->
    <style is="custom-style" include="shared-styles">
      :host {
        display: block;
      }
    </style>


### Anything else

**TODO: FIX this chapter following [my rewrite of Polymer's styling guide](https://github.com/mercmobily/docs/blob/betterstyling/1.0/docs/devguide/styling.md).**


## Events

AS IS IN OFFICIAL DOCUMENTATION

## Data binding

### Data binding basics

    <dom-module id="host-element">
        <template>
          <child-element childName="{{hostName}}"></child-element>
        </template>
    </dom-module>

**When there is an `{{annotation}}`, Polymer treats this things in a special way: child-element's _property_ hostName, and not its _attribute_, is bound to the host's `hostName` property**

This will do the following:

* **Setup**. `host-element` will assign the host's `hostName` property to the child's `childName` property

* **`child-element`** It will listen for the event `hostName-changed` from `host-element`. When it happens, it will assign `e.detail.value` to its own (child-element's) `childName` property.
  * **EXCEPT**: If `child-element`'s property is read-only. In which case, any further changes to `hostName` aren't reflected onto the child

* **`host-element`** It will listen for the event `childName-changed` in `child-element`. When it happens, it will assign `e.detail.value` to `hostName`
  *  **EXCEPT** If `child-element`'s `notify` is off. In this case, `childName-changed` will never be generated
  *  **EXCEPT** If `[[ ]]` were used instead of `{{}}` -- in which case listening doesn't happen at all
  *  **EXCEPT** If you are doing compound binding, e.g. `<child-element childName="AH! {{hostName}}"></child-element>`
  *  **EXCEPT** If you are doing computed binding, e.g. `<child-element childName="_f({{hostName}})"></child-element>`
  *  **EXCEPT** If you are doing attribute binding, e.g. `<child-element childName$="{{hostName}}"></child-element>`

* **NOTE**: You can decide what event the `child-element` will listen to

### Custom event names

Sometimes, especially when dealing with standard HTML elements, you need to listen to specific events, rather than `variableName-changed`. You can do this:

    <dom-module id="host-element">

      <template>
        <p>Hello {{myName}}</p>

        <input id='ME' value="{{myName::input}}">
    </dom-module>

In this case, `host-element` will initially assign `myName` to input`'s `value`. Then, it will listen to the event `input` from the element `input` -- and reflect `e.value` back to `host-element`'s `myName`.

### Binding to text content

You can also bind to text content from `host-element`:

    <template>
       First: {{firstName}}<br>
       Last: {{lastName}}
     </template>

In this case, there is a 1-way direct binding between `host-element`'s properties and the template's contents.

# Expressions in binding annotations

Currently there’s no general support for expressions in binding annotations. The two exceptions are:

1) Negation using `!`:. For example: `<template><div hidden="{{!enabled}}"></div></template>`
2) Computed bindings
3) Compound bindings

### Computed bindings

You can define a function that will be called to compute a value.

**Computed binding is always one-way**

    <dom-module id="host-element">
      <template>
        My name is <span>{{computeFullName(first, last)}}</span>
      </template>
      <script>
        Polymer({
          is: 'host-element',
          properties: {
            first: String,
            last: String
          },
          computeFullName: function(first, last) {
            return first + ' ' + last;
          }
        });
      </script>
    </dom-module>

If a computed binding has no dependent properties (e.g. `<span>{{randomNumber()}} `), it is only evaluated once.

Finally, if you have to pass a literal value as a parameter, make sure you escape any comma `,` with `\,`.


### Binding to attributes (Annotated attribute binding)

**Annotated attribute binding is always one-way**

In the vast majority of cases, binding data to other elements should use property binding described above, where changes are propagated by setting the new value to the JavaScript property on the element. However, sometimes you need to set an attribute on an element, as opposed to a property. These include when attribute selectors are used for CSS or for interoperability with elements that require using an attribute-based API. To bind to an attribute, use `$=` rather than `=`. This results in a call to: `element.setAttribute(attr, value);` as opposed to: `element.property = value;`

    <template>
      <!-- Attribute binding -->
      <my-element selected$="{{value}}"></my-element>
      <!-- results in <my-element>.setAttribute('selected', this.value); -->

      <!-- Property binding -->
      <my-element selected="{{value}}"></my-element>
      <!-- results in <my-element>.selected = this.value; -->

    </template>

Values are serialized according to the value’s current type, like for attribute serialization. Again, as values must be serialized to strings when binding to attributes, it is always more performant to use property binding for pure data propagation.

#### Special cases: when bind to attributes

There are a handful of extremely common native element attributes which can also be modified as properties. Due to cross-browser limitations with the ability to place binding braces {{...}} in some of these attribute values, as well as the fact that some of these attributes map to differently named JavaScript properties, it is recommended to always use attribute binding (using $=) when binding dynamic values to these specific attributes, rather than binding to their property names.

Normal attribute assignment to static values:

    <!-- class -->
    <div class="foo"></div>
    <!-- style -->
    <div style="background: red;"></div>
    <!-- href -->
    <a href="http://foo.com">
    <!-- label for -->
    <label for="bar"></label>
    <!-- dataset -->
    <div data-bar="baz"></div>

Attribute binding to dynamic values (use `$=`):

    <!-- class -->
    <div class$="{{foo}}"></div>
    <!-- style -->
    <div style$="{{background}}"></div>
    <!-- href -->
    <a href$="{{url}}">
    <!-- label for -->
    <label for$="{{bar}}"></label>
    <!-- dataset -->
    <div data-bar$="{{baz}}"></div>

**TODO: Include path bindings here, once the array binding dilemma is resolved and observers on paths is practiced**

https://github.com/Polymer/polymer/issues/2122

https://github.com/Polymer/polymer/issues/2127

## Behaviours

AS IS IN OFFICIAL DOCUMENTATION

## Utility function

AS IS IN OFFICIAL DOCUMENTATION

**TODO: Check that API is completely committed to memory**

## Global settings

AS IS IN OFFICIAL DOCUMENTATION

**TODO: Memorise how to use shadow DOM

## Experiental features

AS IS IN OFFICIAL DOCUMENTATION

**TODO: Check that this is fully understood, memorised, and tested**

## API REFERENCE

AS IS IN OFFICIAL DOCUMENTATION

**TODO: Check that API is completely committed to memory**
