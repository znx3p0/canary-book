# Structure

We'll start by defining how our distributed fibonacci calculator
looks like on the conceptual level.

This distributed fibonacci calculator will have a master server
and computing nodes connected to it, requests will be sent to
the master server and the master server will ask a computing
node to compute fibonacci, send it back and finally send the
result to the peer.

The channel representation of this protocol should be the following:
```
enum Kind {
    Client, Server
}

top:
    1 receive Kind,

    match 1 {
        Server => server,
        client => client,
    }

server:
    1 receive u32
    2 send u32

client:
    1 send u32
    2 receive u32
```

the channel representation is from the perspective of the cluster,
so there are three possible kinds of nodes in this distributed system:
- Cluster
- Node
- Client

The cluster receives requests from clients,
the nodes compute the requests and clients send
requests to the cluster.
