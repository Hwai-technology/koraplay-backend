# 🎮 KoraPlay — Enterprise Realtime Interactive Learning & Quiz Platform
### Built 100% natively using the Kora Programming Language & Framework

---

## 🌟 Project Vision & Overview

**KoraPlay** is an enterprise-grade, server-authoritative, real-time interactive learning and quiz platform inspired by global platforms like *Kahoot!* and *Quizizz*, engineered with a distinct technical identity and strict architectural standards built **100% natively using the Kora Programming Language**.

The primary objective is not merely a quiz application, but rather the **Definitive Real-World Benchmark** proving Kora's capabilities in tackling advanced backend engineering challenges:
- **High Concurrency:** Managing thousands of concurrent participants simultaneously via WebSockets.
- **Server-Authoritative Engine:** Computing scores, measuring millisecond-level time differentials, and preventing client-side tampering or cheating.
- **Strict Clean Architecture:** Strict layer separation (Controllers, Services, Repositories, Domain Models, Enums) compiler-enforced by Kora.
- **Reliable Hybrid Persistence:** PostgreSQL for durable, non-volatile data alongside Redis for high-speed ephemeral state management.

---

## 🏛️ System Architecture

KoraPlay adopts a **Modular Clean Monolith** architecture, where domain features are partitioned into fully isolated modules that can be extracted into microservices in the future if required:

```text
                                  ┌──────────────────────────┐
                                  │   Participants / Hosts   │
                                  │   (Web, Mobile, Screens) │
                                  └─────────────┬────────────┘
                                                │
                                HTTP REST / WS  │  Events (JSON)
                                                ▼
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                                    KORAPLAY ENGINE                                     │
│                                                                                        │
│   [Presentation Layer]                                                                 │
│     ├── REST Controllers (Auth, Quizzes, Games, Analytics)                            │
│     └── Realtime Gateway (WebSocket Pub/Sub, Room Sessions, Heartbeats)                │
│                                                                                        │
│   [Application & Domain Logic]                                                         │
│     ├── Identity & RBAC (Argon2id, JWT Rotation, Device Sessions)                      │
│     ├── Quiz & Question Domain (Snapshots, Versioning, Dynamic Question Types)         │
│     ├── Game Session State Machine (PIN Gen, Room Lifecycle, Reconnection)             │
│     ├── Scoring & Streak Engine (Deterministic decay curve, Bonus Multipliers)        │
│     └── Anti-Cheat & Rate Limiting Guards                                              │
│                                                                                        │
│   [Infrastructure Layer]                                                               │
│     ├── PostgreSQL Repository (Kora ORM, ACID Transactions, Immutable Results)        │
│     └── Redis Realtime State Hub (Room Presence, Ephemeral Game Caches)                │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Core Capabilities

### 1. Quiz & Assessment Engine
- Support for diverse question types: Single Choice, Multiple Choice, True/False, Short Answer, Ordering, Polls, and Media Questions (Images/Video/Formulas).
- **Immutable Quiz Snapshots:** Upon launching a live game session, an immutable snapshot of the quiz is captured, preventing live games from being affected by subsequent teacher edits.
- Intelligent evaluation criteria with speed decay bonuses and streak multipliers.

### 2. Live Game Sessions Engine
- Unique 6-digit Game PIN generation with collision prevention and brute-force protection.
- Instant guest participant onboarding using a nickname without requiring account registration.
- Deterministic Finite State Machine (FSM):
  `CREATED` ➔ `WAITING_FOR_PLAYERS` ➔ `STARTING` ➔ `QUESTION_ACTIVE` ➔ `QUESTION_CLOSED` ➔ `LEADERBOARD` ➔ `FINISHED`.
- High reconnection tolerance for participants and hosts without losing accumulated scores.

### 3. Security & Anti-Cheating
- **Zero-Knowledge Answer Distribution:** The correct answer is never transmitted or hinted to clients until the server officially closes the question timer.
- **Server-Authoritative Timestamping:** Answer timing tampering is prevented by calculating arrival deltas against the server clock.
- Idempotency keys and replay attack defense mechanisms.

---

## 📂 Project Directory Structure

```text
real project/koraplay/
├── kora.toml                         # Kora project manifest and dependencies
├── Dockerfile                        # Production container runtime configuration
├── docker-compose.yml                # PostgreSQL and Redis orchestration
├── docs/                             # Comprehensive architectural and technical documentation
│   ├── architecture.md               # Clean architecture design and module boundaries
│   ├── quiz-system-concept.md        # In-depth quiz and assessment engine specification
│   ├── game-engine-lifecycle.md      # Live game engine lifecycle & state machine
│   ├── websocket-protocol.md         # WebSocket protocol specification and event contracts
│   ├── database-schema.md            # Relational database schema and index optimization
│   ├── api-specification.md          # REST API v1 endpoint specifications
│   └── KORA_LANGUAGE_EVALUATION.md   # Kora language capabilities and compiler benchmarks
├── src/
│   ├── main.kora                     # Application entry point, server boot & routing
│   ├── enums/                        # System constants and domain states
│   │   ├── http_status.kora
│   │   ├── user_role.kora
│   │   ├── question_type.kora
│   │   └── session_status.kora
│   ├── models/                       # Shared DTOs and response envelopes
│   │   └── response_envelope.kora
│   ├── shared/services/              # Cross-cutting services (Logging, Crypto)
│   │   └── logger_service.kora
│   └── features/                     # Clean architecture domain feature modules
│       ├── users/                    # Identity, authentication, and RBAC
│       ├── quizzes/                  # Quiz authoring and question bank management
│       ├── games/                    # Live game sessions and room state coordination
│       └── scoring/                  # Point computation, streaks, and leaderboards
```

---

## 🛠️ Quick Start with Kora CLI

KoraPlay is built using official tools from the **Kora Toolchain (v0.2.0)**:

```bash
# 1. Type check, syntax verification, and ownership analysis
./target/release/kora check "real project/koraplay/src/main.kora"

# 2. Run automated test suite
./target/release/kora test "real project/koraplay"

# 3. Start development server
./target/release/kora run "real project/koraplay/src/main.kora"

# 4. Start development server with hot-reload
./target/release/kora watch "real project/koraplay/src/main.kora"

# 5. Generate OpenAPI 3.1 schema and Swagger UI documentation
./target/release/kora swagger "real project/koraplay/src/main.kora"
```

---

## 📈 Roadmap & Development Milestones

- [x] **Milestone 1:** Project scaffolding, Clean Architecture setup, and complete architectural documentation.
- [ ] **Milestone 2:** Authentication system, user identity, and Argon2id password security.
- [ ] **Milestone 3:** Question bank module, multi-type questions, and snapshot versioning engine.
- [ ] **Milestone 4:** Live game session engine, PIN generation, and real-time room lifecycle.
- [ ] **Milestone 5:** Real-time WebSocket gateway and event streaming for questions and timers.
- [ ] **Milestone 6:** Scoring engine, time-decay bonus calculation, and live leaderboard broadcast.
- [ ] **Milestone 7:** Advanced analytics, achievement badges, and tournament mode.
- [ ] **Milestone 8:** High-concurrency load testing, security hardening, and production deployment.
