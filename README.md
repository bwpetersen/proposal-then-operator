# The `then` operator

## Description
The `then` operator is a unary operator for promises, which allows the promise to be treated as if it were its resolved value. The result of any manipulation would be a new promise.

## Example
```js
let count = Promise.resolve(5)
let doubledCount = then count * 2 // doubledCount is a promise resolving to 5 * 2
console.log(then doubledCount) // logs 10
```
We can think of the above code as being equivalent to:
```js
let count = Promise.resolve(5)
let doubledCount = count.then(x => x * 2)
doubledCount.then(x => console.log(x))
```
We could also allow multiple `then`s in a single expression.
```js
let area = then width * then length
```
Which would behave like:
```js
let area = Promise.all([width, length]).then(([width, length]) => width * length)
```
Or alternatively like:
```js
let area = (async () => await width * await length)()
```

## Detailed explanation
The `then` operator notifies its top level expression that it has then dependencies.
Any top level expression with dependencies is converted to an immediately invoked async function expression.
This iiafe executes each 'then' operator's argument, storing them as promises.
The iiafe returns an expression similar to the original top level expression, but `then [[argument]]`s are changed to corresponding `await [[promise]]`.
