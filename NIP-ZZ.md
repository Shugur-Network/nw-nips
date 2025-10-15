# NIP-ZZ

## DNS Bootstrap for Nostr Web Pages

`draft` `optional`

This NIP standardizes DNS TXT record format for bootstrapping Nostr Web Pages clients with site metadata including author pubkey, relay URLs, and optional service endpoints.

## Abstract

To make Nostr Web Pages accessible via traditional domain names, this NIP defines a DNS TXT record format at `_nweb.<domain>` that provides:

- Site author's public key (for event verification)
- Relay URLs where site events are published
- Optional Blossom endpoints for media assets
- Optional site index event ID for faster bootstrap

## Motivation

While Nostr events are identified by pubkey and event IDs, users expect to navigate via domain names (e.g., `example.com`). DNS bootstrap provides:

- **Human-readable addresses** - Users type domains, not npubs
- **Author pinning** - DNS record pins the canonical site pubkey, preventing impersonation
- **Relay discovery** - Clients know where to fetch events
- **Media gateway hints** - Advertise Blossom endpoints for large assets

## DNS TXT Record Format

### Record Name

```
_nweb.<domain>
```

Examples:

- `_nweb.example.com`
- `_nweb.blog.example.com`
- `_nweb.mysite.org`

### Record Value

A single-line JSON string with the following fields:

| Field        | Type    | Required | Description                                         |
| ------------ | ------- | -------- | --------------------------------------------------- |
| `v`          | integer | Yes      | Schema version (currently `1`)                      |
| `pk`         | string  | Yes      | Site author pubkey (npub or hex)                    |
| `relays`     | array   | Yes      | WSS relay URLs (at least one)                       |
| `blossom`    | array   | No       | Blossom endpoint URLs (HTTPS)                       |
| `site_index` | string  | No       | Site index event ID (kind 34236) for fast bootstrap |
| `policy`     | object  | No       | Client hints (reserved for future use)              |

### Example Records

**Minimal record:**

```json
{
  "v": 1,
  "pk": "npub1abc123...",
  "relays": ["wss://relay.damus.io"]
}
```

**Complete record:**

```json
{
  "v": 1,
  "pk": "5e56a2e48c4c5eb902e062bc30f92eabcf2e2fb96b5e7...",
  "relays": ["wss://relay.damus.io", "wss://nos.lol", "wss://relay.nostr.band"],
  "blossom": ["https://cdn.satellite.earth", "https://blossom.primal.net"],
  "site_index": "a1b2c3d4e5f6...",
  "policy": {
    "min_relays": 2
  }
}
```

## Public Key Format

The `pk` field accepts two formats:

**npub (bech32):**

```json
"pk": "npub1abc123def456..."
```

**Hex:**

```json
"pk": "5e56a2e48c4c5eb902e062bc30f92eabcf2e2fb96b5e7..."
```

Clients MUST support both formats.

## Relay URLs

**Format:**

- MUST use `wss://` scheme (WebSocket Secure)
- MUST be valid WebSocket URLs
- SHOULD be publicly accessible

**Examples:**

```json
"relays": [
  "wss://relay.damus.io",
  "wss://relay.nostr.band",
  "wss://nos.lol"
]
```

**Recommendations:**

- Include at least 2-3 relays for redundancy
- Use well-known public relays
- Ensure relays support NIP-YY event kinds (40000-40003, 34235, 34236)

## Blossom Endpoints

Optional HTTPS URLs for Blossom media servers:

```json
"blossom": [
  "https://cdn.satellite.earth",
  "https://blossom.primal.net"
]
```

Clients use these for resolving `blossom://<hash>` references in HTML/CSS.

## Site Index Hint

Optional event ID of kind 34236 site index for faster bootstrap:

```json
"site_index": "a1b2c3d4e5f6789..."
```

When provided, clients can:

1. Fetch specific event by ID (faster than filter query)
2. Validate it matches site pubkey and has correct `d` tag
3. Fall back to filter query if event not found or invalid

## Policy Object

Reserved for future client hints:

```json
"policy": {
  "min_relays": 2,
  "ttl": 3600,
  "cache_strategy": "aggressive"
}
```

Current version ignores this field. Future NIPs may define standard policy fields.

## Client Behavior

### DNS Lookup

1. User navigates to `example.com`
2. Client checks for `_nweb.example.com` TXT record
3. If found, parse JSON and validate schema
4. If not found or invalid, handle as regular HTTP navigation

### Record Validation

**Required checks:**

- `v` field equals `1`
- `pk` field is valid npub or hex pubkey
- `relays` array has at least one valid `wss://` URL

**If validation fails:**

- Ignore record and handle as regular navigation
- Log warning for debugging

### Relay Connection

1. Connect to all relays in parallel
2. Subscribe to site index filter
3. Aggregate results from multiple relays
4. Use most recent event by `created_at`

### Author Verification

**Critical security requirement:**

All fetched events MUST be authored by the pubkey from DNS record:

```javascript
if (event.pubkey !== dnsRecord.pk) {
  throw new Error("Event author does not match DNS pubkey");
}
```

This prevents relay impersonation attacks.

## DNS Provider Considerations

### JSON Formatting

Some DNS providers require specific formatting:

**Single-line JSON:**

```
{"v":1,"pk":"npub1...","relays":["wss://relay.damus.io"]}
```

**Escaped quotes (automatic by some providers):**

```
{\"v\":1,\"pk\":\"npub1...\",\"relays\":[\"wss://relay.damus.io\"]}
```

Verify the actual TXT record value resolves to valid JSON.

### Record Length Limits

TXT records have size limits:

- Single string: 255 characters
- Multiple strings: Concatenated to ~4KB

For large records, consider:

- Using hex pubkey instead of npub (shorter)
- Limiting relay/blossom array length
- Omitting optional fields

## DNSSEC

DNSSEC is **strongly recommended** to prevent DNS spoofing:

```bash
# Enable DNSSEC for your domain
# (Provider-specific configuration)
```

Clients SHOULD verify DNSSEC signatures when available.

## Example Implementation

### Setting DNS Record

**Cloudflare:**

```
Type: TXT
Name: _nweb
Content: {"v":1,"pk":"npub1...","relays":["wss://relay.damus.io"]}
TTL: Auto
```

**Route 53:**

```
Type: TXT
Name: _nweb.example.com
Value: "{"v":1,"pk":"npub1...","relays":["wss://relay.damus.io"]}"
TTL: 300
```

### Client Lookup

```javascript
// DNS-over-HTTPS lookup
const response = await fetch(
  `https://dns.google/resolve?name=_nweb.example.com&type=TXT`
);
const data = await response.json();

// Parse TXT record
const txtRecord = data.Answer?.[0]?.data;
const nwebConfig = JSON.parse(txtRecord.replace(/^"|"$/g, ""));

// Validate
if (nwebConfig.v !== 1) throw new Error("Unsupported version");
if (!nwebConfig.pk || !nwebConfig.relays) throw new Error("Invalid record");

// Use config
const pubkey = parseNpub(nwebConfig.pk);
const relays = nwebConfig.relays;
```

## Security Considerations

### Author Pinning

The DNS record acts as a trust anchor:

- **MUST** verify all events match DNS pubkey
- **MUST** reject events from other authors
- Prevents relay injection attacks

### DNS Spoofing

Without DNSSEC:

- Attacker could serve fake DNS record
- Points to malicious pubkey
- User sees impersonated site

**Mitigation:**

- Enable DNSSEC
- Use DNS-over-HTTPS (DoH) for client lookups
- Cache validated records

### Relay Censorship

If all listed relays censor content:

- Site becomes inaccessible via DNS
- Users can manually specify alternative relays
- Consider including diverse relay set

## Caching Strategy

**DNS TTL:**

- Respect DNS TTL from provider
- Re-query on TTL expiration
- Cache offline for fallback only

**Record Updates:**

- Users must wait for DNS propagation (usually minutes to hours)
- Consider low TTL for frequently changing records

## Reference Implementation

- DNS lookup: https://github.com/Shugur-Network/nw-nips/tree/main/extension
- Record generation: https://github.com/Shugur-Network/nw-nips/tree/main/publisher
