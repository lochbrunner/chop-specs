# Chop Language Specifications

## Features

* Dynamic memory allocation on the stack (bind to the scope)
* Static and structural typed
* Compile time reflection (tbd)
* Zero runtime cost abstraction
* Few keywords
* Allows expressive code

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
x <- 1      // definition
```

is the same as:

```
x: int <- 1     // declaration and definition
y = 1           // Short notation using type deduction
```

#### Numbers

These lines are equivalent

```
stack let a = 12
let a = 12
let a = {
    foo(2)  // Doing some computation
    4       // Gets ignored here
    12      // Last expression gets returned
}
let a = {
    public default 12   // 12 gets returned (= anonym public member)
}
```

Allocating on the heap

```
heap let b = 12
```

Note
> `stack` allocated variables have block lifetime  
> `heap` allocated variables use reference counting

#### Strings

```
let a = "Hello, World"
```

#### Objects

```
let obj = { 
    public a = 12
    public b = a * 3
}
```

of

```
let factory(arg: int) = { 
    public a = arg
    public b = arg * 3
}

let obj = factory(12)

```

Anonymous members

```
let base = { 
    public a = 12
    public b = a * 3
}

let obj = {
    public base
    public c = 4
    public d = "Hello"
}
```

here `obj` is equivalent to

```
let obj = { 
    public a = 12
    public b = a * 3
    public c = 4
    public d = "Hello"
}
```

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
let foo = (a: int) => {
    a * a
}
```

with explicit type

```
let foo: int -> int = (a: int) => {
    a * a
}
```

or

```
let foo(a: int) = {
    a * a
}
```

You can create template function when using the `any` type

```
let foo(a: any) = {
    a * a
}
```

> Note: If the given type has no definition for the `*` operator, you get a compiler error.

```
let foo(a: any) = {
    a * a
}

s := foo("Hello")       // Compiler error
t := foo(3)             // Ok
```

Using a type as anonymous argument:

```
type Argument = {a: int}
let foo = (Argument) => {
    a * a
}
```

is not the same as

```
type Argument = {a: int}
let foo = (arg: Argument) => {
    arg.a * arg.a
}
```

> Hint: You can skip the curly bracket if there is only one expression.

#### Higher order functions

```
let foo = (arg1: int) => (arg2: int) => arg1 * arg2
```

#### Argument binding

```
let foo = (arg1: int) => (arg2: int) => arg1 * arg2
let foo1 = foo(12)
```

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

### Environment Variables

* Inspired by unix shell
* Can be used for *dependency injection*


```
let foo() =  {
    require a: logger: (string) => void
    require a: int
    logger("Entering foo")
    let b = a              // Accessing variable of caller
}

{   // Defining a new scope
    export let logger(msg: string) = stderr.write(msg.toString)
    export let a = 12
    foo()
}
```

Sourcing

```
lazy let env = {
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

```
outer_loop: for i in [1..10] {
    inner_loop: for j = [1..i] {
        if ... continue inner_loop;
        if ... break outer_loop;
    }
}
```

### Pattern Matching

```
y := match x = {
    case t: Type1 => t.foo()            // Match concrete type
    case s: any & {m: int} => s.m       // `x` contains integer member `m`
    case u: any & {m: 12} => 34         // `x` member `m` has value `12`
    case x > 10 => 34                   // `x` is larger than 10
} 
```

Open points:
* Using *JavaScript* keywords `let` and `const` or from *Scala* `var` and `val` ? Or `mut` for mutables? Or only operators (`:=`, `:`, `=` and `<-`)
* Default access modifier `public` or `private` ?