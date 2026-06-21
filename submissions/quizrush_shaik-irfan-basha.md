# QuizRush ⚡️

---

## Attendee/Team Details

**Name:** Shaik Irfan Basha
**GitHub Username:** IrfanCodesBTW
**LinkedIn Profile:** https://www.linkedin.com/in/shaik-irfan-basha
**GitHub Project Repository:** https://github.com/IrfanCodesBTW/QuizRush

---

## Problem Statement Selected

```txt
Problem statement 61: Gaming — Kahoot-like Quiz Game
```

---

## Project Description

QuizRush is a blazing-fast, real-time multiplayer quiz game (Kahoot-style) designed as a high-performance web application. 

* **What is the project about?** It is an interactive, real-time multiplayer quiz arena where players compete in quiz games with instant latency synchronization and real-time leaderboard updates.
* **Who is it for?** It is built for hosts, educators, event organizers, and communities who want to run live quiz arenas with low latency and without fearing server crash bottlenecks.
* **What problem does it solve?** Typical real-time quiz systems fail or lag under heavy database pressure or networking hiccups. Storing live states directly in transactional databases leads to bottlenecks, transaction locks, and delays. QuizRush solves this by running all hot paths (live gameplay, leaderboards, timers, answer submissions, presence logs) on a high-throughput, low-latency Valkey caching layer while keeping PostgreSQL as a durable but non-blocking persistent secondary store.
* **How does it help the user?** It allows hosts to spin up rooms instantly and lock/unlock them. Players can join frictionlessly as guest users with zero sign-up requirements. Connected clients receive sub-millisecond updates via WebSockets backed by Valkey Pub/Sub. Hosts also get AI-powered analytics and engagement commentary, as well as a live Valkey streams log directly inside their dashboard console to observe underlying cache primitives (Hashes, Sets, Sorted Sets, Streams, Pub/Sub, TTL, and Atomic updates) in real time.

---

## Approach

* **How we understood the problem:** Real-time multiplayer gaming requires sub-millisecond data synchronization. Relational databases like Postgres are too slow for high-frequency reads/writes like ticking timers, concurrent answer submissions, and instant ranking. We needed a split architecture: Valkey for volatile, high-frequency, low-latency live operations, and PostgreSQL for persistent user and room history records.
* **What user flow we designed:**
  1. A host registers/logs in or plays as a guest.
  2. The host creates a quiz room, generating a unique 4-letter room code (e.g., `QR-ABCD`). This sets up room state hashes, set trackers, and starts a TTL expiry in Valkey.
  3. Players join the lobby via the room code (supporting guest players).
  4. The host starts the game: questions are broadcasted, and a real-time countdown timer starts.
  5. Players submit answers in real-time, receiving points based on answer correctness, speed, and streaks.
  6. The host reveals the answer: answer distributions and AI analytics insights are displayed on client dashboards.
  7. The host advances to the next question or the leaderboard podium.
  8. Once the game ends, final scores are saved to PostgreSQL (if available), top achievements are unlocked, and a summary is presented.
* **What features we decided to build:**
  * **Dual-Tier State Resilience:** If the Postgres database crashes, the application automatically degrades to a Valkey-only `Fallback` mode. Stored users can still log in using Valkey caches, new guest sessions can be spun up, rooms created, and games played to completion without interruption.
  * **Valkey Console & Activity Log:** A visual pane that prints real-time Valkey operations (HSET, SADD, ZINCRBY, XADD, etc.) and lists active primitives, showing judges exactly how Valkey is utilized under the hood.
  * **AI-Powered Agent Insights:** Simulated agents (Orchestrator, Analytics, and Engagement agents) that analyze player statistics and answer patterns to provide dynamic text-commentary.
* **How AI is used in our solution:**
  * Automated AI agents assess the difficulty of questions, generate commentary on player accuracy (e.g. "80% of players got this right!"), highlight speed streak accomplishments, and reward achievements ("Quiz Champion") based on real-time room data.
* **What makes our approach useful or different:**
  * **Resilience Testing:** We wrote an E2E test suite using Playwright and Docker that literally kills the Postgres container mid-game to verify that the game continues running smoothly in fallback mode.

---

## Tech Stack and Tools Used

**Frontend:** Next.js (App Router), React 19, Tailwind CSS, Framer Motion, Recharts, Canvas-Confetti, React-PDF
**Backend:** Node.js, Express, Socket.IO, TypeScript (tsx execution engine)
**Database:** PostgreSQL (pg client Pool)
**Caching/Real-time:** Valkey (ioredis client)
**AI Tools/API:** Heuristic Multi-Agent Insights (Game Orchestrator, Analytics Agent, Engagement Agent)
**Cloud/Deployment:** Docker & Docker Compose
**Other Tools:** Cursor, GitHub Copilot, Playwright, Vitest

---

## Key Features

1. **Dual-Tier State Resilience (Valkey-Primary, Postgres-Secondary):** The hot path is completely decoupled from PostgreSQL. If PostgreSQL goes offline, the app enters `Fallback` mode, allowing users to keep playing, guest login, and host games uninterrupted.
2. **Sub-Millisecond Multiplayer Sync:** Powered by Valkey Pub/Sub and Streams over WebSockets (Socket.IO).
3. **Interactive Valkey Activity Feed:** A real-time log panel showing Valkey stream events and active primitives (Hash, Set, Sorted Set, Stream, Pub/Sub, TTL, Atomic Update) that updates as players submit answers and ranks shift.
4. **Sorted-Set Leaderboards:** Leverages Valkey's sorted sets (`ZADD` and `ZINCRBY`) to recalculate and display player rankings dynamically and instantly.
5. **AI-Agent Commentary & Performance Analytics:** Generates real-time engagement reports, response speed charts, and rewards players with badges.

---

## What is Working?

* **Authentication & Session Management:** User registration/login through Postgres, falling back to cached credentials in Valkey when Postgres goes down. Guest authentication works out of the box via Valkey.
* **Room Orchestration:** Room creation, lock/unlock rooms, player joining, and player kicking.
* **Gameplay Flow:** Live game transitions (lobby -> question start -> answer submissions -> reveal answers -> leaderboard podium -> final summary and statistics).
* **Valkey Activity Feed:** Live checkbox updates and activity logging powered by Valkey Stream (`XADD` and `XREVRANGE`) and Pub/Sub (`PUBLISH`).
* **Playwright E2E Tests:** Complete automated flows verifying multiplayer functionality and simulating a hard database crash using Docker sockets (`/var/run/docker.sock`).

---

## What is Still in Progress?

* **Native Valkey GLIDE Client Driver Integration:** Currently using `ioredis` for compatibility and faster prototyping during the hackathon. We plan to migrate to the first-party GLIDE client for Node.js once all native bindings are configured.
* **Custom Quiz Creator Interface:** Currently using a pre-defined 4-question interactive demo quiz specifically designed to teach users about Valkey primitives. We plan to add a custom builder UI.

---

## Screenshots or Demo

* **Deployed Link:** Local development stack (Dockerized)
* **Demo Video Link:** [Local Demo Video]
* **Screenshots:**
  * **Lobby & Room Joining Interface:**
    ![Lobby Screen](https://github.com/IrfanCodesBTW/QuizRush/blob/main/showcase/Screenshot%202026-06-21%20154733.png?raw=true)
  * **Interactive Quiz Active Question:**
    ![Active Question](https://github.com/IrfanCodesBTW/QuizRush/blob/main/showcase/Screenshot%202026-06-21%20154744.png?raw=true)
  * **Lively Gameplay & Host Console Overview:**
    ![Host Console](https://github.com/IrfanCodesBTW/QuizRush/blob/main/showcase/Screenshot%202026-06-21%20154754.png?raw=true)
  * **Dynamic Answer Response Statistics:**
    ![Answer Stats](https://github.com/IrfanCodesBTW/QuizRush/blob/main/showcase/Screenshot%202026-06-21%20154804.png?raw=true)
  * **Real-time Leaderboard Standings:**
    ![Leaderboard Screen](https://github.com/IrfanCodesBTW/QuizRush/blob/main/showcase/Screenshot%202026-06-21%20154813.png?raw=true)
  * **Podium Ceremony & AI commentary insights:**
    ![Winner Podium](https://github.com/IrfanCodesBTW/QuizRush/blob/main/showcase/Screenshot%202026-06-21%20154823.png?raw=true)

---

## Challenges Faced

* **Managing Connection Timeouts in Node.js:** When PostgreSQL went down, the `pg` client's default connection pooling blocked the main thread or hung trying to resolve connections. We solved this by implementing a connection cooldown check and lazy-initializing the DB schemas.
* **Shared Valkey instances in Next.js:** Because Next.js runs API routes on demand, ensuring that the WebSocket server and Next.js APIs shared a single Valkey client state required placing the client instance on the `globalThis` object (`globalThis.quizRushValkey`).

---

## Learnings

* **Sorted Sets for Leaderboards:** Learned how using Valkey Sorted Sets makes real-time ranking computationally trivial and highly performant compared to costly SQL queries.
* **Valkey Streams as Audit Logs:** Streams are incredibly useful for building replayable event systems and logging real-time administrative actions.
* **Docker Sockets for Resilient Tests:** Learned how mounting `/var/run/docker.sock` in containerized testing allows tests to dynamically terminate system components to prove high-availability.

---

## Future Improvements

* **Valkey Cluster Scaling:** Scale the rooms horizontally across Valkey clusters for enterprise-level scale.
* **Dynamic AI Quizzes:** Connect to a live LLM API (such as Gemini or Claude) to generate custom quiz questions dynamically based on any topic selected by the host.

---

## Final Note

QuizRush shows how Valkey can act as the main driver for high-performance, real-time interactive apps. Decoupling the game state from SQL allows developers to build systems that are not only faster but incredibly resilient to database outages, resulting in a flawless user experience.
