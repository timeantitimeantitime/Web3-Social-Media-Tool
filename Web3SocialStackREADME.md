# Web3Social Stack

> **Post once, reach everywhere.**

A TypeScript/Node.js system that publishes a single message across multiple decentralized social platforms simultaneously. No paid APIs for Farcaster — uses direct hub protocol with `NobleEd25519Signer`.

## Supported Platforms

| Platform | Protocol | SDK / Library | Char Limit | Media |
|----------|----------|---------------|------------|-------|
| Farcaster | Hub Protocol | `@farcaster/hub-nodejs` | 320 | Images |
| Lens Protocol | Lens Chain | `@lens-protocol/client` | 5,000 | Images + Video |
| Nostr | WebSocket Relays | `nostr-tools` | 64,000 | Images + Video |
| Mastodon | ActivityPub | `megalodon` | 500 | Images + Video |
| Bluesky | AT Protocol | `@atproto/api` | 300 | Images |
| Twitter/X | REST API v2 | Native fetch | 280 | Images + Video |
| Threads | Graph API | Native fetch | 500 | Images + Video |

## Features

- **One input, seven outputs** — Write once, publish everywhere
- **Smart truncation** — Each adapter formats text to its platform's character limit
- **Auto-discovery** — Only instantiates adapters whose environment variables are present
- **RSS/Atom feeds** — Auto-updates `feeds/feed.xml` with every post
- **Torrent distribution** — Creates `.torrent` files for media posts, seeds via WebTorrent, includes magnet URIs
- **REST API** — Hono server on port 3847 with full CRUD endpoints
- **CLI** — `post`, `platforms`, `seed` commands
- **Type-safe** — Strict TypeScript with Zod validation

## Quick Start

```bash
# Clone or extract
cd web3social

# Install dependencies
npm install

# Copy and configure credentials
cp .env.example .env
# Edit .env with your real API keys

# Build
npm run build

# Verify CLI
node dist/cli.js --help
```

## CLI Usage

```bash
# Publish to all configured platforms
node dist/cli.js post "Hello decentralized world!" --tags web3 social

# With images (auto-creates torrent)
node dist/cli.js post "Check this out"   --images https://example.com/pic1.jpg https://example.com/pic2.jpg   --tags nft art

# Reply to a post
node dist/cli.js post "Great point!" --reply-to 0xabc123 --channel crypto

# List platform status
node dist/cli.js platforms

# Show active torrent seeding status
node dist/cli.js seed
```

## REST API

```bash
npm run server
# → API running at http://localhost:3847
```

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Service health + available platforms |
| GET | `/platforms` | All platforms with config/enabled status |
| POST | `/publish` | Publish to all enabled platforms |
| POST | `/publish/:platform` | Publish to a single platform |
| GET | `/feed.xml` | Generated RSS/Atom feed |
| GET | `/torrents` | Active torrent seeding status |

### POST /publish

```bash
curl -X POST http://localhost:3847/publish   -H "Content-Type: application/json"   -d '{
    "text": "Hello from Web3Social!",
    "images": ["https://example.com/image.jpg"],
    "tags": ["web3", "social"],
    "replyTo": "0xabc123"
  }'
```

Response:

```json
{
  "success": true,
  "results": [
    {
      "platform": "farcaster",
      "success": true,
      "id": "0x...",
      "url": "https://warpcast.com/~/conversations/0x...",
      "timestamp": "2026-07-22T23:44:00.000Z"
    }
  ],
  "torrent": {
    "infoHash": "abc123...",
    "magnetUri": "magnet:?xt=urn:btih:abc123..."
  }
}
```

## Configuration

### Environment Variables

Copy `.env.example` to `.env` and fill in your credentials:

```bash
# Farcaster
FARCASTER_MNEMONIC=your twelve word mnemonic phrase here
FARCASTER_HUB_URL=https://hub.farcaster.standardcrypto.vc:2281

# Lens Protocol
LENS_PRIVATE_KEY=0x...
LENS_ENVIRONMENT=production

# Nostr
NOSTR_PRIVATE_KEY=nsec1...
NOSTR_RELAYS=wss://relay.damus.io,wss://relay.nostr.band,wss://nos.lol

# Mastodon
MASTODON_ACCESS_TOKEN=your_access_token
MASTODON_BASE_URL=https://mastodon.social

# Bluesky
BLUESKY_IDENTIFIER=handle.bsky.social
BLUESKY_PASSWORD=your_app_password

# Twitter/X
TWITTER_API_KEY=your_api_key
TWITTER_API_SECRET=your_api_secret
TWITTER_ACCESS_TOKEN=your_access_token
TWITTER_ACCESS_SECRET=your_access_secret

# Threads
THREADS_ACCESS_TOKEN=your_long_lived_access_token
THREADS_USER_ID=your_user_id
```

### Platform Enable/Disable

Edit `web3social.config.json`:

```json
{
  "platforms": {
    "farcaster": true,
    "lens": true,
    "nostr": true,
    "mastodon": false,
    "bluesky": true,
    "twitter": true,
    "threads": false
  },
  "torrent": {
    "enabled": true,
    "announceList": [
      ["udp://tracker.opentrackr.org:1337/announce"],
      ["udp://tracker.openbittorrent.com:6969/announce"]
    ],
    "seedTimeout": 3600000
  },
  "rss": {
    "enabled": true,
    "outputPath": "./feeds/feed.xml",
    "title": "Web3 Social Feed",
    "description": "Aggregated posts from decentralized social platforms",
    "link": "https://yourdomain.com",
    "language": "en"
  },
  "server": {
    "port": 3847,
    "host": "0.0.0.0"
  }
}
```

## Project Structure

```
web3social/
├── package.json              # ESM modules, dependencies
├── tsconfig.json             # Strict TypeScript
├── web3social.config.json    # Platform & feature toggles
├── .env.example              # Credential template
├── .gitignore
├── src/
│   ├── index.ts              # Library exports
│   ├── cli.ts                # CLI entry point
│   ├── server.ts             # Hono REST API
│   ├── publisher.ts          # Orchestrator — fans out to adapters
│   ├── rss-generator.ts      # RSS/Atom feed writer
│   ├── types.ts              # Zod schemas (PostRequest, PostResult)
│   ├── types.d.ts            # Type declarations for webtorrent, create-torrent
│   ├── utils.ts              # Config loader, env helpers
│   ├── adapters/
│   │   ├── base.ts           # PlatformAdapter interface
│   │   ├── farcaster.ts      # Hub protocol + Ed25519 signing
│   │   ├── lens.ts           # Lens Protocol SDK
│   │   ├── nostr.ts          # Nostr kind-1 events
│   │   ├── mastodon.ts       # Megalodon client
│   │   ├── bluesky.ts        # AT Protocol records
│   │   ├── twitter.ts        # REST API v2
│   │   └── threads.ts        # Graph API containers
│   └── torrent/
│       ├── creator.ts        # Create .torrent, extract info hash
│       └── seeder.ts         # WebTorrent background seeding
└── feeds/
    └── feed.xml              # Generated RSS output
```

## Architecture

```
┌─────────────┐     ┌─────────────┐     ┌─────────────────┐
│   CLI       │────→│  Publisher  │────→│  Farcaster      │
│   REST API  │     │  (Router)   │     │  Lens           │
│             │     │             │     │  Nostr          │
└─────────────┘     └─────────────┘     │  Mastodon       │
                                        │  Bluesky        │
                                        │  Twitter        │
                                        │  Threads        │
                                        └─────────────────┘
                                                  │
                                        ┌─────────┴─────────┐
                                        ↓                   ↓
                                   ┌─────────┐        ┌──────────┐
                                   │ RSS Gen │        │ Torrent  │
                                   │ feed.xml│        │ Creator  │
                                   └─────────┘        │ + Seeder │
                                                      └──────────┘
```

Each adapter implements the `PlatformAdapter` interface:

```typescript
interface PlatformAdapter {
  platform: Platform;
  maxChars: number;
  supportsImages: boolean;
  supportsVideo: boolean;
  isConfigured(): boolean;     // Checks env vars present
  post(request: PostRequest): Promise<PostResult>;
  formatText(text: string): string;  // Truncate to limit
}
```

The `Publisher` only instantiates adapters where `isConfigured()` returns `true` **and** the platform is enabled in `web3social.config.json`.

## Scripts

| Script | Command |
|--------|---------|
| `npm run build` | Compile TypeScript to `dist/` |
| `npm run dev` | Run CLI via tsx (no build needed) |
| `npm run cli` | Run compiled CLI |
| `npm run server` | Start REST API |
| `npm start` | Build + start server |
| `npm run typecheck` | Type-check without emitting |

## Dependencies

- `@farcaster/hub-nodejs` — Farcaster hub protocol
- `@lens-protocol/client` — Lens Protocol SDK
- `nostr-tools` — Nostr event signing and relay publishing
- `megalodon` — Mastodon client
- `@atproto/api` — Bluesky AT Protocol
- `hono` — Lightweight web framework
- `rss` — RSS/Atom feed generation
- `zod` — Runtime schema validation
- `create-torrent` — .torrent file creation
- `webtorrent` — BitTorrent seeding
- `commander` — CLI framework
- `dotenv` — Environment variable loading

## License

MIT
