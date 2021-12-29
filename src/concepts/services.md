# Services

Services are functions that are stored on a route.

They take two parameters: metadata and a channel.
They also return a result, but failures are silently logged.

A simple service looks like this:
```rust , no_run
use canary::{service, Channel, Result};

#[service]
async fn my_service(channel: Channel) -> Result<()> {
    Ok(())
}
```

Although barebones services might be enough for some use cases,
most use cases need to have context with the service (e.g. atomic counter, etc.)

For example, let's say Alice needs to build a counter service.
The service must count every call to the service and send back the current number.
A service like that could be implemented like this:
```rust , no_run
use canary::{service, Channel, Result};
use std::sync::atomic::AtomicU64;
use std::sync::atomic::Ordering;
use std::sync::Arc;

#[service]
async fn counter_service(counter: Arc<AtomicU64>, mut peer: Channel) -> Result<()> {
    let current_val = counter.fetch_add(1, Ordering::Relaxed);
    peer.send(current_val).await?;
    Ok(())
}
```

These services can be registered on a route and exposed through a provider,
and they are designed to be embarrasingly parallel.

It's also important to know how these services get expanded, since the service macro does not cover all
use cases of services, for example, the macros in `SRPC` don't expand to service macros for generic support.

A simple service
```rust , no_run
#[service]
async fn get_id(counter: Arc<u64>, mut peer: Channel) -> Result<()> {
    peer.send(counter).await?;
    Ok(())
}
```

expands to this
```rust , no_run
#[allow(non_camel_case_types)]
struct get_id;
impl ::canary::service::Service for get_id {
    const ENDPOINT: &'static str = "get_id";
    type Pipeline = ();
    type Meta = Arc<u64>;
    fn service(
        __canary_inner_meta: Arc<u64>,
    ) -> Box<dyn Fn(::canary::igcp::BareChannel) + Send + Sync + 'static> {
        async fn get_id(counter: Arc<u64>, mut peer: Channel) -> Result<()> {
            peer.send(counter).await?;
            Ok(())
        }
        ::canary::service::run_metadata(__canary_inner_meta, get_id)
    }
}
```

and a simple SRPC implementation
```rust , no_run
#[srpc::rpc]
struct Counter {
    counter: u64
}

#[srpc::rpc]
impl Counter {
    async fn increase(&mut self) -> u64 {
        self.counter += 1;
        self.counter
    }
}
```

expands to this

```rust

struct Counter {
    counter: u64,
}
impl ::srpc::canary::routes::RegisterEndpoint for Counter {
    const ENDPOINT: &'static str = "counter";
}
struct CounterPeer(pub ::srpc::canary::Channel, ::core::marker::PhantomData<()>);
impl From<::srpc::canary::Channel> for CounterPeer {
    fn from(c: ::srpc::canary::Channel) -> Self {
        CounterPeer(c, ::core::marker::PhantomData::default())
    }
}
impl ::srpc::Peer for Counter {
    type Struct = CounterPeer;
}
const _: () = {
    impl Counter {
        async fn increase(&mut self) -> u64 {
            self.counter += 1;
            self.counter
        }
    }
    #[allow(non_camel_case_types)]
    #[derive(Serialize_repr, Deserialize_repr)]
    #[repr(u8)]
    enum __srpc_action {
        increase,
    }
    impl ::srpc::canary::service::Service for Counter {
        const ENDPOINT: &'static str = "counter";
        type Pipeline = ();
        type Meta = ::std::sync::Arc<::srpc::RwLock<Counter>>;
        fn service(
            __srpc_inner_meta: ::std::sync::Arc<::srpc::RwLock<Counter>>,
        ) -> Box<dyn Fn(::srpc::canary::igcp::BareChannel) + Send + Sync + 'static> {
            ::canary::service::run_metadata(
                __srpc_inner_meta,
                |__srpc_inner_meta: ::std::sync::Arc<::srpc::RwLock<Counter>>,
                 mut __srpc_inner_channel: ::srpc::canary::Channel| async move {
                    loop {
                        match __srpc_inner_channel.receive::<__srpc_action>().await? {
                            __srpc_action::increase => {
                                let res = __srpc_inner_meta.write().await.increase().await;
                                __srpc_inner_channel.send(res).await?;
                            }
                        }
                    }
                },
            )
        }
    }
    impl ::srpc::canary::service::StaticService for Counter {
        type Meta = ::std::sync::Arc<::srpc::RwLock<Counter>>;
        type Chan = ::srpc::canary::Channel;
        fn introduce(
            __srpc_inner_meta: ::std::sync::Arc<::srpc::RwLock<Counter>>,
            mut __srpc_inner_channel: ::srpc::canary::Channel,
        ) -> ::srpc::canary::runtime::JoinHandle<::srpc::canary::Result<()>> {
            ::srpc::canary::runtime::spawn(async move {
                loop {
                    match __srpc_inner_channel.receive::<__srpc_action>().await? {
                        __srpc_action::increase => {
                            let res = __srpc_inner_meta.write().await.increase().await;
                            __srpc_inner_channel.send(res).await?;
                        }
                    }
                }
            })
        }
    }
    impl CounterPeer {
        pub async fn increase(&mut self) -> ::srpc::canary::Result<u64> {
            self.0.send(__srpc_action::increase).await?;
            self.0.receive().await
        }
    }
};
```

