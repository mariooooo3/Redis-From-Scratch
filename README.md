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
| [06](06/) | Event loop — poll(), non-blocking IO, per-connection state, pipelining | [06/](06/) |
| [07](07/) | Basic server — structured request encoding, GET / SET / DEL, status codes | [07/](07/) |

## Concepts

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
- Per-connection state (`struct Conn`) — `want_read`, `want_write`, `want_close`, `incoming`, `outgoing` buffers persist between loop iterations
- `fd2conn` — flat array indexed by fd; denser and faster than a hashtable because the kernel assigns fds as the smallest available integer
- Pipelined requests — a client can send N requests before reading any response; correct servers loop the parser to drain the whole input buffer, never assume at most 1 message per `read()`
- `EINTR` — a Unix signal during a blocking syscall causes it to return early; not an error, retry the call
- Optimistic write — in request-response, the socket is likely writable right after a request arrives; write immediately and handle `EAGAIN` rather than waiting for the next `poll()`
- Multi-string request encoding — `[nstr | len | str1 | len | str2 | ...]`; each argument is length-prefixed, the body starts with a 4-byte count
- `parse_req()` — cursor-based parser with `read_u32` / `read_str` helpers; rejects trailing garbage and enforces an argument count limit
- Status-coded response — `[4-byte len][4-byte status][data...]`; `RES_OK` (0), `RES_ERR` (1 unrecognized command), `RES_NX` (2 key not found)
- `do_request()` — command dispatch: matches the first argument against `"get"` / `"set"` / `"del"` with arity checks
- `std::map<std::string, std::string>` — O(log n) balanced BST used as the key-value store

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

# chapter 06
cd 06
g++ -Wall -Wextra -O2 -g 06_server.cpp -o server
g++ -Wall -Wextra -O2 -g 06_client.cpp -o client

# chapter 07
cd 07
g++ -Wall -Wextra -O2 -g 07_server.cpp -o server
g++ -Wall -Wextra -O2 -g 07_client.cpp -o client
```

Run `./server` in one terminal, `./client` in another.
