---
layout: post
title: "My first steps on Rust"
date: 2016-12-07
---

Has been a couple of days since I started to try Rust. I was scared about the syntax which I found a little strange. I'm not a programmer, I'm an eletronics engineer that somehow learn to use Linux and code a bit and now is working as a _Software development engineer_.

My first touch with a programming language was with C when I was at high school, I made some projects in college with C and microcontrollers but nothing fancy. Some years later I met Python and did some projects for profit. After that I got my current job and know I use C/C++ and Python as my daily work.

So I live coding but without any serious background, I just like to learn new things and make some experiments. This year I started with Go which I found very interesting (I'll post later about this) and started looking at Rust.

My first impression on Rust was that is a *complicated* language. The syntax looks weird and has too many symbols that I barely understand. When I was learning Go (I'm still learning...) I was reading more about Rust. The people that I admire and follow on Twitter were excited about Rust.

Mainly [this post of Julia Evans](https://jvns.ca/blog/2014/03/12/the-rust-os-story/) about how start writing a simple Operating System on Rust was very exciting, so I decided to give a try.

## Things that I like about Rust (so far)

1. **The community**: A lot of people that likes open source works on their spare time. The people gathered around Rust seems to be the more passionated people of the open source community (maybe I'm overdoing this comment). The community is great, the support that you can found on the [User forums](https://users.rust-lang.org/) is awesome. There is also a section called [This week in Rust](https://this-week-in-rust.org/) where a summary of blog posts, news, releases and other stuff is published every week.
2. **The compiler**: `rustc` is great, it's very straightforward and gives you all the information that you need to continue your work. The compiler gives you hints on how to fix your compilation problems. Also, compilation is static so the deployment is very easy.
3. **The dependency management**: `cargo` is also a great tool. You just need to put a new dependency in a text file and `cargo` will download and install that dependency for your project.
4. **The test suite capabilities**: Test Driven Development is a must nowadays and with rust is very easy to do it. Just define a test and `cargo test` will handle the rest. I cannot find yet how to mock functions which is something required once you need to interact with external components within your tests, but there should be a way.
5. **The memory management**: I have read a couple of chapters of [The book](https://doc.rust-lang.org/book/) and now I'm in the ownership concept. What I understand so far is that only one binding (variable) can access to an allocated memory at the same time. This reduces race conditions. I have a lot to learn about this but looks very interesting.
