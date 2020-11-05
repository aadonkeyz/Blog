---
title: Executable Code and Execution Contexts
abbrlink: 33ac290
date: 2020-11-04 23:05:21
categories:
  - JavaScript
---

# Types of Executable Code

There are three types of ECMAScript executable code:
- Global code is source text that is treated as an ECMAScript Program. The global code of a particular Program does not include any source text that is parsed as part of a FunctionBody.
- Eval code is the source text supplied to the built-in `eval` function. More precisely, if the parameter to the built-in eval function is a String, it is treated as an ECMAScript Program. The eval code for a particular invocation of `eval` is the global code portion of that Program.
- Function code is source text that is parsed as part of a FunctionBody. The function code of a particular FunctionBody does not include any source text that is parsed as part of a nested FunctionBody. Function code also denotes the source text supplied when using the built-in Function object as a constructor. More precisely, the last parameter provided to the Function constructor is converted to a String and treated as the FunctionBody. If more than one parameter is provided to the Function constructor, all parameters except the last one are converted to Strings and concatenated together, separated by commas. The resulting String is interpreted as the FormalParameterList for the FunctionBody defined by the last parameter. The function code for a particular instantiation of a Function does not include any source text that is parsed as part of a nested FunctionBody.

# Lexical Environment

A Lexical Environment is a specification type used to define the association of Identifiers to specific variables and functions based upon the lexical nesting structure of ECMAScript code. A Lexical Environment consists of an Environment Record and a possibly null reference to an outer Lexical Environment. Usually a Lexical Environment is associated with some specific syntactic structure of ECMAScript code such as a FunctionDeclaration, a WithStatement, or a Catch clause of a TryStatement and a new Lexical Environment is created each time such code is evaluated.

An Environment Record records the identifier bindings that are created within the scope of its associated Lexical Environment.

The outer environment reference is used to model the logical nesting of Lexical Environment values. The outer reference of a (inner) Lexical Environment is a reference to the Lexical Environment that logically surrounds the inner Lexical Environment. An outer Lexical Environment may, of course, have its own outer Lexical Environment. A Lexical Environment may serve as the outer environment for multiple inner Lexical Environments. For example, if a FunctionDeclaration contains two nested FunctionDeclarations then the Lexical Environments of each of the nested functions will have as their outer Lexical Environment the Lexical Environment of the current execution of the surrounding function.

Lexical Environments and Environment Record values are purely specification mechanisms and need not correspond to any specific artefact of an ECMAScript implementation. It is impossible for an ECMAScript program to directly access or manipulate such values.

# Execution Contexts

When control is transferred to ECMAScript executable code, control is entering an execution context. Active execution contexts logically form a stack. The top execution context on this logical stack is the running execution context. A new execution context is created whenever control is transferred from the executable code associated with the currently running execution context to executable code that is not associated with that execution context. The newly created execution context is pushed onto the stack and becomes the running execution context.

An execution context contains whatever state is necessary to track the execution progress of its associated code. In addition, each execution context has three state components:
- LexicalEnvironment: Identifies the Lexical Environment used to resolve identifier references made by code within this execution context.
- VariableEnvironment: Identifies the Lexical Environment whose environment record holds bindings created by VariableStatements and FunctionDeclarations within this execution context.
- ThisBinding: The value associated with the `this` keyword within ECMAScript code associated with this execution context.

The LexicalEnvironment and VariableEnvironment components of an execution context are always Lexical Environments. When an execution context is created, its LexicalEnvironment and VariableEnvironment components initially have the same value. The value of the VariableEnvironment component never changes while the value of the LexicalEnvironment component may change during execution of code within an execution context.

An execution context is purely a specification mechanism and need not correspond to any particular artefact of an ECMAScript implementation. It is impossible for an ECMAScript program to access an execution context.

# 参考文献

- [Annotated ES5](https://es5.github.io/#x10)
