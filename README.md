# moog

[![Build Status](https://travis-ci.org/punkave/moog.svg?branch=master)](https://travis-ci.org/punkave/moog)

Moog is for creating objects that can be subclassed.

```javascript
var moog = require('moog')();

moog.define('baseclass', {
  color: 'blue',
  // sync constructor
  construct: function(self, options) {
    self._options = options;

    self.jump = function(howHigh) {
      return 'I jumped ' + howHigh + ' pixels high';
    };
  }
});

moog.define('subclass', {
  color: 'red',
  // async constructor
  construct: function(self, options, callback) {
    return goGetTheCandy(function(err, results) {
      if (err) {
       return callback(err);
      }
      self.candy = results;
      return callback(null);
    });
  }
});

moog.create('subclass', { age: 20 }, function(err, obj) {
  assert(obj._options.color === 'red');
  assert(obj.jump(5) === 'I jumped 5 pixels high');
});
```

`moog` synthesizes objects with full support for inheritance. You can define them with any combination of synchronous and asynchronous constructors, specify default options easily, and modify options before they are seen by base classes.

### Factory function

To create an instance of `moog`:

```javascript
var moog = require('moog')();
```

You may also pass options:

```javascript
var moog = require('moog')({
  defaultBaseClass: 'superclass'
});
```

### moog.define(type, definition)

Defines a new type. `type` is a string. A definition looks like:

```javascript
moog.define('baseclass', {
  // Set the default value of an option
  color: 'red',
  // Simple synchronous constructor
  construct: function(self, options) {
    self._options = options;
  }
});
```

`construct` is invoked by `create` at a specific time, see below. `beforeConstruct` and `extend` are also specialized as described below. All other properties are treated as defaults for the `options` object provided when constructing an instance of the type.

To subclass another type, just `extend` it by name in the definition of your subclass:

```javascript
moog.define('subclass', {
  // Change the default value of an option
  color: 'blue',
  extend: 'baseclass'
});
```

**If you define the same class twice** without setting `extend` the second time, an *implicit subclass* is created. The new version subclasses the old one, effectively "patching" it with new options and behavior without having to redefine everything. All other types that subclass that name now subclass the new version.

#### Defining many types at once

For convenience, you may pass an object containing properties that define many different types:

```javascript
moog.define({
  // This is equivalent to calling moog.define for each of these types
  'baseclass': {
    // See above for example of a definition
  },
  'subclass': {
    // See above for example of a definition
  }
});
```

### moog.redefine(type, definition)

Explicitly replaces any previous definition of `type` with a new one. Does *not* subclass the old type. If there was no old definition, this method is equivalent to `moog.define`.

### moog.isDefined(type)

Returns true if the type is defined, whether explicitly or via the autoloader option. That is, `moog.create` will succeed for `type`, provided that the constructor does not signal an error.

### moog.create(type, options, /* callback */)

Creates an object of the specified `type`, passing `options`, which may be modified first by the default option values given in type definitions beginning with the deepest subclass, then by any `beforeConstruct` methods present, which are called for the deepest subclass first. Then the `construct` methods are called, if present, starting with the base class and ending with the final subclass.

If a callback is given, the callback receives the arguments `err, obj` where `obj` is the object created. If a callback is not given, an exception is thrown in the event of an error, otherwise the object is returned. **There must not be any asynchronous beforeConstruct or construct methods if you create the object synchronously.** `moog` will throw an exception in that situation.

`obj` will always have a `__meta` property, which contains an array of metadata objects describing each module in the inheritance chain, starting with the base class. The metadata objects will always have a `name` property. [moog-require](https://github.com/punkave/moog-require) also provides `dirname` and `filename`. This is useful to implement template overrides, or push browser-side javascript and styles defined by each level.

### moog.createAll(globalOptions, specificOptions, /* callback */)

Creates one object of each type that has been defined via `moog.define` or via the `definitions` option given when configuring `moog`. Only types explicitly defined in this way are created, but they may extend types available via the `autoloader` option given when configuring `moog`.

The options passed for each object consist of `globalOptions` extended by `specificOptions[type]`.

If you pass a callback, it will receive an error and, if no error, an object with a property for each type name. If you do not pass a callback, such an object is returned directly. **If you do not pass a callback, then you must not define any types that have asynchronous `construct` and `beforeConstruct` methods.**

## Using moog in the browser

`moog` works in the browser, provided that `async` and `lodash` are already global in the browser. `moog` defines itself as `window.moog`. Currently it is not set up for use with browserify but this would be trivial to arrange.
