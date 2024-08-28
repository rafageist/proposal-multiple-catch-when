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

### Static Semantics: Early Errors

```plaintext
Block: { StatementList } CatchClause[?Yield, ?Await, ?Return]? FinallyClause[?Yield, ?Await, ?Return]?
```
- It is a Syntax Error if `BoundNames` of `CatchParameter` contains any duplicate elements.
- It is a Syntax Error if any element of the `BoundNames` of `CatchParameter` also occurs in the `LexicallyDeclaredNames` of `Block`.
- It is a Syntax Error if any element of the `BoundNames` of `CatchParameter` also occurs in the `VarDeclaredNames` of `Block`.
- It is a Syntax Error if the `Expression` in the `when` clause is not a valid Boolean expression.

### Runtime Semantics: CatchClauseEvaluation

```plaintext
CatchClause: catch ( CatchParameter ) when ( Expression ) Block
```

1. Let `oldEnv` be the running execution context's `LexicalEnvironment`.
2. Let `catchEnv` be `NewDeclarativeEnvironment(oldEnv)`.
3. For each element `argName` of the `BoundNames` of `CatchParameter`, do:

    a. Perform `! catchEnv.CreateMutableBinding(argName, false)`.
4. Set the running execution context's `LexicalEnvironment` to `catchEnv`.
5. Let `status` be `Completion(BindingInitialization)` of `CatchParameter` with arguments `thrownValue` and `catchEnv`.
6. If `status` is an abrupt completion, then:

    a. Set the running execution context's `LexicalEnvironment` to `oldEnv`. 

    b. Return `? status`.

7. If the result of evaluating `Expression` is `false`, return `undefined`.
8. Let `B` be `Completion(Evaluation)` of `Block`.
9. Set the running execution context's `LexicalEnvironment` to `oldEnv`.
10. Return `? B`.

### Runtime Semantics: Block Evaluation

```plaintext
Block : { StatementList } CatchClause[?Yield, ?Await, ?Return]? FinallyClause[?Yield, ?Await, ?Return]?
```

1. Let `B` be `Completion(Evaluation)` of `StatementList`.
2. If `B` is a `throw` completion, then: 

   a. If a `CatchClause` is present, evaluate `CatchClauseEvaluation` with `B.[[Value]]`.

   b. If `CatchClauseEvaluation` returns `undefined`, proceed to step 3.

   c. Otherwise, let `C` be the result of `CatchClauseEvaluation`.

3. If no `CatchClause` is present or all `CatchClause` evaluations result in `undefined`, rethrow the exception.

4. If a `FinallyClause` is present, evaluate it and:

   a. If the `FinallyClause` evaluation results in an abrupt completion, return that result.

   b. Otherwise, proceed with the value from the previous step.

5. Return `? C` or `B`, as appropriate.

## Comparison with Existing Syntax

### Current

The current `try-catch` syntax is limited to the `try` block, which can be followed by one or more `catch` blocks and an optional `finally` block. This structure is restrictive and does not allow for `catch` blocks to be attached to other blocks of code.

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

The proposed syntax allows for `catch` blocks to be attached to any block of code, not just `try` blocks. This flexibility enables developers to handle errors more precisely and conditionally, improving the readability and control of error handling.

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

No `try` block is required to have a `catch` block.

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

No `try` block is required to have a `catch` block inside an `if` statement.

```js
if (condition) {
    // ...
} catch (error) {
    console.log("Caught an error in block:", error.message);
} else {
    // here we are sure that there is no error in the block
    console.log("No error in block.");
}
```

### `if-catch` with `when` Clause

Like the previous example, but with a `when` clause to conditionally handle errors.

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

Functions can have their own `catch` blocks.

```js
function fetchData() {
    throw new Error("Error in block");
} catch (error) when (error.message.includes("block")) {
    console.log("Caught an error in block:", error.message);
}
```

### `catch-catch`

Nested `catch` blocks can handle errors within error-handling logic.

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

Catch exceptions thrown in any `case` of a `switch` statement.

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

Catch exceptions must be before the `while` statement.

```js
do {
    throw new Error("Error in block");
} catch (error) when (error.message.includes("block")) {
    console.log("Caught an error in block:", error.message);
} while (false);
```

## `finally-catch`

Catch exceptions thrown in the `finally` block.

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

Catch exceptions thrown in the `catch` block.

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

## `try-cath` with `if-catch-when` inside

Combine `try-catch` with `if-catch-when` inside.

```js
try {
    
    // ...
    if (condition) {
        throw new Error("Error in block");
    } catch (error) when (error.message.includes("block")) {
        
        // enter here if error.message.includes("block") is true
        // else the exception will be catched by the outer catch block
        console.log("Caught an error in if block:", error.message);
    }
    // ...

} catch (error) {
    console.log("Caught an error in try block:", error.message);
}
```

## Alignment with Current Exception Handling

In JavaScript, when an exception is thrown, it is captured by the first catch block encountered in the hierarchy. This behavior remains consistent with the proposed changes. If a block does not catch the exception—either because it lacks a catch block or because the when condition evaluates to false—the exception will propagate up the call stack, where it can be caught by a higher-level catch block.

This ensures that the traditional flow of exception handling is preserved. The flexibility introduced by allowing any block to have a catch (and potentially a when condition) simply extends this existing mechanism, giving developers more control over how and where exceptions are handled, without altering the fundamental principles of exception propagation.

## Benefits

- **Precision**: Handle specific error types or conditions directly.
- **Clarity**: The `when` clause makes the error-handling logic clear and concise.
- **Simplicity**: Reduces the need for complex nested conditions within `catch` blocks.
- **Expressiveness**: Offers a more powerful way to handle different error scenarios.

## Conclusion

This proposal expands JavaScript’s error handling capabilities by allowing any block of code to have its own `catch` block, with the option to conditionally execute those blocks using the `when` keyword. This enhancement provides developers with more control, clarity, and flexibility while maintaining compatibility with existing JavaScript syntax.
