# Service

Now that we have the following channel representation:
```
receive:
    1 String,
send:
    2 String,
```
We can make a service:

```rust , no_run
#[service]
async fn ping_service(mut channel: Channel) -> Result<()> {
    // receive 1, type String
    let ping: String = channel.receive().await?;
    println!("received {ping}");

    // send 2, type String. str and String have equivalent wire representations,
    // so this is safe.
    channel.send("Pong!").await?;
    Ok(())
}
```

