# Chapter 06 — Event Loop

> Build Your Own Redis with C/C++ — chapters 06 & 06b

The echo server from chapter 04 rewritten as a non-blocking event loop. A single thread handles many connections concurrently by waiting on all sockets with `poll()` and reacting only when a socket is ready.

## Concepts

- `poll()` / `struct pollfd` — readiness API; `POLLIN` readable, `POLLOUT` writable, `POLLERR` errors
- `O_NONBLOCK` + `EAGAIN` — non-blocking sockets return immediately instead of blocking the thread
- `struct Conn` — per-connection state: `want_read`, `want_write`, `want_close`, `incoming`/`outgoing` buffers
- `fd2conn` — flat `vector<Conn*>` indexed by fd; faster than a hashtable for kernel-assigned fds
- Pipelined requests — loop `try_one_request()` until false; never assume at most 1 message per `read()`
- Optimistic write — call `handle_write()` immediately after a request arrives, skip the next `poll()`

## Build & run

```bash
g++ -Wall -Wextra -O2 -g 06_server.cpp -o server
g++ -Wall -Wextra -O2 -g 06_client.cpp -o client

# terminal 1        # terminal 2
./server            ./client
```
