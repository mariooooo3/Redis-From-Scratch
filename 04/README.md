# Chapter 04 — Protocol: Length-Prefixed Messages

> Build Your Own Redis with C/C++ — chapter 04

Same TCP server and client as chapter 03, extended with a simple binary protocol. Multiple requests are sent over a single connection using a 4-byte length header to frame each message.

## Concepts

- Length-prefixed framing: `[4-byte len][body]` — solves the TCP stream boundary problem
- `read_full()` — loops until exactly `n` bytes are read, handles partial `read()` returns
- `write_all()` — loops until exactly `n` bytes are written, handles partial `write()` returns
- Multiple requests per connection — server loops on `one_request()` until error or EOF
- `k_max_msg = 4096` — explicit message size cap to reject oversized input

## Build & run

```bash
g++ -Wall -Wextra -O2 -g 04_server.cpp -o server
g++ -Wall -Wextra -O2 -g 04_client.cpp -o client

# terminal 1        # terminal 2
./server            ./client
# server says: client says: hello1
#              client says: hello2
#              client says: hello3
```
