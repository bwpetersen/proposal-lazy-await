# Lazy Await
An await expression will always pause an async function's execution until its promise is reloved or rejected. This proposal introduces `await*` expressions that do not wait until the awaited value is needed.

For example:
```javascript
const user = await* user.find()
const messages = await* message.findAll()
const data = { user, messages }
```
would be equivalent to:
```javascript
const _user = user.find()
const _messages = message.findAll()
const data = { user: await _user, messages: await _messages }
```
It would only make sense for `await*` expressions to be used in assignments, since the assigned variable will ultimately decide when the actual `await` needs to happen. 
