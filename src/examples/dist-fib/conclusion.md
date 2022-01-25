# Conclusion

Having covered the node and cluster implementation, and having built
the CLI, this is what the code looks like.

```rust
use canary::prelude::*;
use srpc::prelude::*;

use structopt::StructOpt;

#[derive(StructOpt, Debug)]
struct Opt {
    #[structopt(subcommand)]
    command: Command,
}

#[derive(StructOpt, Debug)]
enum Command {
    Node {
        #[structopt(short, long, parse(try_from_str = Addr::new))]
        addr: Addr
    },
    Cluster {
        #[structopt(short, long, parse(try_from_str = Addr::new))]
        addr: Addr
    },
    Client {
        #[structopt(short, long, parse(try_from_str = Addr::new))]
        addr: Addr,
        number: u64
    }
}

#[canary::main]
async fn main() -> Res<()> {
    let opt = Opt::from_args();

    match opt.command {
        Command::Cluster { addr } => {
            GLOBAL_ROUTE.add_service_default_at::<DistributedFibCluster>("cluster")?;

            println!("starting cluster at {:?}", addr);
            addr.bind().await?;

            std::future::pending::<()>().await;
        },
        Command::Node { addr } => {
            GLOBAL_ROUTE.add_service_default_at::<DistributedFibNode>("node")?;
            println!("starting node and connecting to {:?}", addr);
            let cluster = addr.service("cluster")
                .connect()
                .await?
                .client::<DistributedFibCluster>();

            let cluster: Channel = cluster.insert_node().await?;
            GLOBAL_ROUTE.switch("node", cluster.bare()).ok();
        },
        Command::Client { addr, number } => {
            println!("calculating fib of {} remotely", number);

            let mut cluster = addr.service("cluster").connect().await?.client::<DistributedFibCluster>();
            let res = cluster.calculate_fib(number).await?;

            println!("calculated `{res}` remotely");
        },
    }
    Ok(())
}



#[srpc::rpc]
#[derive(Default)]
struct DistributedFibCluster {
    nodes: Vec<DistributedFibNodePeer>,
}

#[srpc::rpc]
impl DistributedFibCluster {
    async fn calculate_fib(&mut self, num: u64) -> u64 {
        if self.nodes.is_empty() {
            println!("calculating fib locally");
            // no nodes :(
            // calculate fib locally instead
            return fibonacci(num)
        }
        let node_num = fastrand::usize(..self.nodes.len());
        let node = &mut self.nodes[node_num];
        println!("sending op to node");
        match node.fibonacci(num).await {
            Ok(res) => res,
            Err(_) => {
                println!("calculating fib locally and removing node");
                // error :(
                // calculate fib locally instead
                // and remove this node since it is unreliable
                self.nodes.remove(node_num);
                fibonacci(num)
            }
        }
    }
    #[consume]
    async fn insert_node(&mut self, chan: Channel) -> Res<()> {
        let chan = chan.client::<DistributedFibNode>();
        self.nodes.push(chan);
        Ok(())
    }
}
```
