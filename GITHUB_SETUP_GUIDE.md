ARCHITECTURE.md

Version: 1.0.0
Status: In Development
Network: Solana Mainnet-beta
Framework: Anchor (Rust)

---

## Executive Summary

DCTR is a Web3 Gamified SocialFi platform that simulates real-world nations under satirical dictatorial regimes. Players compete through on-chain trading performance, unlocking exclusive Dynamic NFT Trophies ("Nuclear Bombs") that grant strategic leaderboard advantages.

This document provides:
1. Complete system architecture and technology stack
2. Smart contract boilerplate (Anchor/Rust)
3. Frontend integration setup (React/Next.js)
4. Real-time indexing infrastructure
5. Deployment and security guidelines

---

## Part 1: System Architecture & Tech Stack

### 1.1 High-Level Architecture Diagram

┌────────────────────────────────────────────────────────────────────┐
│                      Frontend Layer (React/Next.js)                │
│  ┌──────────────────┬──────────────────┬──────────────────────┐   │
│  │  Wallet Connect  │  Game Map UI     │  Leaderboard Sync    │   │
│  │  (Phantom, Solflare) │  (Faction   │  (Real-time WebSocket)   │
│  │                  │  Visualization)  │                      │   │
│  └──────────────────┴──────────────────┴──────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘
                              ↓
┌────────────────────────────────────────────────────────────────────┐
│                   Web3 Adapter Layer (@solana/web3.js)              │
│  ┌──────────────────┬──────────────────┬──────────────────────┐   │
│  │ Wallet Provider  │  Transaction     │  State Management    │   │
│  │ Integration      │  Signing         │  (Zustand/Redux)     │   │
│  └──────────────────┴──────────────────┴──────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘
                              ↓
┌────────────────────────────────────────────────────────────────────┐
│                    Indexing & Data Layer                            │
│  ┌──────────────────┬──────────────────┬──────────────────────┐   │
│  │  DEX Indexer     │  Jupiter API     │  Birdeye Indexer     │   │
│  │  (Volume Track)  │  (Trade Track)   │  (Holder Analysis)   │   │
│  └──────────────────┴──────────────────┴──────────────────────┘   │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │        Solana RPC Nodes + History API                        │  │
│  │  (Mainnet, Testnet Endpoints)                                │  │
│  └──────────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────┘
                              ↓
┌────────────────────────────────────────────────────────────────────┐
│                  Blockchain Layer (Solana)                          │
│  ┌──────────────────┬──────────────────┬──────────────────────┐   │
│  │ DCTR Program     │  NFT Program     │  Token Program       │   │
│  │ (Anchor)         │  (Metaplex)      │  (SPL-Token)         │   │
│  │ (State Tracking) │  (Dynamic Minting)│ (Faction Rewards)    │   │
│  └──────────────────┴──────────────────┴──────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘
                              ↓
┌────────────────────────────────────────────────────────────────────┐
│              Backend Services (Node.js/Rust)                        │
│  ┌──────────────────┬──────────────────┬──────────────────────┐   │
│  │ Indexer Service  │  WebSocket       │  Scheduled Jobs      │   │
│  │ (Transactions)   │  Event Server    │  (Streak Detection)  │   │
│  │                  │  (Real-time)     │  (NFT Minting)       │   │
│  └──────────────────┴──────────────────┴──────────────────────┘   │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │        Database Layer (PostgreSQL + Redis)                   │  │
│  │  • Faction State (PostgreSQL)                                │  │
│  │  • Leaderboards (Redis for speed)                            │  │
│  │  • Streak Tracking (PostgreSQL)                              │  │
│  │  • NFT Metadata Cache (Redis)                                │  │
│  └──────────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────┘

### 1.2 Technology Stack Breakdown

FRONTEND STACK
- Framework: React 18 + Next.js 14 — UI framework with SSR
- Wallet Connection: @solana/web3.js + @solana/wallet-adapter — Phantom, Solflare support
- State Management: Zustand + TanStack Query — App state + server sync
- Real-time Updates: Socket.io / WebSocket — Live leaderboard sync
- UI Components: Shadcn/ui + Tailwind CSS — Component library
- Charts & Visualization: Recharts + Mapbox GL — Game map & leaderboards
- Build Tool: Turbopack (Next.js 14) — Fast builds

BACKEND STACK
- Runtime: Node.js 18+ — Backend services
- API Framework: Express.js + Fastify — REST/WebSocket servers
- Indexing: Rust (Solana Geyser integration) — On-chain event tracking
- Task Queue: Bull (Redis) — Async job processing
- Database: PostgreSQL 15 — Persistent state
- Cache: Redis 7 — High-speed leaderboards
- Monitoring: Prometheus + Grafana — Performance metrics

SMART CONTRACTS STACK
- Framework: Anchor 0.29.0 — Solana program development
- Language: Rust — Smart contract logic
- NFT Standard: Metaplex Token Metadata — Dynamic NFT minting
- Token Standard: SPL-Token-2022 — Faction rewards
- Testing: Anchor test framework — Unit & integration tests

SOLANA INFRASTRUCTURE
- RPC Nodes: Helius / Triton / QuickNode — State queries
- Historical Data: Solana History API — Transaction lookup
- DEX Data: Jupiter API / Birdeye — Trading volume tracking
- Event Indexing: Yellowstone gRPC Geyser Plugin — Real-time transaction stream

NOTE ON INDEXING: the previous draft referenced an indexing tool ("Chainbeam") that could not be verified as an existing, documented service. It has been replaced with Yellowstone gRPC, the actual, widely-used Geyser streaming interface for Solana — confirm the specific provider (self-hosted vs. Helius/Triton-hosted) with your developer before implementation.

### 1.3 Data Flow Architecture — Game Loop: Trading Streak Detection

┌─────────────────────────────────────────────────────────────┐
│ User executes DEX trade (Jupiter/Orca)                      │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│ Geyser Plugin (Yellowstone gRPC) detects transaction on-chain│
│ • Extracts wallet address → faction mapping                 │
│ • Records trade amount & direction (buy/sell)               │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│ Indexer Service processes transaction                       │
│ • Queries Jupiter API for confirmation                      │
│ • Validates trade meets profit criteria                     │
│ • Increments faction streak counter (Redis)                 │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│ Streak Monitoring Job (every 5 seconds)                     │
│ • Checks if any faction reached 50-trade streak             │
│ • Verifies milestone conditions                             │
└─────────────────────────────────────────────────────────────┘
                         ↓
         ┌───────────────┴───────────────┐
         │                               │
    NO STREAK              STREAK ACHIEVED (50+ trades)
    Continue                     ↓
    Tracking            ┌──────────────────────────┐
                        │ Trigger NFT Minting      │
                        │ • Call Metaplex standard │
                        │ • Mint Nuclear Bomb NFT  │
                        │ • Assign to faction lead │
                        └──────────────────────────┘
                                 ↓
                        ┌──────────────────────────┐
                        │ Update On-Chain State    │
                        │ • Record milestone       │
                        │ • Grant leaderboard buff │
                        └──────────────────────────┘
                                 ↓
                        ┌──────────────────────────┐
                        │ Broadcast WebSocket      │
                        │ • Notify UI of new NFT   │
                        │ • Update leaderboard     │
                        │ • Trigger celebration FX │
                        └──────────────────────────┘

### 1.4 State Management Architecture — PostgreSQL Schema

-- Factions (Nations)
CREATE TABLE factions (
  id UUID PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  regime_type VARCHAR(50),
  leader_wallet VARCHAR(44) NOT NULL,
  founding_date TIMESTAMP DEFAULT NOW(),
  total_volume BIGINT DEFAULT 0,
  current_streak INT DEFAULT 0,
  best_streak INT DEFAULT 0,
  member_count INT DEFAULT 0,
  treasury_balance BIGINT DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(leader_wallet)
);

-- Faction Members
CREATE TABLE faction_members (
  id UUID PRIMARY KEY,
  faction_id UUID NOT NULL REFERENCES factions(id),
  user_wallet VARCHAR(44) NOT NULL,
  joined_date TIMESTAMP DEFAULT NOW(),
  trades_count INT DEFAULT 0,
  total_volume BIGINT DEFAULT 0,
  win_streak INT DEFAULT 0,
  contribution_score DECIMAL(10,2) DEFAULT 0,
  UNIQUE(faction_id, user_wallet)
);

-- Trade Events (Indexed)
CREATE TABLE trade_events (
  id UUID PRIMARY KEY,
  transaction_sig VARCHAR(88) UNIQUE NOT NULL,
  user_wallet VARCHAR(44) NOT NULL,
  faction_id UUID NOT NULL REFERENCES factions(id),
  token_in VARCHAR(44),
  token_out VARCHAR(44),
  amount_in BIGINT,
  amount_out BIGINT,
  profit_loss DECIMAL(20,8),
  dex_platform VARCHAR(50),
  block_time TIMESTAMP NOT NULL,
  indexing_time TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_trade_events_wallet_faction_time
  ON trade_events (user_wallet, faction_id, block_time);

-- Streak Tracking
CREATE TABLE streak_records (
  id UUID PRIMARY KEY,
  faction_id UUID NOT NULL REFERENCES factions(id),
  streak_length INT NOT NULL,
  consecutive_wins INT NOT NULL,
  start_date TIMESTAMP NOT NULL,
  end_date TIMESTAMP,
  milestone_achieved BOOLEAN DEFAULT FALSE
);

CREATE INDEX idx_streak_records_faction_length
  ON streak_records (faction_id, streak_length);

-- NFT Rewards (Dynamic)
CREATE TABLE nft_rewards (
  id UUID PRIMARY KEY,
  faction_id UUID NOT NULL REFERENCES factions(id),
  nft_name VARCHAR(255),
  nft_type VARCHAR(50), -- 'NUCLEAR_BOMB', 'ICBM', etc.
  mint_address VARCHAR(44) UNIQUE NOT NULL,
  metadata_uri VARCHAR(500),
  awarded_date TIMESTAMP DEFAULT NOW(),
  owner_wallet VARCHAR(44),
  is_tradable BOOLEAN DEFAULT TRUE
);

CREATE INDEX idx_nft_rewards_faction_date
  ON nft_rewards (faction_id, awarded_date);

-- Leaderboard Cache
CREATE TABLE leaderboard_snapshots (
  id UUID PRIMARY KEY,
  snapshot_date TIMESTAMP DEFAULT NOW(),
  faction_id UUID NOT NULL REFERENCES factions(id),
  ranking INT,
  total_volume BIGINT,
  member_count INT,
  best_streak INT,
  nft_count INT
);

REDIS SCHEMA (Leaderboard Cache)

# Real-time faction leaderboard
leaderboard:24h:
  1. faction:alpha:volume = 5000000
  2. faction:bravo:volume = 4500000
  3. faction:charlie:volume = 3200000

# Current streaks
streak:faction:alpha = 47
streak:faction:bravo = 23

# Member count
faction:alpha:members = 342
faction:bravo:members = 287

# NFT counts
faction:alpha:nfts:count = 3
faction:alpha:nfts:list = [nft_id_1, nft_id_2, nft_id_3]

---

## Part 2: Smart Contract Boilerplate (Anchor/Rust)

### 2.1 Project Setup

# Initialize Anchor project
anchor init dctr_game --typescript

cd dctr_game

# Create program structure
mkdir programs/dctr_game/src/{instructions,state,utils,errors}

### 2.2 Cargo.toml Configuration

(STUB — not yet drafted in the source document. This section still needs actual Cargo.toml dependency contents, e.g. anchor-lang, anchor-spl, and version pins, before a developer can act on it.)

CORRECTIONS APPLIED IN THIS PASS:
1. Removed ethers.js (Ethereum-only library, incompatible with Solana) — kept @solana/web3.js + wallet-adapter only
2. Replaced all PUBKEY column types (not a real PostgreSQL type) with VARCHAR(44)
3. Split inline INDEX(...) declarations (MySQL syntax) into separate CREATE INDEX statements (valid PostgreSQL)
4. Replaced unverifiable "Chainbeam" indexing tool with Yellowstone gRPC, the real, documented Solana Geyser streaming interface
5. Standardized project naming to "DCTR" throughout (removed "Dictatorship Wars" / "The Dictator's Republic" naming drift)
