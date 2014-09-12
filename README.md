#Austere Javascript

A rigorous and opinionated style guide.

## Strict mode is used everywhere.

If you can't use strict mode for some reason, provide a brief justification in your comments. Otherwise, the top of every file should look like this:

```js
'use strict';

// The rest of your file.
```
**Why?**

1. Code that fails to run in strict mode may be reliant on the (awful) silent error behavior of standard ES5.
2. ES5 code written in strict mode will be easier to convert to ES6, since you won't have issues with ambiguous syntax or new OO keywords that are coming to javascript (*e.g.,* ```private, protected, implements,``` etc.).

Do see the [MDN docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope/Strict_mode) if any of this is news.

## Syntax is clear and explicit.

Brackets and semicolons are required.

### White space is consistent.

1. One blank line above the ```var``` keyword, one above the comment block, one below the ```var``` statement, and no others within a function body.
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
  Additional white space usually indicates that the logic should be in separate functions. The only exception is for hot code, where it's important to avoid incurring additional function call overhead.
2. Indentation depth is 2 spaces. No tabs.
3. No space between parens and arguments.

### Quotes are consistent.

Use single quotes for javascript. Double quotes appear *only within* single quotes.

### Structures known to invite confusion are avoided.

1. Ternary conditionals are generally dispreferred in favor of if-then branched code.
2. Switch statements are not used, since fall-through can lead to conditions that are difficult to anticipate.

## Variables, functions, and comments

### Variables are named for readability.

Name every variable with the shortest possible unnabbreviated, grammatically sensible phrase that suits its value.

### Variable declarations are comma-first.

```js
var verb = function () {
  
  var counter
    , result
    , temp;
  
  // ...
};
```

This tends to prevent accidental declaration of globals. Sole caveat: variables containing a function should have a separate `var` statement.

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

### Comments are jsDoc style.

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

Function declarations store an anonymous function in a variable, as follows:

```js
var verb = function (arg) {
  // ...
};
```

## Objects, methods, and members
### Reference preservation is preferred.
#### Arrays are modified in-place.

When modifying an array, a developer's instinct is to loop the array and construct a new one, and then reassign the variable from the old to the new array. If accessed from different parts of a program, repeated reassignment of an object (Array) variable can lead to broken references and memory leaks.

When changing an array, use ```Array.prototype.splice()``` ([MDN docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/splice)).

#### Objects are modified in-place.

Modify objects with ```delete``` and key (re)assingment.

### Non-public members and methods are underscore-prefixed.

In your API, a member or method might be named ```thing.verb```. If it is not part of the API, it should be referenced as ```thing._verb```. This indicates to a developer writing client code that she should not rely on said member/method because its value will change as the *internal* needs of the object dictate.

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

### Initialization is deferred.

Including logic within the constructor reduces flexibility because it will be impossible to invoke that logic again, while maintaining state that isn't directly affected by it.

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
    that.parent = parent;
    that._node = node;
    return that;
  }
};
```

*All* member variables are set to ```null``` in the function block. An ```init()``` method should be defined to assign values to these and invoke other methods. The ```init()``` method is not invoked from within the constructor, but from client code, as follows:

```js
new Thing().init(parent, node);
```

### Public methods are chainable.

An API function should return a reference to the object on which it executes. For example, when ```new Dog.init()``` is invoked, ```init()``` returns a reference to the ```Dog``` object, allowing other methods to be chained to ```init()``` the call, resulting in readable client code: 

```js
new Dog().init('pug').bark();
```

### ```this``` always equals ```that```.

Do not use the ```this``` keyword to reference an object from within its methods. When inside of a function nested within the method (*e.g.,* a callback), ```this``` won't refer to the object. Aliasing the object within each function scope is an error-prone approach. Just set ```that``` equal to ```this``` at the top of every method body.

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
    that.parent = parent;
    that._node = jqNode;
    that._node.on('touchstart', function () {
      that._node.empty();
    });
    return that;
  }
};
```

One can now refer to ```that._node``` from within the ```$.on()``` callback scope, making for a pleasant consistency.

### Composed objects have a reference chain.

When one object contains others, which in turn contain others, it can be very helpful when implementing a hotfix to have the ability to reference any part of the object from any other part by the following means: ```that.parent.parent._someThing._someMethod()```.

Make the first argument of the ```init()``` method a reference to the parent object, and store it in a ```.parent``` member. These references are not to be abused; unless creating a hotfix, do not use them to move more than one level up the chain. Any need to go further should be recognized as a code smell.

### Dynamic objects have a custom ```toString()``` method.

Dynamic objects have a custom method to create a string representation of the object (for storing and reviving). In cases where ```JSON.stringify()``` is sufficient, write the ```toString()``` method as a trivial wrapper, for consistency's sake.

### Dynamic objects can be revived.

Writing revival methods for large composed objects can be difficult, since one has to revive each dynamic descendant, missing none. To avoid messes, all dynamic objects have a  ```revive()``` method, which can be invoked by the parent object's ```revive()``` method.

## Code is well-organized.
### Node.js and CommonJS

In Node, you have no choice but to use the CommonJS module standard. Keep your files small, containing minimal individual modules.

```js
var verb = function () {
  // ...
}

module.exports = verb;
```

### AMD is preferred in the browser.

In the browser, AMD is preferred. So use [RequireJS](http://requirejs.org/) where suitable (but it's not always suitable, *e.g.,* when using Angular).

```js
// Async module definition with RequireJS
define('moduleName', ['jquery', 'tweenLite'], function ($, TweenLite) {
  
  // ...
  return {
    // ...
  }
});
```

Fallback to CommonJS modules (use [Browserify](http://browserify.org)). If that won't do, fallback to modules defined with an object literal, but don't make it a common practice.

## Code is prepared for production.

Create an uglified and concatenated build of all javascript that will be loaded project-wide. Page-specific files should be uglified as well. Use Grunt or Gulp.
