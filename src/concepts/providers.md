# Providers

Providers are the most way to get channels.
You can bind them and then simply iterate over their Channels.

```rust , no_run
let tcp = Tcp::bind("127.0.0.1:8080").await?;
while let Ok(chan) = tcp.next().await {
    let mut chan = chan.encrypted().await?; // choose if the channel should be encrypted
    chan.send("hello!").await?;
}
```

There are also addresses which encapsulate providers.
```rust , no_run
let addr = "tcp@127.0.0.1:8080".parse::<Addr>()?;
let mut chan = addr.connect().await?;
chan.send("hello!").await?;
```

At the moment, Canary supports the following providers:
- TCP        (works on non-wasm platforms)
- Unix       (works on unix platforms)
- WebSockets (works on all platforms)
