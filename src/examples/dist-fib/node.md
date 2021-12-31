# Node

The protocol defines the node as a service that computes
fibonacci numbers.

Implementing this in SRPC looks like this:
```rust
#[srpc::rpc]
struct Node;

#[srpc::rpc]
impl Node {
    async fn fibonacci(&self, number: u32) -> u32 {
        fibonacci(number)
    }
}

#[inline(always)]
fn fibonacci(number: u32) -> u32 {
    let mut a = 0;
    let mut b = 1;
    for _ in 0..number {
        let tmp = a;
        a = b;
        b = a + tmp;
    }
    b
}
```

The macro automatically generates a peer and a service implementation.
You can use `&mut self` and `&self` in rpc methods. RPC methods by default
use a RwLock, but one can use a Mutex (`#[srpc::rpc(mutex)]`) instead or nothing (`#[srpc::rpc(none)]`), but that means that
you'll only have read-only access to the struct.
