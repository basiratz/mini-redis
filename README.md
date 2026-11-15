# miniredis

A lightweight, Redis-compatible in-memory key-value server written in modern C++20.

## Goals

I'm building this to learn how a real network database works from the inside. My previous project was an LSM-tree key-value engine that ran as a library. This one turns the storage idea into a server that many clients can talk to at the same time.

What I want to learn and demonstrate:

- TCP socket programming with POSIX sockets
- Designing and parsing a wire protocol (RESP)
- Thread-safe data structures with `std::shared_mutex`
- Time-based key expiration (TTL)
- Crash recovery through an append-only file
- Testing and benchmarking with numbers, not guesses

## Planned Features

**Protocol and store**
- [ ] RESP parser that handles incomplete, fragmented, and pipelined input
- [ ] RESP response serializer
- [ ] In-memory store (hash table)
- [ ] Command dispatcher
- [ ] Commands: `PING`, `SET`, `GET`, `DEL`, `EXISTS`

**Networking and concurrency**
- [ ] TCP server with multiple simultaneous clients
- [ ] Thread pool for handling connections
- [ ] Thread-safe store with `std::shared_mutex`
- [ ] Graceful shutdown (SIGINT)
- [ ] Works with the real `redis-cli`

**Expiration and persistence**
- [ ] TTL: `EXPIRE`, `TTL`, `SET key value EX n`
- [ ] Lazy expiration (check on access)
- [ ] Active expiration (background sweeper thread)
- [ ] Append-only file (AOF) with replay on startup

**Quality and performance**
- [ ] Unit tests (GoogleTest)
- [ ] Clean runs under AddressSanitizer and ThreadSanitizer
- [ ] Load generator with throughput and p50/p99 latency
- [ ] Comparison against `redis-benchmark`
- [ ] GitHub Actions CI

**Stretch goals**
- [ ] `INCR`, `KEYS`
- [ ] Sharded locks to reduce contention
- [ ] `epoll`-based event loop

## Architecture

*Diagram coming soon.*

Each layer only knows about the layer below it:

```
Client  --TCP-->  Server  -->  RESP Parser  -->  Command Dispatcher  -->  Store
                                                                    -->  AOF (later)
Client  <--TCP--  Server  <--  RESP Serializer  <--  Response
```

| Layer | Responsibility |
|---|---|
| `server` | Sockets, accepting clients, per-client read buffers |
| `protocol` | Turning bytes into commands, and replies into bytes |
| `commands` | Mapping a command name to its handler |
| `store` | Key-value storage, TTL, locking |
| `persistence` | Writing and replaying the append-only file |

## Build and Run

Requirements: a C++20 compiler (GCC 11+ or Clang 14+), CMake 3.20+, Linux or WSL2.

```bash
cmake -S . -B build
cmake --build build -j
ctest --test-dir build --output-on-failure
```

Sanitizer builds (they can't be combined in one build):

```bash
cmake -S . -B build-asan -DENABLE_ASAN=ON
cmake --build build-asan -j
ctest --test-dir build-asan --output-on-failure

cmake -S . -B build-tsan -DENABLE_TSAN=ON
cmake --build build-tsan -j
ctest --test-dir build-tsan --output-on-failure
```

Running the server (available after Week 2):

```bash
./build/miniredis --port 6379
redis-cli -p 6379
```

## Project Structure

```
src/
  protocol/    RESP parser and serializer
  store/       key-value store
  commands/    command dispatcher and handlers
tests/         unit tests
docs/          design notes and benchmark results
```

## Status

**Week 1:** protocol and store in progress (no networking yet).

## Known Limitations

- Linux/POSIX only; no Windows support
- Supports only a small subset of Redis commands
- Single-node only: no replication or clustering
- No authentication or TLS
- Persistence guarantees depend on the chosen fsync policy

*This list will grow as I learn where the design falls short.*

## Design Notes

Decisions and tradeoffs will be written up in `docs/DESIGN.md`.

## License

MIT