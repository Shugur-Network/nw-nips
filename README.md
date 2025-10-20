# Nostr Web NIPs

> Protocol specifications and documentation for Nostr Web

This repository contains the Nostr Implementation Possibilities (NIPs) that define the Nostr Web protocol for publishing and browsing decentralized static websites on Nostr.

## NIPs

### NIP-YY: Nostr Web Pages

Defines the event kinds and structure for web content on Nostr:

- **Kind 1125**: Asset - All web assets (HTML, CSS, JavaScript, fonts, etc.) with MIME type
- **Kind 1126**: Page Manifest - Links assets for a specific page
- **Kind 31126**: Site Index - Maps routes to page manifests (addressable)
- **Kind 11126**: Entrypoint - Points to the current site index (replaceable)

📄 [Read NIP-YY](./NIP-YY.md)

### NIP-ZZ: DNS Bootstrap

Defines how Nostr Web sites are discovered via DNS TXT records:

- DNS TXT record format at `_nweb.<domain>`
- Site author's public key (for event verification)
- Relay URLs where site events are published
- Automatic updates without DNS changes

📄 [Read NIP-ZZ](./NIP-ZZ.md)

## Website

This repository also hosts the landing page for [nweb.shugur.com](https://nweb.shugur.com), which serves as both:

- **Static landing page** when accessed via a normal browser
- **Nostr Web example** when accessed via the Nostr Web browser extension

<table>
  <tr>
    <td align="center" width="50%">
      <a href="https://chromewebstore.google.com/detail/nostr-web-browser/hhdngjdmlabdachflbdfapkogadodkif">
        <img src="https://img.shields.io/badge/Chrome-4285F4?style=for-the-badge&logo=google-chrome&logoColor=white" alt="Chrome Web Store" />
      </a>
    </td>
    <td align="center" width="50%">
      <a href="https://addons.mozilla.org/en-US/firefox/addon/nostr-web-browser/">
        <img src="https://img.shields.io/badge/Firefox-FF7139?style=for-the-badge&logo=firefox&logoColor=white" alt="Firefox Add-ons" />
      </a>
    </td>
  </tr>
</table>

The dual behavior demonstrates the seamless integration of Nostr Web with existing web infrastructure.

## Related Projects

- **[nw-extension](https://github.com/Shugur-Network/nw-extension)**: Cross-browser extension for browsing Nostr Web sites
- **[nw-publisher](https://github.com/Shugur-Network/nw-publisher)**: CLI tool for publishing static sites to Nostr (available on npm)

## License

MIT License - see [LICENSE](./LICENSE) for details.

---

**Nostr Web by [Shugur](https://shugur.com)** — Decentralizing the web, one site at a time 🌐
