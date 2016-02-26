# Background

Stack trace is the report of current active call frames, and it is a 
very useful tool for developers during debugging. Tt can also be used
to support bug reports and diagonistics.

Most programming languages have stack traces built-in for getting the
current stack trace when an error occurs. Some examples are Java, C#,
PHP and Python, and many other languages.

Stack traces are also widely implemented in ECMAScript engines,
including those for server and those in modern browsers.

Despite the fact that this is already widely supported, it has not
been standardized, which causes some problems, including leak
information in sandbox implementation. 

Previous strawman: http://wiki.ecmascript.org/doku.php?id=strawman:error_stack

Previous discussion: https://esdiscuss.org/topic/standardizing-error-stack-or-equivalent and https://esdiscuss.org/topic/error-stack-strawman

# Current Support

All modern browsers have support for stack trace, however some details are different. Test can be conducted by test.html in the same folder.

|                 | Internet Explorer | Edge | Chrome | Firefox | Safari | Opera |
|-----------------|-------------------|------|--------|---------|--------|-------|
| create on new   |                   | X    | X      | X*      |        | X     |
| on prototype    |                   |      |        | X       |        |       |
| create on throw | X**               | X*** |        |         |        |       |
| get             | X                 | X    | X      | X       |        | X     |
| set             | X                 | X    | X      | X       |        | X     |
| value           |                   |      |        |         |        |       |
| writable        |                   |      |        |         |        |       |
| configurable    | X                 | X    | X      | X       |        | X     |
| enumerable      | X                 |      |        |         |        |       |

(Data needed for Safari)

\* Though defined on prototype, Firefox generates line information when object is created

** Only when Error.prototype is in the prototype chain of the thrown object

*** When throwing, the stack will be re-generated


Here are some sample output from IE, Edge, Chrome and Firefox:

IE:
```
Error: some error is thrown
   at testErrorStack (file:///***/test.html:11:5)
   at Global code (file:///***/test.html:25:1)
```

Edge:
```
Error: some error is thrown
   at testErrorStack (file:///***/test.html:11:5)
   at Global code (file:///***/test.html:25:1)
```

Chrome:
```
Error: some error is thrown
    at testErrorStack (file:///***/test.html:7:17)
    at file:///***/test.html:25:1
```

Firefox:
```
testErrorStack@file:///***/test.html:7:17
@file:///***/test.html:25:1
```

All browsers add `stack` as the property and all of them use
same format for file and line/column information:`url:line:column`.

IE reports the line/column information where the Error object is thrown.

Edge reports the line/column information where the Error object is
thrown, or, when the Error object is not thrown, the information where
it is created.

Chrome and Firefox reports line/column information where the Error object
is created.

# Concerns

**Cross Realm Invocation**

In cross-realm invocation, stack traces crossing the boundary should be
sanitized, preventing information from being leaken.

**Native Function Invocation**

Native methods should be displayed differently.

Current implementation:

IE: Not displayed

Edge: `at Array.prototype.forEach (native code)`

Chrome: `at Array.forEach (native)`

Firefox: Not displayed

**Global code**

Global code should be displayed differently.

Current implementation:

IE & Edge: `at Global code`

Chrome: Only file name gets displayed, and no paranthesis surrounds it.

Firefox: The place where function name should be is empty.

**eval and new Function**

IE & Edge: `at eval code (eval code:evalLine:evalColumn)` and `at Function code (Function code:evalLine:evalColumn)`

Chrome: `at eval (eval at callerName (url:callerLine:callerColumn), :evalLine:evalColumn)`, new Function are represented as eval as well.

Firefox: `@url line callerLine > eval:evalLine:evalColumn` and `@url line callerLine > Function:evalLine:evalColumn`

**Anonymous function and lambda expression**

IE & Edge: `at Anonymous function`

Chrome: only file name get displayed with no parenthese. (But in debugger, `(anonymous function)` is displayed)

Firefox: Surrounding function name suffixed by `/<`

**Tail calls**

None of the modern browsers support tail calls yet, so it cannot be compared.
Theoretically when a tail call is performed, its stack frame is removed,
so it should not be in the final stack string. However, this makes programs
harder to debug, so I suggest that at least an indication that a tail call is
performed should exist.

**Asynchronous calls**

None of stable releases of modern browsers include a history of previous stack frames for asynchronous calls.

The Nightly build of Firefox includes stack frame history, in following format
```
wrapper@file:///***/test.html:78:19
setTimeout handler*testTimeout@file:///***/test.html:76:5
@file:///***/test.html:84:1
```
Similar stack trace for Promise:
```
wrapper@file:///***/test.html:89:19
promise callback*testPromise@file:///***/test.html:87:5
@file:///***/test.html:96:5
```

**Generators**

None of modern browsers include the stack frame of the function which creates the generator.

**Source maps**

Most browser's debugger tools support source map, but this is not standardized. Stack traces can provide
different position information when source maps exist.

# API

It is suggested the API can be made accessible via [Built-in Modules](https://github.com/tc39/ecma262/issues/395) if possible.

Details to be discussed


