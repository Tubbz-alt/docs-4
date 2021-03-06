---
title: Protobuf messages - gRPC for WCF Developers
description: Learn how Protobuf messages are defined in the IDL and generated in C#.
author: markrendle
ms.date: 09/09/2019
---

# Protobuf messages

This section covers how to declare Protobuf messages in `.proto` files, explains the fundamental concepts of field numbers and types, and looks at the C# code that is generated by the `protoc` compiler. The rest of the chapter will look in more detail at how different types of data are represented in Protobuf.

## Declaring a message

In WCF, a `Stock` class for a stock market trading application might be defined like the following example:

```csharp
namespace TraderSys
{
    [DataContract]
    public class Stock
    {
        [DataMember]
        public int Id { get; set;}
        [DataMember]
        public string Symbol { get; set;}
        [DataMember]
        public string DisplayName { get; set;}
        [DataMember]
        public int MarketId { get; set; }
    }
}
```

To implement the equivalent class in Protobuf, it must be declared in the `.proto` file. The `protoc` compiler will then generate the .NET class as part of the build process.

```protobuf
syntax "proto3";

option csharp_namespace = "TraderSys";

message Stock {

    int32 id = 1;
    string symbol = 2;
    string display_name = 3;
    int32 market_id = 4;

}  
```

The first line declares the syntax version being used. Version 3 of the language was released in 2016 and is the recommended version for gRPC services.

The `option csharp_namespace` line specifies the namespace to be used for the generated C# types. This option will be ignored when the `.proto` file is compiled for other languages. It is common for Protobuf files to contain language-specific options for several languages.

The `Stock` message definition specifies four fields, each with a type, a name, and a field number.

## Field numbers

Field numbers are an important part of Protobuf. They're used to identify fields in the binary encoded data, which means they can't change from version to version of your service. The advantage is that backward and forward compatibility is possible. Clients and services will simply ignore field numbers they don't know about, as long as the possibility of missing values is handled.

In the binary format, the field number is combined with a type identifier. Field numbers from 1 to 15 can be encoded with their type as a single byte; numbers from 16 to 2047 take 2 bytes. You can go higher if you need more than 2047 fields on a message for any reason. The single byte identifiers for field numbers 1 to 15 offer better performance, so you should use them for the most basic, frequently used fields.

## Types

The type declarations are using Protobuf's native scalar data types, which are discussed in more detail in [the next section](protobuf-data-types.md). The rest of this chapter will cover Protobuf's built-in types and show how they relate to common .NET types.

> [!NOTE]
> Protobuf doesn't natively support a `decimal` type, so double is used instead. For applications that require full decimal precision, refer to the [section on Decimals](protobuf-data-types.md#decimals) in the next part of this chapter.

## The generated code

When you build your application, Protobuf creates classes for each of your messages, mapping its native types to C# types. The generated `Stock` type would have the following signature:

```csharp
public class Stock
{
    public int Id { get; set; }
    public string Symbol { get; set; }
    public string DisplayName { get; set; }
    public int MarketId { get; set; }
}
```

The actual code that is generated is far more complicated than this, because each class contains all the code necessary to serialize and deserialize itself to the binary wire format.

### Property names

Note that the Protobuf compiler applied `PascalCase` to the property names although they were `snake_case` in the `.proto` file. The [Protobuf style guide](https://developers.google.com/protocol-buffers/docs/style) recommends using `snake_case` in your message definitions so that the code generation for other platforms produces the expected case for their conventions.

>[!div class="step-by-step"]
>[Previous](protocol-buffers.md)
>[Next](protobuf-data-types.md)
