# Sentinel Network

**Status:** Draft
**Last Updated:** 2026-01-02

---

## Why Sentinel Exists

The Laniakea infrastructure (PAU pattern, Configurator Unit, LCTS) provides the on-chain machinery for capital deployment. But machinery needs operators. Sentinel is the answer to: **who operates this machinery, and how?**

Key problems Sentinel solves:
1. **Continuous operation** — LCTS queues need settle() called every epoch. PAUs need monitoring. Strategies need execution.
2. **Risk visibility** — Primes and Halos hold diverse assets. Someone must verify solvency, calculate risk capital requirements, detect anomalies.
3. **Governance efficiency** — Manual operations don't scale. 1 new Prime per week × 1 new Halo per week requires autonomous systems.
4. **Commercial incentives** — Private actors need ways to generate alpha without exposing proprietary strategies.

Sentinel is the answer: a network of autonomous agents that hold pBEAMs and operate within governance-defined bounds.

---

## Core Concepts

### Sentinel = Interface Specification

A Sentinel is not just software—it's an **interface specification** that any conforming system can implement.

**Short-term:** Shared open-source implementations. Everyone uses the same codebase for transparency, security auditing, and rapid iteration.

**Long-term:** Diverse implementations (different languages, different architectures, AGI systems) that conform to Sentinel interface specs. This creates maximum resilience by eliminating single-point software failures.

### Sentinel Interfaces (stl-*) vs Sentinel Toolkit (stk-*)

The architecture distinguishes between:

**Sentinel Interfaces (stl-*)** — Continuously running AGI systems that hold pBEAMs:
- Long-running processes
- Make autonomous decisions
- Execute operations directly on-chain

**Sentinel Toolkit (stk-*)** — Batch/utility operations bundled into governance Sentinels:
- One-time or periodic operations
- Used by governance Sentinels (stl-gen, stl-prime, stl-halo, stl-exec) as tools
- No autonomous decision-making

---

## Sentinel Taxonomy

| Category | Sentinels | Purpose |
|----------|-----------|---------|
| **Governance** | stl-gen, stl-prime, stl-halo, stl-exec | Layer/entity governance, settlement, verification, staking, buybacks |
| **Operations** | stl-unit, stl-erc, stl-auction, stl-exchange | Per-unit/per-token LCTS operation, resource allocation auctions, exchange orderbook |
| **Execution** | stl-base, stl-stream, stl-trade | Strategy execution (public baseline, private optimizer, private trader) |
| **Halo Bridge** | stl-control, stl-connect, stl-extend, stl-identity | Real-world asset interface, data bridging, remote operation, identity registry |
| **Infrastructure** | stl-synome | Synome node operation, transaction propagation |

---

## Governance Sentinels

### stl-gen (Generator Governance Sentinel)

**Level:** Generator layer
**Operator:** Accordant GovOps

Responsibilities:
- Generator-level governance operations
- Settlement (uses stk-settle)
- Verification (uses stk-verify)

### stl-prime (Prime Governance Sentinel)

**Level:** Per Prime
**Operator:** Accordant GovOps

Responsibilities:
- Settlement (uses stk-settle)
- Verification (uses stk-verify)
- Carry calculation (uses stk-carry)
- Staking rewards distribution
- Token buyback of Prime agent tokens
- Routine governance and artifact maintenance

### stl-halo (Halo Governance Sentinel)

**Level:** Per Halo
**Operator:** Accordant GovOps

Responsibilities:
- Settlement (uses stk-settle)
- Verification (uses stk-verify)
- Staking rewards distribution
- Token buyback of Halo agent tokens
- Routine governance and artifact maintenance

### stl-exec (Executor Governance Sentinel)

**Level:** Per Executor
**Operator:** Executor team

Responsibilities:
- Executor-level governance operations
- **When Executor has Core Council seat:** Synome override/privileged write permissions
- Can write directly to CC Synome with authoritative updates

---

## Operations Sentinels

### stl-unit (Halo Unit Sentinel)

**Level:** Per Halo Unit
**Operator:** Halo GovOps / Asset Manager

Responsibilities:
- Asset data reporting to Synome
- Risk calculations for the unit's positions
- Listening for deployment intents from stl-prime/stl-halo
- **Operating the unit's LCTS-pBEAM** — calls settle() on SubscribeQueue/RedeemQueue, determines capacity allocation per epoch

Connection to LCTS:
- Holds the pBEAM that can call settle() on the unit's LCTS queues
- Decides how much subscribe/redeem capacity to allocate each settlement
- Operates within bounds set by cBEAM guardrails

### stl-erc (External Risk Capital Sentinel)

**Level:** Per ERC token
**Operator:** Dedicated team per token

ERC tokens require dedicated Sentinels due to:
- **High hardness requirement** — External Risk Capital absorbs losses; errors are unacceptable
- **Low error tolerance** — Mistakes affect external capital providers, not just internal systems

Instances:
- **Generator level:** srUSDS (one stl-erc for senior risk capital across all USDS)
- **Prime level:** TEJRC, TISRC (one stl-erc per Prime's junior/isolated senior risk tokens)

Responsibilities:
- Operating the token's LCTS-pBEAM
- Managing exchange rate updates
- Coordinating with Holding System (for srUSDS)
- Yield distribution and haircut application
- Coordinating with stl-auction for settlement timing

### stl-auction (Auction Sentinel)

**Level:** Per Generator
**Operator:** Accordant GovOps

Runs sealed-bid auctions for resource allocation. New Sentinel type introduced for weekly settlement cycle.

Responsibilities:
- **OSRC Auction** — Allocate Senior Risk Capital capacity to Primes (weekly, uniform-price)
- **SPTP Bucket Auction** — Allocate SPTP capacity reservations to Primes (per bucket, uniform-price)
- **Bid collection** — Receive and store sealed bids from stl-base instances
- **Settlement coordination** — Trigger stl-erc settlement after auction completion

Architecture:
- **Bid Database** — Private storage for sealed bids until settlement
- **Auction Engine** — Matching algorithm (sort by price, match until capacity exhausted)
- **Settlement Coordinator** — Publishes results, triggers downstream settlement

Security:
- Bids signed by stl-base private keys (authenticity)
- Sealed until settlement (no front-running)
- Deterministic matching (verifiable results)

See `weekly-settlement-cycle.md` for full auction mechanics.

### stl-exchange (Exchange Orderbook Sentinel)

**Level:** Per Exchange Halo
**Operator:** Exchange Halo GovOps

Operates the off-chain orderbook and matching engine for an Exchange Halo. Enables fair, MEV-free trading with on-chain intent settlement.

Components:
- **Order Gateway** — Receives orders, validates, rate limits
- **Matching Engine** — Price-time priority matching, fair execution
- **Settlement Builder** — Creates batches for on-chain settlement

Responsibilities:
- **Order lifecycle** — Accept, validate, place, match, cancel orders
- **Fair matching** — Price-time priority, no front-running or preferential treatment
- **Settlement batching** — Group matched trades for efficient on-chain settlement
- **Identity verification** — Check Identity Network attestations for restricted markets
- **Cancellation handling** — Remove cancelled orders, delete unfilled intents

Security:
- Off-chain orderbook eliminates MEV
- Centralized matching enables deterministic fairness
- On-chain settlement preserves self-custody
- Auditable match records for verification

See `sky-intents.md` for full Exchange Halo mechanics.

---

## Execution Sentinels

### stl-base (Baseline Execution Sentinel)

**Level:** Per Prime
**Operator:** Accordant GovOps
**Access:** Public / Open Source

The "public autopilot" — the only entity holding keys to the Execution Engine.

Responsibilities:
- **Execution Engine holder** — Controls BEAMs, executes transactions
- **Base Strategy execution** — Runs the public strategy defined in Prime Artifact
- **Fail-safe** — If stl-stream fails or sends invalid data, automatically reverts to Base Strategy
- **Counterfactual simulation** — Continuously simulates "what would we have earned with just the Base Strategy?" for carry comparison

Three parallel processes:
1. **Process A: Counterfactual Simulation** — Virtual simulation of Base Strategy, uploads results to Synome
2. **Process B: Streaming Monitor** — Receives intent from stl-stream, validates against Risk Tolerance Interval, executes if compliant
3. **Process C: Active Management** — Engages when stl-stream disconnects or sends invalid data

### stl-stream (Proprietary Optimization Sentinel)

**Level:** Per Ecosystem Actor
**Operator:** Ecosystem Actors (DevCos, Trading Firms)
**Access:** Private / Proprietary

The commercial engine for alpha generation. Does NOT have an Execution Engine.

Responsibilities:
- Ingests public data + proprietary data sources (off-chain order books, sentiment analysis, credit models)
- Streams trading **intent** (not transactions) to stl-base
- Earns carry only when outperforming the Base Strategy

Incentive structure:
- Carry = (Actual PnL - Simulated Baseline PnL) × Performance Fee Ratio
- No carry if underperforming baseline
- Streaming Accord defines performance fee ratio

### stl-trade (Proprietary Trader Sentinel)

**Level:** Per Ecosystem Actor
**Operator:** Ecosystem Actors
**Access:** Private / Proprietary

Like stl-base but for assets directly owned by private entities (via multisig), not Prime capital.

- **Direct Execution** — Holds keys to assets, executes on external DeFi platforms (Spark, Aave, Morpho)
- **Data Synergy** — Integrates with stl-stream for harmonized strategies
- **Commercial Autonomy** — Private firms deploy Sentinel stack for their own operations
- **Sky Ecosystem Synergy** — Deep knowledge of Sky assets (SKY, Prime tokens, Halo tokens, Risk Capital tokens)

---

## Halo Bridge Sentinels

### stl-control (Halo Bridge Sentinel)

**Operator:** Ecosystem Actors contributing to Halos
**Purpose:** Translation layer between Sentinel and real-world asset systems

- Connects to proprietary data feeds and control systems of Halo's underlying assets
- Can operate as standalone auditing system or autonomous algorithmic manager
- Foundation for stl-connect and stl-extend

### stl-connect (Secure Data Link Sentinel)

**Purpose:** Private data bridging without public leakage

- Connects stl-control to stl-stream or stl-trade via shared data infrastructure
- Shares proprietary data (potentially anonymized/aggregated) from Halo to Streamer
- Enables information advantage while preserving privacy
- Requires high-trust off-chain legal agreements

### stl-extend (Remote Operator Sentinel)

**Purpose:** Active control delegation from Sentinel to off-chain systems

- Connects stl-stream or stl-trade to stl-control
- Allows Sentinel to operate assets outside the blockchain
- Example: Market Making Halo allowing Sentinel to execute trades on CEX
- Requires robust legal frameworks

### stl-identity (Identity Registry Sentinel)

**Level:** Per Identity Network
**Operator:** Identity Network GovOps / Legal Entity Operations

Manages an Identity Network's simple on-chain address list. Bridges between off-chain KYC (stored in Synome) and on-chain membership checks.

Flow:
```
Legal Entity → Synome → stl-identity → On-chain Registry
   (KYC)      (state)    (executor)      (simple list)
```

Responsibilities:
- **Add members** — When Synome shows new valid attestation, call `add(address)` on-chain
- **Remove members** — When Synome shows expiry/revocation, call `remove(address)` on-chain
- **Expiry monitoring** — Watch Synome for upcoming expiries, remove when they hit
- **Batch updates** — Process multiple adds/removes efficiently

The on-chain contract is minimal — just `mapping(address => bool)` and `isMember()`. All complexity (attestation types, expiry dates, verification levels) lives in Synome.

Security:
- **Sole writer** — Only stl-identity can modify the on-chain registry
- **Synome as source of truth** — stl-identity only acts on Synome state
- **Audit trail** — All changes logged in Synome
- **Rate limits** — Maximum updates per period

See `identity-network.md` for full Identity Network specification.

---

## Infrastructure Sentinels

### stl-synome (Synome Node Sentinel)

**Level:** Per operator (GovOps, Ecosystem Actor, Core Council)
**Operator:** Anyone running a Synome node

Responsibilities:
- Runs and serves Synome database to other Sentinels
- Propagates transactions to other stl-synome nodes
- Syncs with CC Synome (authoritative source)
- Validates cryptographic signatures on incoming transactions

---

## Sentinel Toolkit

### stk-verify (Universal Auditor)

**Purpose:** Independent verification and risk consensus

Operations:
- Cross-checks self-reported data from Halos against on-chain observations
- Compares "what the Halo claims" vs "what the Prime sees"
- Flags anomalies and discrepancies
- Calculates risk metrics: CRR, TRRC, TRC, Encumbrance Ratio

Used by: stl-gen, stl-prime, stl-halo, stl-exec

### stk-settle (Accountant)

**Purpose:** Financial settlement and enforcement

Operations:
- Calculates monthly PnL
- Calculates interest payable: Average Base Rate × Average Outstanding Debt
- Historical penalty enforcement for CRR breaches

Used by: stl-gen, stl-prime, stl-halo

### stk-carry (Incentive Engine)

**Purpose:** Performance verification and fee distribution

Operations:
- Compares actual returns (stl-stream strategy) vs simulated returns (stl-base baseline)
- Calculates carry payments per Streaming Accord
- Facilitates fee transfer to Ecosystem Actors

Used by: stl-prime

---

## Synome Architecture

The Synome is Sky's shared brain — a decentralized database for operational data and artifacts.

### Transaction-Based State

- **Append-only transaction log** with cryptographic signatures
- State is built from transactions, not stored directly
- No finality concern — eventually consistent via gossip propagation
- Like a blockchain, but without consensus on ordering

### CC Synome (Core Council Synome)

The authoritative source of truth:
- Directly updated by Core Council decisions
- Other stl-synome nodes sync from it
- GovOps operational data writes directly to CC Synome

### Write Flows

| Writer | Writes to | Propagates to |
|--------|-----------|---------------|
| Core Council decisions | CC Synome | → all stl-synome nodes |
| GovOps operational data | CC Synome directly | → all stl-synome nodes |
| stl-prime/stl-halo/stl-unit | Via their GovOps | → CC Synome → all nodes |
| stl-exec (with CC seat) | CC Synome with override permissions | → all nodes |

### Sync Model

- stl-synome nodes share transactions with each other
- Valid cryptographic signatures → incorporate transaction
- CC Synome is authoritative; other nodes sync from it
- Eventually all nodes converge to same state

### Data Infrastructure Requirements

The underlying system must provide:
- **Verifiable computation** — Full audit trails for all calculations
- **Structured data versioning** — Track changes to artifacts, parameters, strategies
- **Decentralized storage** — No single point of failure
- **Cross-actor data interchange** — Primes, Halos, Ecosystem Actors share data securely
- **Traceable provenance** — Know exactly where any data point came from

---

## Other Infrastructure Components

### Execution Engine

Component held by stl-base that controls BEAMs and executes transactions.

- Only stl-base holds this
- stl-stream cannot execute directly—only stream intent
- Ensures all execution goes through public, auditable code

### Streaming Accord

Smart contract framework governing the stl-stream ↔ stl-base relationship.

- Defines Risk Tolerance Interval for streamed instructions
- Specifies Performance Fee Ratio for carry calculation
- Governs termination conditions

---

## Connection to Laniakea Infrastructure

### PAU Pattern

Sentinels operate PAUs within governance-approved bounds:
- stl-gen/stl-prime/stl-halo hold cBEAMs to configure their PAUs
- stl-unit/stl-erc hold pBEAMs to operate LCTS queues
- Rate limits constrain all operations

### Configurator Unit

- aBEAM (Core Council) → grants cBEAMs to GovOps
- cBEAM (GovOps/stl-gen/stl-prime/stl-halo) → sets guardrails for pBEAMs
- pBEAM (stl-unit/stl-erc/stl-base) → directly executes on contracts

SORL constraint (max 20% increase per 18h) applies to all rate limit changes.

### LCTS

stl-unit and stl-erc are the operators of LCTS queues:
- Call settle() each epoch
- Determine subscribe/redeem capacity allocation
- Set exchange rates (for ERC tokens)
- Operate within cBEAM-defined bounds

---

## Minimum Viable Sentinel (MVS) Phased Rollout

The immediate goal: service all Primes with a unified open-source solution.

### Phase 1: Simple Verify

Standalone, hardcoded version of stk-verify for initial Primes.

- Position scraping from on-chain data
- Hardcoded risk model calculations
- Hardcoded technical risk parameters (from GitHub code reviews, audits)
- Output: v0.1 Dashboard showing CRR, TRRC, TRC, Encumbrance Ratio

### Phase 2: Simple Settlement (Narrow)

Add stk-settle for Prime1 and Prime2.

- Monthly PnL calculation
- Interest payable calculation
- Relies on Phase 1 scraping infrastructure
- Hardcoded logic tailored to Prime1/Prime2

### Phase 3: Simple Settlement (Full)

Extend stk-settle to all Primes (Spark, Grove, Obex, Keel, etc.).

- Unified accounting layer across all Primes
- Eliminates fragmented reporting standards
- Still using legacy scraping methods

### Phase 4: Data Integration

Introduce Synome and Halo self-reporting.

**Traceable Verification:**
- stk-verify runs through data infrastructure with full audit trails
- Every calculation step recorded
- Full provenance tracking

**Shared Brain:**
- Synome comes online as central database
- GovOps deposit verification results into Prime Artifacts
- Standardized records for every Prime

**Halo Self-Reporting:**
- stl-unit begins reporting via new reporting logic
- Cross-checking: compare self-reported data against independent observations

### Phase 5: Core Halos & Standardization

Address underlying data fragmentation.

**Core Halos:**
- Core Council creates interim wrappers for every collateral type
- Standardized Artifact for each asset type (WBTC, ETH-Lido, etc.)
- Core Council operates reporting for these Core Halos

**Result:**
- Primes subscribe to standardized data streams
- Eliminates custom scraper debt
- stk-verify can assess any portfolio from standardized inputs

---

## Key Risk Metrics

| Metric | Definition |
|--------|------------|
| **CRR** | Capital Ratio Requirement — risk weight per position |
| **TRRC** | Total Required Risk Capital — sum of required safety capital |
| **TRC** | Total Risk Capital — actual safety capital held |
| **Encumbrance Ratio** | TRC / TRRC — critical health score |

---

## Glossary Alignment

See master doc glossary for authoritative definitions:
- Sentinel, Sentinel Interface (stl-*), Sentinel Toolkit (stk-*)
- stl-gen, stl-prime, stl-halo, stl-exec, stl-unit, stl-erc, stl-base, stl-stream, stl-trade
- stl-control, stl-connect, stl-extend, stl-synome
- stk-verify, stk-settle, stk-carry
- Synome, CC Synome, Data Infrastructure
- Execution Engine, Base Strategy, Streaming Accord
- PAU, BEAM, pBEAM, cBEAM, aBEAM
- LCTS, LCTS-pBEAM, SubscribeQueue, RedeemQueue

---

*This document defines Sentinel Network requirements and architecture. For implementation details, see phase-specific specifications.*
