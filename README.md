# ReplitClient.js

A promise-based JavaScript client library used to connect to the server-side code execution service.

## ReplitClient(hostname, port, language, token)

Is the constructor and takes the following required paramaters:

* hostname: the hostname string
* port: the hostname port
* language: the language name. For now it's one of:
  * python
  * ruby
  * java
  * go
* token: an object representing the authentication token and has the following properties:
  * time_created: a JS unix timestamp of the time the token was created
  * msg_mac: HMAC authentication code based on a secret and the `time_created` property

Example:

```js
var repl = new ReplitClient('localhost', 8080, 'ruby', token);
```

## .connect()

Returns a promise that would resolve or reject depending on whether we were able to connect to the server.

Example:

```js
repl.connect().then(
  function() { console.log('connected'); },
  function() { console.log('failed to connect'); }
);
```

## .evaluate(code, options)

Returns a promise that would resolve with an object representing the result of the evaluation of `code`. The result object has the following properties:

* error: is the error string, most likely to be a stack trace
* data: if the evaluation didn't produce any errors this would be the result of the evaluation

`.evaluate` takes the following options:
* code: is a string of the program to be executed
* options: is an object with the following optional properties:
  * stdout: a function that would recieve a string to be printed

Examples:

```js
// Hello world in ruby
repl.evaluate('puts "hello world"', {stdout: function(out) { console.log(out); }})
    .then(function(result) {
       console.log('error', result.error); // null in this case
       console.log('result', result.data); // 'nil' (ruby's representation for null)
    });
```

## .write(str)

Write `str` to the interpreter's stdin.

Examples:

```js
// Using stdin get an integer and add it to 3
repl.evaluate('gets.to_i + 3')
    .then(function(result) {
      console.log(result.data); // 13
    });
repl.write('10');
```

## .disconnect()

Disconnects.
