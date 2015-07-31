# Asynchronous JSON Methods Proposal

JavaScript programs are single-threaded and therefore must streadfastly avoid blocking on IO operations. `JSON.parse` and `JSON.stringify` are both blocking operations. For large JSON strings to parse or objects to stringify the blocking time can be very significant.

To avoid blockage of the main thread by `JSON.parse` and `JSON.stringify` we are proposing for asynchronous and non-blocking JSON methods that move the work to a background thread.

#### `JSON.parseAsync`

`JSON.parseAsync` will take a string and return a promise. The promise will then resolved to the result of parsing the string as a JSON string.


```js
JSON.parseAsync('{"foo": 1}').then(result => console.log(result));
{foo: 1}
```

#### `JSON.stringifyAsync`

`JSON.stringifyAsync` will take an object or array and return a promise. The promise will then resolved to a JSON string.

```js
JSON.stringifyAsync({foo: 1}).then(result => console.log(result));
// '{"foo": 1}'
```

#### With `await`
Because `JSON.stringifyAsync` and `JSON.parseAsync` are returning promises, they can be used with `await` as a drop-in replacement of sync methods.

```js
async ()=> {
  let result = await JSON.parseAsync(jsonString);
  let object = await JSON.stringifyAsync(anObject);
}();
```


#### Transpilation / Polyfill

There is no way of transpilating the actual effect of this proposal. But it's very easy to write a polyfill:


```js
JSON.stringifyAsync = JSON.stringifyAsync || function (object) {
  return new Promise((resolve, reject) => {
    try {
      resolve(JSON.stringify(object));
    } catch (error) {
      reject(error);
    }
  });
}

JSON.parseAsync = JSON.parseAsync || function (string) {
  return new Promise((resolve, reject) => {
    try {
      resolve(JSON.parse(string));
    } catch (error) {
      reject(error);
    }
  });
}
```
