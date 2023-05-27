# C# Type and Memory Model, Type Conversion

![test img](https://github.com/jurimullercloud/blogs/blob/draft/blogs-topics/porgramming-languages/csharp/test.jpg)
## Introduction

One of the key pillars of learning any programming language is to understand the underlying type structure of the language.
Being a .NET runtime implementation, C# has interesting type structure that may seem easy to grasp at the first sight.
However, not having deep understanding of internal type system can result in critical flaws during software development.
In a few the upcoming blog series, I will try to summarize this topic through the lenses of my own findings.

## Contents

- [Type Implementations in C#](#type-implementations-in-c#)
  - [Value Types](#value-types)
  - [Reference Types](#value-types)
- [Scratching .NET memory model](#contents)

## Type Implementations in C\#

As the name of this section suggests types in C# are implementions. This is because, C# is a language that is implemented
ontop of .NET runtime. For any language that is implemented ontop .NET runtime (which is also called **Common Language Runtime**),
the runtime provides **Common Type System**. The CTS provides an object-oriented model for types that all
.NET languages uses, including the Common Intermediate Language (MSIL). In short there are 5 type categories in
.NET:  

- Classes (implements `System.Object`)
- Structures (implements `System.ValueType` which implements `System.Object`)
- Enumerations (implements `System.Enum` which implements `System.ValueType`)
- Interfaces (maps one to one with C# `interface`)
- Delegates (TODO)

Each prededifned or custom type in C# is the implementation of either a Class type, or Structure or Enumeration types, while
`delegate`'s and `interface`'s have one to one mapping (TODO) between the language and runtime. Therefore, all of the C# types
are implementing `System.Object` class. C# further categorizes those types as:

- Value types
- Reference types  

> **Note**:
    There are also generic type parameters and pointer types that will be covered in the later blogs.

### Value Types

Value types are direct storage of data in the given memory model. They usually have small memory requirements, that's why 
they are subtypes of .NET Structures (therefore implementing `System.ValueType` class.). C# has predefined value types,
which are categorized as follows:

- **Numeric Types**
  - Integrals (`sbyte`, `byte`, `short`, `ushort`, `int`, `uint`, `long`, `ulong`)
  - Floating point types (`float`, `double`, `decimal`)
- **Boolean** (`bool`)
- **Character** (`char`)

Apart from predefined value types, a custom value type can be created using `struct` keyword. The following example below
shows how a complex number can be modelled as a value type using `struct`:

```c#
public struct ComplexNum { public double Real; public double Imaginary;}
```

In fact, all c# predifined value types are *type aliases* of a corresponding runtime type which is defined as a struct. 
See an extract from the [runtime source code for System.Boolean type](src/libraries/System.Private.CoreLib/src/System/Boolean.cs)

```c#
    [Serializable]
    [TypeForwardedFrom("mscorlib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089")]
    public readonly struct Boolean
        : IComparable,
          IConvertible,
          IComparable<bool>,
          IEquatable<bool>,
          ISpanParsable<bool>
    {
        ...
    }
```

> **Note**:
Apart from `decimal` type, all predefined value types are called **runtime primitives**. This is because they are
compiled to a set of intructions that are directly understood by underlying processor. (more on `decimal` vs `double` vs 
`float` in the next blogs.).  

Because value types are (implicitly)inheriting `System.ValueType` class of the runtime, there are few constraints on them:

- They cannot directly inherit from other types.
- No other type can be derived from them (thus all of them get `sealed` modifier when compiled to IL code).

One specificity of value types is that the assignment operator always copies the value to a new memory location.

```c#
int a = 3;
int b = a;
b++;

Console.WriteLine(a); // will output 3
Console.WriteLine(b); // will output 4
```

### **Reference Types**

Reference types are comprised of two components:

- The object that stores the data
- The reference to that object's memory location

A reference type variable stores the reference to the object.
C# has following built-in reference types:  

- `object`(type alias of runtime's `System.Object` class).
- `string`(type alias of runtime's `System.String` class).

All reference types are implementing runtime's `System.Object`
class. Unlike value types, reference types can inherit other reference types and can be inherited by other reference types.

Because a variable of reference type, stores the reference to the object's memory location, variable assignment copies the
value of the reference, instead of object itself. This allows to have multiple references to a same object. See the example below

```c#
public class Todo {
    private string[] _items;
    public int ItemCount {
        get =>  _items.Count();
    }

    public Todo AddItem(string item) {
        _items.Add(item);
        return this;
    }
}

var todo1 = new Todo();
var todo2 = myTodo;

todo2.AddItem("Wake up,");
todo2.AddItem("Sleep again.");

Console.WriteLine(todo1.ItemCount); // 2
```

In the example above, both `todo1` and `todo2` variables are referencing to the same object, thus the state of the object
can be modified using any of them.

A reference type can be assigned the value of `null` indicating that it does not reference to any object.

## Scratching .NET memory model

Having explained, value types and reference types, let's understand how they're stored in the memory.
For a program execution the memory model comprises of two major regions:

- **Stack**  
  Stack is the fixed sized memory chunk that is allocated per execution thread. During program execution, the stack is used
  to order / host function calls. It achieves this ordering through LIFO mechanism (*Last in, First out*). A function is
  represented in the stack as a block inside of which all the data the function needs is stored (i.e. the perameters, the variables defined inside, the operationS on them, and some other bookkeeping metadata). When a function returns, it is popped from
  the stack (this, actually, is the reason why stack works with LIFO mechanism), thus the allocated memory block for the function
  is freed.
  The stack is the structured chunk of memory, and as such it is comperatively faster to store and access data. That being said,
  as allocated parts are freed through popping, the size of stack is relatively smaller. In some cases, when for example, a function
  does infinite loop or infinitire recursion, the stack gets overflown, resulting in famous StackOverflowException.

- **Heap**  
  In comparison to stack, the heap is dynamically allocated chunk of memory. It does not have a particular ordering structure
  as in stack, and is suitable to store the data that is persistent throughout program execution. The objects are such persistent
  data structures whose state might be accessed, or modified at different points of execution. As heap does not have a structure
  for memory allocation and deallocation, the unused (which is also called *unreferenced*) data, is deallocated by the **Garbage
  Collector** (more on Garbage Collection in the next blogs.).

 One of the misconceptions about memory model of C# is that value types are stored on the stack, and reference types are
 stored on the heap. While this is true for the latter that is (the reference are always stored on the heap), it is not
 always true for the value types. See the example below:

```c#
public class Salary {
    public decimal Tax = 0.12;

    public decimal GetNetSalary(decimal gross) {
        decimal net = gross * (1 - 0.12);
        return net;
    }
}

// in Program.cs

var mySalary = new Salary();
mySalary.GetNetSalary(40_000);
```

In this example, when an instance of `Salary` class is being instantiated, `mySalary` object will be stored on the heap, 
while the refernce to the location of `mySalary` object is stored on the stack (because `var mySalary = ...` expression runs
inside the `Main` function). Because `Tax` field is defined inside Salary object, despite being a value type, it will also be stored
on the heap. When user calls the `GetNetSalary` function on the instance, it will be populated on the stack, thus the variables
`gross` and `net`. As you can see while all 3 of these data are of value type (`decimal`), one gets stored on the heap,
while the other 2 on the stack.


