# Build your own redis

## Respond to ping

- Ping msg: `PING <msg>`
- returns `PONG` if no arg is passed
- used to test if connection is alive, if server can serve data and measure latency

```
redis> PING
"PONG"
redis> PING "hello world"
"hello world"
```

- Our response must be encoded using `RESP -> Redis Serialization Protocol`
    - This protocol is what Redis clients use to interact with redis

- Adv of RESP:
    - simple
    - fast to parse
    - human readable

- `\r\n` (CRLF) is the protocol's terminator -> separates its parts

### Simple strings

- Encoded as `+` character followed by string
- mustnt contain `\r` or `\n` and is terminated by `\r\n`

```
+OK\r\n

+PONG\r\n
```

### Simple errors

- Instead of `+` we use `-`

```
-Error msg\r\n
```


