# 🗄️ Relational Database Schema & Data Architecture

**KoraPlay** utilizes **PostgreSQL** via Kora ORM for persistent data storage, guaranteeing ACID transactions, concurrency constraints, and referential integrity.

---

## 1. Core Entities & Relational Tables

### A. Users & Authentication (`users` & `refresh_tokens`)
- **`users`**:
  - `id`: `BIGSERIAL PRIMARY KEY`
  - `email`: `VARCHAR(255) UNIQUE NOT NULL`
  - `password_hash`: `VARCHAR(255) NOT NULL` (Argon2id)
  - `full_name`: `VARCHAR(128) NOT NULL`
  - `role`: `VARCHAR(32) NOT NULL` (`ADMIN`, `TEACHER`, `STUDENT`)
  - `created_at`: `TIMESTAMP WITH TIME ZONE DEFAULT NOW()`
  - `updated_at`: `TIMESTAMP WITH TIME ZONE DEFAULT NOW()`
- **`refresh_tokens`**:
  - `id`: `BIGSERIAL PRIMARY KEY`
  - `user_id`: `BIGINT REFERENCES users(id) ON DELETE CASCADE`
  - `token_hash`: `VARCHAR(255) UNIQUE NOT NULL`
  - `expires_at`: `TIMESTAMP WITH TIME ZONE NOT NULL`
  - `revoked`: `BOOLEAN DEFAULT FALSE`

---

### B. Quizzes & Question Bank (`quizzes`, `quiz_versions`, `questions`, `question_answers`)
- **`quizzes`**:
  - `id`: `BIGSERIAL PRIMARY KEY`
  - `owner_id`: `BIGINT REFERENCES users(id) ON DELETE RESTRICT`
  - `title`: `VARCHAR(255) NOT NULL`
  - `description`: `TEXT`
  - `category`: `VARCHAR(64) DEFAULT 'General'`
  - `status`: `VARCHAR(32) DEFAULT 'DRAFT'` (`DRAFT`, `PUBLISHED`, `ARCHIVED`)
  - `is_public`: `BOOLEAN DEFAULT TRUE`
  - `created_at`: `TIMESTAMP WITH TIME ZONE DEFAULT NOW()`
- **`quiz_versions`**:
  - `id`: `BIGSERIAL PRIMARY KEY`
  - `quiz_id`: `BIGINT REFERENCES quizzes(id) ON DELETE CASCADE`
  - `version_number`: `INT NOT NULL`
  - `snapshot_data`: `JSONB NOT NULL` (Complete immutable freeze of all questions and choices)
  - `published_at`: `TIMESTAMP WITH TIME ZONE DEFAULT NOW()`
- **`questions`**:
  - `id`: `BIGSERIAL PRIMARY KEY`
  - `quiz_id`: `BIGINT REFERENCES quizzes(id) ON DELETE CASCADE`
  - `question_type`: `VARCHAR(32) NOT NULL` (`SINGLE_CHOICE`, `MULTIPLE_CHOICE`, `TRUE_FALSE`, `ORDERING`, ...)
  - `prompt`: `TEXT NOT NULL`
  - `media_url`: `VARCHAR(512)`
  - `time_limit_seconds`: `INT DEFAULT 20`
  - `base_points`: `INT DEFAULT 1000`
  - `sort_order`: `INT NOT NULL`
- **`question_answers`**:
  - `id`: `BIGSERIAL PRIMARY KEY`
  - `question_id`: `BIGINT REFERENCES questions(id) ON DELETE CASCADE`
  - `answer_text`: `TEXT NOT NULL`
  - `is_correct`: `BOOLEAN NOT NULL`
  - `sort_order`: `INT NOT NULL`

---

### C. Live Game Sessions & Participant State (`game_sessions`, `game_participants`, `participant_answers`)
- **`game_sessions`**:
  - `id`: `BIGSERIAL PRIMARY KEY`
  - `quiz_version_id`: `BIGINT REFERENCES quiz_versions(id) ON DELETE RESTRICT`
  - `host_id`: `BIGINT REFERENCES users(id) ON DELETE RESTRICT`
  - `game_pin`: `VARCHAR(6) NOT NULL`
  - `status`: `VARCHAR(32) NOT NULL` (`CREATED`, `WAITING`, `ACTIVE`, `FINISHED`, `CANCELLED`)
  - `current_question_index`: `INT DEFAULT 0`
  - `started_at`: `TIMESTAMP WITH TIME ZONE`
  - `finished_at`: `TIMESTAMP WITH TIME ZONE`
  - **Conditional Unique Index:**
    `CREATE UNIQUE INDEX idx_unique_active_pin ON game_sessions(game_pin) WHERE status NOT IN ('FINISHED', 'CANCELLED');`
- **`game_participants`**:
  - `id`: `BIGSERIAL PRIMARY KEY`
  - `session_id`: `BIGINT REFERENCES game_sessions(id) ON DELETE CASCADE`
  - `nickname`: `VARCHAR(64) NOT NULL`
  - `participant_token`: `VARCHAR(128) UNIQUE NOT NULL`
  - `total_score`: `INT DEFAULT 0`
  - `current_streak`: `INT DEFAULT 0`
  - `is_connected`: `BOOLEAN DEFAULT TRUE`
  - `joined_at`: `TIMESTAMP WITH TIME ZONE DEFAULT NOW()`
  - **Unique Constraint:** `UNIQUE(session_id, nickname)`
- **`participant_answers`**:
  - `id`: `BIGSERIAL PRIMARY KEY`
  - `session_id`: `BIGINT REFERENCES game_sessions(id) ON DELETE CASCADE`
  - `participant_id`: `BIGINT REFERENCES game_participants(id) ON DELETE CASCADE`
  - `question_id`: `BIGINT REFERENCES questions(id) ON DELETE CASCADE`
  - `submitted_answer_ids`: `JSONB NOT NULL`
  - `is_correct`: `BOOLEAN NOT NULL`
  - `points_awarded`: `INT DEFAULT 0`
  - `time_taken_ms`: `INT NOT NULL`
  - `submitted_at`: `TIMESTAMP WITH TIME ZONE DEFAULT NOW()`
  - **Strict Unique Constraint (Anti-Duplicate Submission):**
    `UNIQUE(session_id, participant_id, question_id)`

---

## 2. High-Throughput Performance Indexes

```sql
-- Fast session lookup by PIN and active status
CREATE INDEX idx_game_sessions_pin_status ON game_sessions(game_pin, status);

-- Fast retrieval of session participants and leaderboard ordering
CREATE INDEX idx_participants_leaderboard ON game_participants(session_id, total_score DESC);

-- Fast evaluation of answers for a specific session question at timer expiration
CREATE INDEX idx_answers_session_question ON participant_answers(session_id, question_id);
```
