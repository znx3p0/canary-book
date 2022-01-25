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

After defining the command line interface, we can now build the CLI.

```rust
#[canary::main]
async fn main() -> Res<()> {
    let opt = Opt::from_args();

    match opt.command {
        Command::Cluster { addr } => {
            GLOBAL_ROUTE.add_service_default_at::<DistributedFibCluster>("cluster")?;
            println!("starting cluster at {addr:?}");
            addr.bind().await?;
            std::future::pending::<()>().await;
        },

        Command::Node { addr } => {
            GLOBAL_ROUTE.add_service_default_at::<DistributedFibNode>("node")?;

            println!("starting node and connecting to {addr:?}");
            let cluster = addr.service("cluster")
                .connect()
                .await?
                .client::<DistributedFibCluster>();
            let cluster: Channel = cluster.insert_node().await?;

            GLOBAL_ROUTE.switch("node", cluster.bare()).ok(); // inserts the channel into the "node" service
        },

        Command::Client { addr, number } => {
            println!("calculating fib of {} remotely", number);

            let mut cluster = addr.service("cluster")
                .connect()
                .await?
                .client::<DistributedFibCluster>();
            let res = cluster.calculate_fib(number).await?;

            println!("calculated `{res}` remotely");
        },
    }
    Ok(())
}
```
