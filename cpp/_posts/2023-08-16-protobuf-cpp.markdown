---
layout: post
title:  "Protobuf in C++"
date:   2023-08-16 16:30:00 +0200
categories: cpp update
---

## Introduction

Google Protobuffers aka. Protobuf is both language and platform agnostic standard of serialization and deserialization of data. Unless like JSON or XML is binary which is not readable for ape-like species. However silicon-like species can read it really well.

## In the beginning there was Compiler

To effectively use protobuf you need to install the compiler. Yes compiler - since protobuf
the designers decided to create their own language to describe the data. The language is called proto and the compiler is called protoc. Compiler is available for many platforms and allows for compilation of proto files describing the objects you want to serialize/deserialize into many languages.

Yes it means you for example can serialize in C++ and deserialize in Ruby.

Yes it means you write the object definition only once.

For more details about proto files just look [here](https://protobuf.dev/programming-guides/proto3/).

When you have the files ready then just call the compiler (for C++ output):

```
protoc -I=$SRC_DIR --cpp_out=$DST_DIR <.proto files>
```

For C++ it will generate respective .h and .cc files.

## The code

The generated files contain classes with accessor functions based on the definition in .proto files. the classes also contain methods to serialize and deserialize the data.

## Summary

This post is just high-level information about Protocol Buffers. If you want to learn more, just head up to the protobuf [documentation](https://protobuf.dev). You can find there more information about the proto language itself and examples of use in different languages.

Repeating the Internet over the places is just a waste of time. Better use pointers to correct places.

## References

[Protobuf documentation](https://protobuf.dev)