# ⚡ Realtime WebSockets Protocol & Event Specification

---

## 1. Standardized Event Envelope

All messages transmitted between clients and the KoraPlay server conform to a unified event envelope:

```json
{
  "eventId": "evt_98fbc12a-3e91",
  "eventType": "SUBMIT_ANSWER",
  "gameSessionId": "sess_102938",
  "participantId": "part_847291",
  "timestamp": 1773004500123,
  "payload": {},
  "correlationId": "req_001a"
}
```

---

## 2. Client-to-Server Events (Client ➔ Server)

| Event Name | Description | Key `payload` Fields |
|---|---|---|
| `JOIN_GAME` | Request to join a game room using a PIN | `{ "pin": "849201", "nickname": "Tariq", "avatar": "falcon" }` |
| `LEAVE_GAME` | Participant voluntarily disconnects from session | `{ "reason": "user_exit" }` |
| `SUBMIT_ANSWER` | Submit participant answer for active question | `{ "questionId": "q_42", "selectedAnswerIds": ["ans_1"], "clientTimeMs": 4200 }` |
| `REQUEST_RECONNECT` | Reconnection request following a network disconnection | `{ "participantToken": "tok_sec_991823" }` |
| `HOST_START_GAME` | Host instruction to initiate the game countdown | `{ "countdownSeconds": 5 }` |
| `HOST_NEXT_QUESTION` | Host instruction to transition to the next question | `{ "delaySeconds": 3 }` |
| `HOST_PAUSE_GAME` | Host temporarily halts the game session | `{ "reason": "host_break" }` |
| `HOST_RESUME_GAME` | Host resumes the paused game session | `{}` |
| `HOST_KICK_PLAYER` | Host ejects a participant from the room | `{ "targetParticipantId": "part_847291" }` |
| `HOST_END_GAME` | Host terminates the competition and triggers podium | `{}` |

---

## 3. Server-to-Client Events (Server ➔ Client)

| Event Name | Delivery Scope | Description & `payload` Contents |
|---|---|---|
| `GAME_JOINED` | Unicast | Confirms admission and delivers the secure `participantToken`. |
| `PLAYER_JOINED` | Room Broadcast | Notifies room participants/lobby that a new player joined. |
| `PLAYER_LEFT` | Room Broadcast | Notifies room that a player disconnected or was kicked. |
| `GAME_STARTED` | Room Broadcast | Initiates the synchronized game start countdown. |
| `QUESTION_STARTED` | Room Broadcast | Delivers active question, randomized options, and timer (zero answer hints). |
| `QUESTION_TIMER_TICK` | Room Broadcast | Periodic synchronization tick for remaining seconds (1 Hz). |
| `ANSWER_ACCEPTED` | Unicast | Confirms receipt of submitted answer and private speed bonus. |
| `QUESTION_ENDED` | Room Broadcast | Closes submission window, reveals correct answer, and presents answer distribution. |
| `LEADERBOARD_UPDATED` | Room Broadcast | Broadcasts updated top 5 standings, ranks, and score differentials. |
| `GAME_FINISHED` | Room Broadcast | Celebrates top 3 podium winners and dispatches complete session statistics. |
| `ERROR` | Unicast | Communicates error states (e.g., timeout, duplicate submission, invalid PIN). |

---

## 4. Realtime Error Codes

- `ERR_PIN_NOT_FOUND`: Game PIN does not exist or has expired.
- `ERR_NICKNAME_TAKEN`: Nickname is already in use within this session.
- `ERR_ROOM_FULL`: Session participant capacity has been reached.
- `ERR_QUESTION_EXPIRED`: Answer submitted after official timer expiration.
- `ERR_DUPLICATE_SUBMISSION`: Answer already submitted for the current question.
- `ERR_UNAUTHORIZED_HOST`: Operation restricted to session host/teacher.
