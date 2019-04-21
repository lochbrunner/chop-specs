# Chop Language Specifications

Goal:

> Combine the top features of existing programming languages into one consistent and non-verbose and plausible programming language.

* System language (inspired by Rust)
* Unifies the syntax of similar use cases.
* Extendable by the User
* Using the same language for program, meta-programming and build configuration

All features of that language are specified by examples.

## Features

* Static and structural typed (Most type and lifetime annotations can be omitted due to inferring capability of the compiler)
* Meta programming with ease and language server support
* Zero runtime cost abstraction (-> No *GC*)
* Few keywords
* Allows expressive code (less boilerplate than *rust*)
* Also suitable for creating proprietary and/or certified libraries
* Generating docs out of the code (empowered by meta-programming)
* Research topic: Dynamic memory allocation on the stack (bind to the scope)

Tools

* Compiler (no need for linter)
* REPL
* Formatter
* Language Server

Inspired by

* rust: syntax, move semantic and checkers
* Go: syntax and module system
* Scala: syntax
* JavaScript (TypeScript): module system, syntax

## Examples

### Variables

Define and declare a local variable:

```code
x := 1
```

with explicit type

```code
x := cast<int32>(1)    // or
x := 1.as<int32>       // or
x: int32 := 1
```

Define a public variable:

```
x :+ 1
```

which can be accessed from outside the block:

```
a := {b:+12}.b  // a is now 12
```

### Mutable variables

```code
mut x := 1      // declaration
x <- 2          // assignment
```

> Note: `<-` is just an operator defined for integers. For instance sending messages to a channel.
> This could be have other behavior for other types.

### Shadowing

Inspired by rust

```
a := "12.34"
a := parse(a)       // Shadows the variable declaration above
```

### Ownership, Borrowing and Box

Taken from rust.

### Numbers

These lines are equivalent

```code
a := 12
a := {
    foo(2)  // Doing some computation
    4       // Gets ignored here
    12      // Last expression gets returned as a default
}
```

### Strings

```code
a := "Hello, World"
```

String-interpolation (Scala)

```code
a:= s"Hello $name ${obj.foo}"
```

Gets expanded to

```code
a:= "Hello " + name.to_string + obj.foo.to_string
```

Using different formatters

```code
a:= "Today is ${date|"YYYY/MM/DD"d}"
```

Where `"YYYY/MM/DD"d` creates a function which formats the given date into a string or object which contains the `to_string` method. This must be registered via:

```code
postfix d := (code: str) => (date: Date) => {
    // Some code returning the formated date string
}
```

or:

```code
String.d := (code: str) => (date: Date) => {
    // Some code returning the formated date string
}
```

Which defines the method `.d` to Strings.

String concatenation

```code
a := "Hello "
b := "World!"
c := a + b
```

### Alias

You can give many "things" an alias name, which get resolved at compile time

```code
x = 12
y :- x                      // This is an alias for x

y <- 14
stderr.write(x)             // Prints 14 to the std err
```

### Objects

```code
obj := {
    a :+ 12
    b :+ a * 3
}
```

of

```code
factory(arg: int) := {
    a :+ arg
    b :+ arg * 3
}

obj := factory(12)

```

Anonymous members

```code
base = {
    a :+ 12
    b :+= a * 3
}

obj = {
    ...base
    c :+ 4
    d :+ "Hello"
}
```

here `obj` is equivalent to

```code
obj := {
    a :+ 12
    b :+ a * 3
    c :+ 4
    d :+ "Hello"
}
```

### Special Method: Destructor

In order to specify the behavior when some object lifetime reaches it's end.

```code
obj := {
    ~ :+ () => {
        // Do some clean up stuff
    }
}
```

#### Open Point on Movement and object construction

```code
i := 12
obj := {
    a :+ i
}
```

Is `i` then moved into the object and no longer valid?

## Enums

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

> Note: Consider using the type constraints as values too. Using meta programming.

## Builtin Types

* Integral types
  * `int8` or `i8`
  * `int16` or `i16`
  * `int32` or `i32` Using `int` as alias?
  * `int64` or `i64`
  * `uint8` or `u8`
  * `uint16` or `u16`
  * `uint32` or `u32`
  * `uint64`  or `u64`
* Floating point types
  * `float8` or `f8`
  * `float16` or `f16`
  * `float32` or `f32` Using `float` as alias?
  * `float64` or `f64`

## Arrays

```code
a := int[10]
b := int[n]         // Figure out, if this is possible
```

> Are arrays always extendable?

## Containers

Should be implemented as "template" types in **chop** itself.

* String: `string`
* String Builder: `string_builder` ?
* Hash map: `map<Type>`
* C++ `std::vector` : `vector`

## Custom Types

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
    ...MyType
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

* Difference between types and scopes:
  * In types everything is public

### Combining types

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

### Constraints on types

```
type Evens = 2*i with i: int32
```

### Object Destructuring

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
{a} := obj                  // a is moved out of obj
{a} :- obj                  // a is now a short descriptor for obj.a
// With renaming
{x :- a, y :- c.d} :- obj
```

## Dimensional Analysis

Inspired by: [nholthaus/units](https://github.com/nholthaus/units)

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

## Functions

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

> The keyword `any` can be omitted in most situations due this does not provide useful information.
>
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

### Generic functions

> Note: By default the functions have generic argument types.
They get constraint by the compiler identifying their use.
This means that all constraints must be available in the intermediate language description of the libraries.  

### Higher order functions

```code
foo := (arg1: int) => (arg2: int) => arg1 * arg2
```

### Argument binding

```code
foo := (arg1: int) => (arg2: int) => arg1 * arg2
foo1 := foo(12)
```

### Scope of Parameters

By default parameters have block scope and therefor it uses "call by value"

```code
foo := (x: int) => {}

faa := (x: &int) => {}

i := 12
j := 13
foo(i.clone)    // A copy of i gets moved to foo
foo(i)          // i itself gets moved to foo
foo(i)          // error: i was moved before
faa(&j)         // j gets borrowed
```

### Extensions

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

```code
MyType.foo <- (a:int) => {}     // error <- to undeclared foo is not allowed
```

Virtual functions are only allowed to instances and not to types.



#### Abstract Extension Definitions

```code
infix .addTwice := (left, right) => left + right*2
```

Which can be attached to each type which has `+` operation defined.

> Hint: This can be used for creating iterators used by for loops.
>
> Open Points
>
> * How to namespace Operators?  
>   Proposal: Operators must be imported (explicitly)  
>   What about standard arithmetic operators (e.g. `2+3`) ?
> * How to specify the precedence of the new operator?
> * How to specify the type (e.g. infix, postfix or prefix) of the new operator?

## Piping

Inspired by Unix Pipes

> Note: Many open points here!

### Basics: Named pipe

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

### Pipe compositions

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

### Implementation

> Pipes are defined via ordinary operator definitions

## Environment Variables

* Inspired by unix shell
* Can be used for *dependency injection*
* Keywords: `set` and `require`

```code
foo(x) =  {
    require a: logger: (string) => void
    require a: int
    logger("Entering foo")
    b := a              // Accessing variable of caller
}

{   // Defining a new scope
    set logger(msg: string) := stderr.write(msg.toString)
    set a := 12
    foo(12);
}
```

> TODO: Using alternative syntax `:>` and `:<` ?

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

## Control Statements

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

**Consequence**

```code
a :- if x :+ foo() > 0 {}
x = a.x                     // x has now the value of foo()
```

## Pattern Matching

```code
y := match x {
    case t: Type1 => t.foo()            // Match concrete type
    case s: any & {m: int} => s.m       // `x` contains integer member `m`
    case u: any & {m: 12} => 34         // `x` member `m` has value `12`
    case x > 10 => 36                   // `x` is larger than 10
}
```

## Module Concept

Goal: Merge EcmaScript5 module concept with access-level concept of objects.

The module concept is similar to that of EcmaScript5.

The source files or IL files of each library defines its own module.

You can import other modules either if their are available as source or as IL files with the function `import`:

Example: Importing local file *./package/module.chop*

```code
module_x :- import ./package/module_x     // The extension is skipped
```

`import` does not need be implemented in the compiler itself. It is implemented in a default imported module using meta-programming.
You can think of that as if the compiler would insert a `{` in the first line and `}` at the last line of the imported file and paste them into the importing file.

This means the imported file gets its own scope named *module_x* here and therefore only the declarations marked as `public` (`internal` see later) can be used here.

Similar to the Unix path convention you can import "global" modules when you leaf the `./` prefix of the module name. Then the compiler tries to find that file at some specific paths.

Note that you can rename imported modules via

```code
new_module_name :- import old_module_name
```

It is also possible to import modules from public repositories. For instance

```code
module_x :- import www.github.com/publisher/package/module_x     // The extension is skipped
```

Even if the source code is not public, it is also possible to import the IR code of the package without limitations.

### Creating bundles

Unfortunately it is not convenient to import a bunch of modules only to use several functionalities of on library.

To avoid the author of the library can bundle there functionality via re-imports.

The main_module might look as:

```code
sub_module_a :+ import ./sub_module_a
sub_module_b :+ import ./sub_module_b
```

This can be imported as

```code
import ./main_module

let x:= main_module.foo_a
```

You can make them public under a new name

```code
import ./sub_module_a

public feature_a := sub_module_a
```

If one module has the name *index.ext* it gets implicit imported when you specify the containing folder in the import statement.

For instance you have the following folder structure in your library:

```text
my_library
|- index.ext
|- sub_module_a
|  |- index.ext
|  |- any_sub_sub_module.ext
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

### Internal modules

In order to safe IP you can hide internal code with the keyword `internal`.

`internal` declarations are not public from JL files.

> TODO: It might be better that everything must be marked explicit as public.

## Meta-Programming

tbd

Some ideas:

* Using `$` as prefix? Pro: the `$` Symbol means always go one meta-layer deeper: `a := "env: $$$USER"` is the shell environment variable of the compiler process placed in a string of the compiled program.
* Replacing code generators. (e.g. no need for `protoc` anymore)
* Annotating code with custom qualifiers, which checking (e.g. `real-time` or `license`; or guaranties safety to a norm *ISO26262*)
* It should also be possible to read and write files during compilation with the standard File API. Use Cases: Generating schemas and documents during compiling out of the code. 

### Meta programming

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
    member_loop :- $for {name :- key} in type.members {     // Done in compile-time Note "type.members" is hashmap
        json += s"\"$name\": "
        json += match member.type {                 // Match gets evaluated during compile-time
            case int | float: $obj[name]            // toString get optional called
            case string: s"\"${$obj[name]}\""
            case object: jsonify($obj[name])
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

### Custom Annotations

```code
foo_realtime(x: int) = @realtime {
    x+2
}

foo_no_realtime(x: int) = {
    // Doing some stuff like
    // dynamic memory allocation/de-allocation
    // Calling non real-time functions
    // Loops with dynamic number cycles
}

foo() = @realtime {
    foo_no_realtime      // Compiler Error: Function, which calls no realtime
                         // function, can not be realtime anymore
}
```

Implementation via meta-programming: tbd

TODO: Security

### Injecting custom compiler information into to IL

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

> Note `$_["a"]` is the same as `$a`  
> Note: Compiler implementation detail: `$` is a hashmap where all compiler relevant information about that scope is stored.  
> TODO: Is is it also possible to archive that with `const` which declares variables which are only visible by the compiler front-end?

## Code sanitizing

In order to avoid cryptic, verbose and unreadable compiler errors known from some C++ templates, the author of meta-functions should be able to check the arguments and create custom error messages on invalid arguments.

## Developer Experience

* The compiler should detect as much type (and lifetime) information out of the code as possible in order to reduce the amount of annotations written by the developer and make the code more concise.

## Use Cases

### Objects of different type in a vector

```c++
class Base{
 public:
  virtual foo() = 0; 
};

class A : public Base{
 public:
  foo() override { /* ... */};
 private:
  int a;
};

class B : public Base{
 public:
  foo() override { /* ... */};
 private:
  int b,c;
};

int main(int, char**) {
    std::vector<Base*> v;
    v.push_back(new A());
    v.push_back(new B());

    for(auto item : v) {
        item->foo();
    }
    return 0;
}
```

```code
type Base {
    foo : () => void;
}

create_a := () => {
    a :+ ...
    foo :+ () => ...
}

create_b := () => {
    b :+ ...
    c :+ ...
    foo :+ () => ...
}

create_c := () => {
    fii :+ () => ...
}

postfix .faa := (obj) => obj.foo  // Compiler knows that obj must have member foo

v := Vector<Box<Base>>::new()

v.push(Box.new(create_a))      // Compiler aligns memory layout to Base here
v.push(Box.new(create_b))      // Compiler aligns memory layout to Base here
v.push(Box.new(create_c))      // error: member foo is missing

create_c.faa                    // error: member foo is missing

for item in v.iter {
    item.unwrap.foo             // Brackets () are optional
    item.unwrap.faa
}
```

## Notes on compiler implementation

### Intermediate Language

* Format:
  * binary
  * fast searchable
* Motivation:
  1. Caching parsers work
  1. Storing function's signature with predicates
  1. Protecting IP for proprietary libraries
* Can be translated to LLVM-IR
* No runtime code optimization (this gets done by LLVM back-end)

## Open points

* Default access modifier `public` or `private` ?
  * Suggestion: `private`
* Abbreviation:
  * `public` vs `->`
  * `require x: Type` vs `-> x: Type` or something else?
