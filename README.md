# JavaScript `this` parameter

> `this` isn't really a keyword, it is the natural "main" function argument in JavaScript.
> - (@spion)[https://github.com/mindeavor/es-pipeline-operator/issues/2#issuecomment-162348536]

This proposal extends the `function` declaration syntax to allow explicit `this` parameter, which TypeScript/Flow already support.

```ts
// TypeScript this parameter
function getX<T>(this: T) {
	return this.x
}
```

```js
// JavaScript this parameter
function getX(this) {
	return this.x
}
```

Status: Stage 0

Author: @hax (HE Shi-Jun), @gilbert

There is an old proposal [gilbert/es-explicit-this](https://github.com/gilbert/es-explicit-this) presented in Feb 2020 TC39 meeting but not advanced. This revised version drop the features of renaming and destructuring `this` in the old proposal, only focus on the basic feature of `this` parameter syntax which TypeScript/Flow already support.

[Type Annotation proposal](https://github.com/tc39/proposal-type-annotations/blob/a4315be8a311980ca525dc539585b10b7478a63e/README.md#this-parameters) also include this feature, but strictly speaking, `this` parameter itself is not annotation, and could have [runtime semantic](#runtime-errors) which type annotation proposal intentionally avoid, so it's better to spec `this` parameter in this seperate proposal. 

## Motivation

- Standardize `this` parameter syntax of TS/Flow/etc, simpify tool chains of them
- Narrow the gap between JavaScript and TypeScript, eliminate the confusion of newcomers
- Effort of reducing the syntax burdens of Type Annotation proposal
- Provide syntax for "method" (function which should be called with receiver)

## Use cases

Explicit `this` parameter allow type annotation or parameter decorators be added just like normal parameter.

```ts
// type annotation (TypeScript, FlowType)
function handleClick(this: HTMLElement) {
	this.innerText = "clicked!"
}
element.addEventListener("click", handleClick)
```

```ts
function toHex(this: number) {
  return this.toString(16)
}
42~>toHex() // call-this syntax (stage 1 proposal)
```

```ts
// Parameter decorators (future proposal)
function handleClick(@Type(HTMLElement) this) {
	this.innerText = "clicked!"
}
element.addEventListener("click", handleClick)
```

```ts
// utils.js
export function toHex(@numeric this) {
  return this.toString(16) 
  // return typeof this == 'number || typeof this == 'bigint"
  //   ? this : Number(this)
}

// usage
import ::{toHex} from "./utils.js"
42::toHex() // Extensions (stage 1 proposal)
```

Explicit `this` parameter also provide a syntax/semantic for real "method" (function which should be called with receiver). This also allow linters improve the `this`-related rules (eg. https://eslint.org/docs/rules/no-invalid-this and https://eslint.org/docs/rules/prefer-arrow-callback#allowunboundthis).

## Early errors

`this` parameter can only be the first parameter.
```js
function f(a, this /* syntax error */ ) {}
```

Arrow functions can not have `this` parameter because `this` in arrow functions are always lexical.
```js
let f = (this /* syntax error */ ) => 0
```
Class constructors can not have `this` parameter.
```js
class C {
  constructor(this /* syntax error */) {}
}
```

Class constructor can't use `this` parameter because class constructor can only be used via `new` and `this` in the constructor is never an argument passed by caller, the `this` value in constructor is generated by the constructor or its base class.

`this` parameter can not have default value.
```js
function f(this = globalThis /* syntax error */ ) {}
```

Note, TS and Flow (and even Java 8) all report static errors for such cases.

## Issue of non-strict mode

Non-strict functions have the implicit coercion semantic like `this = Object(this ?? globalThis)` which always confuse the newcomers and easy to cause silent bugs. Consider non-strict functions as the legacy of ES3 era, not common these days, and if you specify something explicitly, you normally don't want any implicit effect, so when functions have explicit `this` parameter we should drop the coercion semantic (make its `[[ThisMode]]` behave like "strict"). Another option is ban `this` parameter syntax in non-strict mode at all. See [issue in the old proposal](https://github.com/gilbert/es-explicit-this/issues/9) for other options.

## Runtime errors

If a function has `this` parameter, it could be useful to throw an error when that function gets called without one or used via `new`. For example:

```js
function zip (this, otherArray) {
  return this.map( (a, i) => [a, otherArray[i]] )
}

zip() //=> Throws a TypeError on invocation,
      //   before `array.map(...)` is run.

new zip() //=> Also throws a TypeError
```

This runtime semantic also useful to call-this proposal or Extensions proposal, see [examples](invoke-as-method.md).


## Future consideration

This proposal focus on the minimal syntax and runtime semantics, advanced features like renaming, destructuring and default value of `this` parameter could be added in the future.

## Prior Arts:
- [Java 8+ (receiver parameter)](https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html#jls-8.4.1-220)
- [TypeScript](https://www.typescriptlang.org/docs/handbook/2/classes.html#this-parameters)
- [Flow](https://flow.org/en/docs/types/functions/#toc-function-this)

## Old disussions
- [`function (this | arg1, arg2) syntax`](https://esdiscuss.org/topic/inner-functions-and-outer-this-re-that-hash-symbol#content-25)

