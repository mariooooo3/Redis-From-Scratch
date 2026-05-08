# Chapter 06 — Event Loop

> Build Your Own Redis with C/C++ — chapters 06 & 06b

The echo server from chapter 04 rewritten as a non-blocking event loop. A single thread handles many connections concurrently by waiting on all sockets with `poll()` and reacting only when a socket is ready. The chapter has two parts that share the same source files.

## Concepts

### Part 1 — The event loop

- `poll()` / `struct pollfd` — readiness API; `POLLIN` for readable, `POLLOUT` for writable, `POLLERR` for errors; `events` is the request, `revents` is the result
- `O_NONBLOCK` + `fcntl()` — marks a socket so `read()`/`write()` return `EAGAIN` immediately instead of blocking the thread
- `EINTR` — a Unix signal during a blocking syscall makes it return early with `EINTR`; not an error, just retry `poll()`
- `struct Conn` — per-connection state that persists between loop iterations: `want_read`, `want_write`, `want_close`, `incoming` buffer, `outgoing` buffer
- `fd2conn` — flat `vector<Conn*>` indexed by fd; denser and faster than a hashtable because the kernel allocates fds as the smallest available integer
- Input buffering — `read()` is non-blocking so data arrives in fragments; accumulate into `Conn::incoming` and parse only when enough bytes have arrived
- Output buffering — responses are staged in `Conn::outgoing` and drained across multiple loop iterations if the socket is not ready
- `try_one_request()` — protocol parser; returns `false` if there is not enough data, `true` after consuming one complete message from `incoming`
- State machine — `want_read = true` → waiting for a request; `want_write = true` → flushing a response; transitions happen at the end of each callback

### Part 2 — Pipelining & optimisations

- Pipelined requests — a client sends N requests before reading any response; a single `read()` can return multiple messages at once
- Treating input as a byte stream — loop `try_one_request()` until it returns `false`; never assume at most 1 message arrived per `read()`; clearing the whole buffer after 1 message is a bug
- Optimistic write — in a request-response protocol the socket is likely writable right after a request arrives; call `handle_write()` immediately and skip the next `poll()`, saving one syscall per request
- `EAGAIN` on optimistic write — the assumption can fail with a pipelined client whose read buffer is full; `handle_write()` must check `EAGAIN` and return without error
- `buf_append` / `buf_consume` — append to back is amortised O(1); remove from front shifts all remaining data, making it O(N) per call and O(N²) overall when draining many pipelined messages
- Better buffer — using an offset pointer or a ring buffer makes `buf_consume` O(1)

## Build & run

```bash
g++ -Wall -Wextra -O2 -g 06_server.cpp -o server
g++ -Wall -Wextra -O2 -g 06_client.cpp -o client

# terminal 1        # terminal 2
./server            ./client
# client sends 3 small messages + 1 large (32 MiB) + 1 more, all pipelined
# server echoes each back; the large message spans multiple loop iterations
```
