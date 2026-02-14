# PocketBase Technical Architecture Documentation

This repository contains comprehensive technical documentation covering PocketBase's architecture, backend capabilities, real-time data infrastructure, authentication framework, and API integration layer.

## Table of Contents

1. [Single-Binary Deployment Model](./docs/single-binary-deployment.md)
   - Embedded SQLite database with real-time subscriptions
   - Built-in authentication and user management
   - Integrated file storage (local and S3-compatible)
   - Admin dashboard UI

2. [Real-Time Data Infrastructure](./docs/realtime-data-infrastructure.md)
   - WebSocket-based live data synchronization
   - Event-driven subscription model
   - WAL mode for SQLite optimization

3. [Authentication and Security Framework](./docs/authentication-security.md)
   - Email/password authentication with verification
   - OAuth2 provider integration (15+ providers)
   - Role-based access control (RBAC)
   - Automatic TLS certificate handling

4. [API and Integration Layer](./docs/api-integration.md)
   - RESTful API design
   - Official JavaScript SDK and Dart SDK
   - Extensibility through Go and JavaScript hooks

## Quick Start

```bash
# Download PocketBase
wget https://github.com/pocketbase/pocketbase/releases/latest/download/pocketbase_{version}_linux_amd64.zip
unzip pocketbase_{version}_linux_amd64.zip

# Run PocketBase
./pocketbase serve
```

Access the admin dashboard at `http://127.0.0.1:8090/_/`

## Key Features Overview

| Feature | Description |
|---------|-------------|
| **Database** | Embedded SQLite with WAL mode, 10,000+ reads/sec, 100+ writes/sec |
| **Real-time** | WebSocket subscriptions with sub-100ms propagation |
| **Authentication** | Email/password, OAuth2 (15+ providers), magic links |
| **Storage** | Local filesystem or S3-compatible backends |
| **API** | RESTful API with JavaScript and Dart SDKs |

## Documentation Structure

```
pocketbase-docs/
├── README.md
├── docs/
│   ├── single-binary-deployment.md
│   ├── realtime-data-infrastructure.md
│   ├── authentication-security.md
│   └── api-integration.md
└── LICENSE
```

## License

This documentation is provided for educational purposes.
