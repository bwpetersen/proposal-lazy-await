# Lazy Await
An await expression will always pause an async function's execution until its promise is reloved or rejected. This proposal introduces `await*` expressions that do not wait until the awaited value is needed.

For example:
```javascript
const user = await* user.find()
const messages = await* message.findAll()
// ...
const data = { user, messages }
```
would be equivalent to:
```javascript
const _user = user.find()
const _messages = message.findAll()
// ...
const data = { user: await _user, messages: await _messages }
```
It would only make sense for `await*` expressions to be used in assignments, since the assigned variable will ultimately decide when the actual `await` needs to happen. 

When used as a child of a parent expression, the parent expression will be lazily evaluated as well.
```javascript
const totalCount = (await* dog.count()) + (await* cat.count())
// ...
console.log('totalCount', totalCount)
```
This would be equivalent to:
```javascript
const _totalCount = Promise.all([dog.count(), cat.count()]).then(([x, y]) => x + y)
// ...
console.log('totalCount', await _totalCount)
```
## Error handling for rejections
One downside to this method is that `await*` can't throw when they are rejected. The first access to the variable would need to throw an error. This would be a bit awkward since developers don't expect a variable access to throw. I think the best practice would be for developers to treat the `await*` as if it throws. 
