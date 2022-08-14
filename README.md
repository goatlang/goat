# Goat

<img align="right" alt="Goat Logo" src="https://raw.githubusercontent.com/goatlang/goat/main/goat-logo.svg" height="200">

Extended flavor of the Go programming language, aiming for increased value safety and maintainability.

The motivation behind the Goat project is presented in a series of blog posts presenting the upsides and downsides of
Golang and setting the scene for Goat.

Goat specification is based on the premise of keeping things simple. We must comply with a very
basic idea of Go: **There shouldn't be more than one way to do something**. Our basic rule is to not create
fragmentations in Goat syntax.

Goat implementation will most likely be delivered as a code generation tool or as a transpiler producing regular `go`
files. However, full implementation details should be designed once the specification provided in this document
is finalized.

## Goat Specification

Goat syntax and rules are similar to those in Go, with the exception of the proposals presented below.

### Visibility Modifiers

**Motivation**

The main vulnerability of the current visibility system of Go is naming collisions. Whenever you store a private type into a private variable or a public type into a public variable you may run into the following:
```go
type user struct {
  name string
}

func main() {
  user := &user{name: "John"}
  anotherUser := &user{name: "Jane"} // compilation error: user is not a type
}
```
In addition, the current visibility system of Go supports only two visibility modifiers - public and private. This prevents a more fine-grained control over visibility and produces another namespace pollution problem - all symbols in a package are automatically visible from all other files. This creates highly cluttered package namespaces, especially in big packages and big projects.

**Solution**

Goat should support three visibility modifiers:
- `private` - visible only within the current file
- `package` - visible only within the current package
- `public` - visible everywhere

Each symbol declaration **must** contain a visibility modifier. Name casing will not affect visibility.
To prevent naming collisions, type names should be declared in upper case, and variables or functions should be declared in lower case.

**Example**

```go
private type User struct {} // visible only within the current file

package func doSomething() {} // visible only within the current package

public var answer = 42 // visible everywhere

var anotherAnswer = 43 // compilation error: visibility modifier required
```

### Eliminate Built-in Functions

**Motivation**

Built-in functions contribute to namespace pollution and require additional cognitive load when making sure to avoid shadowing. Consider the following:
```go
func larger(a, b []string) []string {
  len := len(a)
  if len > len(b) { // compilation error: invalid operation: cannot call non-function len (variable of type int)
     return a
  }
  return b
}
```
To prevent shadowing of built-in functions we can simply convert them to keywords. However, a better solution would be to place them under a contextually oriented namespace when possible. This will both prevent the ability to override them and free those precious words to be used as safe variable names.

**Solution**

The following should be replaced:
- `append` - will be available as a slice method instead (i.e. `slice.append(elems ...T)`).
- `copy` - will be available as a slice method instead (i.e. `slice.copy(dst []T)`).
- `delete` - will be available as a map method instead (i.e. `m.delete(key string)`).
- `len` - will be available as methods of `array`, `slice`, `string` and `channel` (e.g. `slice.len()`).
- `cap` - will be available as methods of `array`, `slice`, `string` and `channel` (e.g. `slice.cap()`).
- `close` - will be available as a channel method instead (i.e. `channel.close()`).
- `make` - will be available using as a standard library function instead (`goat.make`).
- `complex` - will be available using as a standard library function instead (`goat.complex`).
- `real` - will be available using as a standard library function instead (`goat.real`).
- `imag` - will be available using as a standard library function instead (`goat.imag`).
- `print` - will be available using as a standard library function instead (`goat.print`).
- `println` - will be available using as a standard library function instead (`goat.println`).
- `panic` - will be a keyword to prevent shadowing.
- `recover` - will be a keyword to prevent shadowing.
- `new` - will be a keyword to prevent shadowing.
- `error` - will be a keyword to prevent shadowing.

**Example**

```go
import "goat"

private func main() {
  var slice = goat.make([]string)
  slice = slice.append("a")
  goat.println(slice.len())
  goat.println(slice.cap())
  var slice2 = goat.make([]string, 1)
  slice.copy(slice2)

  var m = map[string]string{"key": "value"}
  m.delete("key")
  
  var ch = goat.make(chan bool)
  ch.close()
  panic("panic is a keyword")
}
```

### Strict Nil Checks

discussed in [this issue](https://github.com/goatlang/goat/issues/2).

### Enum Support

**Motivation**

Lacking direct support for enums in Go has several drawbacks:
- No enforcement on valid values (example below)
- No enforcement on exhaustive switch cases
- No native support for iterating all possible values
- No native support for converting to and from strings
- No dedicated namespace for enum values (values are scattered between all other symbols in the package)

```go
type ConnectionStatus int

const (
  Idle ConnectionStatus = iota
  Connecting
  Ready
)

func main() {
  var status ConnectionStatus = 46 // no compilation error
}
```

**Solution**

Introduce an `enum` type to resolve the problems presented above.

**Example**

```go
public type ConnectionStatus enum {
  Idle
  Connecting
  Ready
}

private func example() {
  var status ConnectionStatus // zero value is ConnectionStatus.Idle
  fmt.Println(status == ConnectionStatus.Idle) // true
  status = ConnectionStatus.Connecting
  status.String() // "Connecting"
  ConnectionStatus.allValues() // []ConnectionStatus{ConnectionStatus.Idle, ConnectionStatus.Connecting, ConnectionStatus.Ready}
  ConnectionStatus.fromString("Idle") // ConnectionStatus.Idle
  status = 0 // compilation error: invalid enum value
  switch status { // compilation error: non-exhaustive enum switch statement
  case ConnectionStatus.Idle:
   return  
  }
}
```

### Struct Initial Values

In some cases, structs may be more than simply a collection of variables, but rather, a concise entity with state and
behavior. In such cases, it may be required for some fields to initially hold meaningful values and not simply their
zero values. We might need to initialize int values to -1 rather than 0, or to 18, or to a calculated value derived
from other values. Go does not provide any realistic approach to enforce initial state in structs.

**Solution**

Provide initial value capability to struct literals.

**Example**

discussed in [this issue](https://github.com/goatlang/goat/issues/1).

### Const Assignments

In most other use cases, assignment to a variable is a single-time operation. Const assignments allow for preventing
accidental shadowing and accidental rewriting of variables, and also allow code authors to convey intent.

discussed in [this issue](https://github.com/goatlang/goat/issues/3).

### Error Handling
- add `?` operator
  to be filled

### Promise
- `go` keyword should return a promise
  to be filled

## Goat Conventions

Goat conventions are similar to those in Go, with the exception of the proposals presented below.

### Type Names Should Begin With Uppercase

to be filled

### Lowercase Acronyms

to be filled

### Receiver Names
- should be `self` by default
  to be filled

### Error Types For Non Sentinel Errors

to be filled

## Contribution

**Discussion** of ideas and proposals should be done via **issues**. Issues can relate to an existing proposal or a discussion
of new proposals.

**Modifications** to the spec should be done by submitting **pull requests** modifying this document with the required
change. Each proposal should include:
- Motivation (why do we absolutely need it?)
- Solution (how will it work? will it preserve Go's simplicity?)
- Examples

<img alt="Goat Programming Language" src="https://raw.githubusercontent.com/goatlang/goat/main/goat.svg" height="300">

### Original work / attributions

The Go gopher eyes were designed by [Renee French](https://reneefrench.blogspot.com/).  
Her design is licensed under the Creative Commons 3.0 Attributions license.
