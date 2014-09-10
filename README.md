#Austere Javascript

A rigorous and opinionated style guide.

## Strict mode is used everywhere.

If you can't use strict mode for some reason (for example, the strictly interpreted code fails to run in a targeted browser), provide a brief justification in your comments. Otherwise, the top of every file should look like this:

```js
'use strict';

// The rest of your file.
```
**Why?**

1. Code that fails to run in strict mode is usually just sloppy, and may be reliant on the (awful) silent error behavior of standard ES5.
2. ES5 code written in strict mode will be easier to convert to ES6, since you won't run into issues with ambiguous syntax or have to debug conflicts with classical OO keywords that are coming to javascript (*e.g.,* ```protected, constant, implements, private, interface,``` etc.).

Do see the [MDN docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope/Strict_mode) if any of this is news.

## Variables, functions, and comments

### Variables are named for readability.

Name every variable with the shortest possible unnabbreviated, grammatically sensible phrase that suits its value. So ```headerOffset``` is good, while ```distance``` is bad if you're storing the distance of the header from the top of its container, for example.

### Variable declarations are comma-first.

With the sole caveat that variables containing functions should be declared with a separate `var` statement.

```js
var verb = function () {
  
  var counter
    , result
    , temp;
  
  // ...
};
```

##### Q: Why comma-first? After all, it's ugly.

A: It prevents a missed comma from causing the accidental declaration of global variables, which can cause troubles that are a challenge to debug. (If you forget a comma, javascript will insert a semicolon, and then your remaining vars are just ignored; in your code, you'll use those ignored vars and they'll be globals.)

Q: Why not multi-var then? A: To enforce the declaration of everything at the top of the function, in keeping with javascript's hoisting.

### Variable declaration and initialization are separate.

```js
var verb = function () {
  
  var counter
    , result
    , temp;
  
  result = [];
  temp = [];
  counter = 0;
  // ...
};
```

### White space is consistent.

There shall be one line of white space above the ```var``` keyword and above the comment block, plus one line of white space below the end of the ```var``` statement and below the comment block. Blank lines are not permitted elsewhere within a function block.

```javascript

var verb = function (arg) {

  /**
   * [comment block]
   */

  var thing
    , otherThing;
  
  // ...
};

```

If the need to include white space is felt, it's probably an indication that the logic above and below the white space should be in separate functions. The only exception is for hot code, where a great deal of logic has to be kept in one function to avoid incurring additional function call overhead. If a function will be called hundreds of thousands of times, ignore the white space rule.

### Quotes and spaces are consistent.

1. Always use single quotes for javascript. Only use double quotes when inside of single quotes.
2. Indentation depth is exactly 2 spaces. No tab characters.
3. No space between parens and arguments. That is:

```js
// Yes.
var verb = function (arg) {
  // ...
};

// No.
var verb = function ( arg ) {
  // ...
};
```

### Comments are rendered in jsDoc style.

It is not important that the comments be parsable by jsDoc, since jsDoc3 still fails to read and render comments properly when the functions they explain are within closures.

Notice that comments are placed *inside* the function body, at the top. Not above the function body.

```js
var verb = function (str, obj) {

  /**
   * @function
   * Breif, 1-sentence description of the function here.
   * @param {string}
   * @param {object}
   * @return {string}
   * @note ... (optional)
   * @todo ... (optional)
   */
   
   // ...
};
```

### Function declarations are safe.

Function declarations always store an anonymous function in a variable, as follows:

```js
var verb = function (arg) {
  // ...
};
```

## Objects, methods, and members
### Reference preservation is preferred.
#### Arrays are modified in-place.

In most cases, when modifying a javascript array, the developer's first instinct is loop the array and construct a new one, and then replace the old with the new by reassigning the variable containing the array. If the array is being accessed by a variety of parts of the software, repeated reassignment will eventually create broken references and memory leaks that are a bear to debug.

So, when removing or adding items to an array, use ```Array.prototype.splice()``` if possible. It's usually possible. ([MDN docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/splice)).

#### Objects are modified in-place.

As with arrays, the loop-and-replace method is the common one. Always prefer modification using the ```delete``` directive and/or new key-value assingment. Unlike array modification, object modification in javascript is quite well-behaved.

### Non-public members and methods are underscore-prefixed.

In your API, a member or method may be named: ```thing.verb```. However, if it is not part of the API, it should be referenced as ```thing._verb```. This indicates to a developer writing client code that she should not rely on said member or method because its value will change as the *internal* needs of the object dictate.

### Constructor declarations are safe and named.

Because it is often useful to be able to check the type of an object without duck typing, constructor functions are named (in addition to being safely declared). This allows for easy examination of ```constructor.name``` if the constructor property is defined on the object's prototype chain.

```js
var Thing = function Thing () {
  
  /**
   * @constructor
   * Creates, stores, and mutates the Thing application state.
   */

};

Thing.prototype = function () {
  constructor: Thing
};
```

Notice how little effort is required to determine the object's type:
```js
var thing;

thing = new Thing();
console.log(thing.constructor.name); // 'Thing'
```

### Constructors are not self-initializing.

Including initialization logic (any logic, in fact) within the constructor body reduces flexibility by eliminating the possibility of re-running that logic while maintaining state that isn't affected by said logic. That's an unnecessary limitation.

```js
var Thing = function Thing () {
  
  /**
   * @constructor
   * Creates, stores, and mutates the Thing application state.
   */
  
  this.parent = null;
  this._node = null;
  this._markup = null;
};

Thing.prototype = function () {
  
  constructor: Thing,
  
  init: function (parent, node) {
    
    /**
     * @method
     * Initializes a Thing object.
     * @param {object}
     * @param {object} A DOM node
     * @return {Thing}
     */
    
    var that;
    
    that = this;
    that._parent = parent;
    that._node = node;
    return that;
  }
};
```

*All* member variables that are to be referenced are set to ```null``` in the function block of the constructor. An ```init()``` method should be defined to assign values to these and invoke other methods to decorate the object. The ```init()``` method should not be invoked from within the constructor, but from client code, as follows:

```js
new Thing().init(parent, node);
```

### Public methods are chainable.

Whenever possible, an API function should return a reference to the object on which the API calls generally execute. For example, if ```new Dog.init()``` is invoked, since the ```init``` method is public, it should return a reference to the ```Dog``` object. That way, other public methods can be chained to ```init()``` calls, resulting in terse *but highly readable* client code like this: 

```js
new Dog().init('pug').bark();
```

### ```this``` always equals ```that```.

Do not use the ```this``` keyword to reference the parent object from its methods. If inside of another function within the method (typically an anonymous callback), ```this``` won't reference the parent object any longer. Changing the means of referencing the object within each function scope is a messy, repetitive, error-prone process. Just set ```that``` equal to ```this``` at the top of every method body.

```js
Thing.prototype = function Thing () {
  
  constructor: Thing,
  
  init: function (parent, jqNode) {
    
    /**
     * @method
     * Initializes a Thing object.
     * @param {object}
     * @param {object} A DOM node
     * @return {Thing}
     */
    
    var that;
    
    that = this;
    that._parent = parent;
    that._node = jqNode;
    that._node.on('touchstart', function () {
      that._node.empty();
    });
    return that;
  }
};
```

Notice that one can refer to ```that._node``` from within the ```$.on()``` callback function's scope; this makes for a pleasant consistency.

##### Why not create bound functions instead of aliasing?

Some developers recommend creating [bound functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) to prevent ```this``` from referring to an unexpected value. In general this is great, but it's unsupported in IE10 and only gained support on Android browser with Android 4.0. While doing this on the server (Node/Rhino/TeaJS) is fine, the polyfill for the ```bind()``` method is not simple, and since the aliasing solution works well in general, it is preferred.

### Composed objects have a reference chain.

When one object contains others, which in turn contain others, it can be very helpful when implementing a hotfix to have the ability to reference any part of the object from any other part by the following means: ```that.parent.parent._someThing._someMethod()```.

Always make the first argument of the ```init()``` method a reference to the parent object, and store the reference in a ```.parent``` member variable. Obviously, these references are not to be abused. If you're not creating a hotfix, do not use them to move more than one level up or down the chain from within a single method. Any need to go further should be recognized as a code smell.

### Dynamic objects can be revived.

Writing revival methods for large composed objects can be difficult, since one has to revive each dynamic descendant, missing none. To avoid messes, all dynamic objects have a  ```revive()``` method, which can be invoked by the parent object's ```revive()``` method.

## Code is well-organized.
### Node.js and CommonJS

In Node, you have no choice but to use the CommonJS module standard. Keep your files relatively small, containing components that make sense as individual modules. Module granularity has no lower limit (as scope decreases, reusability increases in corresponding degree).

```js
var verb = function () {
  // ...
}

module.exports = verb;
```

### AMD is preferred in the browser.

In the browser, asynchronous module definition (AMD) is the preferred solution, as dependency injection completely removes the puzzle of when to load which scripts, while keeping things asynchronous. So use [RequireJS](http://requirejs.org/) where suitable (but it's not always suitable, *e.g.,* when using Angular).

```js
// Async module definition with RequireJS
define('moduleName', ['jquery', 'tweenLite'], function ($, TweenLite) {
  
  // ...
  return {
    // ...
  }
});
```

Fallback to CommonJS modules (usually you'll want to use [Browserify](http://browserify.org)). If that won't do, fallback to modules defined within a simple object literal to avoid using globals, but don't make this a common practice.

## Code is prepared for production.

Create production builds of all javascript that will be loaded project-wide. Concatenate, minify and mangle (=uglify). Page-specific files should be minified and mangled as well. Use [Grunt](http://gruntjs.com/) to do this from the command line.

## Switch statements are not used.

Fall-through occasionally leads to conditions that are difficult to anticipate and think through.
