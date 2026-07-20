# Axona-share

Share images over [Axona](https://axona.net) pub/sub — a
small, dependency-light reference app for the
[protocol](https://github.com/axona-net/axona-protocol).

> Note: a separate, older variant of this app ships inside the kernel demo at
> `demo.axona.net/apps/axona-share`. **This** repo is the standalone app served
> from GitHub Pages; if you're debugging, confirm which URL you're on first.

Channels are pub/sub topics. An image is compressed to under 1 MB, split into
chunks with `@axona/protocol/std/chunk`, published over Axona, and reassembled
on every subscriber. No server holds the images — they travel peer-to-peer
across the mesh, relayed/bootstrapped through a bridge.

## Live (GitHub Pages)

<https://axona-net.github.io/axona-share/> — the published build, on the
**production** network by default.

## Networks — one build, prod or testnet

The same build picks its bridge from the URL, so you can point the live Pages
app at either network:

| URL | network |
|-----|---------|
| `…/axona-share/` | **prod** (`wss://bridge.axona.net`) — default |
| `…/axona-share/?net=testnet` | **testnet** (`wss://testnet.axona.net`) |
| `…/axona-share/?bridge=wss://my.bridge` | any explicit bridge |

The active network is shown in the header version row. Order of precedence:
explicit `?bridge=` → `?net=testnet\|prod` shortcut → default by hostname (a
`*testnet*` host defaults to testnet, otherwise prod).

## Run it

```sh
npm run link-kernel   # installs @axona/protocol and symlinks ./axona-protocol
npm start             # serves on http://localhost:8080
```

`link-kernel` creates the `./axona-protocol` symlink that the import map in
[`index.html`](index.html) resolves the bare `@axona/protocol` /
`@axona/protocol/std/chunk` / `@axona/web` specifiers against. The app is plain
ES modules + an import map — no bundler, no build step.

Requires a published kernel **≥ 2.48.0** (that's where `std/` and the
reliable-publish size guard landed).

## Layout

| file         | role |
|--------------|------|
| `index.html` | markup, styles, and the import map |
| `app.js`     | UI, channels, the share/receive flow |
| `axona.js`   | connects an `AxonaPeer` over the web transport; exposes `pub`/`sub` |
| `region.js`  | resolves the geo region → S2 keyspace for topic IDs |
| `image.js`   | client-side image downsampling/compression |

## Chunking & message size

Chunks are sized to the kernel's **reliable-publish floor (15 KB)** so each
chunk fits in a single pub/sub message that every peer and browser can
receive — WebRTC's interoperable `maxMessageSize` floor is 16 KiB, and a
message larger than the weakest hop can carry is silently dropped. `std/chunk`
handles the manifest, distinct-index completion, and reassembly timeout; the
app just iterates `messages` and calls `axona.pub`.

## Provenance

Promoted from a subdirectory of the Axona kernel monorepo
(`axona-protocol/apps/axona-share`) into its own repository. History starts
fresh here; the kernel is now consumed as a published npm dependency
(github-pinned) rather than via relative `../../src` imports. The original
local `lib/file-transport.js` chunking was migrated onto the protocol's
`std/chunk` helper as part of the move.
