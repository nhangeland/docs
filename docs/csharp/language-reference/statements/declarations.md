---
title: "Declaration statements - local variables and constants, var, ref local variables"
description: "Declaration statements introduce a new local variable, local constant, or ref local variable. Local variables can be explicitly or implicitly typed. A declaration statement can also include initialization of a variable's value."
ms.date: 01/30/2023
f1_keywords: 
  - "var"
  - "var_CSharpKeyword"
helpviewer_keywords: 
  - "var keyword [C#]"
---
# Declaration statements

A declaration statement declares a new local variable, local constant, or [ref local variable](#ref-locals). To declare a local variable, specify its type and provide its name. You can declare multiple variables of the same type in one statement, as the following example shows:

:::code language="csharp" source="snippets/declarations/Declaration.cs" id="Declare":::

In a declaration statement, you can also initialize a variable with its initial value:

:::code language="csharp" source="snippets/declarations/Declaration.cs" id="DeclareAndInit":::

The preceding examples explicitly specify the type of a variable. You can also let the compiler infer the type of a variable from its initialization expression. To do that, use the `var` keyword instead of a type's name. For more information, see the [Implicitly-typed local variables](#implicitly-typed-local-variables) section.

To declare a local constant, use the [`const` keyword](../keywords/const.md), as the following example shows:

:::code language="csharp" source="snippets/declarations/Declaration.cs" id="LocalConstant":::

When you declare a local constant, you must also initialize it.

For information about ref local variables, see the [Ref locals](#ref-locals) section.

## Implicitly-typed local variables

When you declare a local variable, you can let the compiler infer the type of the variable from the initialization expression. To do that use the `var` keyword instead of the name of a type:

:::code language="csharp" source="snippets/declarations/ImplicitlyTyped.cs" id="ImplicitlyTyped":::

As the preceding example shows, implicitly-typed local variables are strongly typed.

> [!NOTE]
> When you use `var` in the enabled [nullable aware context](../builtin-types/nullable-reference-types.md) and the type of an initialization expression is a reference type, the compiler always infers a **nullable** reference type even if the type of an initialization expression isn't nullable.

A common use of `var` is with a [constructor invocation expression](../operators/new-operator.md#constructor-invocation). The use of `var` allows you to not repeat a type name in a variable declaration and object instantiation, as the following example shows:

```csharp
var xs = new List<int>();
```

Beginning with C# 9.0, you can use a [target-typed `new` expression](../operators/new-operator.md#target-typed-new) as an alternative:

```csharp
List<int> xs = new();
List<int>? ys = new();
```

When you work with [anonymous types](../../fundamentals/types/anonymous-types.md), you must use implicitly-typed local variables. The following example shows a [query expression](../keywords/query-keywords.md) that uses an anonymous type to hold a customer's name and phone number:

:::code language="csharp" source="snippets/declarations/ImplicitlyTyped.cs" id="VarExample":::

In the preceding example, you can't explicitly specify the type of the `fromPhoenix` variable. The type is <xref:System.Collections.Generic.IEnumerable%601> but in this case `T` is an anonymous type and you cannot provide its name. That's why you need to use `var`. For the same reason, you must use `var` when you declare the `customer` iteration variable in the `foreach` statement.

For more information about implicitly-typed local variables, see [Implicitly-typed local variables](../../programming-guide/classes-and-structs/implicitly-typed-local-variables.md).

In pattern matching, the `var` keyword is used in a [`var` pattern](../operators/patterns.md#var-pattern).

## Ref locals

You add the `ref` keyword before the type of a variable to declare a `ref` local. A `ref` local is a variable that *refers to* other storage. Assume the `GetContactInformation` method is declared as a [ref return](jump-statements.md#ref-returns):

```csharp
public ref Person GetContactInformation(string fname, string lname)
```

Let's contrast these two assignments:

```csharp
Person p = contacts.GetContactInformation("Brandie", "Best");
ref Person p2 = ref contacts.GetContactInformation("Brandie", "Best");
```

The variable `p` holds a *copy* of the return value from `GetContactInformation`. It's a separate storage location from the `ref` return from `GetContactInformation`. If you change any property of `p`, you are changing a copy of the `Person`.

The variable `p2` *refers to* the storage location for the `ref` return from `GetContactInformation`. It's the same storage as the `ref` return from `GetContactInformation`. If you change any property of `p2`, you are changing that single instance of a `Person`.

You can access a value by reference in the same way. In some cases, accessing a value by reference increases performance by avoiding a potentially expensive copy operation. For example, the following statement shows how one can define a ref local value that is used to reference a value.

```csharp
ref VeryLargeStruct reflocal = ref veryLargeStruct;
```

The `ref` keyword is used both before the local variable declaration *and* before the value in the second example. Failure to include both `ref` keywords in the variable declaration and assignment in both examples results in compiler error CS8172, "Can't initialize a by-reference variable with a value."

```csharp
ref VeryLargeStruct reflocal = ref veryLargeStruct; // initialization
refLocal = ref anotherVeryLargeStruct; // reassigned, refLocal refers to different storage.
```

Ref local variables must still be initialized when they're declared.

The following example defines a `NumberStore` class that stores an array of integer values. The `FindNumber` method returns by reference the first number that is greater than or equal to the number passed as an argument. If no number is greater than or equal to the argument, the method returns the number in index 0.

:::code language="csharp" source="./snippets/declarations/NumberStore.cs" id="Snippet1":::

The following example calls the `NumberStore.FindNumber` method to retrieve the first value that is greater than or equal to 16. The caller then doubles the value returned by the method. The output from the example shows the change reflected in the value of the array elements of the `NumberStore` instance.

:::code language="csharp" source="./snippets/declarations/NumberStore.cs" id="Snippet2":::

Without support for reference return values, such an operation is performed by returning the index of the array element along with its value. The caller can then use this index to modify the value in a separate method call. However, the caller can also modify the index to access and possibly modify other array values.  

The following example shows how the `FindNumber` method could be rewritten to use ref local reassignment:

:::code language="csharp" source="./snippets/declarations/NumberStoreUpdated.cs" id="Snippet1":::

This second version is more efficient with longer sequences in scenarios where the number sought is closer to the end of the array, as the array is iterated from end towards the beginning, causing fewer items to be examined.

The compiler enforces scope rules on `ref` variables: `ref` locals, `ref` parameters, and `ref` fields in `ref struct` types. The rules ensure that a reference doesn't outlive the object to which it refers. See the section on scoping rules in the article on [method parameters](../keywords/method-parameters.md#scope-of-references-and-values).

### ref and readonly

The `readonly` modifier can be applied to `ref` local variables and `ref` fields. The `readonly` modifier affects the expression to its right. See the following example declarations:

```csharp
ref readonly int aConstant; // aConstant can't be value-reassigned.
readonly ref int Storage; // Storage can't be ref-reassigned.
readonly ref readonly int CantChange; // CantChange can't be value-reassigned or ref-reassigned.
```

- *value reassignment* means the value of the variable is reassigned.
- *ref assignment* means the variable now refers to a different object.

The `readonly ref` and `readonly ref readonly` declarations are valid only on `ref` fields in a `ref struct`.

## scoped ref

The contextual keyword `scoped` restricts the lifetime of a value. The `scoped` modifier restricts the [*ref-safe-to-escape* or *safe-to-escape* lifetime](../keywords/method-parameters.md#scope-of-references-and-values), respectively, to the current method. Effectively, adding the `scoped` modifier asserts that your code won't extend the lifetime of the variable.

You can apply `scoped` to a parameter or local variable. The `scoped` modifier may be applied to parameters and locals when the type is a [`ref struct`](../builtin-types/ref-struct.md). Otherwise, the `scoped` modifier may be applied only to local variables that are [ref types](#ref-locals). That includes local variables declared with the `ref` modifier and parameters declared with the `in`, `ref` or `out` modifiers.

The `scoped` modifier is implicitly added to `this` in methods declared in a `struct`, `out` parameters, and `ref` parameters when the type is a `ref struct`.

## C# language specification

For more information, see the [Declaration statements](~/_csharpstandard/standard/statements.md#136-declaration-statements) section of the [C# language specification](~/_csharpstandard/standard/README.md).

For more information about the `scoped` modifier, see the [Low-level struct improvements](~/_csharplang/proposals/csharp-11.0/low-level-struct-improvements.md) proposal note.

## See also

- [C# reference](../index.md)
- [Object and collection initializers](../../programming-guide/classes-and-structs/object-and-collection-initializers.md)
- [ref keyword](../keywords/ref.md)
- [Reduce memory allocations using new C# features](../../advanced-topics/performance/index.md)
- ['var' preferences (style rules IDE0007 and IDE0008)](../../../fundamentals/code-analysis/style-rules/ide0007-ide0008.md)
