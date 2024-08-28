# ECMAScript Universal and Conditional "catch" blocks with the new "when" Keyword

## Introduction

JavaScript's `try-catch` structure is a fundamental tool for error handling, but it can be enhanced for greater flexibility and clarity. This proposal introduces the concept of allowing any block of code, not just `try` blocks, to have associated `catch` blocks. Furthermore, each `catch` block can have a `when` clause to conditionally handle specific errors, providing a more controlled and expressive approach to managing exceptions.

## Key Concepts

1. **Universal `catch` Blocks**: Any block of code, including functions, loops, and even other `catch` blocks, can have its own `catch` statement.
2. **Nested `catch` Blocks**: Since a `catch` block is just another block of code, it can also have its own `catch` to handle errors within error-handling logic.
3. **Conditional `catch` with `when`**: The `when` clause allows for conditional execution of `catch` blocks, improving the readability and control of error handling.

## Motivation

### Improving JavaScript’s Error Handling

Currently, JavaScript lacks the ability to type or conditionally handle errors directly in `catch` blocks, leading to complex and less readable code. This proposal introduces a way to manage errors more precisely and clearly, inspired by similar features in languages like C#.

## Proposed Syntax

The proposed syntax allows for `catch` blocks to be attached to any code block and for those blocks to conditionally execute based on the `when` clause:

```txt
/* any block of code */ 
{
    // Code that may throw an error
    // ...
} 
catch (<var>) when (<boolean expression) 
{
    // Error-handling logic
    // ...
    // catch is also a block!
}
catch (<var>) when (<boolean expression) 
{
    // Error-handling logic
    // ...
}
finally 
{
    // Cleanup code, executed regardless of success or failure

    // ... but finally is also a block!
} 
catch (<var>) when (<boolean expression) 
{
    // Error-handling logic
    // ...
}
...
```

## Grammar

```grammar
14.2 Block 

Syntax 

    Block[Yield, Await, Return] :
        { StatementList[?Yield, ?Await, ?Return] } CatchClause[?Yield, ?Await, ?Return]? FinallyClause[?Yield, ?Await, ?Return]?

    CatchClause[Yield, Await, Return] :
        catch ( CatchParameter[?Yield, ?Await] ) when ( Expression ) Block[?Yield, ?Await, ?Return]
        catch ( CatchParameter[?Yield, ?Await] ) Block[?Yield, ?Await, ?Return]
        catch when ( Expression ) Block[?Yield, ?Await, ?Return]
        catch Block[?Yield, ?Await, ?Return]

    FinallyClause[Yield, Await, Return] :
        finally Block[?Yield, ?Await, ?Return]

14.15 The try Statement

Syntax

TryStatement[Yield, Await, Return] :
    Block[?Yield, ?Await, ?Return]
    
```

## Comparison with Existing Syntax

### Current

```js
try 
{
    // Code that may throw an error
    throw new Error("Error in block");
} 
catch (error) 
{
    if (error.message.includes("block")) {
        console.log("Caught an error in block:", error.message);
    } else {
        throw error;
    }
}
```

### Proposed

```js
/* any block of code */ 
{
    // Code that may throw an error
    throw new Error("Error in block");
} 
catch (error) /* optional */ when (error.message.includes("block")) 
{
    console.log("Caught an error in block:", error.message);
}
```

## Examples

### `try-catch` (the current specification)

This proposal is compatible with the existing `try-catch` syntax.

```js
try 
{
    // Code that may throw an error
    throw new Error("Error in block");
} 
catch (error) 
{
    console.log("Caught an error in block:", error.message);
}
```

### `try-catch` with `when` clause

Users can conditionally handle errors based on the `when` clause.

```js
try {
    data = fetchData();
} 
catch (err) when (err instanceof NetworkError) {
    console.error('Network error:', err.message);
} 
catch (err) when (err instanceof SyntaxError) {
    console.error('Syntax error in response:', err.message);
} 
finally {
    console.log('Cleanup code, executed regardless of success or failure.');
}
```

### `anonymous-catch` (no `try` block)

```js
{
    // Code that may throw an error
    throw new Error("Error in block");
} 
catch (error) when (error.message.includes("block")) 
{
    console.log("Caught an error in block:", error.message);
}
```

### `if-catch`

```js
if (condition) {
    throw new Error("Error in block");
} catch (error) {
    console.log("Caught an error in block:", error.message);
}
```

### `if-catch` with `when` Clause

```js
if (condition) {
    throw new Error("Error in block");
} catch (error) when (error.message.includes("block")) {
    console.log("Caught an error in block:", error.message);
}
```

### `for-catch`

```js
for (let i = 0; i < 3; i++) {
    throw new Error("Error in block " + i);
} catch (error) when (error.message.includes("block 1")) {
    console.log("Caught an error in block 1:", error.message);
}
```

### `while-catch`

```js

let i = 0;
while (i < 3) {
    throw new Error("Error in block " + i);
} catch (error) when (error.message.includes("block 1")) {
    console.log("Caught an error in block 1:", error.message);
    i++;
}
```

### `function-catch`

```js
function fetchData() {
    throw new Error("Error in block");
} catch (error) when (error.message.includes("block")) {
    console.log("Caught an error in block:", error.message);
}
```

### `catch-catch`

```js
/* ... any block of code ... */ {
    throw new Error("Error in block");
} catch (error) when (error.message.includes("block")) {
    console.log("Caught an error in block:", error.message);
} catch (nestedError) {
    console.log("Caught a nested error:", nestedError.message);
}
```

## `class-catch`

This example demonstrates how a `catch` block can be attached to a class definition to handle errors during the execution of any class method.

```js
class MyClass {
    constructor() {
        throw new Error("Error in block");
    } 

    method() {
        throw new Error("Error in block");
    } catch (error) when (error.message.includes("block")) {
        console.log("Caught an error in block:", error.message);
    }

    static staticMethod() {
        throw new Error("Error in block");
    }
} catch (error) when (error.message.includes("block")) {
    console.log("Caught an error in block:", error.message);
}
```

## `switch-catch`

```js
switch (value) {
    case 1:
        throw new Error("Error in block 1");
    case 2:
        throw new Error("Error in block 2");
} catch (error) when (error.message.includes("block 1")) {
    console.log("Caught an error in block 1:", error.message);
}
```

## `do-catch`

```js
do {
    throw new Error("Error in block");
} catch (error) when (error.message.includes("block")) {
    console.log("Caught an error in block:", error.message);
} while (false);
```

## `finally-catch`

```js
try {
    throw new Error("Error in block");
} catch (error) when (error.message.includes("block")) {
    console.log("Caught an error in block:", error.message);
} finally {
    console.log("Finally block executed.");
    throw new Error("Error in finally block");
} catch (nestedError) {
    console.log("Caught a nested error:", nestedError.message);
}
```

## `try-catch-throw-catch`

```js
try {
    throw new Error("Error in block");
} catch (error) when (error.message.includes("block")) {
    console.log("Caught an error in block:", error.message);
    throw new Error("Error in catch block");
} catch (nestedError) {
    console.log("Caught a nested error:", nestedError.message);
}
```

## Benefits

- **Precision**: Handle specific error types or conditions directly.
- **Clarity**: The `when` clause makes the error-handling logic clear and concise.
- **Simplicity**: Reduces the need for complex nested conditions within `catch` blocks.
- **Expressiveness**: Offers a more powerful way to handle different error scenarios.

## Conclusion

This proposal expands JavaScript’s error handling capabilities by allowing any block of code to have its own `catch` block, with the option to conditionally execute those blocks using the `when` keyword. This enhancement provides developers with more control, clarity, and flexibility while maintaining compatibility with existing JavaScript syntax.
