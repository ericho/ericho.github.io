---
layout: post
title: "Calculate SHA1 sum to all iso files using a threadpool"
date: 2017-10-02
---

The [rust-cookbook](https://rust-lang-nursery.github.io/rust-cookbook/) provides examples on the usage of the most common Rust crates. As a learning task I sent a pull request to this repository to provide a new example of the usage of the [threadpool](https://docs.rs/threadpool/1.7.0/threadpool/) crate.

In this example, the [Walkdir](https://docs.rs/walkdir/1.0.7/walkdir/) crate is used to get all the `iso` files in a folder tree and then calculates the SHA1 sum for every file found.

The full code can be found [here](https://rust-lang-nursery.github.io/rust-cookbook/concurrency.html#ex-threadpool-walk) and the raw version can be seen [here](https://github.com/rust-lang-nursery/rust-cookbook/blob/master/src/concurrency.md).

## Setup

If you see the raw version, you'll notice the `#` commenting out some parts of the code. This is used to hide code from the rendered example.

The example is based on [error_chain](https://github.com/rust-lang-nursery/error-chain) crate in order to use the `?` operator to return `Result` types.

## Code explanation

### Creating the pool

```rust
let pool = ThreadPool::new(num_cpus::get());
let (tx, rx) = channel();
```

Here the threadpool is created using the number of cpus available on the system. To get that number, the [num_cpus](https://docs.rs/num_cpus/1.7.0/num_cpus/) crate is used. Then a channel is created to perform the communication between threads, in this example in a Multiple Producers Single Consumer way.

### Walking the folder and launching threads.

```rust
for entry in WalkDir::new("/home/user/Downloads")
    .follow_links(true)
    .into_iter()
    .filter_map(|e| e.ok())
    .filter(|e| !e.path().is_dir() && is_iso(e.path())) {
        let path = entry.path().to_owned();
        let tx = tx.clone();
        pool.execute(move || {
            let digest = compute_digest(path);
            tx.send(digest).expect("Could not send data!");
        });
    }
```

A lot of things are happening in this code block (the beauty of Rust). First `Walkdir::new("/the/folder").follow_links(true).into_iter()` returns an iterator for every file and folder under the supplied path. Then `.filter_map(|e| e.ok())` filters only the files that are able to be opened.

`.filter(|e| !e.path().is_dir() && is_iso(e.path()))` filters the directoies and the non iso files.

#### Launching threads

Now we have only the interested "iso" files and we are ready to launch the threads.

```rust
let path = entry.path().to_owned();
let tx = tx.clone();
pool.execute(move || {
    let digest = compute_digest(path);
    tx.send(digest).expect("Could not send data!");
});
```

`path` and `tx` needs to be cloned as the threads runs in a different scope. `pool.execute(move ||)` moves the variables in the scope into the lamda.

Inside the pool only two lines are run. The first one runs the `compute_digest(path)` function and then `tx.send` sends the result to the receiver.

### Check if is iso

```rust
fn is_iso(entry: &Path) -> bool {
    match entry.extension() {
        Some(e) if e.to_string_lossy().to_lowercase() == "iso" => true,
        _ => false,
    }
}
```

This function receives a [Path](https://doc.rust-lang.org/std/path/struct.Path.html) reference and returns a `bool` if the extension is "iso".

A match statement for `entry.extension()` checks is the file has "iso extension".

### Compute digest

```rust
fn compute_digest<P: AsRef<Path>>(filepath: P) -> Result<(Digest, P)> {
    let mut buf_reader = BufReader::new(File::open(&filepath)?);
    let mut context = Context::new(&SHA1);
    let mut buffer = [0; 1024];

    loop {
        let count = buf_reader.read(&mut buffer)?;
        if count == 0 {
            break;
        }
        context.update(&buffer[..count]);
    }

    Ok((context.finish(), filepath))
}
```

This function returns a referenced Path and returns a `Result<(Digest, P)>` where P is the received Path.

Then a `BufReader` is used to read the file. The `Context` type helps tu read the by in chunks of 1024 bytes. This is useful to read big files without loading all file in memory.

### Receive results

```rust
drop(tx);
for t in rx.iter() {
    let (sha, path) = t?;
    println!("{:?} {:?}", sha, path);
}
```

`rx.iter()` is used to receive all the results from the threads. The received result is unwrap and printed.

The `drop(tx)` at the beginning helps to avoid that `rx.iter()` hangs forever.
