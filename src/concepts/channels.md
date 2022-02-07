# Channels

Channels are a backend-agnostic way of communicating with peers.
You can think of it as a wrapper around a stream (tcp, websockets or any other backend) that allows
you to send and receive objects or messages.

For example, let's assume you have connected Alice's machine and Bob's machine.

```rust , no_run
use canary::Channel;
use canary::Result; // result is equivalent to std::io::Result, but it implements `Serialize` / `Deserialize`

// runs on Alice's machine
async fn alice(mut chan: Channel) -> Result<()> {
    chan.send("Hey Bob!").await?;
    Ok(())
}

// runs on Bob's machine
async fn bob(mut chan: Channel) -> Result<()> {
    let message: String = chan.receive().await?;
    println!("alice says: `{}`", message); // alice says: `Hey Bob!`
    Ok(())
}
```

You can send objects that implement `Serialize`
and receive objects that implement `Deserialize`.

Channels by default use Bincode for serialization, but they support various other
formats such as JSON, BSON and Postcard.


