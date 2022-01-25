# Examples

To prepare yourself to build custom distributed systems,
you will have to build a set of distributed systems with Canary.

Some examples have been prepared for you to build along,
and a final challenge for you to try building individually.

The examples you have to build are the following (in order):
- A ping-pong server.
- A distributed fibonacci calculator.
- A shollz/croc replica.

If you still don't understand how things work, don't be afraid to take a look
at the code, since the codebase is relatively small; Canary is implemented in ~1600 sloc (most of which is boilerplate),
and igcp (the library that provides channels) is implemented in ~1400 sloc.

You can also take a look at the code from [SailDB](https://github.com/znx3p0/saildb), a key-value
in-memory database designed to be extremely simple which uses SRPC as a backend.
