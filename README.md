# ReplitClient.js

A promise-based JavaScript client library used to connect to the server-side code execution service.

## ReplitClient(hostname, port, language, token)

Is the constructor and takes the following required paramaters:

* hostname: the hostname string
* port: the hostname port
* language: the language name. For now it's one of:
  * python3
  * python
  * ruby
  * php
  * java
  * go
  * nodejs
  * csharp
  * fsharp
  * cpp
  * cpp11
  * c
  * swift
  * lua
  * rust
* token: an object representing the authentication token and has the following properties:
  * time_created: a JS unix timestamp of the time the token was created
  * msg_mac: HMAC authentication code based on a secret and the `time_created` property

Example:

```js
var repl = new ReplitClient('localhost', 8080, 'ruby', token);
```

_An optional 4th argument `socketCreator` function can be passed that allows you to intercept socket creation. You have to return a socket from this function that implements the same interface as the w3c `WebSocket`. You can use this to provide fallbacks if your users don't have websockets enabled or for testing_

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
repl.evaluate('gets.to_i + 3', {stdout: function(out) { console.log(out); }})
    .then(function(result) {
      console.log(result.data); // 13
    });
repl.write('10\n');
```

## .reset() 

Will restart your interepter/compiler while maintaining the connection. Returns a promise.

## .evaluateOnce(code, options)

Very similar to `.evaluate` but is http based and doesn't require an persistent connection to the server. In other words, *no* need to call `.connect` before calling this. 

Additionally, it takes a timeout option that disconnects from the server and rejects the returning promise:
* options:
  * timeout:
    * time: timeout in milliseconds
    * callback: a callback that is expected to return `true` or `false` on whether we should abort the call

## .runProject(files, options) (beta and subject to change)

Runs code with multiple files or modules.

* files: an array of file objects with a `name` and `content` string properties
* options:
  * stdout: optional stdout callback

## .runUnitTests({ files, suiteCode }) (beta and subject to change)

Runs unit tests tests against a set of files. Both takes an options object with two required arguments:

* files: an array of file objects with a `name` and `content` string properties
* suiteCode: code for the test suite

Language support:
- Java: uses JUnit and the suite class must be called `UnitTests`
- Python and Python3: Uses the builtin `unittest` module and the unit tests class must be called `UnitTests`.

## .runSingleUnitTests({ code, suiteCode }) (beta and subject to change)

Convinience method that calls into `runUnitTests` with the a single "main" file with `code` as the content.

## .disconnect()

Disconnects.

## .stop()

Will stop any currently running jobs.

# Events

## 'connecting'

Emitted when trying to connect

## 'connected'

Emitted when connection and handshake is done

## 'disconnected'

When the socket is closed. It is passed an object with the following properties:

* retry (boolean): indicating whether we are going to retry
* delay (milliseconds): we are going to wait before attempting to retry

Retries are currently capped at 7 times and with exponential backoff starting at 500ms
