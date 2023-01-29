---
title: The Rust Slice Type
date: 2023-01-28 17:22:10
updated:
categories: [Rust]
tags: [Rust, slice]
---

# The Rust Slice Type

Slice is a kind of data structure that store heap memory, it contains two fields, one representing the memory address and the other representing the length. Our most common string literal is a kind of slice.

Slice let us reference a contiguous sequence of elements in a collection rather than the whole collection. A slice is a kind of reference, so it does not have ownership. And the slice ensures that the data it references is always valid. 

## String Slices

A string slice is a reference to part of a String, and it looks like this:

```rust
let s = String::from("hello world");
let world = &s[6..11];
```
<!-- more -->
The s is a reference to the entire String `hello world`, but the world is a slice, it reference to part of the String `hello world`. String slice referring to part of a `String`

We can use slices to refer to any part of a string with Rust's `..`syntax, for example:

```rust
let s = String::from("hello world");
let world = &s[6..11];    ==> "world"
let s1 = &s[3..];     ==> "lo world"
let s2 = &s[..];      ==> "hello world"
let s3 = &s[..2];      ==> "he"
```

The string slice type is represented by `&str`, it can be used as a function return type, for example:

```rust
fn first_world(s:&String) -> &str {
    &s[..]
}
```

## Other Slices

In addition to string slicing, there are other collection slicing types, such as array slicing.

```rust
let a:[i8;4]=[2, 3, 4, 5];     ==> &[i8]
let slice = &a[1..3];
```

