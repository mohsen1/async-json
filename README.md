# Asynchronous JSON methods for ECMAScript proposal

JavaScript programs are single-threaded and therefore must streadfastly avoid blocking on IO operations. `JSON.parse` and `JSON.stringify` are both blocking operations. For large JSON strings to parse or objects to stringify the blocking time can be very significant.

To avoid blockage of the main thread by `JSON.parse` and `JSON.stringify` we are proposing for asynchronous and non-blocking JSON methods that move the work to a background thread.

The asynchronous JSON methods keep the function signature the same with synchronous methods except they return a promise instead of value/string. This helps the overall API consistency.

## `JSON.parseAsync( text [ , reviver ] )`

The `parseAsync` function parses a JSON text (a JSON-formatted String) and produces a `Promise` [(ECMA-262 §25.4)][promise] object that then resolves to a ECMAScript value. It uses `JSON.parse` [(ECMA-262 §24.3.1)][parse] in a background thread to produce the result to avoid blockage of the main thread during the parse task.

The optional *`reviver`* parameter is a function that takes two parameters, *key* and *value*. It can filter and transform the results. It is called with each of the *key/value* pairs produced by the parse, and its return value is used instead of the original value. If it returns what it received, the structure is not modified. If it returns **`undefined`** then the property is deleted from the result.

##### Example
```js
JSON.parseAsync('{"foo": 1}').then(result => console.log(result));
// {foo: 1}
```

## `JSON.stringifyAsync( value [ , replacer [ , space ] ] )`

The `stringifyAsync` function returns a `Promise` [(ECMA-262 §25.4)][promise] object that then resolves to a String in UTF-16 encoded JSON format representing an ECMAScript value. It uses `JSON.stringify` [(ECMA-262 §24.3.2)][stringify] in a background thread to produce the result to avoid the blockage of the main thread during the stringify task.

It can take three parameters. The *`value`* parameter is an ECMAScript value, which is usually an object or array, although it can also be a String, Boolean, Number or **`null`**. The optional *`replacer`* parameter is either a function that alters the way objects and arrays are stringified, or an array of Strings and Numbers that acts as a white list for selecting the object properties that will be stringified. The optional `space` parameter is a String or Number that allows the result to have white space injected into it to improve human readability.

##### Example
```js
JSON.stringifyAsync({foo: 1}).then(result => console.log(result));
// '{"foo": 1}'
```

#### With `await`
Because `JSON.stringifyAsync` and `JSON.parseAsync` are returning promises, they can be used with `await` as a drop-in replacement of sync methods.

```js
async ()=> {
  let object = await JSON.parseAsync('{"foo": 1}');
  let string = await JSON.stringifyAsync({foo: 1});
}();
```

#### Transpilation / Polyfill

There is no way of transpilating the actual effect of this proposal. But instead we can write a polyfill that keeps API consistent while it's using the main thread for parsing and stringifying tasks. We are using promises to perform the sync operation in the next processor tick.


```js
JSON.parseAsync = JSON.parseAsync || function parseAsync(text, reviver) {
 return Promise.resolve().then(()=> JSON.parse());
}

JSON.stringifyAsync = JSON.stringifyAsync || function stringifyAsync(value, replacer, space) {
 return Promise.resolve().then(()=> JSON.parse(value, replacer, space));
}
```

[parse]: http://www.ecma-international.org/ecma-262/6.0/#sec-json.parse
[promise]: http://www.ecma-international.org/ecma-262/6.0/#sec-promise-objects
[stringify]: http://www.ecma-international.org/ecma-262/6.0/#sec-json.stringify
