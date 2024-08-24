# ECMAScript Multiple and conditional "catch" blocks with new "when" keyword

## Introduction

JavaScript’s `try-catch` structure is a powerful tool for error handling, but it can be improved to offer more flexibility and clarity. Unlike statically-typed languages, JavaScript does not allow developers to specify the type of the error caught in a `catch` block. This limitation can lead to unnecessary complexity and less precise error handling. This proposal introduces the ability to have multiple `catch` blocks, each with a `when` clause, allowing developers to apply specific conditions to each `catch` block. This enhancement provides a more controlled and expressive approach to managing different types of exceptions.

## Motivation

### Learning from CSharp

In languages like C#, the `when` keyword is used within `catch` blocks to add conditions that must be true for the block to execute (https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/when). This allows for more precise error handling without the need for nested `if` statements, making code cleaner and more maintainable. By introducing a similar `when` clause to JavaScript, this proposal aims to bring the same level of control and clarity to JavaScript’s error handling.

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

```plaintext
try {
  // Code that may throw an exception 
}  
catch (<error variable 1>) when (<boolean expression 1>) {   
  // Handle error
}  
catch (<error variable 2>) when (<boolean expression 2>) {
  // Handle error
}  
finally {   
  // Code that always runs 
}
```

## Example

Here’s a practical example:

```plaintext
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

## Current Workarounds in JavaScript

### Using `if-else` Statements

One common approach to handling multiple error types within a single `catch` block is to use `if-else` statements to check the type or other properties of the caught error. Although this method is straightforward, it can lead to less readable and more cluttered code when handling many different error types.

```js
try {
    // Code that may throw an exception
} catch (err) {
    if (err instanceof TypeError) {
        console.error('TypeError occurred:', err.message);
    } else if (err instanceof ReferenceError) {
        console.error('ReferenceError occurred:', err.message);
    } else if (err.message.includes('custom message')) {
        console.error('Custom error detected:', err.message);
    } else {
        console.error('Unhandled error:', err.message);
    }
} finally {
    console.log('Cleanup code, executed regardless of success or failure.');
}

```

### Using `switch(true)`

As an alternative to multiple `if-else` statements, a `switch(true)` structure can be used to organize the error handling code. This approach can make the code more readable and maintainable by reducing the repetitive nature of `if` conditions and by clearly segregating the handling logic for different error conditions.

```js
try {
    // Code that may throw an exception
} catch (err) {
    switch (true) {
        case err instanceof TypeError:
            console.error('TypeError occurred:', err.message);
            break;
        case err instanceof ReferenceError:
            console.error('ReferenceError occurred:', err.message);
            break;
        case err.message.includes('custom message'):
            console.error('Custom error detected:', err.message);
            break;
        default:
            console.error('Unhandled error:', err.message);
    }
} finally {
    console.log('Cleanup code, executed regardless of success or failure.');
}

```


## Adding `when` Keyword for Conditional Catch Blocks in ECMAScript

This proposal suggests modifications to the ECMAScript specification, specifically to section 14.15 (https://tc39.es/ecma262/#sec-try-statement), to introduce the `when` keyword for conditional `catch` blocks. This would allow more granular and conditional error handling in JavaScript, improving the language's robustness and clarity.

This modification introduces the `when` keyword for conditional `catch` blocks in JavaScript, allowing developers to handle specific exceptions based on conditions directly within the `catch` block. This enhancement brings JavaScript closer to the error-handling flexibility found in languages like C# while maintaining backward and forward compatibility with existing and future language features.

```plaintext
14.15 The try Statement (Modified)

Syntax (Proposed Change)

TryStatement[Yield, Await, Return] :
    try Block[?Yield, ?Await, ?Return] CatchSequence[?Yield, ?Await, ?Return]
    try Block[?Yield, ?Await, ?Return] Finally[?Yield, ?Await, ?Return]
    try Block[?Yield, ?Await, ?Return] CatchSequence[?Yield, ?Await, ?Return] Finally[?Yield, ?Await, ?Return]

CatchSequence[Yield, Await, Return] :
    Catch[?Yield, ?Await, ?Return]
    CatchSequence[?Yield, ?Await, ?Return] Catch[?Yield, ?Await, ?Return]

Catch[Yield, Await, Return] :
    catch ( CatchParameter[?Yield, ?Await] ) when ( Expression ) Block[?Yield, ?Await, ?Return]
    catch when ( Expression ) Block[?Yield, ?Await, ?Return]
    catch ( CatchParameter[?Yield, ?Await] ) Block[?Yield, ?Await, ?Return]
    catch Block[?Yield, ?Await, ?Return]

Finally[Yield, Await, Return] :
    finally Block[?Yield, ?Await, ?Return]

CatchParameter[Yield, Await] :
    BindingIdentifier[?Yield, ?Await]
    BindingPattern[?Yield, ?Await]

14.15.1 Static Semantics: Early Errors (Proposed Change)

(...)

Catch : catch ( CatchParameter ) when ( Expression ) Block

- It is a Syntax Error if BoundNames of CatchParameter contains any duplicate elements.
- It is a Syntax Error if any element of the BoundNames of CatchParameter also occurs in the LexicallyDeclaredNames of Block.
- It is a Syntax Error if any element of the BoundNames of CatchParameter also occurs in the VarDeclaredNames of Block.
- It is a Syntax Error if the Expression is not a valid Boolean expression.

14.15.2 Runtime Semantics: CatchClauseEvaluation (Proposed Change)

(...)

Catch : catch ( CatchParameter ) when ( Expression ) Block

1. Let oldEnv be the running execution context's LexicalEnvironment.
2. Let catchEnv be NewDeclarativeEnvironment(oldEnv).
3. For each element argName of the BoundNames of CatchParameter, do
   a. Perform ! catchEnv.CreateMutableBinding(argName, false).
4. Set the running execution context's LexicalEnvironment to catchEnv.
5. Let status be Completion(BindingInitialization of CatchParameter with arguments thrownValue and catchEnv).
6. If status is an abrupt completion, then
   a. Set the running execution context's LexicalEnvironment to oldEnv.
   b. Return ? status.
7. If the result of evaluating Expression is false, return undefined.
8. Let B be Completion(Evaluation of Block).
9. Set the running execution context's LexicalEnvironment to oldEnv.
10. Return ? B.

(...)

14.15.3 Runtime Semantics: Evaluation (Proposed Change)

(...)

TryStatement : try Block CatchSequence Finally

1. Let B be Completion(Evaluation of Block).
2. Let S be the List of CaseClause items in CatchSequence, in source text order.
2. If B is a throw completion, For each CatchClause A of S, do:
   a. Let C be Completion(CatchClauseEvaluation of CatchClause with argument B.[[Value]]).
   b. If C is not undefined, exit the loop and proceed to the next step.
3. If all Catch of CatchSequence result in undefined, rethrow the exception.
4. Otherwise, let C be B.
5. Let F be Completion(Evaluation of Finally).
6. If F is a normal completion, set F to C.
7. Return ? UpdateEmpty(F, undefined).
```

## Conclusion

Enhancing JavaScript’s error handling with multiple `catch` blocks and a `when` clause offers a natural and powerful way to manage exceptions. This proposal builds on the existing `try-catch` structure, providing developers with more control and flexibility while maintaining code clarity.

## Further Discussion

We invite feedback and suggestions from the community to refine and improve this proposal.
