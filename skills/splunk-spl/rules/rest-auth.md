---
title: Splunk REST API Authentication
impact: HIGH
tags: rest, auth, basic-auth, session-token, ssl, curl
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/use-the-splunk-rest-api/access-splunk-search-results-using-the-rest-api"
---

## Splunk REST API Authentication

The Splunk REST API supports Basic Auth and Session Token authentication.

**Basic Auth:**
```bash
curl -k -u admin:changeme \
  https://localhost:8089/services/search/jobs \
  -d search="search index=web_logs | head 10"
```

**Session Token (recommended for multi-request scripts):**
```bash
# Step 1: Get session token
TOKEN=$(curl -sk -u admin:changeme \
  https://localhost:8089/services/auth/login \
  -d username=admin -d password=changeme \
  | grep -o '<sessionKey>[^<]*' | sed 's/<sessionKey>//')

# Step 2: Use token
curl -k -H "Authorization: Splunk $TOKEN" \
  https://localhost:8089/services/search/jobs \
  -d search="search index=web_logs | head 10"
```

**Notes:**
- **`-k` flag** — skip SSL certificate verification for self-signed certs.
  For production, use proper certificates and remove `-k`.
- **Session token TTL:** Default 1 hour (configurable in `web.conf` `sessionTimeout`).
- **Token auth also supported:** Splunk authentication tokens (Settings > Tokens)
  for long-lived API access without username/password.
  Header: `Authorization: Bearer <token>`.
- Management port: default `8089` (configured in `server.conf`).
- REST API base URL: `https://splunk-host:8089/services/`.
