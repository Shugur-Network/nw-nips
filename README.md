# Nostr Web NIPs

> Protocol specifications and documentation for Nostr Web

This repository contains the Nostr Implementation Possibilities (NIPs) that define the Nostr Web protocol for publishing and browsing decentralized static websites on Nostr.

## NIPs

### NIP-YY: Nostr Web Pages

Defines the event kinds and structure for web content on Nostr:

- **Kind 40000**: HTML content (immutable)
- **Kind 40001**: CSS stylesheets (immutable)
- **Kind 40002**: JavaScript modules (immutable, requires SRI)
- **Kind 40003**: Components (immutable, reusable HTML fragments)
- **Kind 34235**: Page Manifest (replaceable, per-route metadata)
- **Kind 34236**: Site Index (replaceable, route mapping)

📄 [Read NIP-YY](./NIP-YY.md)

### NIP-ZZ: DNS Bootstrap

Defines how Nostr Web sites are discovered via DNS TXT records:

- DNS TXT record format at `_nweb.<domain>`
- Public key pinning for author verification
- Relay configuration
- Optional Blossom media server endpoints

📄 [Read NIP-ZZ](./NIP-ZZ.md)

## Website

This repository also hosts the landing page for [nweb.shugur.com](https://nweb.shugur.com), which serves as both:

- **Static landing page** when accessed via a normal browser
- **Nostr Web example** when accessed via the Nostr Web browser extension

Install links:

- Chrome Web Store: [nostr-web-browser](https://chromewebstore.google.com/detail/nostr-web-browser/hhdngjdmlabdachflbdfapkogadodkif)
- Firefox Add-on: [nostr-web-browser on AMO](https://addons.mozilla.org/en-US/firefox/addon/nostr-web-browser/)

The dual behavior demonstrates the seamless integration of Nostr Web with existing web infrastructure.

## Related Projects

- Nostr Web extension (Cross-browser):
  - Chrome Web Store: [nostr-web-browser](https://chromewebstore.google.com/detail/nostr-web-browser/hhdngjdmlabdachflbdfapkogadodkif)
  - Firefox Add-on: [nostr-web-browser on AMO](https://addons.mozilla.org/en-US/firefox/addon/nostr-web-browser/)
- **[nw-publisher](https://github.com/Shugur-Network/nw-publisher)**: CLI tool for publishing static sites to Nostr (available on npm)

## Contributing

Contributions to the NIP specifications are welcome! Please:

1. Fork this repository
2. Create a feature branch
3. Submit a Pull Request with your proposed changes

For discussions about the protocol, visit the [GitHub Discussions](https://github.com/Shugur-Network/nw-nips/discussions).

## License

MIT License - see [LICENSE](./LICENSE) for details.

## Links

- **Documentation**: [docs.shugur.com/nostr-web](https://docs.shugur.com/nostr-web)
- **Demo Site**: [nweb.shugur.com](https://nweb.shugur.com)
- **npm Package**: [nw-publish](https://www.npmjs.com/package/nw-publish)

---

**Nostr Web by [Shugur](https://shugur.com)** — Decentralizing the web, one site at a time 🌐
