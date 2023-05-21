# C# Type and Memory Model, Type Conversion

## Introduction
One of the key pillars of learning any programming language is to understand the underlying type structure of the language. 
Being a .NET runtime implementation, C# has interesting type structure that may seem easy to grasp at the first sight. 
However, not having deep understanding of internal type system can result in critical flaws during software development.
In a few the upcoming blog series, I will try to summarize this topic through the lenses of my own findings. 


## Contents
 - [Type Implementations in C#](#type-implementations-in-c)
 - [Scratching .NET memory model]() 
 - [Type Conversions in C#]()
 - [Conclusion]()


## Type Implementations in C#

As the name of this section suggests types in C# are implementions. This is because, C# is a language that is implemented 
ontop of .NET runtime. For any language that is implemented ontop .NET runtime (which is also called **Common Language Runtime**), 
the runtime provides **Common Type System**. The CTS provides an object-oriented model for types that the IL code 
(which is the common language that all .NET languages are compiled into) uses. In short there are 5 type categories in 
.NET:  
 - Classes (implements `System.Object`)
 - Structures (implements `System.ValueType` which implements `System.Object`)
 - Enumerations (implements `System.Enum` which implements `System.ValueType`)
 - Interfaces (TODO)
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
they are subtypes of .NET Structures (therefore implementing `System.ValueType` class.).
A value type can be created using `struct` keyword:

```c#
public struct ComplexNum { public double Real; public double Imaginary;}
```

Apart from custom value types, there are also **predefined value types**. These include:
 - **Numeric Types**
  - Integrals (`sbyte`, `byte`, `short`, `ushort`, `int`, `uint`, `long`, `ulong`)
  - Floating point types (`float`, `double`, `decimal`) 
 - **Boolean** (`bool`)
 - **Character** (`char`)

> **Note**:   
Apart from `decimal` type, all predefined value types are called **runtime primitives**. This is because they are
compiled to a set of intructions that are directly understood by underlying processor. (more on `decimal` vs `double` vs `float` in the next blogs.).

Because value types are inheriting `System.ValueType` class of the runtime, there are few constraints on them:
 - They cannot directly inherit from other types.
 - No other type can be derived from, thus all of them are sealed. 



### Reference Types












