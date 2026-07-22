# Web3Social Stack

> **Post once, reach everywhere.**

A TypeScript/Node.js system that publishes a single message across **21 decentralized platforms** simultaneously. No paid APIs for Farcaster — uses direct hub protocol with `NobleEd25519Signer`.

## Supported Platforms (21 total)

### Original Social (7)
| Platform | Protocol | SDK | Char Limit | Media |
|----------|----------|-----|------------|-------|
| Farcaster | Hub Protocol | `@farcaster/hub-nodejs` | 320 | Images |
| Lens Protocol | Lens Chain | `@lens-protocol/client` | 5,000 | Images + Video |
| Nostr | WebSocket Relays | `nostr-tools` | 64,000 | Images + Video |
| Mastodon | ActivityPub | `megalodon` | 500 | Images + Video |
| Bluesky | AT Protocol | `@atproto/api` | 300 | Images |
| Twitter/X | REST API v2 | Native fetch | 280 | Images + Video |
| Threads | Graph API | Native fetch | 500 | Images + Video |

### Additional Federated Social (6)
| Platform | Protocol | Reach | Notes |
|----------|----------|-------|-------|
| Pixelfed | ActivityPub | ~1M | Federated Instagram alternative |
| PeerTube | ActivityPub + WebTorrent | ~600K | Decentralized video hosting |
| Lemmy | ActivityPub variant | ~100K | Reddit alternative |
| Misskey | ActivityPub | ~300K | Microblogging with reactions |
| Friendica | ActivityPub + Diaspora | ~50K | Multi-protocol federation |
| Diaspora | Diaspora protocol | ~20K | Privacy-focused pods |

### Blockchain Publishing (2)
| Platform | Protocol | Notes |
|----------|----------|-------|
| Mirror.xyz | Ethereum + Arweave | NFT essays + crowdfunding |
| Steemit | Steem blockchain | Token rewards for content |

### Real-Time / Messaging (2)
| Platform | Protocol | Notes |
|----------|----------|-------|
| Matrix | Federated + E2E encrypted | Rooms, VoIP, bridges |
| XMPP | Jabber (RFC 6120) | What WhatsApp is built on |

### Direct Channels (3)
| Platform | Protocol | Reach | Notes |
|----------|----------|-------|-------|
| Email | SMTP/IMAP | 4B+ | The original decentralized network |
| Web Push | RFC 8030 | Universal | Direct to browser, no gatekeeper |
| SMS | Twilio API | 5B+ phones | Highest open rates |

### Storage / Distribution (1)
| Platform | Protocol | Notes |
|----------|----------|-------|
| IPFS | libp2p | Content-addressed P2P storage |

## Quick Start

```bash
npm install
cp .env.example .env
# Edit .env with your credentials
npm run build
node dist/cli.js --help
```

## CLI Usage

```bash
# Publish to all configured platforms
node dist/cli.js post "Hello Web3" --tags web3 social

# With images (auto-creates torrent)
node dist/cli.js post "Check this out" \
  --images https://example.com/pic1.jpg \
  --tags nft art

# List platform status
node dist/cli.js platforms

# Show active torrents
node dist/cli.js seed
```

## REST API

```bash
npm run server
# → http://localhost:3847
```

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Service health + available platforms |
| GET | `/platforms` | All 21 platforms with config/enabled status |
| POST | `/publish` | Publish to all enabled platforms |
| POST | `/publish/:platform` | Publish to a single platform |
| GET | `/feed.xml` | Generated RSS/Atom feed |
| GET | `/torrents` | Active torrent seeding status |

## Architecture

```
CLI / REST API → Publisher → 21 Platform Adapters → RSS + Torrent
```

Each adapter implements `PlatformAdapter`:
- `isConfigured()` — checks env vars, only instantiates if present
- `post(request)` — publishes with platform-specific SDK
- `formatText(text)` — truncates to platform character limit

## Project Structure

```
web3social/
├── src/
│   ├── cli.ts              # CLI entry point
│   ├── server.ts           # Hono REST API
│   ├── publisher.ts        # Orchestrator — 21 adapters
│   ├── rss-generator.ts    # RSS feed writer
│   ├── types.ts            # Zod schemas
│   ├── utils.ts            # Config loader
│   ├── adapters/
│   │   ├── base.ts         # PlatformAdapter interface
│   │   ├── farcaster.ts    # Hub protocol + Ed25519
│   │   ├── lens.ts         # Lens SDK
│   │   ├── nostr.ts        # Kind-1 events
│   │   ├── mastodon.ts     # Megalodon
│   │   ├── bluesky.ts      # AT Protocol
│   │   ├── twitter.ts      # REST API v2
│   │   ├── threads.ts      # Graph API
│   │   ├── pixelfed.ts     # ActivityPub
│   │   ├── peertube.ts     # ActivityPub + WebTorrent
│   │   ├── lemmy.ts        # ActivityPub variant
│   │   ├── misskey.ts      # ActivityPub
│   │   ├── friendica.ts    # Multi-protocol
│   │   ├── diaspora.ts     # Diaspora protocol
│   │   ├── mirror.ts       # Ethereum + Arweave
│   │   ├── steemit.ts      # Steem blockchain
│   │   ├── matrix.ts       # Federated E2E
│   │   ├── xmpp.ts         # Jabber
│   │   ├── email.ts        # SMTP
│   │   ├── webpush.ts      # RFC 8030
│   │   ├── sms.ts          # Twilio
│   │   └── ipfs.ts         # libp2p
│   └── torrent/
│       ├── creator.ts      # .torrent + magnet URI
│       └── seeder.ts       # WebTorrent seeding
├── package.json
├── tsconfig.json
├── web3social.config.json
└── .env.example
```

## Environment Variables

See `.env.example` for all 21 platform configurations.

## Scripts

| Script | Command |
|--------|---------|
| `npm run build` | Compile TypeScript to `dist/` |
| `npm run dev` | Run CLI via tsx |
| `npm run cli` | Run compiled CLI |
| `npm run server` | Start REST API |
| `npm start` | Build + start server |

## License

MIT
