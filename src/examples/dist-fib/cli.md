# Command line

Now that we've got the code for the cluster and the node,
we've got to create the interface.

For this we'll use structopt.

First we'll define a struct with our commands:
```rust , no_run
#[derive(StructOpt, Debug)]
struct Opt {
    #[structopt(subcommand)]
    command: Command,
}

#[derive(StructOpt, Debug)]
enum Command {
    // create a node worker and connect it to cluster with address
    Node {
        #[structopt(short, long, parse(try_from_str = Addr::new))]
        addr: Addr
    },
    // create a cluster at address
    Cluster {
        #[structopt(short, long, parse(try_from_str = Addr::new))]
        addr: Addr
    },
    // create a cluster at address
    Client {
        #[structopt(short, long, parse(try_from_str = Addr::new))]
        addr: Addr,
        number: u32
    }
}
```

[WIP]
