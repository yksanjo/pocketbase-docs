# Authentication and Security Framework

PocketBase provides a comprehensive authentication and security framework with industry-best practices implementation.

## 2.3.1 Email/Password Authentication with Verification Flows

Security implementations follow industry best practices:

| Security Control | Implementation |
|-----------------|----------------|
| Password hashing | bcrypt with adaptive cost factors (default 10, configurable) |
| Verification tokens | 15-minute expiration, cryptographically random |
| Rate limiting | Per-IP and per-account thresholds with exponential backoff |
| Session management | JWT tokens with configurable expiration, automatic refresh |
| Transport security | Automatic TLS via Let's Encrypt or manual certificate configuration |

### Authentication Flow

1. **Registration**: User creates account with email/password
2. **Verification**: Email verification token sent (15-minute expiration)
3. **Login**: JWT token issued upon successful authentication
4. **Session**: Token refreshes automatically before expiration

### Password Security

- **bcrypt hashing** with adaptive cost factors
- Default cost factor: 10 (configurable)
- Protection against rainbow table attacks
- Salting handled automatically

---

## 2.3.2 OAuth2 Provider Integration (15+ Providers)

The OAuth2 abstraction handles provider-specific complexity while maintaining protocol compliance.

### Supported Providers

| Provider Category | Examples | Special Handling |
|-------------------|----------|------------------|
| Major platforms | Google, Apple, GitHub, Microsoft, Facebook | PKCE, state validation, nonce verification |
| Developer platforms | GitLab, Bitbucket, Gitea | Scope negotiation, profile normalization |
| Social/entertainment | Discord, Twitch, Spotify, Twitter/X | Token refresh, rate limit handling |
| Enterprise/regional | Generic OIDC, SAML2, Kakao, Line | Custom claim mapping, attribute synchronization |

### OAuth Features

- **PKCE (Proof Key for Code Exchange)** for enhanced security
- **State validation** to prevent CSRF attacks
- **Token refresh** handling
- **Userinfo normalization** across providers
- **Account linking** with matching verified email addresses

### Configuration Example

```javascript
// OAuth2 login
pb.collection('users').authWithOAuth2({
    provider: 'google'
});
```

---

## 2.3.3 Role-Based Access Control and Permission Management

PocketBase implements attribute-based access control (ABAC) through predicate expressions evaluated at query time.

### Permission Levels

| Permission Level | Control Mechanism |
|-----------------|-------------------|
| Collection-level | Create/read/update/delete rules per collection |
| Record-level | Dynamic filters based on record content and user attributes |
| Field-level | Visibility and editability restrictions per field |
| Ownership-based | Automatic grants for record creators |

### Predicate Language Variables

The predicate language supports:
- `@request.auth.*` - Current user attributes
- `@request.data.*` - Incoming data fields
- `@now` - Current timestamp
- Cross-collection relationships

### Example Permission Rules

```
// Only the record creator can update their record
@request.auth.id = user.id

// Users can only see active records
status = "active"

// Organization members can view their organization's records
@collection.organizations.members.id ?= @request.auth.id
```

### Rule Types per Collection

| Operation | Rule Purpose |
|-----------|--------------|
| List rule | Who can list/filter records |
| View rule | Who can view individual records |
| Create rule | Who can create new records |
| Update rule | Who can modify records |
| Delete rule | Who can delete records |

---

## 2.3.4 Automatic TLS Certificate Handling

Let's Encrypt integration eliminates certificate management operational burden.

### Features

- **ACME v2 protocol** with HTTP-01 challenge validation
- **Automatic renewal** 30 days before expiration with graceful rotation
- **Multi-domain certificates** (SAN) for deployments serving multiple subdomains
- **HTTP-to-HTTPS redirection** and HSTS header injection

### Internal Deployment Options

For internal deployments, manual certificate configuration supports:
- Corporate CA-signed certificates
- Self-signed development certificates
- Custom PKI integration

### Configuration

```yaml
// pb_config.yaml
http:
  tls:
    enabled: true
    letsencrypt:
      enabled: true
      hosts: ["example.com", "api.example.com"]
```

### Certificate Renewal

- Automatic renewal triggered 30 days before expiration
- Zero-downtime certificate rotation
- Fallback to existing certificate if renewal fails

---

## Security Best Practices

### Password Requirements

- Minimum length: 8 characters
- Configurable strength policies
- Optional character complexity requirements

### Rate Limiting

| Limit Type | Threshold | Behavior |
|-----------|-----------|----------|
| Per-IP | 100 requests/minute | Exponential backoff |
| Per-account | 10 failed attempts | Account lockout |

### Session Management

- JWT tokens with configurable expiration (default: 14 days)
- Automatic token refresh
- Single-device or multi-device sessions
