# ECMAScript Universal and conditional "catch" blocks with the new "when" keyword

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

## Why Braces `{}` Matter

While some proposals seek to move away from the traditional `try-catch` structure, often resorting to `if` statements or introducing new operators, this proposal embraces and expands upon the existing use of braces `{}` to maintain consistency with the current JavaScript syntax and control flow structures.

### Importance of Braces

Braces `{}` are a fundamental part of JavaScript's syntax, serving as the primary means to define code blocks. By leveraging this familiar structure, the proposal ensures that developers can manage errors within the same framework they use for other control flows like `if`, `for`, and `while` loops.

### Control Flow Integrity

At its core, error handling is about controlling the flow of execution in the presence of unexpected conditions. By expanding the capabilities of blocks with optional `catch` and `finally` clauses, this proposal provides a powerful yet intuitive way to manage errors without introducing new or unfamiliar syntax. The focus remains on enhancing existing structures, ensuring that the language remains coherent and that the learning curve for developers is minimal.

### Avoiding Redundant Constructs

Moving away from `try-catch` often results in the use of `if` statements or other control structures that, while functional, can lead to redundant or less expressive code. This proposal addresses error handling in a more integrated manner, allowing developers to manage exceptions within the same block structure that controls their program's logic.

### A Structured Solution for a Structured Problem

Error handling is inherently about structuring your code to handle the unexpected. This proposal keeps the focus on structure by using control flow blocks, rather than introducing operators or new constructs that might disrupt the logical flow of code. By sticking with braces, we ensure that error handling remains a natural extension of the language's existing syntax and philosophy.

This proposal advocates for an evolution of JavaScript's error-handling capabilities that respects and enhances the language's foundational structures, ensuring that developers can write cleaner, more maintainable code without sacrificing familiarity or simplicity.

## Conclusion

This proposal expands JavaScript’s error handling capabilities by allowing any block of code to have its own `catch` block, with the option to conditionally execute those blocks using the `when` keyword. This enhancement provides developers with more control, clarity, and flexibility while maintaining compatibility with existing JavaScript syntax.

## Inspiration

This proposal was inspired by the need to improve JavaScript's error-handling mechanisms, drawing from various languages and systems that have implemented similar concepts:

1. **C# and F# `when` Clause**: Both C# and F# provide a `when` clause in their `catch` blocks, allowing developers to handle exceptions based on specific conditions. This inspired the idea of bringing a similar conditional mechanism to JavaScript.

2. **Scala's Pattern Matching in `catch`**: Scala's ability to use pattern matching within `catch` blocks, combined with conditional logic, influenced the design of conditional `catch` blocks with `when` in this proposal.

3. **PL/pgSQL's `EXCEPTION` Handling**: In PL/pgSQL, the EXCEPTION section within functions allows for specific error handling based on the type of exception, providing a structured approach to managing errors in procedural code. This inspired the idea of enhancing JavaScript's error handling within blocks and functions.

4. **BASIC's `On Error` Statement**: The `On Error` mechanism in BASIC languages like Visual Basic offers a way to direct the flow of control when an error occurs, similar to the concept of catch blocks. This inspired the proposal to allow more flexible and conditional error handling in JavaScript.

5. **Ruby's `rescue` with Conditions**: Ruby's elegant error-handling using `rescue`, which can include conditional logic within the block, inspired the flexibility and readability goals of this proposal.

6. **Real-World Scenarios**: The need for more granular control over error handling in complex JavaScript applications highlighted the limitations of the current `try-catch` structure and motivated the development of this more flexible approach.

By synthesizing these ideas and experiences from various languages and systems, this proposal aims to provide a more powerful and flexible approach to error handling in JavaScript, while maintaining the simplicity and dynamism that the language is known for.

## References

1. Crockford, Douglas. *JavaScript: The Good Parts*. O'Reilly Media, 2008. ISBN: 978-0596517748.

2. Simpson, Kyle. *You Don't Know JS: Scope & Closures*. O'Reilly Media, 2014. ISBN: 978-1449335588.

3. Hunt, Andrew, and David Thomas. *The Pragmatic Programmer: Your Journey to Mastery*. Addison-Wesley Professional, 1999. ISBN: 978-0201616224.

4. Scott, Michael L. *Programming Language Pragmatics*. Morgan Kaufmann, 2009. ISBN: 978-0123745149.

5. McConnell, Steve. *Code Complete: A Practical Handbook of Software Construction*. Microsoft Press, 2004. ISBN: 978-0735619678.

## Author

Rafael Rodríguez Ramírez

rafageist@divengine.com

[rafageist.com](https://rafageist.com)

## License

This proposal is licensed under the [MIT License](./LICENSE).
