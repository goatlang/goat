# Goat

Following the ideas presented in the Goat article series (link to be added soon),
We've created this repository to start forging the specification of the Goat programming language.

The Goat syntax aspires to be as close as possible to Go's, with minor variations to provide better
maintainability, safer runtime, and a better compile time experience.

Goat specification is based on the premise of keeping things simple. We must comply with the very
basic idea of Go: There shouldn't be more than one way of doing something. We do not wish to create
new types of syntactical fragmentations in Go syntax. With this, each compile time or convention
variation presented below should include the following:
- Motivation (why do we absolutely need it?)
- Solution (how will it work? will it preserve Go's simplicity?)
- Examples

## Compile Time Variations

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
In addition, the current visibility system of Go supports only two visibility modifiers - public and private. This prevents a more fine-grained control over visibility and produces another namespace polution problem - all symbols in a package are automatically visible from all other files. This creates highly cluttered package namespaces, especially in big packages and big projects.

**Solution**

Goat should support three visibility modifiers:
- `private` - visibile only within the current file
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

Built-in functions contribute to namespace pollution and require additional cognative load when making sure to avoid shadowing. Consider the following:
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

func main() {
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

to be filled

### Enum Support

to be filled

### Struct Initial Values

to be filled

### Const Assignments
- var
- val
- remove shorthand assignments
- add ternary expression
to be filled

### Error Handling
- add `?` operator
to be filled

### Promise
- `go` keyword should return a promise
to be filled

### Short Lambda Expression

to be filled

### Interface Field Support

to be filled

## Convention Variations

### Type Names Should Begin With Uppercase

to be filled

### Lowercase Acronyms

to be filled

### Receiver Names
- should be `self` by default
to be filled

### Error Types For Non Sentinel Errors

to be filled

