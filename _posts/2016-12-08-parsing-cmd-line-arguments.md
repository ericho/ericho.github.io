---
layout: post
title: "Parsing command line arguments on Rust"
date: 2016-12-08
---

Command line parsing is needed on every program that receives parameters from the user. The [clap crate](https://crates.io/crates/clap) helps us to have a really nice command line parser with a lot of [features enabled](http://clap.rs/).

As an example this is a really simple application that takes two numbers from the command line and return the sum of them.

The Cargo.toml content
```
[package]
name = "cmd_parser"
version = "0.1.0"
authors = ["Erich Cordoba <erich.cm@yandex.com>"]

[dependencies]
clap = "2.19.2"
```

The code

```rust
#[macro_use] extern crate clap;

use clap::{Arg, App};

fn main() {
    let matches = App::new("myapp")
					.version("0.1.0")
					.author("Erich Cordoba")
					.about("Arithmetic operations between two numbers")
					.arg(Arg::with_name("num1")
						.short("n")
						.long("num1")
						.value_name("n")
						.help("The first number")
						.takes_value(true)
    					.required(true))
					.arg(Arg::with_name("num2")
						.short("m")
						.long("num2")
						.value_name("m")
						.help("The second number")
						.takes_value(true)
						.required(true))
					.get_matches();

    let n = value_t!(matches.value_of("num1"), i32).unwrap();
    let m = value_t!(matches.value_of("num2"), i32).unwrap();

    println!("{} + {} = {}", n, m, n + m);
}

```

The first section enabled the use of clap crate in our program. We tell the compiler that we will use macros from `clap` and also the Arg and App components will be used.

```rust
#[macro_use] extern crate clap;

use clap::{Arg, App};
```
This section defines the options an arguments.

```rust
let matches = App::new("myapp")
      .version("0.1.0")
      .author("Erich Cordoba")
      .about("Arithmetic operations between two numbers")
      .arg(Arg::with_name("num1")
        .short("n")
        .long("num1")
        .value_name("n")
        .help("The first number")
        .takes_value(true)
          .required(true))
      .arg(Arg::with_name("num2")
        .short("m")
        .long("num2")
        .value_name("m")
        .help("The second number")
        .takes_value(true)
        .required(true))
      .get_matches();
```

This will be used to get help message when run with `--help`.

At the end the macro `value_t!` is used to parse the argument as the defined type, in this case i32. `value_t!` returns a `Result` type so we'll need to unwrap it.

And finally we print the result.

```rust-lang
let n = value_t!(matches.value_of("num1"), i32).unwrap();
let m = value_t!(matches.value_of("num2"), i32).unwrap();

println!("{} + {} = {}", n, m, n + m);
```
With this simple example we can get an output like this when the `--help` is supplied.

```
myapp 0.1.0
Erich Cordoba
Arithmetic operations between two numbers

USAGE:
    cmd_parser --num1 <n> --num2 <m>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -n, --num1 <n>    The first number
    -m, --num2 <m>    The second number
```

And also, if an argument is not supplied the following error will appear.

```
error: The following required arguments were not provided:
    --num1 <n>
    --num2 <m>

USAGE:
    cmd_parser --num1 <n> --num2 <m>

For more information try --help
```
