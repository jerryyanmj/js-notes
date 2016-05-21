# Javascript Notes by example

This is some notes about Javascript and it is largely based on examples of code.  The best way of evaluating and playing with the example code is by using the Chrome Developer Tools.  In MAC OSX, CMD+OPT+I is the shortcut and Shift+Enter is the trick to enable multi-line statements.

### Javascript is functional.
* Functions are first-class objects in Javascript Language.
* Javascript functions are higher-ordered functions.  They can take a function as an input and they can also return a function as an output.  Sorting is an example of higher-ordered function.
```javascript
var values = [1,4,3,23,11,67,22,12,11];
values.sort(function(v1,v2) {return v2 - v1;});
```
* Javascript event loop is single-threaded.  Function invocations, including callback functions, are registered in the event loop and gets executed in this single thread queue **(FIFO)**.
```javascript
function useless(callback) {return callback();}
```

### Fundamentals about function declarations and Scoping
Example 1 - Declarations and name parameter
```javascript
/*
  Declares a named function.  The name is available throughout the current scope
  and is implicitly added as a property of window.
*/
function isNimble() {return true;}
// Test that the window property is established.
typeof window.isNimble === "function"; // true
// Test that the name property of the function is set.
isNimble.name === "isNimble"; // true

/*
  Declares an anonymous function that is assigned to the variable canFly.
  It is a window property and the name of the function is empty.
*/
var canFly = function() {return true;}
typeof window.canFly === "function"; // true
canFly.name === ""; // true
// Declares another anonymous function that is assigned to the window property.
window.isDeadly = function() {return true;};
typeof window.isDeadly === "function" // true

function outer() {
  // function inner can be referenced before its declaration.
  typeof inner === "function"; // true
  function inner() {}
  // function inner can be referenced after its declaration.
  typeof inner === "function"; // true
  // function inner is not a global scoped function.
  window.inner === undefined; // true
}

outer();  // outer can be referenced in the global scope, but not inner.
window.inner === undefined;

window.jerry = function chloe() {return true;}
window.jerry.name === 'chloe'; // true
```
Example 2 - Variable Scoping
```javascript
// All our tests assert that each item is in scope, so all but the tests for items that can be forward-referenced will fail.
// At top level, only outer is defined, inner is not defined in the same scope as outer.
console.log('|---------------- BEFORE OUTER ----------------|');
console.log(typeof outer==='function', typeof inner==='function', typeof a==='number', typeof b==='number', typeof c==='number'); // true false false false false
function outer() {
  // The outer function is still in scope as is the inner function, which is defined within the outer function.
  // Functions can be forward-referenced, but not variable declarations, so all other tests fail.
  console.log("|---------------- INSIDE OUTER, BEFORE a ----------------|")
  console.log(typeof outer==='function', typeof inner==='function', typeof a==='number', typeof b==='number', typeof c==='number'); // true true false false false
  var a = 1;
  // Functions outer/inner continue to be in scope.  Since a is declared before, test for a is valid now.
  console.log("|---------------- INSIDE OUTER, AFTER a ----------------|")
  console.log(typeof outer==='function', typeof inner==='function', typeof a==='number', typeof b==='number', typeof c==='number'); // true true true false false
  function inner(){}

  var b = 2;
  // Functions outer/inner continue to be in scope.  Since a and b are declared before, tests for a and b are valid now.
  console.log("|---------------- INSIDE OUTER, AFTER inner() AND b ----------------|");
  console.log(typeof outer==='function', typeof inner==='function', typeof a==='number', typeof b==='number', typeof c==='number'); // true true true true false

  if (a ==1) {
    // Functions outer/inner continue to be in scope, so are a and b.  Since c is not declared yet, test for c fails.
    console.log("|---------------- INSIDE OUTER, INSIDE if ----------------|");
    console.log(typeof outer==='function', typeof inner==='function', typeof a==='number', typeof b==='number', typeof c==='number'); // true true true true false
    var c = 3;
  }
  // Variable declarations extend from the point of declaration to the end of the function, crossing any block boundaries.
  console.log("|---------------- INSIDE OUTER, OUTSIDE if ----------------|");
  console.log(typeof outer==='function', typeof inner==='function', typeof a==='number', typeof b==='number', typeof c==='number'); // true true true true true

  outer();
  // Same as test 1
  console.log("|---------------- AFTER OUTER ----------------|");
  console.log(typeof outer==='function', typeof inner==='function', typeof a==='number', typeof b==='number', typeof c==='number');// true false false false false
}
```

### Function parameters
* Functions can take a list of parameters.  If the number of arguments is different from that of a function declaration, it is fine.
* 2 implicit parameters are always passed to functions
  * arguments - a collection of all the arguments and it has a length property.
  * this - function execution context.
  ```javascript
  function test(t1,t2) {
    console.log(t1,t2,arguments);
  }
  ```

### Function invocation
There are 4 different ways of invoking a function.
* Invoked as a function
  ```javascript
  function test(){};
  test(); // Invoke as function

  var test2 = function() {};
  test2();
  ```
* Invoked as a method
  ```javascript
  var obj = {};
  obj.test = function() {};
  obj.test(); // Invoke as a method of obj.
  ```
  Example to illustrate the difference between function and method invocations
  ```javascript
  function doSth(){return this;}
  console.log(doSth() === window); // true

  // doSthElse is variable which only reference the original function doSth.
  var doSthElse = doSth;
  console.log(doSthElse() === window); // true

  // Invoke the function through aMethod property, thus invoking it as a member of anObj.
  // The function context is no longer window, but is anObj.  That's object oriented.
  var anObj = {
    aMethod: doSth
  }
  console.log(anObj.aMethod() === window); // false
  console.log(anObj.aMethod() === anObj); // true

  var anotherObj = {
    aMethod: doSth
  }
  console.log(anotherObj.aMethod() === window); // false
  console.log(anotherObj.aMethod() === anObj); // false
  console.log(anotherObj.aMethod() === anotherObj); // true
  ```
* Invoked as a constructor
  Example - Illustrate the difference between constructor and function
  ```javascript
  function AnObj() {
    this.doSomething = function() { return this; };
  }

  var obj1 = new AnObj();
  var obj2 = new AnObj();
  console.log(obj1.doSomething() === obj1); // true
  console.log(obj2.doSomething() === obj1); // false
  console.log(obj2.doSomething() === obj2); // true

  var weirdObj = AnObj();
  console.log(weirdObj.doSomething() === obj1); // Invalid because doSomething is not defined on the weirdObj itself.  It is on window.
  console.log(window.doSomething() === window) // true
  ```
* Invoked with apply/call methods
  * For every function, apply() and call() exist.
  * apply() takes two arguments: function context object and an array of values for invocation arguments.
  * call() also takes parameters: function context object and values for invocation arguments in consecutive order (NOTE: not array).


  Example apply/call - 1
  ```javascript
  function test(t1,t2) { console.log(this,t1,t2,arguments); }
  test.apply(window, [1,2,3]);
  test.call(window, 1,2,3);
  ```

  Example - apply/call - 2
  ```javascript
  function juggle() {
    var result = 0;
    for (var i=0; i<arguments.length; i++) {
      result += arguments[i];
    }
    this.result = result;
  }
  var obj1 = {};
  var obj2 = {};
  juggle.apply(obj1, [1,2,3,4]);
  juggle.call(obj2, 5,6,7,8);
  console.log(obj1);
  console.log(obj2);
  ```

  Example illustrating setting a function context
  ```javascript
  function forEach(list,callback) {
    for (var i=0; i<list.length; i++) {
      callback.call(list[i],i);
    }
  }

  forEach([1,2,3,4,5], function(index) { console.log(this*10); });
  ```
### Anonymous Functions
* Store it in variable
* Establish it as a method of an object
* Callback Functions

Examples
```javascript
// Store a function with no name to a variable
window.onload = function() {};

// Create a function as a method
var obj = {
  doSth: function() { console.log("Method"); }
}
obj.doSth();

// Callback example
setTimeout(function() { console.log("callback"); }, 500);
```

### Recursion
A set of examples below are used to illustrate recursion, its issues and how they can be resolved.
Recursion with named function
```javascript
// create a named function which is called recursively.
function chirp(n) {
  return n>1 ? chirp(n-1) + "-chirp":"chirp";
}
console.log(chirp(10));
```
Recursion with a method of an object
```javascript
var ninja = {
  chirp: function(n) { return n>1 ? ninja.chirp(n-1) + "-chirp":"chirp"; }
}
console.log(ninja.chirp(5));
var ninja2 = {chirp: ninja.chirp};
ninja = {};
ninja2.chirp(3);
```
__Everything should up to now.  Issue starts to happen when ninja is assigned to an empty object.  Reference to ninja.chirp becomes invalid after ninja is empty.__
Improve the chirp by using **function context**.
```javascript
var ninja = {
  chirp: function(n) { return n>1 ? this.chirp(n-1) + "-chirp":"chirp"; }
}
console.log(ninja.chirp(5));
var ninja2 = {chirp: ninja.chirp};
ninja = {};
console.log(ninja2.chirp(4));
```
Problem resolved.  Yet, new problem is introduced.  The solution relied upon the fact that the function would be a method named chirp() of any object within which the method is defined.  
* What if the properties don't have the same name?  
* What if one of the references to the function isn't even an object property?

Inline function makes the example more elegant.  **We assign the name signal to the inline function and use that name for the recursive reference within the function body.**
```javascript
// We assign the name signal to the inline function and use that name for the recursive reference within the function body.
var ninja = {
  chirp: function signal(n) { return n>1 ? signal(n-1) + "-chirp":"chirp"; }
}
console.log(ninja.chirp(5));
var ninja2 = {chirp: ninja.chirp};
ninja = {};
console.log(ninja2.chirp(4));
```
**Inline function identify** - Even though inline functions can be named, those names are only visible within the functions themselves.  This is why top-level functions are created as methods on window.  Without the window properties, we would have no way to reference the function.
```javascript
// We assign the name signal to the inline function and use that name for the recursive reference within the function body.
var ninja = function myNinja() {
  console.log(ninja == myNinja)
}
// Invoke ninja
ninja();
console.log(myNinja); // undefined
console.log(ninja.name) // myNinja
```
**callee** property - A special approach which isn't future proof.
```javascript
var ninja = {
  chirp: function(n) { return n>1 ? arguments.callee(n-1) + "-chirp":"chirp"; }
}
console.log(ninja.chirp(5));
var ninja2 = {chirp: ninja.chirp};
ninja = {};
console.log(ninja2.chirp(4));
```
**Coding practice tip** - It is always a good practice to have semicolons at the end of all statements, and especially after variable assignments.  When compressing code, properly placed semicolons will allow for greater flexibility in compression techniques.

### Function as object
Just like any object, we can attach any properties to a Javascript function.
Example - Illustrates how we can attach properties to a function
```javascript
var obj = {};
var fn = function() {};
obj.property = "hello";
fn.property = "world";
```
Example - Storing function by using the technique of setting property to a function.  In this example, we set an identifier to the function object itself so that the property can be used to dentermine if a function is stored in our toolbox or not.
```javascript
var toolbox = {
  nextId: 1,
  cached: {},
  add: function(fn) {
    if (!fn.id) {
      fn.id = toolbox.nextId++;
	  // !! constrcut is a simple way of turning any Javascript expression into its Boolean equivalent.
      return !!(toolbox.cached[fn.id] = fn);
    }
    return false;
  }
};
function tool(){};

console.log(toolbox.add(tool)); // true
console.log(toolbox.add(tool)); // false - it is added already.
```
Example - Storing previously calculated values inside the function
```javascript
function isPrime(value) {
  if (!isPrime.answers) isPrime.answers = {};
  if (isPrime.answers[value] != null) {
    // console.log("The answer was cached!");
    return isPrime.answers[value];
  }

  var prime = value != 1;
  for (var i = 2; i < value; i++) {
    if (value % i == 0) {
	  prime = false;
	  break;
	}
  }
  return isPrime.answers[value] = prime;
}
```



var elems = {
  length: 0,
  add: function(elem) {
    Array.prototype.push.call(this, elem);
  },
  gather: function(elem) {
    this.add(elem);
  }
};


sss
































*This text will be italic*

_This will also be italic_
**This text will be bold**
__This will also be bold__
*You **can** combine them*


# This is an <h1> tag
## This is an <h2> tag
###### This is an <h6> tag


As Grace Hopper said:
> I’ve always been more interested
> in the future than in the past.

http://github.com - automatic!
[GitHub](http://github.com)

\*literal asterisks\*


```javascript
function test() {
 console.log("look ma’, no spaces");
}
```
#1
github-flavored-markdown#1
defunkt/github-flavored-markdown#1

* Item 1
* Item 2
 * Item 2a
 * Item 2b

![GitHub Logo](/images/logo.png)
Format: ![Alt Text](url)

First Header | Second Header
------------ | -------------
Content cell 1 | Content cell 2
Content column 1 | Content column 2

- [x] this is a complete item
- [ ] this is an incomplete item
- [x] @mentions, #refs, [links](),
**formatting**, and <del>tags</del>
supported
- [x] list syntax required (any
unordered or ordered list
supported)
