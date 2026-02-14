# API and Integration Layer

PocketBase provides a comprehensive RESTful API with official SDKs and extensibility options for custom business logic.

## 2.4.1 RESTful API Design with Predictable Endpoints

PocketBase's REST API follows consistent conventions for easy integration.

### API Endpoints

| Pattern | Endpoint | Purpose |
|---------|----------|---------|
| Collection operations | `GET /api/collections/{name}/records` | List with filter, sort, pagination |
| Create record | `POST /api/collections/{name}/records` | Create new record |
| Record operations | `GET /api/collections/{name}/records/{id}` | Retrieve single record |
| Update record | `PATCH /api/collections/{name}/records/{id}` | Partial update |
| Delete record | `DELETE /api/collections/{name}/records/{id}` | Remove record |
| Real-time | `wss://.../api/collections/{name}/records` | WebSocket subscription |

### Query Capabilities

- **Filter predicates**: Advanced filtering with logical operators
- **Multi-field sorting**: Sort by multiple fields in specified order
- **Relation expansion**: `expand=author,comments`
- **Field projection**: `fields=id,title`
- **Cursor-based pagination**: Efficient pagination for large datasets

### Example API Calls

```bash
# List records with filtering
curl -X GET "http://127.0.0.1:8090/api/collections/posts/records?filter=status='published'&sort=-created"

# Create a record
curl -X POST "http://127.0.0.1:8090/api/collections/posts/records" \
  -H "Content-Type: application/json" \
  -d '{"title": "My Post", "content": "Hello World"}'

# Update a record
curl -X PATCH "http://127.0.0.1:8090/api/collections/posts/records/RECORD_ID" \
  -H "Content-Type: application/json" \
  -d '{"title": "Updated Title"}'

# Delete a record
curl -X DELETE "http://127.0.0.1:8090/api/collections/posts/records/RECORD_ID"
```

---

## 2.4.2 Official JavaScript SDK and Dart SDK for Cross-Platform Development

### SDK Comparison

| SDK | Platform Coverage | Key Features |
|-----|------------------|--------------|
| JavaScript/TypeScript | Browser, Node.js, React Native, Electron | 2,800+ GitHub stars; reactive subscriptions, auth state management, file upload progress |
| Dart/Flutter | iOS, Android, Web, Desktop | 680+ GitHub stars; native async/await, offline support foundation |

### SDK Features

Both SDKs provide:
- **Automatic token refresh**
- **Request deduplication**
- **Optimistic updates**
- **Framework-specific integrations** (React/Vue/Svelte reactivity adapters)

### JavaScript SDK Usage

```javascript
import PocketBase from 'pocketbase';

const pb = new PocketBase('http://127.0.0.1:8090');

// Authentication
await pb.collection('users').authWithPassword('test@example.com', 'password');

// Create a record
const record = await pb.collection('posts').create({
    title: 'My Post',
    content: 'Content here'
});

// Update with optimistic update
await pb.collection('posts').update(record.id, {
    title: 'Updated Title'
});

// Real-time subscription
pb.collection('posts').subscribe('*', (e) => {
    console.log(e.action, e.record);
});

// File upload with progress
const formData = new FormData();
formData.append('title', 'Post with image');
formData.append('image', fileInput.files[0]);

await pb.collection('posts').create(formData, {
    onUploadProgress: (progress) => {
        console.log(`Upload: ${progress}%`);
    }
});
```

### Dart SDK Usage

```dart
import 'package:pocketbase/pocketbase.dart';

final pb = PocketBase('http://127.0.0.1:8090');

// Authentication
await pb.collection('users').authWithPassword('test@example.com', 'password');

// Create a record
final record = await pb.collection('posts').create({
    'title': 'My Post',
    'content': 'Content here',
});

// Real-time subscription
pb.collection('posts').subscribe('*', (e) {
    print('${e.action}: ${e.record}');
});
```

---

## 2.4.3 Extensibility Through Go and JavaScript Hooks (pb_hooks)

The dual hook system enables customization without core modification.

### Hook Types Comparison

| Hook Type | Implementation | Best For |
|-----------|---------------|----------|
| Go hooks | Compiled into binary | Performance-critical paths, complex logic, external Go ecosystem integration |
| JavaScript hooks (pb_hooks) | Runtime-interpreted via Goja engine | Rapid prototyping, team without Go expertise, frequently changing logic |

### JavaScript Hooks (pb_hooks)

Hooks are placed in the `pb_hooks` directory:

```
pb_hooks/
├── collections/
│   └── posts/
│       ├── before_create.js
│       ├── after_create.js
│       ├── before_update.js
│       └── after_delete.js
├── users/
│   └── before_auth.js
└── http/
    └── before_request.js
```

### Hook API Example

```javascript
// pb_hooks/collections/posts/before_create.js
onBeforeCreateRequest((e) => {
    // Access the record data
    const data = e.getData();
    
    // Add timestamp
    data.created_at = new Date().toISOString();
    
    // Validate
    if (!data.title || data.title.length < 3) {
        throw new Error('Title must be at least 3 characters');
    }
    
    // Set default values
    if (!data.status) {
        data.status = 'draft';
    }
    
    // Continue with the request
    return e.next();
});
```

### Go Hooks

For performance-critical operations, Go hooks can be compiled:

```go
// main.go
package main

import (
    "log"
    
    "github.com/pocketbase/pocketbase"
    "github.com/pocketbase/pocketbase/core"
)

func main() {
    app := pocketbase.New()
    
    app.OnRecordBeforeCreateRequest().Add(func(e *core.RecordCreateEvent) error {
        // Custom logic here
        log.Printf("Creating record: %v", e.Record.Collection().Name)
        return nil
    })
    
    if err := app.Start(); err != nil {
        log.Fatal(err)
    }
}
```

### Hook Event Types

| Event | Description |
|-------|-------------|
| OnModelBeforeCreate | Before creating any record |
| OnModelAfterCreate | After creating any record |
| OnModelBeforeUpdate | Before updating any record |
| OnModelAfterUpdate | After updating any record |
| OnModelBeforeDelete | Before deleting any record |
| OnModelAfterDelete | After deleting any record |
| OnRecordAuth* | Authentication-related events |
| OnFile* | File upload/download events |
| OnMailer* | Email sending events |

### Filter Predicates

Hook registration supports filter predicates for conditional execution:

```javascript
// Only run for published posts
onBeforeCreateRequest({
    filter: 'status = "published"'
}, (e) => {
    // Logic only runs for published posts
    return e.next();
});
```

### Middleware Chaining

Use `e.Next()` delegation for composable handler architectures:

```javascript
onBeforeCreateRequest((e) => {
    // First middleware
    console.log('Log: Creating record');
    return e.next(); // Continue to next handler
});

onBeforeCreateRequest((e) => {
    // Second middleware
    e.getData().processed = true;
    return e.next();
});
```

---

## API Authentication

### Using API Keys

```bash
# Admin API key (full access)
curl -H "Authorization: ADMIN_API_KEY" http://127.0.0.1:8090/api/collections

# Authenticated user token
curl -H "Authorization: USER_TOKEN" http://127.0.0.1:8090/api/collections/posts/records
```

### Token Refresh

```javascript
// Automatically handled by SDK
await pb.collection('users').authRefresh();

// Manual refresh
await pb.collection('users').authWithPassword('email', 'password');
const newToken = pb.authStore.token;
```
