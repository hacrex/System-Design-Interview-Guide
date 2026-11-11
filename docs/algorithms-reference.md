# Algorithms reference

Short, language-agnostic pseudocode for the algorithms that come up in system design interviews. Use these as a starting point when an interviewer asks "how would you actually implement that?" — adapt to your language of choice.

## 1. Token bucket rate limiter

Module 05 — rate limiting. Allows bursts up to `capacity`, refills at `refill_rate` tokens per second.

```
function allow(client_id):
    now = current_time()
    bucket = load(client_id)        # { tokens, last_refill }
    
    # Refill tokens since last request
    elapsed = now - bucket.last_refill
    bucket.tokens = min(capacity, bucket.tokens + elapsed * refill_rate)
    bucket.last_refill = now
    
    if bucket.tokens >= 1:
        bucket.tokens = bucket.tokens - 1
        save(client_id, bucket)
        return ALLOW
    else:
        return REJECT_429
```

Run the whole `allow` as a single Redis Lua script (or equivalent atomic transaction) so the read-compute-write cannot interleave across app servers.

Key fields to store per client: `tokens` (float) and `last_refill` (timestamp). Two values → memory light.

---

## 2. Consistent hashing ring with virtual nodes

Module 02 — consistent hashing. Place nodes and keys on a ring; a key belongs to the next node clockwise. Adding/removing a node moves only the keys next to it.

```
class ConsistentHashRing:
    # ring maps hash position -> node
    ring = SortedMap()
    # virtual_nodes_per_node controls how many ring positions each physical node owns
    
    function add_node(node):
        for i in 0 .. virtual_nodes_per_node:
            position = hash(node.name + "#" + i)
            ring[position] = node
    
    function remove_node(node):
        for i in 0 .. virtual_nodes_per_node:
            position = hash(node.name + "#" + i)
            ring.remove(position)
    
    function get_node(key):
        position = hash(key)
        # Find the next node clockwise (smallest position >= key position)
        node_entry = ring.ceiling_entry(position)
        if node_entry is null:
            node_entry = ring.first_entry()   # wrap around
        return node_entry.value
```

Use `virtual_nodes_per_node` in the dozens to hundreds so load evens out and larger machines can own more positions.

---

## 3. LRU cache eviction

Module 01 — caching. Evict the least recently used item when the cache is full.

```
class LRUCache:
    capacity = N
    map = HashMap(key -> node)        # O(1) lookup
    list = DoublyLinkedList()         # most-recent at head, least-recent at tail
    
    function get(key):
        if key not in map:
            return MISS
        node = map[key]
        list.move_to_head(node)       # mark as most recently used
        return node.value
    
    function put(key, value):
        if key in map:
            node = map[key]
            node.value = value
            list.move_to_head(node)
            return
        
        node = new Node(key, value)
        map[key] = node
        list.add_to_head(node)
        
        if list.size > capacity:
            # Evict least recently used (tail)
            lru = list.remove_tail()
            map.remove(lru.key)
```

Combined with a hash map, both `get` and `put` are O(1).

---

## 4. Exponential backoff with jitter

Module 05 — failure handling. Retry a failed call, waiting progressively longer, with random jitter to prevent retry storms.

```
function call_with_retry(operation, max_attempts = 5, base_delay_ms = 100, max_delay_ms = 30000):
    attempt = 0
    while true:
        try:
            return operation()        # success
        except RetryableError as e:
            attempt = attempt + 1
            if attempt >= max_attempts:
                throw e               # give up
            
            # Exponential backoff: base * 2^(attempt-1), capped
            delay = min(base_delay_ms * 2^(attempt-1), max_delay_ms)
            
            # Jitter: randomize within [0, delay] so clients don't retry in lockstep
            sleep(random(0, delay))
        except NonRetryableError:
            throw e                   # do not retry (e.g. 400 Bad Request)
```

Only retry idempotent or safe operations. Cap attempts so a transient failure doesn't turn into a permanent outage.

---

## 5. Leader election heartbeat (simplified Raft-style)

Module 03 — leader election. Followers start an election if they don't hear from the leader within a timeout.

```
state = FOLLOWER
current_term = 0
voted_for = null
election_timeout = random(150ms, 300ms)
last_heartbeat = now

loop:
    if state == FOLLOWER:
        if now - last_heartbeat > election_timeout:
            state = CANDIDATE
    
    if state == CANDIDATE:
        current_term = current_term + 1
        voted_for = self
        votes = 1 (self)
        
        for peer in peers:
            if peer.request_vote(current_term, self.id):
                votes = votes + 1
        
        if votes > majority(peers):
            state = LEADER
            last_heartbeat = now
    
    if state == LEADER:
        for peer in peers:
            peer.append_entries(current_term, self.id, log_entries)
        sleep(heartbeat_interval)     # e.g. 50ms
        last_heartbeat = now
    
    # On RPC from a leader with a higher term:
    #   current_term = rpc.term; state = FOLLOWER; voted_for = null
```

A cluster of `2f + 1` nodes survives `f` failures because a majority must agree. Terms prevent split brain; fencing tokens stop an old leader from writing after it lost leadership.

---

## 6. Circuit breaker

Module 05 — failure handling. States: closed → open → half-open → closed.

```
states = { CLOSED, OPEN, HALF_OPEN }
state = CLOSED
failure_count = 0
last_failure_time = null

function call(dependency):
    if state == OPEN:
        if now - last_failure_time > cooldown:
            state = HALF_OPEN
        else:
            return FAIL_FAST          # protect the dependency
    
    try:
        result = dependency.call()
        if state == HALF_OPEN:
            state = CLOSED            # recovery confirmed
            failure_count = 0
        return result
    except Failure:
        failure_count = failure_count + 1
        last_failure_time = now
        
        if state == HALF_OPEN or failure_count >= threshold:
            state = OPEN
        raise
```

Closed: requests flow normally. Open: fail fast. Half-open: trial requests test recovery.

---

## How to use in an interview

- Sketch the algorithm on the whiteboard; you don't need perfect syntax.
- State the tradeoff: e.g., "LRU is O(1) but needs a hash map plus a doubly linked list; that's more code than a simple FIFO queue."
- Mention where state lives: Redis for distributed counters/buckets, in-memory for per-process caches.

Related: see [module 05](05-reliability-and-protection.md) for rate limiting and failure handling, [module 02](02-data-storage.md) for consistent hashing, [module 03](03-consistency-and-distributed-data.md) for leader election.
