# 📡 REST API v1 Specification

All REST API responses follow the standard Kora response envelope: `ResponseEnvelope<T>`.

**Base URL:** `/api/v1`

---

## 1. Authentication & Identity

| Method | Endpoint | Description | Access / Authorization |
|---|---|---|---|
| `POST` | `/auth/register` | Register a new user account (Teacher / Admin) | Public |
| `POST` | `/auth/login` | Authenticate user and issue Access & Refresh Tokens | Public |
| `POST` | `/auth/refresh` | Rotate and refresh an Access Token | Refresh Token Bearer |
| `POST` | `/auth/logout` | Revoke current user session and invalidate token | Authenticated User |
| `GET` | `/auth/me` | Fetch current user profile and role permissions | Authenticated User |

---

## 2. Quizzes & Question Bank Management

| Method | Endpoint | Description | Access / Authorization |
|---|---|---|---|
| `POST` | `/quizzes` | Create a new quiz draft | Teacher / Admin |
| `GET` | `/quizzes` | List teacher's quizzes with filtering and pagination | Teacher / Admin |
| `GET` | `/quizzes/{id}` | Retrieve quiz details and question list | Owner / Public if published |
| `PUT` | `/quizzes/{id}` | Update quiz metadata and settings | Owner Only |
| `DELETE` | `/quizzes/{id}` | Delete quiz and its questions | Owner Only |
| `POST` | `/quizzes/{id}/publish` | Publish quiz and generate an immutable version snapshot | Owner Only |
| `POST` | `/quizzes/{id}/duplicate` | Clone an existing quiz into a new editable draft | Teacher |
| `POST` | `/quizzes/{id}/questions` | Add a new question to the quiz | Owner Only |
| `PUT` | `/quizzes/{id}/questions/{qId}` | Update an existing question and its answer choices | Owner Only |
| `DELETE` | `/quizzes/{id}/questions/{qId}` | Remove a question from the quiz | Owner Only |

---

## 3. Live Game Sessions Engine

| Method | Endpoint | Description | Access / Authorization |
|---|---|---|---|
| `POST` | `/games` | Initialize a live game session and generate unique PIN | Teacher |
| `GET` | `/games/{id}` | Retrieve game session state and joined participant count | Host Only |
| `POST` | `/games/join` | Validate game PIN and nickname before WebSocket connect | Public (Participants) |
| `GET` | `/games/{id}/leaderboard` | Fetch current top leaderboard standings | Host |
| `GET` | `/games/{id}/results` | Fetch final game summary, analytics, and podium results | Host |
