# Conclusion

We have now covered how to set up a service on a route,
how to expose a route and how to connect to a service on a different machine.

This is pretty much everything you need to know to develop distributed systems
on Canary, since the fundaments were covered.

The code written in this example is the following:
```rust , no_run
use canary::{service, Channel, Result, routes::GLOBAL_ROUTE, providers::Tcp};

#[canary::main]
async fn main() -> Result<()> {
    Tcp::bind("127.0.0.1:8888").await?;
    GLOBAL_ROUTE.add_service_at::<ping_service>("ping", ())?;

    let mut channel = Tcp::connect("127.0.0.1:8888", "ping").await?;
    channel.send("Ping!").await?;
    let pong: String = channel.receive().await?;
    println!("received {pong}");
    Ok(())
}

#[service]
async fn ping_service(mut channel: Channel) -> Result<()> {
    let ping: String = channel.receive().await?;
    println!("received {ping}");
    channel.send("Pong!").await?;
    Ok(())
}
```
