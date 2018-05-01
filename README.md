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

Note
> `stack` allocated variables have block lifetime  
> `heap` allocated variables use reference counting

#### Strings

```
a := "Hello, World"
```

String-interpolation (Scala)

```
a:= s"Hello $name ${obj.foo()}"
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

Switch

```
// this function has 2 unnamed output pipes
switch: FooOutputType -> &2 := (input: pipe<FooOutputType>) => {
    loop {
        i :<< input
        if ...
            &0 << 12        // Write to first pipe
        else
            &1 << 34        // Write to second pipe
    }
}

composition := 
    foo
    | switch
    \ fuu   // Uses output of the first pipe of the switch  
    \ fee   // Uses the output of the second pipe of the switch
```

#### Named pipe

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

## Open points

* Using *JavaScript* keywords `let` and `const` or from *Scala* `var` and `val` ? Or `mut` for mutables? Or only operators (`:=`, `:`, `=` and `<-`)
* Default access modifier `public` or `private` ?