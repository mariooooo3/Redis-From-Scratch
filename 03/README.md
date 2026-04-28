# Chapter 03 — TCP Server and Client

> Build Your Own Redis with C/C++ — chapter 03

A minimal TCP server and client using the BSD socket API. Intentionally simple — no error handling, no protocol, one client at a time.

## Concepts

- TCP socket lifecycle: `socket()` → `bind()` → `listen()` → `accept()` → `read/write` → `close()`
- `SO_REUSEADDR` — prevents "Address already in use" on server restart
- Network byte order (big-endian) and `htons()`/`htonl()` conversions
- `struct sockaddr_in` and the cast-to-`struct sockaddr *` pattern

## Build & run

```bash
g++ -Wall -Wextra -O2 -g 03_server.cpp -o server
g++ -Wall -Wextra -O2 -g 03_client.cpp -o client

# terminal 1        # terminal 2
./server            ./client
# client says: hello    # server says: world
```
