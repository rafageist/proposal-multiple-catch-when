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

## Benefits

- **Precision**: Handle specific error types or conditions directly.
- **Clarity**: The `when` clause makes the error-handling logic clear and concise.
- **Simplicity**: Reduces the need for complex nested conditions within `catch` blocks.
- **Expressiveness**: Offers a more powerful way to handle different error scenarios.

## Conclusion

This proposal expands JavaScript’s error handling capabilities by allowing any block of code to have its own `catch` block, with the option to conditionally execute those blocks using the `when` keyword. This enhancement provides developers with more control, clarity, and flexibility while maintaining compatibility with existing JavaScript syntax.
