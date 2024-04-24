---
author: Davide Ceschia
pubDatetime: 2022-02-21
title: Experimenting with Javascript Objects (Part 1)
postSlug: experimenting-with-javascript-objects-part-1
featured: false
draft: false
tags:
  - javascript
  - typescript
ogImage: ""
description: Introduction to Javascript Objects
---

Introduction to Javascript Objects

## Table of contents

## Experimenting with Javascript Objects (Part 1)


Javascript is a very complicated yet powerful language and objects are in my opinion one of the hardest parts of it, I decided to write this series of articles to help others get a more in-depth understanding and use cases of objects. This first article is just an introduction to objects, in the next ones we will start exploring various practical applications.

## What is a Javascript Object?

A Javascript object is a data type used to store keyed collections, you can create an object using the `Object()` constructor or the **object literal** syntax. There are actually a lot more ways you can create an object, but I’ll just show some of them.

```ts

// Object Literal
const myObject = {
  hi: "mom",
};

// Object constructor
const myObject = new Object({
  hi: "mom",
});

// Object constructor function
const myObjectFunc = function (val) {
  this.hi = val;
};

const myObject = new myObjectFunc("mom");
```

90% of the time what you’ll be using is the object literal syntax, it’s for sure the easiest and cleanest solution. The object constructor is rarely used and you should generally not worry about it. The object constructor function is a bit more useful but still only used in special situations.

## Working with Objects

### Defining/Mutating Properties

There are multiple ways to define properties in an object. You can

*   **Define them using an object literal during initialization**
    ```ts
    const myObject = {
    hi: "mom",
    }
    ```

*   **Use the assignment operator**
    ```ts
    const myObject = {}; // or new Object()

    myObject.hi = "mom";

    // Or

    myObject["hi"] = "mom";

    // Or

    const myString = "hi";

    myObject[myString] = "mom";
    ```

*   **Use Object.defineProperty**

    This is a bit more advanced, but it is very very useful. It allows you to define properties with specific options, for example, you can make the property non-enumerable or non-mutable. It returns the resulting object and throws an error on failure.

    ```ts
    const myObject = {};

    Object.defineProperty(myObject, "hi", {
      value: "mom",
      writable: true, // default: false
      enumerable: true, // default: false
      configurable: true, // default: false
    });

    console.log(myObject.hi); // > mom

    myObject.hi = "medium";
    console.log(myObject.hi); // > medium

    delete myObject.hi;
    console.log(myObject.hi); // > undefined

    Object.defineProperty(myObject, "hi", {
      value: "mom",
      writable: true, // default: false
      enumerable: true, // default: false
      configurable: true, // default: false
    });
    console.log(myObject.hi); // > mom

    Object.defineProperty(myObject, "hello", {
      value: "world",
      writable: false, // default: false
      enumerable: false, // default: false
      configurable: false, // default: false
    });

    console.log(myObject.hello); // > world

    myObject.hello = "medium";
    console.log(myObject.hello); // > world

    delete myObject.hello;
    console.log(myObject.hello); // > world

    // > Uncaught TypeError: Cannot redefine property: hello at Function.defineProperty (<anonymous>)
    Object.defineProperty(myObject, "hello", {
      value: "medium",
      writable: false, // default: false
      enumerable: false, // default: false
      configurable: false, // default: false
    });
    console.log(myObject.hello);
    ```

    There is a lot going on here, let’s break it down.  
    By calling `defineProperty` with all options set to true is functionally equivalent to the property assignment operator `myObject.hi = "mom"` .

    Things change when you switch the options. Settings `writable` to false prevents anyone from mutating the property using the assignment operator, setting `configurable` to false prevents the use of `defineProperty` on the same property again and `enumerable` decides whether the property is enumerable or not. We will see how to loop over enumerable and non-enumerable properties shortly.

    As you can see from the results below, calling defineProperty on a property that was set to not be configurable will result in a `TypeError` .

*   **Reflect.defineProperty**

    This is very similar to `Object.defineProperty` but instead of returning the object, it returns a boolean with the result of the operation. If the property was already defined as non-configurable, instead of throwing an error it will return false.

### Accessing Properties

Accessing properties is very similar to defining them, you just don’t have to use the assignment operator.

```ts
const myObject = {
  hi: "mom",
};

console.log(myObject.hi);

// Or

console.log(myObject["hi"]);

// Or

const myString = "hi";

console.log(myObject[myString]);
```

To access all properties in an object there are a lot of ways to do that, you could

*   **Use for…in**  
    for…in will loop over all **enumerable** properties, **even those contained in the prototype chain**. Non-enumerable properties will not be listed.

    ```ts
    const myObject = {
        hi: "mom",
        hello: "world",
    }

    for (const k in myObject) {
        if(myObject.hasOwnProperty(k)) {
            const value = myObject[k];
            console.log(k, value);
        }
    }
    ```

results:

```
> hi mom  
> hello world
```

`myObject.hasOwnProperty` is required to avoid accessing properties in the prototype chain, we’ll understand that in a moment.

*   **Use Object.keys  
    **Object.keys will only loop over **enumerable** properties contained in the object itself, it will **not traverse the prototype chain**.
    ```ts
    const myObject = {
        hi: "mom",
        hello: "world",
    };

    const keys = Object.keys(myObject);

    keys.forEach((key) => {
        const value = myObject[k];
        console.log(k, value);
    });
    ```

    You could also use `Object.values` or `Object.entries` .

*   **Use Object.getOwnPropertyNames  
    **Object.getOwnPropertyNames will loop over all **enumerable** and **non-enumerable** properties in the object itself, it will **not check the prototype chain**.

    ```ts
    const myObject = {
        hi: "mom",
        hello: "world",
    };

    const keys = Object.getOwnPropertyNames(myObject);

    keys.forEach((key) => {
        const value = myObject[k];
        console.log(k, value);
    });
    ```

### Deleting Properties

Deleting properties is pretty straightforward, but first, we need to make it clear that **deleting** a property **IS NOT** the same as setting the property to **null/undefined**. If you set a property to undefined, iterators will still loop over it and `Object.hasOwnProperty` will still return true.

*   **Use the delete keyword**

    The first delete works because by default property assignments are configurable, but the second delete operation fails silently because the property could not be deleted. That is where `Reflect.deleteProperty` comes.

    ```ts
    const myObject = {
        hi: "mom",
    };

    console.log(myObject.hasOwnProperty("hi")); // > true
    delete myObject.hi;
    console.log(myObject.hasOwnProperty("hi")); // > false

    // PAY ATTENTION NOW

    Object.defineProperty(myObject, "hi", {
        value: "mom",
        configurable: false,
    });

    console.log(myObject.hasOwnProperty("hi")); // > true
    delete myObject.hi;
    console.log(myObject.hasOwnProperty("hi")); // > true
    ```

*   **Use Reflect.deleteProperty**

    `Reflect.deleteProperty` is functionally the same as the delete keyword but it’s a function and it returns the result of the delete operation.

    ```ts

    const myObject = {
        hi: "mom",
    };

    console.log(myObject.hasOwnProperty("hi")); // > true
    let result = Reflect.deleteProperty(myObject, "hi");
    console.log(result, myObject.hasOwnProperty("hi")); // > true false

    // PAY ATTENTION NOW

    Object.defineProperty(myObject, "hi", {
        value: "mom",
        configurable: false,
    });

    console.log(myObject.hasOwnProperty("hi")); // > true
    result = Reflect.deleteProperty(myObject, "hi");
    console.log(result, myObject.hasOwnProperty("hi")); // > false true
    ```

    As you can see we can now check the result of the operation and act accordingly.

## Const vs Let vs Var

All three keywords are used to declare a variable, but what’s the difference?

Before ES6, `var` was the de-facto (and only) way to declare variables, nowadays it’s a way better practice to use `let` and `const` . I won’t go too much into the details about the differences (I might write a dedicated article about it) but the point is that you should never use `var`. You should always use `const` unless you need to reassign the variable, in which case you should use `let` .

What I wanted to emphasize is that `const` prevents reassignments but not mutations to objects! You should still use it if you want the variable to not be reassigned but keep in mind that properties can generally still be mutated/added/deleted (unless explicitly specified using `Object.defineProperty` or `Reflect.defineProperty` ).

## Javascript Objects are references

In javascript only **Number**, **String**, and **Boolean** types are passed around by **value**, everything else is passed by **reference** . That means that the value is not copied, but a reference to the original value is being passed instead, resulting in mutations not being limited to the function context. Let’s see an example.

```ts

const myObject = {};

const doSomething = function (whatever) {
  whatever.hi = "mom";
};

console.log(myObject); // > {}

doSomething(myObject);

console.log(myObject); // > {hi: "mom"}
```

As you can see from the results, the original object is being mutated, this is a very important concept when understanding objects.

## Functions are Objects

As we saw in the beginning, you can create an object using the **Object constructor function** syntax, but all JS functions are just Objects. The only difference between functions and objects is that functions can be called, objects cannot.

```ts
function myFunc() {
  console.log("hi", myFunc.who);
}

myFunc.who = "mom";

myFunc(); // > hi mom
```

## Arrays are Objects

It might be weird but in Javascript, arrays are objects as well. You can define properties on an array. All the functions and methods we talked about in the `Working with Objects` work on arrays as well.

```ts

const myArray = [1, 2, 3];

Object.defineProperty(myArray, "hi", {
  value: "mom",
});
myArray.hello = "world";
console.log(myArray);

for (const key in myArray) {
  console.log("key:", key);
}

for (const value of myArray) {
  console.log("value:", value);
}
```

results:

```
> [1, 2, 3, hello: 'world', hi: 'mom']  
> key: 0  
> key: 1  
> key: 2  
> key: hello  
> value: 1  
> value: 2  
> value: 3
```

As you can see we can easily set properties with the assignment operator or with `Object.defineProperty` to an array. It is important to understand that these properties will not be part of the array itself so most loop methods like `.map` `.filter` or `for...of` will not loop over these properties, they just loop over the values in the array itself. As before, you can make the property enumerable or not and that decides whether or not it will show in the different object loop methods.

## Prototype Chain

The prototype chain is a fundamental concept in javascript. Even though you might not be directly using it, it impacts your day-to-day life even if you don’t notice. Javascript classes are just syntactic sugar on top of prototypes. So let’s understand what we’re talking about.

When you do something like

```ts
const myObject = {};

console.log(myObject.toString())
// in this case it's the same as calling
console.log(Object.getPrototypeOf(myObject).toString())
```

`myObject` doesn’t hold the `toString` method, so the Javascript engine tries to look for it in the prototype chain which in this case looks like

```
myObject <- Object.prototype <- null
```

All objects in Javascript inherit properties from a prototype.

*   `Date` objects inherit from `Date.prototype`

```
myDateVariable <- Date.prototype <- Object.prototype <- null
```

*   `Array` objects inherit from `Array.prototype`

```
myArrayVariable <- Array.Prototype <- Object.prototype <- null
```

*   `myObject` objects inherit from `myObject.prototype`

```
myObject <- Object.prototype <- null
```

The `Object.prototype` is on the top of the prototype inheritance chain and its prototype is `null` , which is the end of the chain.

Before, we said that functions are objects, but with a little caveat when working with object constructor functions.

```ts
function myObjectFunc() {
  this.hi = "mom";
}

const myObject = new myObjectFunc();

// This will not work as intended!
myObjectFunc.hello = "world";
console.log(myObject.hello); // undefined!
```

You never want to assign properties to an object constructor function outside of the constructor itself. You either want to add it inside the constructor, add it to the instance of the object ( `new MyObjectFunc()` ) or add it to the prototype of the constructor.

A function instance does not inherit properties from the function itself but from its prototype, that’s why the above code does not work as expected, the prototype chain of the above function constructor looks like this

```
myObject <- myObjectFunc.prototype <- Function.prototype <- Object.prototype <- null
```

When you call `const myObject = new myObjectFunc()` what javascript is doing under the hood is

1.  Creating a new object
2.  Setting the prototype of this object to the constructor function’s prototype property
3.  Binding the `this` keyword to the newly created object and executing the constructor function
4.  Returning the newly created object

The above snippet is functionally equivalent to

```ts

function myObjectFunc() {
  this.hi = "mom";
}

const myObject = Object.create(myObjectFunc.prototype)
myObjectFunc.call(myObject)

// This will not work as intended!
myObjectFunc.hello = "world";
console.log(myObject); // undefined!
```

### Assigning a Prototype to an Object

As we just saw, you can assign a prototype to an object by calling `Object.create()` and passing the object to use as a prototype. It will return a new object that has your object as a prototype.

Another method also exists which is `Object.setPrototypeOf()` but its use is highly discouraged because of performance reasons.

```ts

const myFirstObject = {
  hi: "mom",
};
const mySecondObject = Object.create(myFirstObject);

console.log(mySecondObject.hi, mySecondObject.hasOwnProperty("hi")); // > mom false
```

In this case the prototype chain of `mySecondObject` looks like this

```
mySecondObject <- myFirstObject <- Object.prototype <- null
```

### Prototypal Inheritance and Prototypal Delegation

Prototypal inheritance states that an object inherits all the properties of its prototype.

There is also another very important concept which is called prototypal delegation which states that when you try to access a property of an object, if it cannot find the property in the object itself, it will traverse the entire prototype chain until it finds that property. If it cannot find it it’ll just return undefined.

### Inheriting methods and the “this” keyword

In JavaScript, any function can be added to an object as a property. Inheriting a function works exactly as inheriting a property, including property shadowing (which works kind of like method overriding).

When a property is inherited, the `this` keyword refers to the context of the parent calling object, not the original object where the property was defined.

```ts
const myFirstObject = {
  hi: "mom",
  sayHi() {
    console.log("hi " + this.hi);
  },
};

const mySecondObject = Object.create(myFirstObject);

mySecondObject.hi = "medium";
console.log(mySecondObject); // > {hi: "medium"}
console.log(Object.getPrototypeOf(mySecondObject)); // > {hi: "mom", sayHi: ƒ}
mySecondObject.sayHi(); // > hi medium
```

As you can see the `sayHi()` function is called from the `mySecondObject` context hence the `this` keyword refers to that. If a `hi` property is not defined in `mySecondObject`, it will be looked for in the prototype chain.

### Property Shadowing

Because of prototypal delegation, when accessing a property, the Javascript engine will try to look for it in the object itself and if it doesn’t find anything, it will keep looking for it in the prototype and so on until the end of the prototype chain.

What happens if both an object and its prototype have a property with the same name? When trying to access that property, the one in the object itself will **shadow** the one in the prototype, which will need to be accessed directly using `myObject.prototype.myProperty` .

Please **DO NOT extend native prototypes** such as `Object.prototype` or `Array.prototype`, it’s considered bad practice and should be avoided.

## Conclusion

For a more in-depth reference, please take a look at [this link](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_Objects)

In the next article, we will see how to correctly compare and copy objects.
