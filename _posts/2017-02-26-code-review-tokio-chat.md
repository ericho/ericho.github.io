---
layout: post
title: Code review on Tokio chat example - Part 1
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
use std::net::SocketAddr;
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
let addr = addr.parse::<SocketAddr>().unwrap();

let mut core = Core::new().unwrap();
let handle = core.handle();
let socket = TcpListener::bind(&addr, &handle).unwrap();
let connections = Rc::new(RefCell::new(HashMap::new()));
```

Let's go line by line:

```rust
let addr = env::args().nth(1).unwrap_or("127.0.0.1:8080".to_string());
```

The [`env::args()`](https://doc.rust-lang.org/std/env/fn.args.html) function returns the arguments used to run the program, an [`Args`](https://doc.rust-lang.org/std/env/struct.Args.html) iterator is returned. `Args` has the `nth` method which returns the `n` element of the iterator. `nth` will return and `Option<Self::Item>` with the supplied argument or will be `None` if there's nothing in the iterator at that index value. 

So `unwrap_or` is a method of that `Option` returned that unlike `unwrap` that will panic if `None` is received, it will set `127.0.0.1:8080` as the default value. String literals are of `str` type, a slice of characters which is not the same as a `String`. The `to_string()` is used to convert a slice of characters into a String. Here is a good article on [str vs String](http://hermanradtke.com/2015/05/03/string-vs-str-in-rust-functions.html).

```rust
let addr = addr.parse::<SocketAddr>().unwrap();
```

The first thing that can be noticed in this line is that the `addr` binding is declared again. This is possible but we'll lose the old value, in this case we don't care about that because we are saving a new value of `addr` from the parsed old one. The `addr` string has a method `parse` that parses the current string into a new one depending on the specified type. The "turbofish" syntax, `::<>`, helps to determine which algorithm will be use for the parsing. In this case the `SocketAddr` type is used to parse a correct IP. Forget about regex to parse a correct IP address, this does the job :). 

If a malformed string is passed the program will panic due to the `unwrap` at the end of the line.

```rust
let mut core = Core::new().unwrap();
let handle = core.handle();
```

The `Core::new()` creates a new event loop which is the core of this program. The `Core` will receive `Futures` to handle them when are ready. Think on a `Future` as something that will be finished in the future. The concept of callback can sound similar, although from a functionality perspective it would be the same, internally is complete different. The good part of this is that a single threaded program can handle multiples futures. More on event loops in tokio can be found [here](https://tokio.rs/docs/getting-started/reactor/).

A `handle` is created to get access to the loop. 

```rust
let socket = TcpListener::bind(&addr, &handle).unwrap();
println!("Listening on: {}", addr);
```

A socket is created by binding the address with the handle of the event loop. Now the socket is also bind with the event loop. `socket` is `TcpListener` type. A message is printed to show that we are listening in the specified connection.

```rust
let connections = Rc::new(RefCell::new(HashMap::new()));
```

The `connections` hashmap is created to be referenced counted by `Rc` and internally mutable with `RefCell`. `connections` will store the list of open connections to the server.

### The stream of incoming sockets

```rust
let srv = socket.incoming().for_each(move |(stream, addr)| {
    // A lot of code here...
});
```

This is a big block in the code, so I'm separating the explanation in parts. `socket` is a `TcpListener` that has a `incoming` method. This method will return a stream of the sockets this listener support. `incoming()` returns an iterator and that is way there is a `for_earch` function that is in charge of handle each received connection. 

The `for_each` function uses a [closure](https://doc.rust-lang.org/book/closures.html) as parameter with two arguments, `stream` and `addr`.

### For each socket

All the code inside the `for_each` will be explained here. Remember, we have two bindings in this scope `stream` and `addr`.

```rust
println!("New Connection: {}", addr);
let (reader, writer) = stream.split();
```

First, the information that a new connection was received is printed. Then the stream is split into two pieces, the reader and the writer.

```rust
let (tx, rx) = futures::sync::mpsc::unbounded();
connections.borrow_mut().insert(addr, tx);
```

In the first line, two channels are created to send and receive data, the sender then is inserted into the `connections` hashmap. At this point we have the `addr` as a key and the sender channel.

```rust
let connections_inner = connections.clone();
let reader = BufReader::new(reader);
```

The `connections` hashmap is duplicated into `connections_inner` for later use. Also a new `BufReader` is created.

```rust
let iter = stream::iter(iter::repeat(()).map(Ok::<(), Error>));
```

The `iter` function of `stream` creates an iterator which is created by `iter::repeat(()).map(Ok::<(), Error>)`. The `iter::repeat(())` creates an infinite iterator of `()` value. 
