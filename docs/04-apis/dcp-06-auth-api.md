# Auth API

| Field | Value |
|-------|-------|
| Doc ID | `dcp-api-06` |
| Category | APIs |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-core-04, dcp-api-01 |

---

## OAuth Authorize

```
GET /v1/oauth/authorize
  ?client_id=app_882
  &redirect_uri=https://app.example/callback
  &response_type=code
  &scope=route:write:fqdn:api.example.com
  &state=...
  &code_challenge=...
  &code_challenge_method=S256
```

User selects domain, environment, and scope presets in consent UI.

---

## Token Exchange

```
POST /v1/oauth/token
```

```json
{
  "grant_type": "authorization_code",
  "code": "...",
  "redirect_uri": "https://app.example/callback",
  "client_id": "app_882",
  "code_verifier": "..."
}
```

### Response

```json
{
  "access_token": "eyJ...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "...",
  "scope": "route:write:fqdn:api.example.com"
}
```

---

## Client Credentials (Machine)

```
POST /v1/oauth/token
```

```json
{
  "grant_type": "client_credentials",
  "client_id": "ci_bot",
  "client_secret": "...",
  "scope": "txn:submit:environment:staging route:write:fqdn:*.staging.example.com"
}
```

Restricted to pre-registered scopes — no user consent.

---

## Token Exchange (Enterprise IdP)

```
POST /v1/oauth/token
```

```json
{
  "grant_type": "urn:ietf:params:oauth:grant-type:token-exchange",
  "subject_token": "{entra_jwt}",
  "subject_token_type": "urn:ietf:params:oauth:token-type:jwt",
  "scope": "intent:read:zone:example.com"
}
```

---

## Revoke Capability

```
POST /v1/capabilities/revoke
```

```json
{
  "token_id": "cap_991"
}
```

Propagates to validators < 5 seconds.

---

## Introspect

```
POST /v1/oauth/introspect
```

```json
{ "token": "eyJ..." }
```

```json
{
  "active": true,
  "scope": "route:write:fqdn:api.example.com",
  "exp": 1719590400,
  "dcp": { "org_id": "org_abc", "capabilities": [] }
}
```

---

## API Keys

```
POST /v1/api-keys
```

```json
{
  "name": "ci-deploy",
  "capabilities": [
    { "action": "txn:submit", "resource": "environment:staging" }
  ],
  "expires_at": "2027-01-01T00:00:00Z"
}
```

Returns `dcp_live_...` once. Maps internally to capability JWT with short TTL refresh.