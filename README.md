# Build Your Own Redis with C/C++

> Following the book by James Smith — build-your-own.org

A Redis-like in-memory key-value store built from scratch in C++, chapter by chapter. No external libraries — raw BSD sockets, manual protocol parsing, and hand-written data structures.

## Chapters

| # | Topic | Code |
|---|-------|------|
| 01 | Introduction — why Redis, why C/C++ | — |
| 02 | Socket programming theory — TCP/IP layers, socket primitives | — |
| [03](03/) | TCP server & client — first working connection | [03/](03/) |
| [04](04/) | Request-response protocol — length-prefixed binary framing | [04/](04/) |
| 05 | Concurrent IO models — event loop, non-blocking IO, poll() | — |

## Concepts covered so far

- TCP socket lifecycle: `socket()` → `bind()` → `listen()` → `accept()` → `read/write` → `close()`
- `SO_REUSEADDR` — prevents "Address already in use" on server restart
- Network byte order and `htons()`/`htonl()` conversions
- Why `read()`/`write()` can return less than requested on TCP — and how to handle it
- Length-prefixed framing: `[4-byte len][body]` — the foundation of binary protocols
- `read_full()` / `write_all()` — correct loops for partial reads and writes
- Multiple requests over a single connection
- Event loop — a single thread waits on all sockets with `poll()`, then reads/writes only the ones that are ready
- `O_NONBLOCK` + `EAGAIN` — non-blocking reads/writes return immediately instead of suspending the thread
- `poll()` / `epoll` — readiness APIs; `epoll` is preferred in production for scalability; disk files are not supported

## Build & run

Each chapter is self-contained. See the README inside each folder:

```bash
# chapter 03
cd 03
g++ -Wall -Wextra -O2 -g 03_server.cpp -o server
g++ -Wall -Wextra -O2 -g 03_client.cpp -o client

# chapter 04
cd 04
g++ -Wall -Wextra -O2 -g 04_server.cpp -o server
g++ -Wall -Wextra -O2 -g 04_client.cpp -o client
```

Run `./server` in one terminal, `./client` in another.
