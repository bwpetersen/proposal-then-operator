# The `then` operator

## Description
The `then` operator is a unary operator for promises, which allows the promise to be treated as if it were its resolved value. The result of any manipulation would be a new promise.

## Examples
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
let area = then getWidth() * then getLength()
```
Which would behave like:
```js
let area = Promise.all([getWidth(), getLength()]).then(([width, length]) => width * length)
```
Or alternatively like:
```js
let area = (async () => {
  // execute the operands once asynchronously
  let width = getWidth()
  let length = getLength()
  // replace then with await 
  return await width * await length
})()
```

## Explanation
When a "top level expression" contains one or more `then` operators, it will first evaluate all of the operands. It an operand results in a non-promise, it will convert it to a promise via `Promise.resolve`. When all of the operand promises resolve, we execute the expression with each `then` expression swapped with their respective resolved value.

## Cutoff point and expressions vs statements
There needs to be some "top level expressison" that is ultimately converted to a promise.

For example it would be bad if the following:
```js
notDone = ! then isDone()
```
were treated as:
```js
isDone().then(done => notDone = !done)
```
since it would be preferable that variables be bound in current scope. What would be desired is:
```js
notDone = isDone().then(done => !done)
```
Thus in determining what would be the "top level expression", it is best not to go higher than the right hand side of an assignment.

When it comes to statements such as:
```js
if (then ready) {
  doStuff()
}
```
at first it would seem desirable to treat it as:
```js
ready.then(ready => {
  if (ready) {
    doStuff()
  }
})
```
but it would cause a lot of confusion if the conditional statement was followed by `doMoreStuff()` which depended on the execution `doStuff()`, the result would be undesirable behavior. Thus it would be better to keep things at the expression level. 

## What is the difference to `await`?
The purpose of this operator is be able to modify a promise asynchronously. It's not meant to wait for a resolution before continuing. For this reason, the `then` operator doesn't need to be restricted to `async` functions.
