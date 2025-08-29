
# Redis Server
## Task
This is a lightweight Redis-compatible in-memory data store written in C++. It supports strings, lists, and hashes, along with the full complete implementation of the Redis Serialization Protocol (RESP) parsing, multi-client concurrency, and periodic disk persistence without any external dependencies.

## Functioning

- **Common Commands:** PING, ECHO, FLUSHALL
- **Key/Value:** SET, GET, KEYS, TYPE, DEL/UNLINK, EXPIRE, RENAME
- **List:** LGET, LLEN, LPUSH/RPUSH (multi-element), LPOP/RPOP, LREM, LINDEX, LSET
- **Hash:** HSET, HGET, HEXISTS, HDEL, HKEYS, HVALS, HLEN, HGETALL, HMSET

Data is automatically dumped to `dump.my_rdb` every 300 seconds and on graceful shutdown; at startup, the server attempts to load from this file.

## Installation
This project uses a simple Makefile. Ensure you have a C++17 (or later) compiler.
- `make`
- `make clean`

```bash
make
```

Or compile manually:
```bash
g++ -std=c++17 -pthread -Iinclude src/*.cpp -o my_redis_server
```

## Usage
After compiling the server, you can run it and use with client.

### Running the Server

Start the server on the default port (6379) or specify a port:

```bash
./my_redis_server            # listens on 6379
./my_redis_server 6380       # listens on 6380
```

To gracefully shutdown and persist immediately, press `Ctrl+C`.

### Using the Server
You can connect with the standard `redis-cli` or custom RESP client `my_redis_cli`.

```bash
redis-cli -p 6379
```

## Design & Architecture

- **Concurrency:** Each client is handled in its own `std::thread`.  
- **Synchronization:** A single `std::mutex db_mutex` guards all in-memory stores.  
- **Data Stores:**  
  - `kv_store` (`unordered_map<string,string>`) for strings  
  - `list_store` (`unordered_map<string,vector<string>>`) for lists  
  - `hash_store` (`unordered_map<string,unordered_map<string,string>>`) for hashes
- **Expiration:** Lazy eviction on each access via `purgeExpired()`, plus TTL map `expiry_map`.  
- **Persistence:** Simplified RDB: text‐based dump/load in `dump.my_rdb`.  
- **Singleton Pattern:** `RedisDatabase::getInstance()` enforces one shared instance.  
- **RESP Parsing:** Custom parser in `RedisCommandHandler` supports both inline and array formats.
