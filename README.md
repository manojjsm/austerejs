#Austere Javascript

A rigorous style guide for javascript.

## Prefatory Comment

Some of the practices endorsed here are something of a pain, but that isn't a worthy objection if we're concerned about producing moderately large, complex, and reasonably maintainable codebases.

## Strict mode is used everywhere.

... And if you cannot use strict mode for some reason (for example, the strictly interpreted code fails to run in a targeted browser), provide a brief justification in your comments. Otherwise, the top of every file should look like this:

```js
"use strict";

// The rest of your file.
```

## Functions, variables, comments

### Variable declarations are comma-first.

With the sole caveat that variables containing functions should be declared with a separate `var` statement.

```js
var verb = function () {
  
  var counter
    , result
    , temp;
  
  ...
};
```

##### Q: Why comma-first? After all, it's ugly.

A: It prevents a missed comma from causing the accidental declaration of global variables, which can cause troubles that are a challenge to debug. (If you forget a comma, javascript will insert a semicolon, and then your remaining vars are just ignored; in your code, you'll use those ignored vars and they'll be globals. It happens.)

Q: Why not multi-var then? A: To enforce the declaration of everything at the top of the function, in keeping with javascript hoisting.

### Variable declaration and initialization are separate.

```js
var verb = function () {
  
  var counter
    , result
    , temp;
  
  result = [];
  temp = [];
  counter = 0;
   
  ...
  
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
   
   ...

};
```

## Objects, methods, and members

### Non-public members and methods are underscore-prefixed.

In your API, a member or method may be named: ```thing.verb```. However, if it is not part of the API, it should be referenced as ```thing._verb```. This indicates to a developer writing client code that she should not rely on said member or method because its value will change as the *internal* needs of the object dictate.

### Constructors are not self-initializing.

Cramming initialization logic into the constructor itself reduces the flexibility of the code by eliminating the possibility of re-initializing the object while maintaining data and context not affected by the initialization logic. It limits your options, in short.

```js
var Thing = function () {
  
  /**
   * @constructor
   * Creates, stores, and mutates the Thing application state.
   */
  
  this.parent = null;
  this._node = null;
  this._markup = null;
};

Thing.prototype = function () {
  init: function (parent, node) {
    
    /**
     * @method
     * Initializes a Thing object.
     * @param {object}
     * @param {object} A DOM node
     */
    
    var that;
    
    that = this;
    that._parent = parent;
    that._node = node;
  }
};

```
*All* member variables that are to be referenced are set to ```null``` in the function block of the constructor. An ```init``` method should be defined to assign values to these and invoke other methods to decorate the object. The ```init``` method should not be invoked from within the constructor, but from client code, as follows:

```js
new Thing().init(parent, node);
```

### Public methods are chainable.

Whenever possible, an API function should return a reference to the object on which the API calls generally execute. For example, if ```new Dog.init()``` is invoked, since the ```init``` method is public, it should return a reference to the ```Dog``` object. That way, other public methods can be chained to ```init()``` calls, resulting in terse *but highly readable* client code like this: 

```js
new Dog().init("pug").bark();
```

### ```this``` always equals ```that```.

Do not use the ```this``` keyword to reference the parent object from its methods. If inside of another function within the method (typically an anonymous callback), ```this``` won't reference the parent object any longer. Changing the means of referencing the object within each function scope is a messy, repetitive, error-prone process. Just set ```that``` equal to ```this``` at the top of every method body.

```js
Thing.prototype = function () {
  init: function (parent, node) {
    
    /**
     * @method
     * Initializes a Thing object.
     * @param {object}
     * @param {object} A DOM node
     */
    
    var that;
    
    that = this;
    that._parent = parent;
    that._node = node;
    $(that._node).on("touchstart", function () {
      that._node.empty();
    });
  }
};
```

Notice that one can refer to ```that._node``` from within the ```$.on()``` callback function's scope; this makes for a pleasant consistency.
