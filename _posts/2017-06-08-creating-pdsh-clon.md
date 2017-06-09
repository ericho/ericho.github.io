---
layout: post
title: "Creating my own pdsh in Rust"
date: 2017-06-08
---

As my learning path in Rust continues, I decided to write a little application that can help me at work. Currently I'm working on building administrative tools for HPC Clusters where it's common to have development environments where a set of nodes is involved (we use [vagrant](https://www.vagrantup.com/) to build a micro-cluster in a devbox).

In this environments is also common to have the need to send a command to a set of nodes at the same time, like change a field in a configuration file or start a program in all nodes at the same time. The [pdsh](https://linux.die.net/man/1/pdsh) is one the most used tools to issue commands in hosts concurrently. So, as a learning task, I started to create [my own version of pdsh in Rust](https://github.com/ericho/jaibon).

The plan is : 

    * To have a tool that solves the same problem as `pdsh`. It's not a clone, at least for now.
    * Send commands through ssh to the hosts. Asumes that the ssh key setup was done before, there's no support for setting ssh passwords.
    * It shall be capable to copy files with scp.
    * Shall support the hostlist format.
    * It would be nice to print results in JSON format.
    
## Jaibon

I called the project `jaibon`, not particular reason, and at this point this the usage supported. 

```
jaibon 0.1.0

USAGE:
    jaibon [OPTIONS] --command <command> --nodes <nodes>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -c, --command <command>    The command to be executed in the remote hosts.
    -n, --nodes <nodes>        Specify the node or list of nodes where the command
                               will be executed. The list of nodes should be set in a
                               separated comma list format.
    -u, --user <user>          Set the username used to perform the SSH connection to
                               the specified hosts. If is not set, the user will be
                               retrieved from the $USER environmental variable.
```

Basically only needs a command and a set of nodes in order to work. The user parameter is optional, if is not provided then the $USER environment variable will be used. An example of the usage will be: 

![Using jaibon](https://raw.githubusercontent.com/ericho/jaibon/master/demo.png)

## How it works

The idea behind this is just to create ssh commands with the parameters provided. For instance, if you want to run the `uptime` command in a remote node, you can do the following: 

```
ssh myuser@host uptime
```

And the process will return the output of the remote execution in your terminal.

### The command type

I created the `command` type to store all the information regarding the ongoing command.

```rust
pub struct Command {
    command: String,
    pub stdout: String,
    pub stderr: String,
    pub node: String,
    user: String,
    pub result: CommandResult,
    pub printer: CommandPrinter,
}
```

The structure is very straigthforward, it should store the command, the user, the stdout and stderr of the process and the overall result execution. Also is possible to define a printer for the command (planning to support JSON in the future).

### Launching commands

A vector stores the list of nodes, so a loop is used to spawn threads for every node.

```rust
    for i in nodes_vec {
        let theuser = user.clone();
        let thecmd = command.clone();
        thread_handlers.push(thread::spawn(move || {
                                               println!("Launching command on node {}", i);
                                               let mut cmd = Command::new(theuser.to_owned(),
                                                                          &i,
                                                                          thecmd.to_owned());
                                               cmd.run();
                                               cmd
                                           }));
    }
```

On every thread a command structure is created and then the `run()` method is executed. All the thread handlers are stored in the `thread_handlers` vector.

Then, to wait for results only the following code is needed.

```rust
    for t in thread_handlers {
        let cmd = t.join().unwrap();
        println!("{}", cmd);
    }
```

In order to use `println!("{}", cmd)`, the `Display` trait was implemented in the `command` structure. 

```
impl fmt::Display for Command {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self.printer {
            CommandPrinter::DefaultPrinter => self.default_formatter(f),
        }
    }
}
```

Here the `CommandPrinter` enum is in charge of dynamically dispatch the needed printer. At this point only the default formatter is enabled.

## Conclusion

These are the things that I've learn in this first version:

    * If the compiler complains a lot, probably the design is wrong.
    * The `Command::new` method works but I'm not sure if that is the best way to create a new `command` structure. Should I use `&str` instead of `String`?
    * I won a battle against the compiler, but definitely I'm far to win the war.
    * Not sure if I should use the [ssh2](https://crates.io/crates/ssh2) crate instead of launching commands.
    * I'm starting to love traits!

I'll continue with the support of `scp` and the hostlist support.
