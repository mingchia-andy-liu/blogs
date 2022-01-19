---
title: "function.length in JavaScript"
date: 2022-01-17T22:47:13-08:00
description: Found a a function.length in wild. What is it?
slug: javascript-function-length
tags:
  - javascript
type: post
---

Recently I found a `function.length` in the real world pacakge. I did not know what it was. To my surprise, it is a legal property given a function. It gives the number of parameters expected by the function. 

```js
const func1 = () => console.log('hello');
const func2 = (a,b,c) => console.log('world');
func1.length; // 0
func2.length; // 3
```

How is this useful? I have never needed to know the number of parameters a function needs before calling it. Turns out some libraries use this `length` property to do some method overloading. 🤯

That is pretty smart way of doing this without using just one big object in the first parameter.

```js
const myFunc = (callback) => {
    const a = 1, b = 2, c = 3;
    const verifyArityArgsMap = {
        1: [a],
        2: [a b],
        3: [a,b,c],
    };
    const arity = callback.length; // Here using length to check how many parameter to use.
    const verifyArgs = verifyArityArgsMap[arity];

    callback(...verifyArgs);
}
```

But there is a caveat. With JavaScript, there is always a catch. [Let\'s see the spec on MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/length).

> This number excludes the rest parameter and only includes parameters before the first one with a default value.

```js
const func3 = (a, b = 1) => console.log('I am weird');
const func4 = (...args) => console.log('I do not have a length');
console.log(func3.length); // 1
console.log(func4.length); // 0
```


### Real world example

🧐 [The wild code I was looking at that uses length for method overloadding.](https://github.com/AzureAD/passport-azure-ad/blob/277f483d29993d1eada1017d5857c382f1c5a94c/lib/oidcstrategy.js#L102-L126)


&nbsp;

### Should you use this? 

I don't know, depends on your situation. But the answer is probably not. That's all. I hope you find this interesting.