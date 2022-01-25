# Conclusion

We have now covered how to set up a service on a route,
how to expose a route and how to connect to a service on a different machine.

This is pretty much everything you need to know to develop distributed systems
on Canary, since the fundamentals were covered.

The code written in this example is the following:
```rust , no_run
use canary::prelude::*;
use canary::providers::Tcp;

#[canary::main]
async fn main() -> Res<()> {
    //                        name of service | location of service
    GLOBAL_ROUTE.add_service_default_at::<ping_service>("ping")?;

    Tcp::bind("127.0.0.1:8888").await?;
    let mut channel = Tcp::connect("127.0.0.1:8888", "ping").await.unwrap();

    channel.send("Hello world!").await?;
    let result: String = channel.receive().await?;
    println!("peer sent {result}");
    Ok(())
}

#[service]
async fn ping_service(mut channel: Channel) -> Res<()> {
    // receive 1, type String
    let ping: String = channel.receive().await?;
    println!("received {ping}");

    // send 2, type String. str and String have equivalent wire representations,
    // so this is safe.
    channel.send("Pong!").await?;
    Ok(())
}
```
