---
layout: post
title: "Understanding Rust Enums"
date: 2017-02-12
---

One of the first things that I saw in Rust is that a lot of API functions returns some `Option<T>` of `Result<T, E>` types. It turns out that these values are `enums`, but they differ from the classical definition, at least the one that I'm used to.

Reading [the documentation](https://doc.rust-lang.org/book/enums.html) I learned that enums can:

  - *Store different types*: The variants can be an integer but also a string or even a struct storing an integer and a string at the same time.
  - *Have functions*: A enum can have functions. This make sense if you receive a enum and you need to know what variant is.
  - *Have fixed size*: The size of a `enum` is the size of the biggest variant plus some bytes for tagging.

Consider that you have a enum Animal

```rust
enum Animal {
     Dog,
     Cat,
     Fish(i32),
     Horse{weight: i32, age: i32}
}
```

This enum has three types of variants:
  1. With no data: Like `Dog` and `Cat`. Those are not declared in other point of the program, just exists here as a variant of `Animal`.
  2. With data: The `Fish` variant can have a `i32` value stored there.
  3. Struct like data: As `Horse` variant, it can be a complex type.

Now that we have a enum we can have a function like this:

```rust
fn guessAnimal(a: Animal) {
   match a {
   	 Animal::Cat => println!("It's a cat!")
	 Animal::Dog => println!("It's a dog!")
	 Animal::Fish(i) => println!("It's a fish with {} eyes", i)
   }
}
```

With `match` we can identify the variant of a enum and proceed accordingly.

Enums are useful to determine the result of a operation, and that's is why `Result<T, E>` exists. Imagine that we want to know if something went wront, what actually was the problem with a detailed string message, for instance.