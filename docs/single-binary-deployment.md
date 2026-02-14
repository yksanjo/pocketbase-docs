# Single-Binary Deployment Model

PocketBase's architecture represents a fundamental departure from traditional client-server database models. By embedding all components into a single executable, it eliminates complexity while providing enterprise-grade capabilities.

## 2.1.1 Embedded SQLite Database with Real-Time Subscriptions

PocketBase's embedded SQLite database represents a fundamental architectural departure from client-server database models. By linking SQLite directly into the application binary through the modernc.org/sqlite pure-Go driver, PocketBase eliminates network round-trips between application and database layers, reducing query latency from milliseconds to microseconds.

### Write-Ahead Logging (WAL) Mode Configuration

The Write-Ahead Logging (WAL) mode configuration transforms SQLite's concurrency characteristics:

| Mode | Read Concurrency | Write Performance | Use Case Fit |
|------|------------------|-------------------|--------------|
| Rollback Journal | Readers block on writers | Slower (random I/O) | Legacy compatibility |
| WAL Mode | Readers don't block writers | Faster (sequential append) | PocketBase default—optimal for real-time |

### Performance Characteristics

WAL mode enables:
- **10,000+ reads/second** concurrent with **100+ writes/second** on consumer SSD storage
- Linear read scaling across connection count
- Sub-100ms propagation latency from commit to client notification

This performance profile supports real-time subscription workloads that would conventionally require more complex database architectures.

### Real-Time Subscription System

The real-time subscription system builds upon database change observation rather than polling or external message queues. When write operations occur, PocketBase broadcasts change events through WebSocket connections to subscribed clients.

**Subscription API supports:**
- Collection-level subscriptions
- Record-level subscriptions
- Filtered subscriptions with server-side permission enforcement

---

## 2.1.2 Built-in Authentication and User Management System

PocketBase's integrated authentication system eliminates external identity provider dependencies while providing comprehensive security capabilities.

### Authentication Methods

| Authentication Method | Implementation Details |
|----------------------|------------------------|
| Email/password | bcrypt hashing, verification flows, password reset, strength policies |
| Username | Alternative or complementary to email authentication |
| OAuth 2.0 | 15+ providers: Google, Apple, GitHub, Microsoft, Facebook, Twitter/X, Discord, Twitch, Spotify, Kakao, Line, Gitea, generic OIDC, SAML2 |
| Magic links | Passwordless authentication via time-limited tokens |

### User Management Features

The user management data model extends beyond authentication to support:
- Rich profiles with custom field definitions
- Role-based access control
- Administrative lifecycle management through the bundled dashboard

OAuth integration handles provider-specific complexities (PKCE, state validation, token refresh, userinfo normalization) while presenting unified user records regardless of authentication source.

---

## 2.1.3 Integrated File Storage with Local and S3-Compatible Options

PocketBase's file storage subsystem provides unified asset management with backend abstraction.

### Storage Modes

| Storage Mode | Configuration | Use Case |
|--------------|---------------|----------|
| Local filesystem | Default—directory path configuration | Development, single-server deployment |
| S3-compatible | AWS S3, MinIO, Wasabi, DigitalOcean Spaces, Cloudflare R2 | Production scalability, durability |
| Backblaze B2 | S3-compatible API | Cost-optimized cold storage |

### Storage Features

- **Automatic image processing**: thumbnail generation, format conversion, EXIF correction
- **Presigned URL generation**: direct browser-to-storage uploads
- **Referential integrity**: automatic cleanup of orphaned files
- **Backend abstraction**: transparent switching between local development and production cloud storage without application code changes

---

## 2.1.4 Admin Dashboard UI Bundled in Single Executable

The embedded administrative dashboard—implemented in Svelte and compiled into static assets embedded via Go's `//go:embed` directive—provides immediate operational capability without separate frontend deployment.

### Dashboard Functions

| Dashboard Function | Capability |
|-------------------|------------|
| Schema design | Visual collection and field creation with 15+ field types |
| Data management | CRUD operations, filtering, sorting, bulk actions |
| User administration | Account lifecycle, role assignment, authentication methods |
| File browser | Visual organization, preview, manual upload/download |
| System configuration | Mail, storage, rate limiting, backup settings |
| Real-time monitoring | Connection observation, log streaming |

### Extension Points

For forked projects, the dashboard is both:
- **Significant asset**: reduces frontend development requirements
- **Extension point**: customizable through Svelte source modification or API-based alternative interfaces
