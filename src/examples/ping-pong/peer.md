# Peer

After exposing our service, we can access it with the following:
```rust , no_run
// get a channel from "127.0.0.1:8080" with the "ping" id
let mut channel = Tcp::connect("127.0.0.1:8080", "ping").await?;
channel.send("Ping!").await?;
let pong: String = channel.receive().await?;
println!("received {pong}"); // received Pong!
```
