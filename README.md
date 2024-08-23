# ECMAScript Multiple and conditional "catch" blocks with new "when" keyword

## Introduction

JavaScript’s `try-catch` structure is a powerful tool for error handling, but it can be improved to offer more flexibility and clarity. Unlike statically-typed languages, JavaScript does not allow developers to specify the type of the error caught in a `catch` block. This limitation can lead to unnecessary complexity and less precise error handling. This proposal introduces the ability to have multiple `catch` blocks, each with a `when` clause, allowing developers to apply specific conditions to each `catch` block. This enhancement provides a more controlled and expressive approach to managing different types of exceptions.

## Motivation

### Learning from C#

In languages like C#, the `when` keyword is used within `catch` blocks to add conditions that must be true for the block to execute. This allows for more precise error handling without the need for nested `if` statements, making code cleaner and more maintainable. By introducing a similar `when` clause to JavaScript, this proposal aims to bring the same level of control and clarity to JavaScript’s error handling.

### Addressing JavaScript’s Limitations

JavaScript currently lacks the ability to type the error variable in a `catch` block, often leading to less precise error handling. Developers must write additional conditional logic within `catch` blocks to check the type of the error or other conditions, which can make the code harder to read and maintain. The `when` clause offers a solution by allowing developers to filter errors directly within the `catch` statement, reducing the need for nested conditions and making the intention of the error handling code clearer.

### Why `when`?

The `when` clause allows developers to conditionally handle exceptions based on custom logic, similar to type-specific `catch` blocks in statically-typed languages. This makes the error-handling code more readable, maintainable, and specific.

### Compatibility

This proposal is designed to be both backward and forward compatible:

- **Backward Compatibility**: The `when` clause is an optional addition and does not affect existing JavaScript code. Without the `when` clause, the `try-catch` structure functions as it currently does.
    
- **Forward Compatibility**: If JavaScript or TypeScript introduces typed `catch` variables in the future, the `when` clause will remain relevant and useful, providing an additional layer of conditional logic that works alongside any future typing features.
    

## Syntax

The proposed syntax introduces the `when` clause within `catch` blocks, enabling developers to specify conditions directly:

```
try {
  // Code that may throw an exception 
}  
catch (err) when (err instanceof TypeError) {   
  // Handle TypeError 
}  
catch (err) when (err instanceof ReferenceError) {
  // Handle ReferenceError 
}  
finally {   
  // Code that always runs 
}
```

## Example

Here’s a practical example:

```
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

- **Precision**: Handle specific error types or conditions more effectively.
- **Clarity**: The `when` clause makes it clear which errors are being handled and under what conditions.
- **Simplicity**: Reduces the need for complex nested conditions within `catch` blocks.
- **Expressiveness**: Provides a more powerful way to handle different error scenarios in a single `try-catch` statement.

## Considerations

- **Performance**: Optimizations may be needed to minimize any potential performance impact from introducing multiple `catch` blocks with conditions.
- **Tooling**: Updates to linters, transpilers, and other tools may be necessary to support the new syntax.

## Conclusion

Enhancing JavaScript’s error handling with multiple `catch` blocks and a `when` clause offers a natural and powerful way to manage exceptions. This proposal builds on the existing `try-catch` structure, providing developers with more control and flexibility while maintaining code clarity.

## Further Discussion

We invite feedback and suggestions from the community to refine and improve this proposal.
