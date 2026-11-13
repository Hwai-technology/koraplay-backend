# 🕹️ Game Engine & Live Session Lifecycle

---

## 1. Finite State Machine (FSM)

Every game session in **KoraPlay** adheres to a deterministic lifecycle managed by a server-authoritative state machine implemented in Kora:

```text
       ┌───────────┐
       │  CREATED  │
       └─────┬─────┘
             │ (Generate PIN & open lobby)
             ▼
  ┌───────────────────────┐
  │  WAITING_FOR_PLAYERS  │ ◄─────── (Players join via PIN)
  └──────────┬────────────┘
             │ (Host clicks 'Start Game')
             ▼
       ┌───────────┐
       │ STARTING  │ (Introductory countdown: 3.. 2.. 1)
       └─────┬─────┘
             │
             ├────────────────────────────────────────┐
             ▼                                        │ (When next questions exist)
  ┌───────────────────────┐                           │
  │    QUESTION_ACTIVE    │ ◄─────────────────────────┘
  └──────────┬────────────┘
             │ (Timer expires or all participants have answered)
             ▼
  ┌───────────────────────┐
  │    QUESTION_CLOSED    │ (Calculate scores + reveal correct answer)
  └──────────┬────────────┘
             │
             ▼
  ┌───────────────────────┐
  │      LEADERBOARD      │ (Display top rankings & broadcast updates)
  └──────────┬────────────┘
             │ (If this was the final question)
             ▼
       ┌───────────┐
       │ FINISHED  │ (Podium celebration & persist final results to DB)
       └───────────┘
```

---

## 2. Game PIN Generation Algorithm & Collision Avoidance

The Game PIN is a 6-digit numeric code that enables fast guest access without requiring prior account registration.

### Design Requirements:
1. **Entropy & Unpredictability:** 6 randomized digits excluding trivial sequential or repeating patterns (e.g., `123456`, `000000`).
2. **Absolute Uniqueness:** 100% unique across all currently active sessions.
3. **Automatic Recycling:** As soon as a session transitions to `FINISHED` or `CANCELLED` (or expires), the PIN is safely released back to the available pool.
4. **Brute-Force Defense:** Rate-limiting wrong PIN attempts per IP address (maximum 5 attempts per minute).

---

## 3. Room Management & Realtime Concurrency (Actor Model)

KoraPlay utilizes an actor-based concurrency model with Kora's native WebSocket engine:

- **Isolated Room Entity:**
  Each active game session is represented as an in-memory actor room identified by `room_{sessionId}`.
- **Broadcast Channel:**
  Used to publish synchronized public events to all connected clients in the room (e.g., question start, timer ticks, leaderboard reveal).
- **Unicast Channel:**
  Used to deliver private, client-specific events (e.g., answer receipt acknowledgment, personal score breakdown, and individual speed bonus without revealing them to peers).

---

## 4. Reconnection Tolerance & Graceful Recovery

Mobile network instability (4G/Wi-Fi drops) is a common challenge in live multiplayer games:

- Upon joining, the server issues a cryptographically secure `participantToken` to the client.
- If the WebSocket connection drops unexpectedly:
  1. The participant is not immediately evicted; their status transitions to `DISCONNECTED` with a 60-second grace period.
  2. The server retains their accumulated score and streak state in memory and Redis.
  3. Upon reconnecting, the client issues a `REQUEST_RECONNECT` event with the `participantToken`.
  4. The server validates the token and re-associates the new WebSocket socket with the existing participant record.
  5. The server immediately dispatches a `STATE_SYNC` packet containing the current question, elapsed time, and state so the participant seamlessly resumes without score loss.
