# NIP-YY

## Nostr Web Pages

`draft` `optional`

This NIP defines a set of event kinds for hosting static websites on Nostr, enabling censorship-resistant web publishing through the Nostr protocol.

## Abstract

Nostr Web Pages (NWP) allows publishing static websites as Nostr events. HTML, CSS, and JavaScript are stored as immutable events, while page manifests and site indexes use parameterized replaceable events to maintain current references. This enables decentralized, verifiable, and censorship-resistant websites.

## Motivation

Traditional web hosting relies on centralized servers that can be censored, taken down, or compromised. By publishing websites as Nostr events:

- Content becomes censorship-resistant through relay replication
- Sites are independently verifiable via cryptographic signatures
- No central origin servers are required
- Publishing works with existing Nostr infrastructure

## Event Kinds

This NIP defines the following event kinds:

| Kind    | Description        | Type                      |
| ------- | ------------------ | ------------------------- |
| `40000` | HTML content       | Immutable                 |
| `40001` | CSS stylesheet     | Immutable                 |
| `40002` | JavaScript module  | Immutable                 |
| `40003` | Component/Fragment | Immutable                 |
| `34235` | Page Manifest      | Parameterized Replaceable |
| `34236` | Site Index         | Parameterized Replaceable |

### Immutable Assets (40000-40003)

Content-addressed assets that never change after publication.

**Required tags:**

- `m` - MIME type (`text/html`, `text/css`, `text/javascript`, etc.)
- `sha256` - Hex-encoded SHA-256 hash of the `content` field (for Subresource Integrity)

**Example HTML event (kind 40000):**

```json
{
  "kind": 40000,
  "pubkey": "<site-author-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["m", "text/html"],
    ["sha256", "a1b2c3..."],
    ["title", "Home Page"]
  ],
  "content": "<!DOCTYPE html><html>...</html>",
  "id": "<event-id>",
  "sig": "<signature>"
}
```

**Example CSS event (kind 40001):**

```json
{
  "kind": 40001,
  "pubkey": "<site-author-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["m", "text/css"],
    ["sha256", "d4e5f6..."]
  ],
  "content": "body { margin: 0; }",
  "id": "<event-id>",
  "sig": "<signature>"
}
```

**Example JavaScript event (kind 40002):**

```json
{
  "kind": 40002,
  "pubkey": "<site-author-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["m", "text/javascript"],
    ["sha256", "g7h8i9..."]
  ],
  "content": "console.log('Hello Nostr Web');",
  "id": "<event-id>",
  "sig": "<signature>"
}
```

### Page Manifest (34235)

Parameterized replaceable event that links to current assets for a specific route.

**Required tags:**

- `d` - Route path (e.g., `/`, `/about`, `/blog/post-1`)
- `e` - Asset event IDs with markers: `html`, `css`, `js`, `component`

**Optional tags:**

- `title` - Page title
- `description` - Page description
- `csp` - Content Security Policy directives
- `default_route` - Mark as default route (boolean flag)

**Example:**

```json
{
  "kind": 34235,
  "pubkey": "<site-author-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["d", "/"],
    ["e", "<html-event-id>", "html"],
    ["e", "<css-event-id-1>", "css"],
    ["e", "<css-event-id-2>", "css"],
    ["e", "<js-event-id>", "js"],
    ["title", "Home"],
    ["description", "Welcome to my Nostr Web site"]
  ],
  "content": "",
  "id": "<event-id>",
  "sig": "<signature>"
}
```

### Site Index (34236)

Parameterized replaceable event that maps routes to their current page manifest IDs.

**Required tags:**

- `d` - Always `"site-index"` (addressable event key)
- `default_route` - Default route (e.g., `/`)

**Content:** JSON object mapping routes to manifest event IDs

**Example:**

```json
{
  "kind": 34236,
  "pubkey": "<site-author-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["d", "site-index"],
    ["default_route", "/"]
  ],
  "content": "{
    \"/\": \"<manifest-event-id-1>\",
    \"/about\": \"<manifest-event-id-2>\",
    \"/blog/post-1\": \"<manifest-event-id-3>\"
  }",
  "id": "<event-id>",
  "sig": "<signature>"
}
```

## Client Behavior

### Publishing

1. Generate content events (40000-40003) with SHA-256 hashes
2. Publish immutable assets to relays
3. Create page manifest (34235) for each route, referencing asset event IDs
4. Create/update site index (34236) with route-to-manifest mapping
5. Generate DNS TXT record (see NIP-YY)

### Fetching and Rendering

1. Query DNS for `_nweb.<domain>` TXT record to get site pubkey and relays
2. Fetch site index (34236) from relays: `{"kinds": [34236], "authors": ["<pubkey>"], "#d": ["site-index"]}`
3. Parse site index to get manifest ID for requested route
4. Fetch page manifest (34235): `{"ids": ["<manifest-id>"]}`
5. Fetch all referenced assets (40000-40003) by event ID
6. Verify SHA-256 hashes of JavaScript assets (Subresource Integrity)
7. Assemble HTML with CSS and JS references
8. Render in sandboxed environment with CSP enforcement

### Security Considerations

**Author Verification:**

- All events MUST be authored by the pubkey specified in DNS TXT record
- Clients MUST reject events from other pubkeys

**Subresource Integrity (SRI):**

- JavaScript assets (40002) MUST include `sha256` tag
- Clients MUST verify hash before execution
- CSS assets (40001) SHOULD include `sha256` tag

**Content Security Policy:**

- Default CSP: `default-src 'self'; script-src 'sha256-<hash>'`
- Per-page CSP can be specified in manifest
- Clients SHOULD enforce CSP to prevent code injection

**Sandboxing:**

- Content SHOULD be rendered in isolated environment (iframe sandbox)
- Network requests outside Nostr/Blossom SHOULD be blocked

## Caching

**DNS Records:**

- Cache only as offline fallback
- Always attempt fresh DNS lookup

**Site Index (34236):**

- Always fetch fresh (TTL = 0)
- Required to detect site updates

**Page Manifests (34235):**

- Cache with short TTL (30-60 seconds)
- Validate against current site index

**Immutable Assets (40000-40003):**

- Cache indefinitely (content-addressed)
- Use SHA-256 for cache key validation

## Media Assets

Large media files (images, videos, fonts) SHOULD use Blossom (see NIP-YY for endpoint discovery):

```html
<img src="blossom://<sha256-hash>" alt="Image" />
```

Clients translate `blossom://` URLs to configured Blossom endpoints.

## Example Workflow

1. **Author creates site:**

   ```bash
   # Generate events
   npx nw-publish ./my-site

   # Output: DNS JSON and event IDs
   ```

2. **Author sets DNS:**

   ```
   _nweb.example.com TXT '{"v":1,"pk":"npub1...","relays":["wss://relay.damus.io"]}'
   ```

3. **User visits site:**
   - Browser extension checks `_nweb.example.com`
   - Fetches site index from relays
   - Loads page manifest for `/`
   - Fetches HTML, CSS, JS assets
   - Renders page

## Reference Implementation

- Browser extension: https://github.com/Shugur-Network/nostr-web/tree/main/extension
- Publisher CLI: https://github.com/Shugur-Network/nostr-web/tree/main/publisher
