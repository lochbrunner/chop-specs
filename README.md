# Chop Language Specifications

## Features

* Dynamic memory allocation on the stack (bind to the scope)
* Static and structural typed
* Compile time reflection (tbd)
* Zero runtime cost abstraction

Inspired by

* Go
* Scala
* JavaScript

## Examples

### Variables

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

### Interfaces

```
interface MyInterface = {
    a: int
    b: string
    c: {
        d: int
    }
}
```

### Functions


```
let foo = (a: int) => {
    a * a
}
```

or

```
let foo(a: int) = {
    a * a
}
```

or using a interface as anonymous argument:

```
interface Argument = {a: int}
let foo = (Argument) => {
    a * a
}
```

is not the same as

```
interface Argument = {a: int}
let foo = (arg: Argument) => {
    arg.a * arg.a
}
```

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
MyInterface.foo = (a: int) => {
    this.a = a
}
```

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

Open points:
* Using *JavaScript* keywords `let` and `const` or from *Scala* `var` and `val` ? Or `mut` for mutabldes?
* Default access modifier `public` or `private` ?