# Chop Language Specifications

## Motivation

A typical day of a software engineer is often wasted with either fighting against the compiler, (e.g. unnessary type annotation is wrong, hard to understand compiler error messages) or late occuring bugs (e.g. a typo in a dynamicly typed language causes an experiment to fail).
This language tries to free software engineers from long and tidious debugging sessions and over complicated source code.

As it is often hard to predict which design choice might be better, there are sometimes mulitple redundant features in this language that addresses one single problem.
After it turned out that one of those does not provide any real benefit, it might been dropped in the future.

> Combine the top features of existing programming languages into one consistent and non-verbose and plausible programming language.

* System language (inspired by Rust)
* Powerful but simple language: Unifies the syntax of similar use cases to be consistent with minimal exceptions.
* Extendable by the User: Using the same language for program, meta-programming and build configuration
* The complexity of the source code should be linear in the complexity of the problem.
* One language for shell scripting and system programing. (Unification of Bash, Python and C++)

All features of that language are specified by examples.

## Addressed Pain Points

* Runtime dispatching makes it hard follow the code path. Compile time dispatching might be a solution.
* Configuration managment is mostly a diffcult tradeoff. (E.g. overwriting, forwarding, ...)
* Adding valueable debug information at different layers of a callstack for thrown an exception.
* Difficult and limited macro definition syntax. 

## Features

* Static and structural typed (Most type and lifetime annotations can be omitted due to inferring capability of the compiler)
* Meta programming should look and feel as the *runtime* code. With language server support
* Zero runtime cost abstraction (-> No *GC*)
* Few keywords and syntactical exceptions.
* Allows expressive code (less boilerplate than *rust*)
* Also suitable for creating proprietary and/or certified libraries (using secret store)
* Consistent and easy build and dependency management.
* Generating docs out of the code (empowered by meta-programming)
* Experimental: Dynamic memory allocation on the stack (bind to the scope)

Tools

* Compiler (no need for linter)
* REPL
* Formatter
* Language Server

Inspired by

* C: The syntactical base
* rust: move semantic, borrowing and lifetime
* Go: syntax and module system
* Scala: syntax
* JavaScript (TypeScript): module system, syntax

See [Proof of Concept](https://github.com/lochbrunner/chop-compiler) implementation.

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

```code
x :+ 1
```

which can be accessed from outside the block:

```code
a := {b:+12}.b  // a is now 12
```

#### Alternative

```code
.x := 1
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

```code
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

An object is nothing else than a scope with public variables.

```code
obj := {
    a :+ 12
    b :+ a * 3
}
```

or alternativly:

```code
obj := {
    .a := 12
    .b := a * 3
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
  * `i8`
  * `i16`
  * `i32`
  * `i64`
  * `u8`
  * `u16`
  * `u32`
  * `u64`
* Floating point types
  * `f32`
  * `f64`

## Arrays

```code
a := i32[10]
b := i32[n]         // Figure out, if this is possible
```

> Are arrays always extendable?

## Containers

Should be implemented as "template" types in **chop** itself.

* String: `string`
* String Builder: `string_builder` ?
* Hash map: `map<Type>`
* C++ `std::vector` : `vector`

## Named Types

Or Interfaces

```code
type MyType = {
    a: i32
    b: string
    c: {
        d: i32
    }
}
```

Alternative syntax

```code
MyType :- {
    a: i32
    b: string
    c: {
        d: i32
    }
}
```

Another alternative syntax

```code
MyType :- {
    pub a: int
    pub(1) b: string
    c: {
        d: int
    }
}
```

While `1` is the number of scope levels.

Extending types

```code
type MyExtendedType = {
    d: float32
    ...MyType
}
```


Questions: Is this possible?
Problems:

* Difference between types and scopes:
  * In types everything is public. Really? Consider scoping of Rust.

### Methods

Types have a special mutability that allows to extend (non virtual) methods.

```code
MyType :- {
    a: i32
}

MyType.foo :- (self, b: i32) => self.a + b

obj := {a:+ 43}
stdout obj.foo(21)
```

### Combining types

Having

```code
type TypeA = {
    a: i32
    c: i32
}

type TypeB = {
    b: i32
    c: i32
}
```

then

```code
type TypeAAndB = TypeA & TypeB
```

results in

```code
type TypeAAndB = {
    a: i32
    b: i32
    c: i32
}
```

and

```code
type TypeAorB = TypeA | TypeB
```

results in

```code
type TypeAorB = {
    c: i32
}
```

> Note you can not create space for every composed type as value on the stack. The full power of composed type are only available when they are of shared ownership.

### Constraints on types

```code
type Evens = 2*i with i: int32
```

### Object Destructuring

Using alias for destructuring

```code
type SampleType = {
    a: i32
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
foo := (a: i32) => {
    a * a
}
```

with explicit type

```code
foo: i32 -> i32 := (a: i32) => {
    a * a
}
```

or

```code
foo(a: i32) := {
    a * a
}
```

or simply

```code
foo(a: i32) := a * a
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
type Argument = {a: i32}
foo := (Argument) => {
    a * a
}
```

is not the same as

```code
type Argument = {a: i32}
foo := (arg: Argument) => {
    arg.a * arg.a
}
```

> Hint: You can skip the curly bracket if there is only one expression.

### Syntactical Sugar

#### Block as argument

> Under discussion

```code
do_twice(code: Body) := {
    code
    code
}
```

Possible use case implementing of *defer*.

#### Leaving round brackets

Similar to Groovy

> Under discussion

```code
print := (text) => {
    // ...
}


// Usage
print "Hello, World!"
```

### Generic functions

> Note: By default the functions have generic argument types.
They get constraint by the compiler identifying their use.
This means that all constraints must be available in the intermediate language description of the libraries.  

### Higher order functions

```code
foo := (arg1: i32) => (arg2: i32) => arg1 * arg2
```

### Argument binding

```code
foo := (arg1: i32) => (arg2: i32) => arg1 * arg2
foo1 := foo(12)
```

### Scope of Parameters

By default parameters have block scope and therefor it uses "call by value"

```code
foo := (x: i32) => {}

faa := (x: &i32) => {}

i := 12
j := 13
foo(i.clone)    // A copy of i gets moved to foo
foo(i)          // i itself gets moved to foo
foo(i)          // error: i was moved before
faa(&j)         // j gets borrowed
```

### Extensions

```code
MyType.foo := (a: i32) => {
    this.a = a
}
```

Assigning extensions to multiple types:

```code
(MyType1 | MyType2).foo := (a: i32) => {
    this.a = a
}
```

> Note Extensions are immutable. Which means you can not assign an extension function with the same name and signature to type.

```code
MyType.foo <- (a:i32) => {}     // error <- to undeclared foo is not allowed
```

Virtual functions are only allowed to instances and not to types.

### Operators

> type `#` precedence operator-name `:=` or `:+` `(` argument(s) `)` `=>` definition.

With

* **type** is one of `infix`,  `postfix` or `prefix`
* **precedence** specifies the order of evaluation compared to other operators. Must be `SUM`, `PRODUCT` or `EXPONENTIAL`. With a leading `<` or `>` you can specify the precedence one lower or higher than the specified one.

Example:

```code
infix#SUM + := (left, right) => left.unwrap + right.unwrap
```

Which can be attached to each type which has `+` operation defined.

> Hint: This can be used for creating iterators used by for loops.

### Experimental

#### Constructor Sugar

Inspired by Scala's primary constructor.

Instead of creating a object with:

```code
factory := (a, b) => {
    a :+ a
    b :+ b
    c :+ a + b
}
```

Write

```code
factory := (a:+, b:+) => {
    c :+ a + b
}
```

Where `a` and `b` gets captured.

#### Batch processing

```code
transform := (x) => x * 2 + 3

a,b := transform(3,5)
// a = 9
// b = 13
```

Better

```code
transform := _ * 2 + 3

a,b := transform(3,5)
// Or
a,b := (3,5)*2+3
// a = 9
// b = 13
```

## Piping

**Experimental!**

Inspired by Unix Pipes

**Needs refinement**

### Channels

```code
my_channel: channel<i32>

foo := () => {
    result: i32;
    // Do some magic
    // ...
    result
}

lazy my_channel << foo()
// Nothing happened so far

i :<< my_channel       // Here foo gets called
```

Using as a generator

```code
fibonacci := () => {
    {receiver, sender} := channel<i32>::new
    i = 0
    j = 1

    // The lazy loop gets only evaluated when the someone reads from the channel
    lazy loop {
        t := i + j
        i <- j
        j <- t
        sender << t                 // This line triggers the lazy loop
    }
    receiver
}

for value in fibonacci.iter.take(5) {
    stderr.print(value)
}
```

Or is it better to use `yield return` ?

Prints the first five Fibonacci numbers to the standard error.

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

### Motivation

Forwarding parameters though a long

### Examples

```code
foo(x) =  {
    require a: logger: (string) => void
    require a: i32
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

### Alternative

Use `**kwargs` concept from Python, but type checked and not implemented as a dictionary.

Maybe something like this:

```code
bar := (...kwargs) => {
    b: i32 := kwargs.b
}

foo := (a: i32, ...kwargs) => {
    bar(..kwargs)
}

foo(a=1, b=2)
```

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
    t: Type1 => t.foo()            // Match concrete type
    s: any & {m: i32} => s.m       // `x` contains integer member `m`
    u: any & {m: 12} => 34         // `x` member `m` has value `12`
    x > 10 => 36                   // `x` is larger than 10
}
```

Note the syntax is very similar to function definitions.
Therefore you can create runtime dispatcher like that.

```code
foo_caller := (t: Type1) => t.foo() 
m_accessor := (s: any & {m: i32}) => s.m
special_m_member := (u: any & {m: 12}) => 34
greater_than := (x > 10) => 36

y := match x {
    foo_caller
    m_accessor
    special_m_member
    greater_than
}

generic_foo := match {
    foo_caller
    m_accessor
    special_m_member
    greater_than
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

This is the most notable feature of that language.
It should allow to extend the language in an easy and powerful way, without introducing too much new syntax.

Some ideas:

* Using `$` as prefix? Pro: the `$` Symbol means always go one meta-layer deeper: `a := "env: $$$USER"` is the shell environment variable of the compiler process placed in a string of the compiled program.
* Replacing code generators. ~~(e.g. no need for `protoc` anymore)~~
* Annotating code with custom qualifiers, which checking (e.g. `real-time` or `license`; or guaranties safety to a norm *ISO26262*)
* It should also be possible to read and write files during compilation with the standard File API. Use Cases: Generating schemas and documents during compiling out of the code.

### Motivation

1. Compiler development is time consuming and needs a big portion of domain knowledge.
In many cases only a few new features are needed.
Extending the compiler using the very similar syntax as for the main language should lower the barrier.
1. The creativity of the Open-Source community has come up with many innovative ideas that were implemented as programming libraries. What if we could leverage the same amount of creativity and productivity to build an amazing compiler?
1. Moving logic from run-time to compile-time has several advantages:
    1. Faster at runtime.
    1. Earlier catch of bugs.
    1. More information can be exposed to the IDE.

### Examples

Example: Writing a JSON serializer

The type to serialize

```code
type MyType = {
    a: i32
    b: string
    c: {
        d: i32
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
            case i32 | float: $obj[name]            // toString get optional called
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

**Needs refinement**

```code
foo_realtime(x: i32) = @realtime {
    x+2
}

foo_no_realtime(x: i32) = {
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

## Certification

In order to simplify safety, security or realtime certifications, the compiler supports several tooling.

Each function has a secrets store which can store some X-critical tags.
These tags can only be set by an authority.
The compiler can verify the correctness of the tags in local secret store by that authority.

The authority can define rules in which can be used to derive these tags to other functions.

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

### Async/Await or Event loops

**[TODO]**

### Encapsulation

```c++
class A {
 private:
  int a;
 public:
  A(): a(0) {}
  int inc() {
   return a++;
  }
};

A obj;
int a = obj.inc();
int b = obj.a;              // error: a is private          

```

```chop
A := {
  new :+ () => {
      mut a :+ 0        // How to make this protected? another syntax ?
  }
  postfix .inc :+ (obj) => {
      obj.a++
  }
}

obj := A.new            // Should this be mut obj := A.new ?
a := obj.inc
b := obj.a              // no error :(
```

**[TODO]**

## Notes on Compiler Implementation

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

## Shell

With config in `~.choprc` as

```config
import unix
```

This should provide you a Unix like shell experience

```code
ls
```

where the result type of `unix.ls` implements the interface

```code
type ConsoleOutout {
    progress: () -> f32,
    result: string
}
```

### Summary

The Unix shell command `ls` is nothing else than a public function in the module unix which returns a type that can be displayed on the console.

Advantages:

* One language for each kind of task: All language features in shell scripts.
* Avoid subprocess calls
* One can use the same function in other programs as well and the function writer does not have to take care about formatting such as progress bar painting and other complex user interactions.

## Open points

* Default access modifier `public` or `private` ?
  * Suggestion: `private`
* Abbreviation:
  * `public` vs `->`
  * `require x: Type` vs `-> x: Type` or something else?


## Other Pain-Points to trackle

### Debugging:

* Very large stacktrace
* Missing context / function arguments in stacktrace
* Cryptical displayed representation of variables.
