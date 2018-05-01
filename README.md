# Chop Language Specifications

## Features

* Dynamic memory allocation on the stack (bind to the scope)
* Static and structural typed
* Compile time reflection (tbd)
* Zero runtime cost abstraction
* Few keywords
* Allows expressive code
* No distinction be space ` ` and newline in the code.

Inspired by

* Go
* Scala
* JavaScript (TypeScript)

## Examples

### Variables

Define immutables:

```
x := 1
``` 

with explicit type

```
x := cast<int32>(1)    // or
x := 1.as<int32>       // or
x: int32 := 1
``` 

Define mutable

```
x: int      // declaration
x <- 1      // assignment
```

is the same as:

```
x: int <- 1     // declaration and definition
y = 1           // Short notation using type deduction
```

#### Numbers

These lines are equivalent

```
stack a := 12
a: = 12
a: = {
    foo(2)  // Doing some computation
    4       // Gets ignored here
    12      // Last expression gets returned
}
a: = {
    public default 12   // 12 gets returned (= anonym public member)
}
```

Kinds of ownership

```
stack a := 14    // Ownership by the scope
shared b := 12   // Shared ownership (reference counting)
unique c := 15   // Ownership will be moved
```

#### Strings

```
a := "Hello, World"
```

String-interpolation (Scala)

```
a:= s"Hello $name ${obj.foo()}"
```

#### Alias

You can give many "things" an alias name, which get resolved at compile time

```
x := 12
y :- x                      // This is an alias for x

y <- 14
stderr.write(x)             // Prints 14 to the std err
```

#### Objects

```
obj := { 
    public a = 12
    public b = a * 3
}
```

of

```
factory(arg: int) := { 
    public a = arg
    public b = arg * 3
}

obj := factory(12)

```

Anonymous members

```
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

```
obj := { 
    public a = 12
    public b = a * 3
    public c = 4
    public d = "Hello"
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

```
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

```
type MyType = {
    a: int
    b: string
    c: {
        d: int
    }
}
```

Extending types

```
type MyExtendedType = {
    d: float32
    MyType
}
```

Alternative syntax

```
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

```
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

```
type TypeAandB = TypeA & TypeB
```

results in

```
type TypeAandB = {
    a: int
    b: int
    c: int
}
```

and 

```
type TypeAorB = TypeA | TypeB
```

results in

```
type TypeAorB = {
    c: int
}
```

### Functions


```
foo := (a: int) => {
    a * a
}
```

with explicit type

```
foo: int -> int := (a: int) => {
    a * a
}
```

or

```
foo(a: int) := {
    a * a
}
```

or simply

```
foo(a: int) := a * a
```

You can create template function when using the `any` type

```
foo(a: any) := {
    a * a
}
```

> Note: If the given type has no definition for the `*` operator, you get a compiler error.

```
foo(a: any) := {
    a * a
}

s := foo("Hello")       // Compiler error
t := foo(3)             // Ok
```

Using a type as anonymous argument:

```
type Argument = {a: int}
foo := (Argument) => {
    a * a
}
```

is not the same as

```
type Argument = {a: int}
foo := (arg: Argument) => {
    arg.a * arg.a
}
```

> Hint: You can skip the curly bracket if there is only one expression.

#### Higher order functions

```
foo := (arg1: int) => (arg2: int) => arg1 * arg2
```

#### Argument binding

```
foo := (arg1: int) => (arg2: int) => arg1 * arg2
foo1 := foo(12)
```

#### Scope of Parameters

By default parameters have block scope and therefor it uses "call by value"

```
foo := (x: int) => {}

i := 12
foo(i)      // Call by value (copy)
foo(:-i)    // Call by reference Should this be possible?
```

Better:

```
shared i := 12
unique j := 14

foo(i)      // Using shared ownership of i
foo(j)      // Now foo is the owner of j
foo(j)      // Error: j is null because you lost its ownership
```

> Note: Libraries are only possible if either:
> 1. There is no final library concept. Which means libraries contain bytecode but not machine code
> 2. All variations of that code get packed in the library
> 
> Prefer 1. solution

#### Extensions

```
MyType.foo = (a: int) => {
    this.a = a
}
```

Assigning extensions to multiple types:

```
(MyType1 | MyType2).foo = (a: int) => {
    this.a = a
}
```

> Note Extensions are immutable. Which means you can not assign an extension function with the same name and signature to type.

### Piping

Inspired by Unix Pipes

#### Basics: Named pipe

```
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

#### Pipe compositions

Unnamed pipe:

```
composition := foo | faa | fuu
```

Which is the same as

```
composition := (arg1 : ArgType1) => {
    fuu(faa(foo(arg1)))
}
```

Using space is equivalent to newline:

```
composition := 
    foo
    | faa
    | fuu
```

Creating a switch:

```
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
      | faa     // Uses the output of fuu. Note: faa is not "multipiped"
    \ fee       // Uses the output of the second pipe of the switch
```

Hints:

You can *name* the unnamed pipes

```
my_pipe :- &1;
my_pipe << ...
```

Accessing the pipes dynamically

```
foo := () => {
    i := 0
    &$i << ..       // You have to specify the number of output pipes when using this
}
```

Returning named pipes:

```
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

### Environment Variables

* Inspired by unix shell
* Can be used for *dependency injection*


```
foo() =  {
    require a: logger: (string) => void
    require a: int
    logger("Entering foo")
    b := a              // Accessing variable of caller
}

{   // Defining a new scope
    export logger(msg: string) := stderr.write(msg.toString)
    export a := 12
    foo()
}
```

Sourcing

```
lazy env := {
    export logger = (msg: string) => stderr.write(msg.toString)
} 

{
    source env  // Getting the logger
    logger("Hello, World!")
}

```

### Control Statements

Declaration inside condition

```
if x := foo() > 0 {     // x is immutable
    stderr.write(x);
}
```

Naming blocks with `:-` get executed directly.
They can be used for accessing with `break`, `continue` and `goto`.

```
named_block :- {
    // So something
}
```

```
outer_loop :- for i in [1..10] {
    inner_loop :- for j = [1..i] {
        if ... continue inner_loop;
        if ... break outer_loop;
    }
}
```

### Pattern Matching

```
y := match x {
    case t: Type1 => t.foo()            // Match concrete type
    case s: any & {m: int} => s.m       // `x` contains integer member `m`
    case u: any & {m: 12} => 34         // `x` member `m` has value `12`
    case x > 10 => 34                   // `x` is larger than 10
} 
```

### Meta-Programming

tbd

Using `#` as prefix?

## Open points

* Default access modifier `public` or `private` ?
  * Suggestion: `private`
* Abbreviation:
  * `public` vs `->`
  * `require x: Type` vs `-> x: Type` or something else?