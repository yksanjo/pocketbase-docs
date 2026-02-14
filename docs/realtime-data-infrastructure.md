# Real-Time Data Infrastructure

PocketBase provides a comprehensive real-time data infrastructure built on WebSocket technology, enabling bidirectional event-driven communication with production-ready connection management.

## 2.2.1 WebSocket-Based Live Data Synchronization

PocketBase's WebSocket implementation enables bidirectional, event-driven communication with production-ready connection management.

### Connection Management Features

- **Automatic reconnection** with exponential backoff
- **Heartbeat/ping handling** for connection health detection
- **Backpressure management** to prevent memory exhaustion from slow consumers
- **Graceful degradation** to polling for restrictive proxy environments

### Subscription Protocol

The subscription protocol implements topic-based routing with server-side filtering:

- Clients subscribe to patterns (e.g., `posts.*`, `posts.record123`)
- Receive only events matching their authorization scope and filter criteria
- Achieves **10,000+ concurrent WebSocket connections** on modest single-server deployments (4 vCPU, 8GB RAM)

### WebSocket Endpoint

```
wss://your-domain.com/api/collections/{collection_name}/records
```

---

## 2.2.2 Event-Driven Subscription Model for Frontend Reactivity

The event-driven architecture extends to server-side hooks enabling custom business logic injection.

### Event Categories

| Event Family | Trigger Points |
|--------------|----------------|
| OnModel* | BeforeCreate, AfterCreate, BeforeUpdate, AfterUpdate, BeforeDelete, AfterDelete |
| OnRecordAuth* | Authentication requests, OAuth flows, password resets |
| OnFile* | Upload, download, token generation |
| OnMailer* | Email sending for verification, reset, notifications |

### Hook Implementation Details

Hook implementations execute within the same database transaction as triggering operations, ensuring atomicity between custom logic and data persistence.

### Frontend SDK Features

Frontend SDKs provide:
- **Optimistic update support** with automatic rollback on rejection
- Sub-100ms perceived performance for user interactions
- Reactive subscriptions with automatic state synchronization

---

## 2.2.3 Write-Ahead Logging (WAL) Mode for SQLite Performance Optimization

The WAL mode technical advantages merit detailed examination.

### WAL Mode Characteristics

| Characteristic | WAL Mode Benefit |
|----------------|------------------|
| Read concurrency | Unlimited readers during writes (vs. single-writer lock in rollback journal) |
| Write performance | Sequential append to WAL file (vs. random database page writes) |
| Crash recovery | WAL replay only (vs. full database scan for incomplete transactions) |
| Checkpointing | Automatic background merge of WAL to main database |

### Performance Claims

> "For the majority of queries, SQLite in WAL mode outperforms traditional databases like MySQL, MariaDB, or PostgreSQL, especially when handling read operations."

While workload-specific validation is advised, the architectural advantages for read-heavy, real-time applications are theoretically sound and empirically supported by demonstrated connection handling capacity.

### Configuration

WAL mode is enabled by default in PocketBase. The configuration provides:
- Optimal performance for real-time workloads
- High concurrency read access
- Improved crash recovery
- Sequential write performance

---

## Real-Time Subscription Example

### JavaScript Client

```javascript
import PocketBase from 'pocketbase';

const pb = new PocketBase('http://127.0.0.1:8090');

// Subscribe to changes in a collection
await pb.collection('posts').subscribe('*', function (e) {
    console.log(e.action); // 'create', 'update', or 'delete'
    console.log(e.record); // the changed record
});

// Subscribe to specific record
await pb.collection('posts').subscribe('post123', function (e) {
    console.log(e.record);
});

// Unsubscribe
await pb.collection('posts').unsubscribe();
```

### Filtered Subscriptions

```javascript
// Subscribe only to records matching a filter
await pb.collection('posts').subscribe('*', {
    filter: 'status = "published"'
}, function (e) {
    console.log(e.record);
});
```
