
# Wire Representation


If you don't care about efficiency or serialization, skip this section.
Wire representation is how your protocol looks like on the wire.

It's important to know about the wire representation and channel representation
of the protocols you build on top of Canary, since ALL distributed systems and
communications have a channel representation (what is sent through the wire and what is
received through the wire in order) and a wire representation (what is sent through
the wire and received through the wire but separated) even those that aren't built on top of Canary.

Wire and channel representations also help debug distributed systems and also help know which types are equivalent on the
wire ( e.g. &str and String have an equivalent wire representation ).

The differences between the wire representation and channel representation is that
the wire representation is divided into inbound (receiving) wire representation and
outbound (sending) wire representation.

An example wire representation:
```
send:
    # position 1
    1 u16,
    # position 3
    3 u16,
receive:
    # position 2
    2 u16,
```

This outbound wire representation looks like this on the wire:
```
(outbound)
    1   3
-> 0x0 0x0

(inbound)
    2
<- 0x0
```

Common wire-equivalent types are:
- `Vec<T>` and `[T]`
- `String` and `str`
- `u16` and `[u8; 2]`
- `u32` and `[u8; 4]`
- `u64` and `[u8; 8]`

Another example of a wire representation:
```
send:
    1 [u8; 2],
    2 String,

receive:
    3 Vec<u16>,
    4 u16
```

Astute readers may have noticed that we run into a problem when trying to
represent this wire representation:
What about `String` and `Vec<u8>`?
They are dynamically sized, so their length is sent first (as an u64),
and then they're serialized and sent, so they look like this:

```
->
    1 # [u8; 2]
    0x0 0x0

    2 # length of serialized object
    0x0 0x0 0x0 0x0

    # a unit struct represents a dynamically sized object
    ()

<-
    3 # Vec<u16>, dynamically sized
    0x0 0x0 0x0 0x0 ()

    4 0x0
```

A condensed version of this wire representation looks like this:
```
->
    1 0x0 0x0
    2 0x0 0x0 0x0 0x0 ()
<-
    3 0x0 0x0 0x0 0x0 ()
    4 0x0
```

Currently due to simplicity, IGCP sends all objects (even wire statically-sized-types) as dynamically sized.
This means that there is space for improvement, but it is still sufficiently efficient for most use cases.

NOTE ABOUT TYPES:
Sending and receiving non-equivalent wire represented types can lead to messaging problems that can be hard to debug.
