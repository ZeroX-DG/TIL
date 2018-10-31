# Proxy
Proxy is a special Object used to define custom behavior for a target object when it's being used.

MDN: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy

## Syntax
```js
var p = new Proxy(target, handler);
```
- **target**: A target object to wrap with Proxy. It can be any sort of object, including a native array, a function or even another proxy.
- **handler**: An object whose properties are functions which define the behavior of the proxy when an operation is performed on it.

## Example
### Object observer
Object observer is a vital part of making state management library. It's basically means that you can notify the component to re-render when the state is modified.

To create an object observer with Proxy, you should first create an object holding all your data in it.
```js
const data = {
  todos: []
}
```
Then pass it to Proxy as the target. 
```js
const p = new Proxy(data, {
  get(target, property) {
    return target[property]
  },
  set(target, property, value) {
    events.emit('object:change')
    target[property] = value
    // always return true
    return true
  }
})
```
By emitting events when the object is changed, we can listen to that event in our component and re-render it to match the data.

### Object validation
By using Proxy we can also check for invalid data input when it being set to our object.
```js
const data = {
  age: 0
}

const person = new Proxy(data, {
  set(object, property, value) {
    if (property === 'age' && isNaN(value)) {
      throw new Error('age must be a number')
    }
    object[property] = value
    return true
  }
});

person.age = 10
person.age = 'a' // Error: age must be a number
```

### Observing change with nested object
Proxy will not observe the nested object value so instead we have to implement it ourselves.

```js
const data = {
  todos: []
}

const handler = {
  get(target, property) {
    const isPropertyObject =
      target[property] && typeof target[property] === "object";
    return isPropertyObject
      ? new Proxy(target[property], handler) // create new proxy for nested object property
      : target[property];
  },
  set(target, property, value) {
    target[property] = value
    return true
  }
}

const todoList = new Proxy(data, handler)
```