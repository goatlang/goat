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

### Eliminate Builtins

to be filled

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

