# Structure

For the ping-pong server, we will need:
- A ping-pong service
- A provider to expose our service

The ping-pong service has the following channel representation:
```
receive String -> send String
```

and the following wire representation:
```
receive:
    1 String,
send:
    2 String,
```

