# Chapter 07 — Basic Server: GET, SET, DEL

> Build Your Own Redis with C/C++ — chapter 07

The event loop from chapter 06 extended with real Redis-like commands. The server parses structured multi-string requests, dispatches `get`/`set`/`del` against an in-memory store, and returns a status-coded response.

## Concepts

- Multi-string request format — `[nstr | len | str1 | len | str2 | ...]`; 4-byte count followed by length-prefixed arguments
- `parse_req()` — cursor-based parser using `read_u32` / `read_str` helpers; rejects trailing garbage and enforces `k_max_args`
- Response format — `[4-byte len][4-byte status][data...]`; status `RES_OK` (0), `RES_ERR` (1), `RES_NX` (2 key not found)
- `do_request()` — dispatches on `cmd[0]`; `get` returns the value or `RES_NX`, `set` inserts/overwrites, `del` erases
- `std::map<std::string, std::string> g_data` — O(log n) BST used as the initial key-value store

## Build & run

```bash
g++ -Wall -Wextra -O2 -g 07_server.cpp -o server
g++ -Wall -Wextra -O2 -g 07_client.cpp -o client

# terminal 1        # terminal 2
./server            ./client set foo bar   # server says: [0]
                    ./client get foo       # server says: [0] bar
                    ./client get missing   # server says: [2]
                    ./client del foo       # server says: [0]
```
