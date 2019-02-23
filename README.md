# Chop Language Specifications

Goal:

> Combine the top features of existing programming languages into one consistent and non-verbose and plausible programming language.

## Features

* Dynamic memory allocation on the stack (bind to the scope; Under discussion)
* Static and structural typed
* Meta programming with ease and language server support
* Zero runtime cost abstraction
* Few keywords
* No distinction of space and newline character in the code.
* Allows expressive code (minimal boilerplate)
* Using much of the C++ STL internally where appropriate
* Also suitable for creating proprietary and/or certified libraries
* Generating docs out of the code (empowered by meta-programming)

Tools

* Compiler (no need for linter)
* REPL
* Formatter
* Language Server

Inspired by

* C: The syntactical base
* Go
* Scala: syntax
* JavaScript (TypeScript): module system
* rust: enums, move, docs (not included now)

## Examples

### Variables

Define immutables:

```code
x := 1
```

with explicit type

```code
x := cast<int32>(1)    // or
x := 1.as<int32>       // or
x: int32 := 1
```

Define mutable

```code
x: int      // declaration
x <- 1      // assignment
```

is the same as:

```code
x: int <- 1     // declaration and definition
y = 1           // Short notation using type deduction. Note: This notation is still under discussion
```

> Open Point: Should `<-` be also to be overloaded for custom behavior? For instance sending messages to a queue?

#### Numbers

These lines are equivalent

```code
stack a := 12
a := 12
a := {
    foo(2)  // Doing some computation
    4       // Gets ignored here
    12      // Last expression gets returned
}
a := {
    public default 12   // 12 gets returned (= anonym public member)
}
```

Kinds of ownership

```code
stack a := 14    // Ownership by the scope
shared b := 12   // Shared ownership (reference counting)
unique c := 15   // Ownership will be moved
```

#### Strings

```code
a := "Hello, World"
```

String-interpolation (Scala)

```code
a:= s"Hello $name ${obj.foo}"
```

String concatenation

```code
a := "Hello "
b := "World!"
c := a + b
```

#### Alias

You can give many "things" an alias name, which get resolved at compile time

```code
x = 12
y :- x                      // This is an alias for x

y <- 14
stderr.write(x)             // Prints 14 to the std err
```

#### Objects

```code
obj := {
    public a = 12
    public b = a * 3
}
```

of

```code
factory(arg: int) := {
    public a = arg
    public b = arg * 3
}

obj := factory(12)

```

Anonymous members

```code
base = {
    public a = 12
    public b = a * 3
}

obj = {
    public base
    public c = 4
    public d = "Hello"
}
```

here `obj` is equivalent to

```code
obj := {
    public a = 12
    public b = a * 3
    public c = 4
    public d = "Hello"
}
```

### Enums

Enums can either be implemented as integers (C++) or as strings (Typescript).

The representation are always strings

```code
type MyEnum = "One" | "Two" | "Three"
```

It is possible to use them for pattern matching (see later).

```code

type MyEnumA = "One" | "Two" | "Three"
type MyEnumB = "Four" | "Five" | "Six"

x: MyEnumA | MyEnumB
// assign some stuff to x

y := match x {
    case a: MyEnumA => fooA(a)
    case b: MyEnumB => fooB(b)
}
```

### Builtin Types

* Integral types
  * `int8`
  * `int16`
  * `int32` Using `int` as alias?
  * `int64`
  * `uint8`
  * `uint16`
  * `uint32`
  * `uint64`
* Floating point types
  * `float8`
  * `float16`
  * `float32` Using `float` as alias?
  * `float64`

### Arrays

Are arrays always appendable?

```code
a := int[10]
shared b := int[n]
c := int[n]         // Figure out, if this is possible
```

### Containers

Should be implemented as "template" types in **chop** itself.

* String: `string`
* String Builder: `string_builder` ?
* Hash map: `map<Type>`
* C++ `std::vector` : `vector`

### Custom Types

```code
type MyType = {
    a: int
    b: string
    c: {
        d: int
    }
}
```

Extending types

```code
type MyExtendedType = {
    d: float32
    MyType
}
```

Alternative syntax

```code
MyType :- {
    a: int
    b: string
    c: {
        d: int
    }
}
```

Questions: Is this possible?
Problems:

* Difference between types and scopes
  * Must there be a `public` before the members

#### Combining types

Having

```code
type TypeA = {
    a: int
    c: int
}

type TypeB = {
    b: int
    c: int
}
```

then

```code
type TypeAandB = TypeA & TypeB
```

results in

```code
type TypeAandB = {
    a: int
    b: int
    c: int
}
```

and

```code
type TypeAorB = TypeA | TypeB
```

results in

```code
type TypeAorB = {
    c: int
}
```

> Note you can not create space for every composed type as value on the stack. The full power of composed type are only available when they are of shared ownership.

#### Object Destructuring

Using alias for destructuring

```code
type SampleType = {
    a: int
    b: string
    c: {
        d: float
    }
}

obj: SampleType = ...

{x :- a, y :- c.d} = obj
```

### Dimensional Analysis

Inspired by: https://github.com/nholthaus/units

Defining units as

```code
unit<u32> Duration { // Supports only u32
    s
}

unit Mass {
    kg
}

unit Length {
    m
}

unit Force {
    N,
    kg*m/s/s
}
```

The dimension `force` is introduced with units abbreviation `N` which is equal to the previous defined units `kg*m/s/s`.
They are applied automatic to each numerical primitive.
Dimensions get computed automatically when using the operators `+`, `-`, `*` and `/`.

```code
foo := (force: Force<int32>) => {
    //...
}

// Calling
foo(12N)

// or
let l = 13m
let m = 3kg
let t = 3s

foo(m*l/t/t)
```

Dimensional analysis is performed during compile time.

> Note: Can this feature be introduced using meta programming?

Possible solution: Each token which can not be parsed with standard tokenizer get tried to match with registered extension implemented via meta programming.

> Open point: Should this be constrained in order to increase readability of the code?

### Functions

```code
foo := (a: int) => {
    a * a
}
```

with explicit type

```code
foo: int -> int := (a: int) => {
    a * a
}
```

or

```code
foo(a: int) := {
    a * a
}
```

or simply

```code
foo(a: int) := a * a
```

You can create template function when using the `any` type

```code
foo(a: any) := {
    a * a
}
```

> Note: If the given type has no definition for the `*` operator, you get a compiler error.

```code
foo(a: any) := {
    a * a
}

s := foo("Hello")       // Compiler error
t := foo(3)             // Ok
```

Using a type as anonymous argument:

```code
type Argument = {a: int}
foo := (Argument) => {
    a * a
}
```

is not the same as

```code
type Argument = {a: int}
foo := (arg: Argument) => {
    arg.a * arg.a
}
```

> Hint: You can skip the curly bracket if there is only one expression.

#### Higher order functions

```code
foo := (arg1: int) => (arg2: int) => arg1 * arg2
```

#### Argument binding

```code
foo := (arg1: int) => (arg2: int) => arg1 * arg2
foo1 := foo(12)
```

#### Scope of Parameters

By default parameters have block scope and therefor it uses "call by value"

```code
foo := (x: int) => {}

i := 12
foo(i)      // Call by value (copy)
foo(:-i)    // Call by reference Should this be possible?
```

Better:

```code
shared i := 12
unique j := 14

foo(i)      // Using shared ownership of i
foo(j)      // Now foo is the owner of j
foo(j)      // Error: j is null because you lost its ownership
```

> Note: Libraries are only possible if either:
>
> 1. There is no final library concept. Which means libraries contain byte-code but not machine code
> 2. All variations of that code get packed in the library
>
> Prefer 1. solution

#### Extensions

```code
MyType.foo := (a: int) => {
    this.a = a
}
```

Assigning extensions to multiple types:

```code
(MyType1 | MyType2).foo := (a: int) => {
    this.a = a
}
```

> Note Extensions are immutable. Which means you can not assign an extension function with the same name and signature to type.

##### Abstract Extension Definitions

```code
operator .addTwice := (left, right) => left + right*2
```

Which can be attached to each type which has `+` operation defined.

### Piping

Inspired by Unix Pipes

#### Basics: Named pipe

```code
my_pipe: pipe<int>

foo := () => {
    result: int;
    // Do some magic
    // ...
    result
}

lazy my_pipe << foo()
// Nothing happened so far

i :<< my_pipe       // Here foo gets called
```

Using as a generator

```code
fibonacci() := {
    shared result: pipe<int>    // named pipes should always have `shared` ownership
    i = 0
    j = 1

    // The lazy loop gets only evaluated when the someone reads from the pipe
    lazy loop {
        t := i + j
        i <- j
        j <- t
        result << t
    }
    result
}
```

#### Pipe compositions

Unnamed pipe:

```code
composition := foo | faa | fuu
```

Which is the same as

```code
composition := (arg1 : ArgType1) => {
    fuu(faa(foo(arg1)))
}
```

Using space is equivalent to newline:

```code
composition :=
    foo
    | faa
    | fuu
```

Creating a switch:

```code
// this function has 2 unnamed output pipes
switch: FooOutputType -> &2 := (input: FooOutputType) => {
    loop {
        i :<< input
        if ...
            &0 << ...        // Write to first pipe
        else
            &1 << ...        // Write to second pipe
    }
}

composition :=
    foo
    | switch
    \ fuu       // Uses output of the first pipe of the switch
      | faa     // Uses the output of fuu. Note: faa is not "multi-piped"
    \ fee       // Uses the output of the second pipe of the switch
```

Hints:

You can *name* the unnamed pipes

```code
my_pipe :- &1;
my_pipe << ...
```

Accessing the pipes dynamically

```code
foo := () => {
    i := 0
    &$i << ..       // You have to specify the number of output pipes when using this
}
```

Returning named pipes:

```code
switch: FooOutputType -> &2 := (input: FooOutputType) => {
    loop {
        i :<< input
        if ...
            good << 12        // Undeclared pipes are output types automatically
        else
            bad << 34
    }
}

composition :=
    foo
    | switch    // Compiler knows that switch has these two *output* pipes
    \bad sorry
    \good hurray

```

> Note: Can this be archived with Monads?

### Environment Variables

* Inspired by unix shell
* Can be used for *dependency injection*
* Keyword: `set`

```code
foo() =  {
    require a: logger: (string) => void
    require a: int
    logger("Entering foo")
    b := a              // Accessing variable of caller
}

{   // Defining a new scope
    set logger(msg: string) := stderr.write(msg.toString)
    set a := 12
    foo()
}
```

Sourcing

```code
lazy env := {
    set logger = (msg: string) => stderr.write(msg.toString)
}

{
    source env  // Getting the logger
    logger("Hello, World!")
}

```

> TODO: Can be converged with the `:-` operator ala: replace `lazy env := {...}` with `env :- {...}` ?  

### Control Statements

Declaration inside condition

```code
if x := foo() > 0 {     // x is immutable
    stderr.write(x);
}
```

Naming blocks with `:-` get executed directly.
They can be used for accessing with `break`, `continue` and `goto`.

```code
named_block :- {
    // So something
}
```

```code
outer_loop :- for i in [1..10] {
    inner_loop :- for j = [1..i] {
        if ... continue inner_loop;
        if ... break outer_loop;
    }
}
```

### Pattern Matching

```code
y := match x {
    case t: Type1 => t.foo()            // Match concrete type
    case s: any & {m: int} => s.m       // `x` contains integer member `m`
    case u: any & {m: 12} => 34         // `x` member `m` has value `12`
    case x > 10 => 34                   // `x` is larger than 10
}
```

### Module Concept

Goal: Merge EcmaScript5 module concept with access-level concept of obects.

The module concept is similar to that of EcmaScript5.

The source files or IL files of each library defines its own module.

You can import other modules either if their are available as source or as IL files with the keyword `import`:

Example: Importing local file *./package/module.chop*

```code
import ./package/module_x     // The extension is skipped
```

You can think of that as if the compiler would insert a `{` in the first line and `}` at the last line of the imported file and paste them into the importing file.

This means the imported file gets its own scope named *module_x* here and therefore only the declarations marked as `public` (`internal` see later) can be used here.

Similar to the Unix path convention you can import "global" modules when you leaf the `./` prefix of the module name. Then the compiler tries to find that file at some specific paths.

Note that you can rename imported modules via

```code
new_module_name :- import old_module_name
```

#### Creating bundles

Unfortunately it is not convenient to import a bunch of modules only to use several functionalities of on library.

To avoid the author of the library can bundle there functionality via re-imports.

The main_module might look as:

```code
import ./submodule_a
import ./submodule_b


public submodule_a
public submodule_b
```

This can be imported as

```code
import ./main_module

let x:= main_module.foo_a
```

You can make them public under a new name

```code
import ./submodule_a

public feature_a := submodule_a
```

If one module has the name *index.ext* it gets implicit imported when you specify the containing folder in the import statement.

For instance you have the following folder structure in your library:

```text
my_library
|- index.ext
|- submodule_a
|  |- index.ext
|  |- any_sub_submodule.ext
.
.
.
```

In the root *index.ext* you can import the "folder" *./sub_module_a* which implicit would import *./sub_module_a/index.ext*.

The client code would import the library via

```code
import my_library
```

can have access to all public elements of the whole library.

#### Internal modules

In order to safe IP you can hide internal code with the keyword `internal`.

`internal` declarations are not public from JL files.

### Meta-Programming

tbd

Some ideas:

* Using `$` as prefix?
* Replacing code generators. (e.g. no need for `protoc` anymore)
* Annotating code with custom qualifiers, which checking (e.g. `realtime` or `license`; or guaranties safety to a norm *ISO26262*)

#### Meta programming

Example: Writing a JSON serializer

The type to serialize

```code
type MyType = {
    a: int
    b: string
    c: {
        d: int
        e: float
    }
}
```

The serializer

```code
jsonify(obj) = {
    json: = "{\n"
    type := $obj.type       // With $ you can access compiler information of a variable or any other symbol
    // type is now a compiler variable (similar to constexpr in C++)
    member_loop :- for {name :- key, member :- value} in type.members {     // Done in compiletime Note "type.members" is hashmap
        json += s"\"$name\": "
        json += match member.type {                 // Match gets evaluted during compiletime
            case int | float: member.value          // toString get optional called
            case string: s"\"${member.value}\""
            case object: jsonify(member.value)
        }
        if !member_loop.isLast                      // Accessing for loops internal states (only allowed because type.members support random access to its items)
            json += ",\n"
    }
    json + "\n}"
}
```

> Note: The for loop gets executed by the compiler as far it can.
> This is similar to C++ template programming, but it acts on an intermediate code representation.
> When using the prefix `$` you can have access to the same information as the compiler uses generating the target code.

Usage

```code
obj: MyType = ...
json := jsonify(obj)        // Not that the compiler will evaluate the template here. (Type Caching possible)
```

#### Custom Annotations

```code
foo_realtime(x: int) = @realtime {
    x+2
}

foo_no_realtime(x: int) = {
    // Doing some stuff like
    // dynamic memory allocation/deallocation
    // Calling non realtime functions
    // Loops with dynamic number cycles
}

foo() = @realtime {
    foo_no_realtime      // Compiler Error: Function, which calls no realtime
                         // function, can not be realtime anymore
}
```

Implementation via meta-programming: tbd

#### Injecting custom compiler information into to IL

You can access compiler variables directly with the `$$` prefix.

Library code

```code
$$vendor = "My Inc"
```

This variable is not visible to the compiler back-end (e.g. LLVM) but it can be used by the compiler front-end when importing it. In contrast to `$a`.

```code
import ./my_lib

lib_vendor := $$my_lib.vendor
```

> Note `$_.a` is the same as `$a`  
> Note: Compiler implementation detail: `$` is a hashmap where all compiler relevant information about that scope is stored.  
> TODO: Is is it also possible to archive that with `const` which declares variables which are only visible by the compiler front-end?

## Notes on compiler implementation

### Intermediate Language

* Format:
  * binary
  * fast searchable
* Motivation:
  1. Caching parsers work
  1. Protecting IP for proprietary libraries
* Allows C++ inspired templates
* Can be translated to LLVM-IR
* No runtime code optimization (this gets done by LLVM back-end)

## Open points

* Default access modifier `public` or `private` ?
  * Suggestion: `private`
* Abbreviation:
  * `public` vs `->`
  * `require x: Type` vs `-> x: Type` or something else?