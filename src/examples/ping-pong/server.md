# Server

Now that we have a service that represents our ping-pong server,
we want to expose it.

To expose a service we have to do two things:
- Add the service to a route
- Expose the route

You can add the service to a route(in this case the global route) like this:
```rust , no_run
//                        name of service | location of service
GLOBAL_ROUTE.add_service_default_at::<ping_service>("ping")?;
```
To expose the route we have to choose a provider,
after choosing a provider, we can expose the route like this:
```rust , no_run
Tcp::bind("127.0.0.1:8080").await?;
```
