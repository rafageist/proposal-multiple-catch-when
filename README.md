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

## Syntax

The proposed syntax allows for `catch` blocks to be attached to any code block and for those blocks to conditionally execute based on the `when` clause:

```js
{
    // Code that may throw an error
    throw new Error("Error in block");

} catch (error) when (error.message.includes("block")) {
    console.log("Caught an error in block:", error.message);

    // Nested catch block
    throw new Error("Error within catch");

} catch (nestedError) {
    console.log("Caught a nested error:", nestedError.message);
}

```

## Example

Here's a practical example using the new syntax:

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

## Examples for  `if-catch` 

```js
if (condition) {
    throw new Error("Error in block");
} catch (error) when (error.message.includes("block")) {
    console.log("Caught an error in block:", error.message);
}
```

## Examples for  `for-catch` 

```js
for (let i = 0; i < 3; i++) {
    throw new Error("Error in block " + i);
} catch (error) when (error.message.includes("block 1")) {
    console.log("Caught an error in block 1:", error.message);
}
```

## Examples for  `while-catch` 

```js

let i = 0;
while (i < 3) {
    throw new Error("Error in block " + i);
} catch (error) when (error.message.includes("block 1")) {
    console.log("Caught an error in block 1:", error.message);
    i++;
}
```

## Examples for  `function-catch` 

```js
function fetchData() {
    throw new Error("Error in block");
} catch (error) when (error.message.includes("block")) {
    console.log("Caught an error in block:", error.message);
}
```

## Examples for  `catch-catch` 

```js
/* ... any block of code ... */ {
    throw new Error("Error in block");
} catch (error) when (error.message.includes("block")) {
    console.log("Caught an error in block:", error.message);
} catch (nestedError) {
    console.log("Caught a nested error:", nestedError.message);
}
```

## Examples for  `class-catch` 

```js
class MyClass {
    constructor() {
        throw new Error("Error in block");
    } 
} catch (error) when (error.message.includes("block")) {
    console.log("Caught an error in block:", error.message);
}
```

## Examples for  `switch-catch` 

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

## Examples for  `do-catch` 

```js
do {
    throw new Error("Error in block");
} catch (error) when (error.message.includes("block")) {
    console.log("Caught an error in block:", error.message);
} while (false);
```

## Examples for  `finally-catch` 

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

## Comparison with Existing Syntax

### Current Syntax

```js
try {
    // Code that may throw an error
    throw new Error("Error in block");
} catch (error) {
    if (error.message.includes("block")) {
        console.log("Caught an error in block:", error.message);
    } else {
        throw error;
    }
}
```

### Proposed Syntax

```js
{
    // Code that may throw an error
    throw new Error("Error in block");
} catch (error) when (error.message.includes("block")) {
    console.log("Caught an error in block:", error.message);
}
```

## Benefits

- **Precision**: Handle specific error types or conditions directly.
- **Clarity**: The `when` clause makes the error-handling logic clear and concise.
- **Simplicity**: Reduces the need for complex nested conditions within `catch` blocks.
- **Expressiveness**: Offers a more powerful way to handle different error scenarios.

## Conclusion

This proposal expands JavaScript’s error handling capabilities by allowing any block of code to have its own `catch` block, with the option to conditionally execute those blocks using the `when` keyword. This enhancement provides developers with more control, clarity, and flexibility while maintaining compatibility with existing JavaScript syntax.
