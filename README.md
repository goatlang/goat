# Goat

[![Issues](https://img.shields.io/github/issues/goatlang/goat?color=green&logo=github)](https://github.com/goatlang/goat/issues)
[![License](https://img.shields.io/github/license/goatlang/goat)](https://github.com/goatlang/goat/blob/main/LICENSE)
[![Commits](https://img.shields.io/github/last-commit/goatlang/goat?color=green&logo=github)](https://github.com/goatlang/goat/commits/main)

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

* [Goat Specification](#goat-specification)
    + [Visibility Modifiers](#visibility-modifiers)
    + [Eliminate Built-in Functions](#eliminate-built-in-functions)
    + [Strict Nil Checks](#strict-nil-checks)
    + [Enum Support](#enum-support)
    + [Struct Initial Values](#struct-initial-values)
    + [Const Assignments](#const-assignments)
    + [Error Handling](#error-handling)
    + [Promises](#promises)
* [Goat Conventions](#goat-conventions)
    + [Type Names Should Begin With Uppercase](#type-names-should-begin-with-uppercase)
    + [Lowercase Acronyms](#lowercase-acronyms)
    + [Receiver Names](#receiver-names)
    + [Error Types For Non Sentinel Errors](#error-types-for-non-sentinel-errors)
* [Contribution](#contribution)

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

**Motivation**

Following up on [this article](https://jesseduffield.com/Gos-Shortcomings-1/).

**Solution**

Error handling in Goat should support the `?` operator. It will not break Go's premise around error handling. Error handling will remain explicit and should not be avoided. Propagation of errors, if needed, may be shortened to `?` operator.

**Example**

```go
func concat1() (string, error) {
	data1, err := fetchURLData("example.com")
	if err != nil {
		return "", err
	}
	data2, err := fetchURLData("domain.com")
	if err != nil {
		return "", err
	}
	return data1 + data2, nil
}

func concat2() (string, error) {
	data1 := fetchURLData("example.com")?
	data2 := fetchURLData("domain.com")?
	return data1 + data2, nil
}

func concat3() string {
	data1 := fetchURLData("example.com")? // compilation error: function must return error
	data2 := fetchURLData("domain.com")? // compilation error: function must return error
	return data1 + data2, nil
}

func fetchURLData(url string) (string, error) {
	// ...
}
```

### Promises

**Motivation**

Following up on [this article](https://jesseduffield.com/Gos-Shortcomings-1/).

**Solution**

In Goat, the `go` keyword should return a promise.

**Example**

```go
func fetchResourcesFromURLs(urls []string) ([]string, error) {
  all := make([]promises.Promise[string], len(urls))
  for i, url := range urls {
     all[i] = go fetchResource(url)
  }
  return promises.All(all)
}

func fetchResource(url string) (string, error) {
  // some I/O operation...
}
```

## Goat Conventions

Goat conventions are similar to those in Go, with the exception of the proposals presented below.

### Type Names Should Begin With Uppercase

Go omits visibility modifier keywords (public, private, etc...) in favor of symbol naming. Symbols starting with an uppercase letter are automatically public and the rest are private. While this is great for simplicity, over time it's becoming clear that this method has a stronger downside than upside: In most other languages, type names, by convention, begin with an uppercase letter, and variable names begin with a lowercase one. This convention has a very powerful implication - it means that variables can never shadow types. Consider the following Go code:

```go
type user struct {
  name string
}

func main() {
  user := &user{name: "John"}
  anotherUser := &user{name: "Jane"} // compilation error: user is not a type
}
```

This is quite common in Go, whenever you store a private type into a private variable or a public type into a public variable - you run into this.
To prevent is we should introduce visibility modifiers, but also make sure that by contentions, types start with uppercase letters and variables with lowercase ones.

### Lowercase Acronyms

Acronyms in Go are [uppercase by convention](https://github.com/golang/go/wiki/CodeReviewComments#initialisms). This convention breaks readability and automatic tools when 2 or more acronyms are connected. For example - representing an HTTPS URL of some resource using the variable name `HTTPSURL` instead of `HttpsUrl`.

### Receiver Names

Following on [this article](https://jesseduffield.com/Gos-Shortcomings-5/#receiver-names), receiver names of a single letter are less meaningful and harder to maintain. There should be no problem with naming receiver variables `self`.

### Error Types For Non Sentinel Errors

Non sentinel errors should never use `errors.New`, `fmt.Errorf`, or similar. Rather, they should *always* define a dedicated error type.

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
