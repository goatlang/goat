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
- Explanation (how does it work?)
- No Fragmentation Clause (how does it not make Goat more complex than Go?)
- Examples

## Compile Time Variations

### Visibility Modifiers
- including package private
to be filled

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

