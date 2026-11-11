# Concise Book on MPLS

A concise, diagram-rich guide to MPLS: label switching, LDP, the BGP-free core, BGP/MPLS L3VPNs, Layer 2 VPNs, Segment Routing (SR-MPLS), fast reroute with TI-LFA, traffic engineering, and SRv6.

**Read it online:** https://khaledmahmudkm.github.io/concisempls/

## Labs

The [`labs/`](labs/) folder holds seven containerlab labs built from FRRouting containers. See [`labs/README.md`](labs/README.md) for requirements (Linux host with the `mpls_router` and `mpls_iptunnel` modules loaded) and how to run them.

## How this repository works

* `chapters/` holds one Markdown file per chapter, with diagrams embedded as SVG, and `chapters.json` for the table of contents.
* `book.json` holds the title, author and palette (`lagoon`).
* `index.html` loads the shared reader, [bookengine](https://github.com/khaledmahmudkm/bookengine), from jsDelivr. When opened from a local web server, it uses `../bookengine-git/` instead if that folder exists.

To preview locally, serve the parent folder (the one containing both `concisempls-git` and `bookengine-git`):

```bash
python3 -m http.server 8000
# then open http://localhost:8000/concisempls-git/
```

© 2026 Khaled Mahmud
