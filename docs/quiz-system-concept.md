# 🧠 Comprehensive Architectural Concept for the Quiz & Assessment Engine

---

## 1. System Philosophy

In **KoraPlay**, a "quiz" is not treated as a trivial survey form, but rather as a **dynamic, competitive learning artifact** balancing three vital design pillars:

1. **Server-Authoritative & Deterministic Fairness:**
   The Kora backend serves as the single source of truth. Clients have no authority to compute their own scores, and under no circumstances do they receive indicators of the correct answers before the server officially closes the question timer.
2. **Cognitive Diversity in Question Types:**
   Moving beyond standard multiple-choice questions to support logical ordering, visual blur-reveal identification, short text matching, and code/math evaluations.
3. **Immutable Live Snapshots:**
   When a teacher initiates a live competition from a quiz, the system takes an instantaneous, immutable copy (`QuizVersionSnapshot`). Even if the author edits or deletes questions in the management dashboard while the competition is underway, the active game session remains fully isolated, running on the frozen version without data drift or scoring corruption.

---

## 2. Quiz Lifecycle

```text
  [ DRAFT ] ───────────► [ PUBLISHED ] ───────────► [ LIVE SNAPSHOT ]
     │                         │                           │
     ▼                         ▼                           ▼
- Add / edit questions    - Searchable & shareable      - Immutable frozen copy
- Continuous autosave     - Configure global settings   - Bound to GameSession
- Cannot run live yet     - Ready for game launches     - Preserves original scoring & state
```

### Quiz States:
- `DRAFT`: Under authoring by the teacher; hidden from live game sessions.
- `PUBLISHED`: Verified and ready for deployment; indexed by category, tags, and difficulty.
- `ARCHIVED`: Preserved for historical analytics; cannot spawn new live sessions, but past performance records remain viewable.

---

## 3. Question Types Spectrum

| Question Type | Enum Identifier | Verification Strategy | Scoring Mechanism |
|---|---|---|---|
| **Single Choice** | `SINGLE_CHOICE` | Single correct option selected from 2 to 6 choices | Full base points + speed decay bonus |
| **Multiple Choice** | `MULTIPLE_CHOICE` | Multiple correct options with distractor choices | Partial or full scoring with penalties for false positives |
| **True / False** | `TRUE_FALSE` | Binary quick decision | Standard points with condensed timer |
| **Short Answer** | `SHORT_ANSWER` | Text entry with flexible normalization (Regex / Case-Insensitive) | Full points upon verified keyword or regex match |
| **Visual Reveal** | `IMAGE_QUESTION` | Image or diagram revealed progressively via blur-to-sharp animation | Higher score for faster identification |
| **Logical Ordering** | `ORDERING` | Sequence 4 to 6 items chronologically or procedurally | Graded on positional accuracy (Levenshtein / relative order match) |
| **Live Poll** | `POLL` | Collect participant sentiment with no right or wrong answer | Flat participation points awarded to all respondents |
| **Code / Math Challenge** | `CODE_CHALLENGE` | Exact output verification for algorithms or mathematical expressions | Premium bonus for syntax precision and rapid solving |

---

## 4. Mathematical Scoring Engine

The scoring engine balances **correctness**, **reaction speed**, and **consecutive correct streaks**:

### 1. Per-Question Score Equation:
$$Score = (BasePoints \times AccuracyFactor) + TimeBonus + StreakBonus$$

### 2. Speed Bonus Decay:
Given a total question time limit $T_{total}$ (e.g., 20 seconds), and a participant response time $T_{response}$ (e.g., 4 seconds):

$$TimeBonus = \left( 1 - \frac{T_{response}}{T_{total}} \right) \times MaxTimeBonus$$

*Practical Example:*
- $BasePoints = 1000$ points.
- $MaxTimeBonus = 500$ points.
- Question Time Limit = 20 seconds.
- Participant Response Time = 5 seconds.
- $TimeBonus = (1 - 5/20) \times 500 = 0.75 \times 500 = 375$ points.
- Total Question Score = $1000 + 375 = 1375$ points.

### 3. Streak Multipliers:
To encourage sustained focus and engagement, participants receive ascending bonuses for consecutive correct answers:
- **Streak 1 - 2:** Standard points ($1.0\times$ multiplier).
- **Streak 3:** $+100$ bonus points (Fire Streak 🔥).
- **Streak 5:** $+250$ bonus points (Super Streak ⚡).
- **Streak 10:** $+500$ bonus points (Unstoppable 🏆).
- On any incorrect answer, the streak counter resets to zero immediately.

---

## 5. Anti-Cheating Architecture

### A. Zero-Knowledge Frontend Distribution
- When the server enters `QUESTION_ACTIVE` and broadcasts the question payload over WebSockets, the payload strictly includes:
  - Question identifier (`questionId`).
  - Question prompt and media asset URLs.
  - Shuffled choices with obfuscated option identifiers (unique order per participant to deter screen-peeking).
  - Time limit.
- **Under no circumstances is `is_correct` or any hint transmitted in this payload.**

### B. Server-Authoritative Timestamping
- The server does not trust client device clocks.
- When broadcasting a question, the server records `question_started_at` with microsecond precision.
- Upon receiving a `SUBMIT_ANSWER` payload, the server computes:
  $$Elapsed = ReceivedAt - StartedAt$$
- If the submission arrives after the official deadline plus a strict network latency tolerance window ($\Delta t \le 200\text{ms}$), the answer is rejected and recorded as a `TIMEOUT`.

### C. Idempotency Protection
- Double submissions from the same participant for the same question are rejected at both the application layer and via the database unique index:
  `UNIQUE(session_id, participant_id, question_id)`
  This prevents race conditions, multi-click exploits, and replay attempts.

---

## 6. Post-Game Analytics

Following game completion, comprehensive metrics are generated for instructors:
1. **Question Difficulty Index:** The percentage of participants who answered correctly.
2. **Distractor Analysis:** Frequency distribution of incorrect choices selected, revealing widespread student misconceptions.
3. **Mean Reaction Time:** Identification of questions requiring deeper cognitive processing versus immediate recall.
4. **Leaderboard Fluctuations:** Trajectory graphing rank volatility from the opening round to the final question.
