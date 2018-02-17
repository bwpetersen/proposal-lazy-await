# Lazy Await
## Simple example

```javascript
const user = await* User.find()
const messages = await* Message.findAll() // does not wait for User.find()
// ...
const data = { user, messages }  // waits for User.find() and Message.findAll()
```
The above code would be behaviorally equivalent to:
```javascript
const _user = User.find()
const _messages = Message.findAll()
// ...
const data = { user: await _user, messages: await _messages }
```

## Explanation
An `await` expression will always pause an `async` function's execution until its promise is resolved or rejected. This proposal introduces `await*` expressions (or **lazy await expressions**) that do not pause. Instead, they immediately evaluate to a **lazy value**. Any variable that is assigned to a lazy value is a **lazy variable**. Whenever an expression tries to access a lazy variable's value, the async function pauses until it is resolved or rejected. 

## Lazy values
The difference between a **lazy value** and a promise is in how it behaves. Whereas `Promise.resolve(x) + y` is invalid, `(await* x) + y` is equivalent to `await* Promise.resolve(x).then(x => x + y)`. In general we would say:
```javascript
f(await* a, await* b, 3, 4)
```
is behaviorally equivalent to:
```javascript
await* Promise.all([a, b]).then(([a, b]) => f(a, b, 3, 4))
```
So, while lazy values don't have access to `Promise.prototype` methods, they can be manipulated as if they were fulfilled.

## Lazy variables
A **lazy variable** is a bit like a wrapper to a promise. When you try to read the value of a lazy variable, the async function will wait until it has resolved or rejected. This means that merely accessing a lazy variable could throw an error if its underlying promise rejects.

If you wish to manipulate a lazy variable without waiting for it to resolve, this is still possible. You may use `await*` to convert it to a lazy value.
```javascript
const user = await* User.find()
// ...
const username = (await* user).name // does not pause
// ...
const greeting = `Hi ${username}` // waits for User.find()
```

## Error handling for rejections
One downside to this proposal is that `await*` expressions can't throw when they are rejected, because the async function must continue without waiting for a resolution or rejection. The first access to the lazy variable would need to throw an error. This would be a bit awkward since developers don't expect a variable access expression to throw. I think the best practice would be for developers to treat the `await*` as if it throws, since it is in the same scope as its lazy variable. 
