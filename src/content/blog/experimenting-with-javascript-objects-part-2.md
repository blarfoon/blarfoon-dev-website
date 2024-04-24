---
author: Davide Ceschia
pubDatetime: 2022-02-22
title: Experimenting with Javascript Objects (Part 2)
postSlug: experimenting-with-javascript-objects-part-2
featured: false
draft: false
tags:
  - javascript
  - typescript
ogImage: ""
description: Comparing and copying Javascript Objects
---

Comparing and copying Javascript Objects

## Table of contents

## Experimenting with Javascript Objects (Part 2)

## Copying Objects

As we said in the previous [article](https://medium.com/experimenting-with-javascript-objects-part-1-16956b97b0c7), Javascript Objects are just references to the underlying data, this means that when you assign an object to multiple variables, all variables will hold the reference to the same underlying object.

```ts
const myFirstObject = {};
const mySecondObject = myFirstObject;

mySecondObject.hi = "mom";

console.log(myFirstObject); // > {hi: "mom"}
console.log(mySecondObject); // > {hi: "mom"}
console.log(myFirstObject === mySecondObject); // > true
```

Both variables change when you mutate one of them, and their comparison returns true, that’s because both variables actually reference the same underlying object.

One thing you can do to prevent this is doing a shallow/deep copy of the object but keep in mind that a shallow copy only copies first-level keys and a deep copy is very (relatively) slow.

## Shallow Copy

Let’s see an example that leverages the javascript **spread** operator to create a shallow copy of an object.

```ts
const myObject = {
  hi: "mom",
  someone: {
    else: "red",
  },
};

function doSomething(whatever) {
  whatever.hi = "medium";
  whatever.someone.else = "blue";
};

console.log(myObject); // > {"hi":"mom","someone":{"else":"red"}}

doSomething({ ...myObject });

console.log(myObject); // > {"hi":"mom","someone":{"else":"blue"}}
```

As you can see `hi: "mom"` did not change because it’s a first-level key, whereas `someone.else` did change because it’s a nested key.

Instead of the **spread** operator, you could have also used the more verbose `Object.assign` function.

## Deep Copy

Deep copies are a bit more _complex._ A lot of people suggest doing

```ts
JSON.parse(JSON.stringify(myObject))
```

but I heavily discourage using that because it could and will probably cause issues in your code. Let’s look at the following example:

```ts

const myObject = {
  hi: "mom",
  changeHi() {
    this.hi = "medium";
  },
  something: undefined,
};

console.log(myObject); // > {hi: "mom", something: undefined, changeHi: ƒ}

const myNewObject = JSON.parse(JSON.stringify(myObject));

console.log(myNewObject); // > {hi: "mom"}
```

As you can see from the results, some values are lost during this operation, this happens because not all javascript types are serializable by `JSON.stringify` .

If you need to deeply copy an object, your best bet is to implement a deep-copy function yourself like:

```ts

function clone(src) {
  const ret = src instanceof Array ? [] : {};
  for (const key in src) {
    if (!src.hasOwnProperty(key)) {
      continue;
    }
    let val = src[key];
    if (val && typeof val == "object") {
      val = clone(val);
    }
    ret[key] = val;
  }
  return ret;
}

const myObject = {
  hi: "mom",
  changeHi() {
    this.hi = "medium";
  },
  something: undefined,
};

console.log(myObject); // > {hi: "mom", something: undefined, changeHi: ƒ}

const myNewObject = clone(myObject);
myNewObject.changeHi();

console.log(myNewObject); // > {hi: "medium", something: undefined, changeHi: ƒ}
```

As you can see from the results

```
> {hi: "mom", something: undefined, changeHi: ƒ}  
> {hi: "medium", something: undefined, changeHi: ƒ}
```

all the values are maintained through the copy and changes to one object do not affect the other object.

The above `clone` function is not exhaustive and is just a proof of concept, it does not handle the deep copy of `Date` , `RegExp` and other things!

## Comparing Objects

Objects comparison is a somewhat complex topic among beginners, but it’s actually quite easy, let’s try to break it down.

As we previously said, objects are just references, and this is extremely important when you compare them.

When you create an instance of an object, it will be equal to nothing but itself, this happens because every object is a reference to the underlying data so when you compare two different objects, you are actually comparing their references which will obviously be different. Let’s see an example:

```ts

const myFirstObj = {};
const mySecondObj = {};

console.log(myFirstObj == mySecondObj, myFirstObj === mySecondObj); // > false false
```

As you can see from the results, no matter how you compare them, they are just different objects. This is actually a behavior that can be exploited for some very interesting use-cases which we will explore in the next articles.

If you intend to compare the content of two objects, you can shallow/deep compare them.

## Shallow Comparison

Shallow comparison only compares first-level keys and it’s a fundamental concept behind many frontend libraries like React. Let’s see an example of shallow comparison:

```ts

function areEqualShallow(a, b) {
  for (const key in a) {
    if (!(key in b) || a[key] !== b[key]) {
      return false;
    }
  }
  for (const key in b) {
    if (!(key in a) || a[key] !== b[key]) {
      return false;
    }
  }
  return true;
}

const myFirstObject = {};
const mySecondObject = {};

console.log(areEqualShallow(myFirstObject, mySecondObject)); // > true

myFirstObject.hi = "mom";
mySecondObject.hi = "mom";

console.log(areEqualShallow(myFirstObject, mySecondObject)); // > true

myFirstObject.nested = {
  hi: "medium",
};
mySecondObject.nested = {
  hi: "medium",
};

console.log(areEqualShallow(myFirstObject, mySecondObject)); // > false
```

As we can see from the results, the two objects are equal as long as all their first-level keys are equal. As soon as we add another nested object the shallow comparison will fail because `myFirstObject.nested` and `mySecondObject.nested` have two different references.

## Deep Comparison

Deep comparison is a lot slower and recursively compares all the nested keys, it might be useful if we need to ensure that two objects have the same values, even nested ones.

```ts

function areEqualDeep(a, b) {
  if (typeof a == "object" && a != null && typeof b == "object" && b != null) {
    const count = [0, 0];
    for (const key in a) count[0]++;
    for (const key in b) count[1]++;
    if (count[0] - count[1] != 0) {
      return false;
    }
    for (const key in a) {
      if (!(key in b) || !areEqualDeep(a[key], b[key])) {
        return false;
      }
    }
    for (const key in b) {
      if (!(key in a) || !areEqualDeep(b[key], a[key])) {
        return false;
      }
    }
    return true;
  } else {
    return a === b;
  }
}

const myFirstObject = {};
const mySecondObject = {};

console.log(areEqualDeep(myFirstObject, mySecondObject)); // > true

myFirstObject.hi = "mom";
mySecondObject.hi = "mom";

console.log(areEqualDeep(myFirstObject, mySecondObject)); // > true

myFirstObject.nested = {
  hi: "medium",
};
mySecondObject.nested = {
  hi: "medium",
};

console.log(areEqualDeep(myFirstObject, mySecondObject)); // > true
```

As you can see from the results, even nested comparisons now will be evaluated.

## Conclusion

For a more in-depth reference, please take a look at [this link](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_Objects).
