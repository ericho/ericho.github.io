---
layout: post
title: Code review on Tokio chat example
data: 2017-02-26
---

In my journey learning Rust I decided to take a look to the [Tokio](https://tokio.rs/) project. Tokio is a project to write fast networking code in Rust, it uses a set of concepts new to me, like [futures](https://tokio.rs/docs/getting-started/futures/) among other things.

It's worth to say that I failed in my intention of create a network application, it was difficult due to my lack of knowledge of Rust and all this new concepts at the same time. So I decide to create a code review of one of their examples, the [chat example](https://github.com/tokio-rs/tokio-core/blob/master/examples/chat.rs). 

## The chat example

This example creates a "chat server" that receives connections and then every client can type messages. All the clients will receive the message but not the sender.

When the application is ready its only needed to run.

```
nc 127.0.0.1 8080
```

And start typing. This can be launched in different terminal to see the messages appear.

## Creating the project

Like any other Rust project we create it using `cargo`, just run:

```
cargo init --bin chatserver
```

We will write all the program content on `main.rs` file. Also we need to update the `Cargo.toml` file the required dependencies.

```
[dependencies]
futures = "0.1"
tokio-core = "0.1"
```

## Looking in the code

### The included crates

This example only uses two crates, so they only require:

```rust
extern crate tokio_core;
extern crate futures;
```

`tokio-core` manages the low level networking as well as other tools, `futures` is the crate that enable us to have asynchronous events.

### Included components

```rust
use std::collections::HashMap;
use std::rc::Rc;
use std::cell::RefCell;
use std::iter;
use std::env;
use std::io::{Error, ErrorKind, BufReader};
```

These are the components needed from the standard library. The `HashMap` will be used to save all the references to the connected clients. 

`Rc` and `RefCell` are type wrappers to provide mutability features. `Rc` let us to share data having multiple "owners" by using a reference count, the data is freed once the counter decreases to zero. `RefCell` provides internal mutability. [Here](http://manishearth.github.io/blog/2015/05/27/wrapper-types-in-rust-choosing-your-guarantees/) is a good article on both topics.

The `iter` library let us use iterators, `env` is to use the command line arguments in this program. `Error` and `ErrorKind` are used to create errors when the connection is lost. `BufReader` its used to read incoming data.

```rust
use tokio_core::net::TcpListener;
use tokio_core::reactor::Core;
use tokio_core::io::{self, Io};

use futures::stream::{self, Stream};
use futures::Future;
```

These are the `tokio` and `futures` use statements. Basically we are using a `TcpListener` to create the sockets, and `Stream` to handle all the incoming connections and some helpers to read and write data.

### Creating the TCP listener

```rust
let addr = env::args().nth(1).unwrap_or("127.0.0.1:8080".to_string());
let addr = addr.parse().unwrap();

let mut core = Core::new().unwrap();
let handle = core.handle();
let socket = TcpListener::bind(&addr, &handle).unwrap();
```

Let's go line by line:

```rust
let addr = env::args().nth(1).unwrap_or("127.0.0.1:8080".to_string());
```

The [`env::args()`](https://doc.rust-lang.org/std/env/fn.args.html) function returns the arguments used to run the program, an [`Args`](https://doc.rust-lang.org/std/env/struct.Args.html) iterator is returned. `Args` has the `nth` method which returns the `n` element of the iterator. `nth` will return and `Option<Self::Item>` with the supplied argument or will be `None` if there's nothing in the iterator at that index value. 

So `unwrap_or` is a method of that `Option` returned that unlike `unwrap` that will panic if `None` is received, it will set `127.0.0.1:8080` as the default value. String literals are of `str` type, a slice of characters which is not the same as a `String`. The `to_string()` is used to convert a slice of characters into a String. Here is a good article on [str vs String](http://hermanradtke.com/2015/05/03/string-vs-str-in-rust-functions.html).




