 - tagline: Supported F# language features and libraries

# Compatibility

Fable provides support for some classes of .NET BCL (Base Class Library) and most of
FSharp.Core library. When possible, Fable translates .NET types and methods to native JS
in order to keep `fable-core` small and improve performance.

## .NET Base Class Library

The following classes are translated to JS and all their methods
(static and instance) should be available in Fable.

.NET                                  | JavaScript
--------------------------------------|----------------------------------------------------------
System.String                         | string
System.Guid                           | string
System.TimeSpan                       | number
System.DateTime                       | Date
System.Timers.Timer                   | fable-core/Timer
System.Collections.Generic.List       | Array
System.Collections.Generic.HashSet    | Set
System.Collections.Generic.Dictionary | Map
System.Text.RegularExpressions.Regex  | RegExp
System.Lazy                           | fable-core/Lazy
System.Random                         | {}
System.Math                           | (native JS functions)

The following static methods are also available:

- `System.Console.WriteLine` (also with formatting)
- `System.Diagnostics.Debug.WriteLine` (also with formatting)
- `System.Diagnostics.Debug.Assert(condition: bool)`
- `System.Diagnostics.Debugger.Break()`
- `System.Activator.CreateInstance<'T>()`

The following types can also be used, but BCL methods (e.g. `System.Int32.Parse`) are not available.
Please use FSharp.Core functions (e.g. `int`) instead to operate them.

.NET              | JavaScript
------------------|----------------------------------------------------------
System.Char       | string
System.Boolean    | boolean
Numeric Types     | number
Arrays            | Array / Typed Arrays
Events            | fable-core/Event

### Caveats

- No bound checks for numeric types.
- Integer division will always produce an integer.
- Numeric arrays are compiled to [Typed Arrays](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray) when possible.
- `Regex` will always behave as if passed `RegexOptions.ECMAScript` flag (e.g., no negative look-behind or named groups).
- Structural equality is not available when using F# unions or records as `Dictionary` keys or `HashSet` values
  (you can use F# `Map` or `Set` instead).

> If you pass the `--clamp` argument to the compiler, byte arrays will be compiled as `Uint8ClampedArray`.

## FSharp.Core

Most of FSharp.Core operators are supported, as well as formatting with
`sprintf`, `printfn` or `failiwthf` (`String.Format` is also available).
The following types and/or corresponding modules from FSharp.Core lib will
likewise translate to JS:

.NET              | JavaScript
------------------|----------------------------------------------------------
Tuples            | Array
Option            | (erased)
Choice            | fable-core/Choice
String            | fable-core/String (module)
Seq               | [Iterable](http://babeljs.io/docs/learn-es2015/#iterators-for-of)
List              | fable-core/List
Map               | fable-core/Map
Set               | fable-core/Set
Async             | fable-core/Async
Event             | fable-core/Event (module)
Observable        | fable-core/Observable (module)
Arrays            | Array / Typed Arrays
Events            | fable-core/Event
MailboxProcessor  | fable-core/MailboxProcessor (limited support)

The following F# semantic and syntactic features are also available:

- Records and Unions
- Structural Equality/Comparison
- Comprehensions (seq, array, list)
- Computation Expressions
- Pattern Matching
- Active Patterns
- Object Expressions
- Units of measure

### Caveats

- Options are **erased** in JS (`None` translates to `null`).
- `IEnumerable` interface cannot be implemented. Use `seq` comprehensions
  or `Seq` module functions to generate lazy sequences.
- `Async.RunSynchronously` is not supported.
- When opening `Fable.Core` you'll have access to `Async.AwaitPromise`
  and `Async.StartAsPromise` extensions for easy interaction with [JS Promises](http://babeljs.io/docs/learn-es2015/#promises).
- `MailboxProcessor` is single-threaded in JS and currently only
  `Start`, `Receive`, `Post` and `PostAndAsyncReply` are implemented
  (`cancellationToken` or `timeout` optional arguments are not supported).



## Object Oriented Programming

The following OOP features are compatible with Fable:

**Interface** methods are compiled as normal object methods (care is needed to prevent name collision)
and it's possible to test against an interface (e.g. `x :? IComparable`) for types defined in F# code.

**Overloads** and **secondary constructors** are allowed in class implementations (not for interfaces),
but they'll have a suffix attached (`_0`, `_1`...) in JS and are not recommended.

**Custom operators** are possible, just note it won't be idiomatic to call
them from JS if necessary (e.g., `Time.op_Addition(ts1, ts2)`).

**Inheritance** is possible and conforms to [ES2015 inheritance](https://github.com/lukehoban/es6features#classes)
but must be done by calling the _primary_ constructor of the base class. Methods can be overridden, but in that case
it won't be possible to access the base methods by casting the object.


## Reflection

Some limited support for reflection is available:

**Type testing** (e.g. `x :? MyType`) is possible, also with custom-defined
interfaces (see above).

**System.Type** can be accessed (either with `.GetType()` or `typeof<MyType>`)
and its `Name`, `FullName` and `Namespace` can be inspected.

**Generic information** disappears in generated code. However, it's accessible
to the compiler, so calls like `typeof<MyType>` are possible with concrete
types or with generics in _inline_ functions.

**Attributes** are only visible to the compiler which uses them, for example, when defining
[foreign interfaces](interacting.html).
