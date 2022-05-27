## Useful to call-this proposal or Extensions proposal

The [call-this](https://github.com/tc39/proposal-call-this) or [Extensions](https://github.com/tc39/proposal-extensions) propose the syntax to invoke a function as method form, they could utilize runtime semantic of the functions with `this` parameter to mitgate the confusion of free methods and protect the users. For example:

```js
// zip-method.js
export function zip (this, otherArray) {
  return this.map( (a, i) => [a, otherArray[i]] )
}
```

```js
// call-this example

import {zip} from "./zip-method.js"

[1, 2]
  .map(x => x * 10)
  ~>zip([1, 2]) // call-this syntax
  .flat()
//=> [10, 1, 20, 2]

zip([10, 20], [1, 2]) // throw TypeError, so avoid misuse
```

Extensions proposal support similar pattern, already protect the users without `this` parameter functions

```js
// Extensions example

// import zip as an extension method
import ::{zip} from "./zip-method.js" 

[1, 2]
  .map(x => x * 10)
  ::zip([1, 2]) // invoke extension method
  .flat()
//=> [10, 1, 20, 2]

zip // throw ReferenceError
```

```js
import {zip} from "./zip-method.js"
[10, 20]::zip([1, 2]) // ReferenceError
```

But Extensions proposal could be revised to only accept functions with `this` parameter, provide even stronger protection.

```js
// zip-util.js
export function zip (array, otherArray) {
  return array.map( (a, i) => [a, otherArray[i]] )
}
```

```js
// 

import ::{zip} from "./zip-util.js" // TypeError
[10, 20]::zip([1, 2])
```
