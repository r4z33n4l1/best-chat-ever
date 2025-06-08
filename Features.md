### 🗂️ Master Feature Checklist (hyper-detailed)

<details>
<summary><strong>1&nbsp;·&nbsp;User & Auth</strong></summary>

* **1.1 Sign-up / Log-in**

  * Social OAuth (Google, GitHub, Discord)
  * Email-link & Passkey (WebAuthn) flows
  * “Guest” mode w/ local UUID (auto-upgrade path)
* **1.2 Session Handling**

  * HttpOnly cookie on Edge
  * Silent token refresh + 401 re-auth guard
* **1.3 Account Settings UI**

  * Name, photo, locale, theme
  * Personal API keys for custom LLMs
  * Billing usage dashboard

</details>

<details>
<summary><strong>2&nbsp;·&nbsp;Core Chat Experience</strong></summary>

* **2.1 Composer**

  * Auto-grow textarea, `/slash` command menu
  * Drag-&-drop file zone, paste-image support
* **2.2 Message Bubble**

  * Streaming tokens w/ markdown + code highlighting
  * “Copy code” & “Copy raw”
  * Cost + token count badge
  * Edit / Regenerate / Delete
* **2.3 Sidebar**

  * Conversation list (title + last msg preview)
  * Model picker & temperature slider
  * New chat ➜ scroll into view animation

</details>

<details>
<summary><strong>3&nbsp;·&nbsp;Chat API Wrapper</strong></summary>

* **3.1 Provider Adapters**

  * OpenAI, Anthropic, Gemini, Mistral, Groq
  * OpenRouter generic adapter
* **3.2 Streaming Interface**

  * `async *generate(params)` returns `{token, index, done}`
  * Heartbeat every 5 s
* **3.3 Fallback Chain**

  * Ordered model list w/ health probes
  * Exponential back-off + jitter retries
* **3.4 Middlewares**

  * Upstash token-bucket rate limiter
  * Prompt/response content filter hooks
  * Telemetry logger → OpenTelemetry exporter

</details>

<details>
<summary><strong>4&nbsp;·&nbsp;Edge Streaming Layer</strong></summary>

* **4.1 Route `/api/chat/stream`**

  * Server-Sent Events (SSE) over HTTP/2
  * `?cursor=` param to resume mid-generation
* **4.2 Redis Token Cache**

  * `SETEX(id:chunk#, 5s, payload)` for tab reload
* **4.3 Pressure Control**

  * Kill-switch header if user > quota

</details>

<details>
<summary><strong>5&nbsp;·&nbsp;Convex Datastore</strong></summary>

| Table / Index                       | Fields                                                          |
| ----------------------------------- | --------------------------------------------------------------- |
| `users`                             | `id`, `email`, `name`, `avatar`, `providerIds[]`                |
| `conversations`                     | `id`, `userId`, `title`, `forkOf?`, `created`, `updated`        |
| `messages` (idx: `convoId_created`) | `id`, `convoId`, `role`, `content`, `tokens`, `cost`, `created` |
| `attachments`                       | `id`, `ownerId`, `fileKey`, `mime`, `vectorIds[]`               |
| `shareLinks`                        | `id`, `convoId`, `slug`, `public`, `created`                    |

* **Convex Functions**

  * `sendMessage()` (optimistic insert + SSE fan-out)
  * `listMessages()` reactive query
  * `forkConversation()`
  * `createShareLink()`
  * CRON: nightly cost summarizer

</details>

<details>
<summary><strong>6&nbsp;·&nbsp;Realtime Sync</strong></summary>

* Convex reactive queries → WebSocket diff patches
* Local cache

  * Browser : IndexedDB via `zustand/persist`
  * Mobile  : Expo SQLite
* Conflict resolution

  * Last-writer-wins for text
  * CRDT (Y.js) for rich branch trees

</details>

<details>
<summary><strong>7&nbsp;·&nbsp;Attachments & RAG</strong></summary>

* **Upload Pipeline**

  * `UploadThing` → Cloudflare R2 signed URL
  * Virus scan (ClamAV Lambda) ✅
* **Text Extraction**

  * PDF: `pdf-parse`
  * Images: Tesseract OCR
* **Embedding & Storage**

  * Split into 1 k-token chunks
  * Store vectors in Convex Vector API
* **Query Tool**

  * k-nearest search ➜ top N chunks merged into system prompt

</details>

<details>
<summary><strong>8&nbsp;·&nbsp;Image Generation</strong></summary>

* `/api/image` proxy to OpenRouter `image/*`
* Streaming base64 → progressive `<img>` placeholder
* Gallery grid with hover → full-size modal

</details>

<details>
<summary><strong>9&nbsp;·&nbsp;Branching & Sharing</strong></summary>

* “Fork Chat” button → duplicates conversation row (`forkOf`)
* Tree view in sidebar (indent by depth)
* `share/:slug`

  * OG-image via Vercel OG API
  * Toggle public/private

</details>

<details>
<summary><strong>10&nbsp;·&nbsp;Web Search Tool</strong></summary>

* Brave Search REST call (top 5 JSON)
* Renders inline citations (superscript numbers)
* Cache responses 5 min in Upstash

</details>

<details>
<summary><strong>11&nbsp;· Prompt Mixer UI</strong></summary>

* Drag-and-drop blocks: **System • User • Tool**
* Live preview of compiled prompt
* Version history diff viewer

</details>

<details>
<summary><strong>12&nbsp;·&nbsp;Model Diff View</strong></summary>

* Parallel calls to selected A & B models
* Side-by-side token streams (left/right align)
* Per-message latency & cost comparison footer

</details>

<details>
<summary><strong>13&nbsp;·&nbsp;Voice Features</strong></summary>

* Record mic ▶ `MediaRecorder` → WebM
* `/api/whisper` (whisper-cpp tiny-en) returns transcript
* Browser TTS (`speechSynthesis`) playback with pause/seek

</details>

<details>
<summary><strong>14&nbsp;·&nbsp;Slash-Command SDK</strong></summary>

* `/commands` dashboard: register name + code snippet
* Convex Action runner executes sandboxed TS
* Autocomplete `/weather`, `/stock AAPL`

</details>

<details>
<summary><strong>15&nbsp;·&nbsp;Offline PWA</strong></summary>

* Service Worker

  * Cache static assets + last 10 conversations
* Outbox queue (IndexedDB) → auto-flush on reconnect
* Install prompt + home-screen icon

</details>

<details>
<summary><strong>16&nbsp;·&nbsp;Observability & Perf</strong></summary>

* OpenTelemetry traces (proxy ↔ LLM)
* Langfuse spans w/ prompt/response
* Grafana Cloud dashboard (p95 latency, cost/min)
* k6 load test script in `/scripts/`

</details>

<details>
<summary><strong>17&nbsp;·&nbsp;Security & Limits</strong></summary>

* RLS-style checks in each Convex function
* Upstash rate-limit middleware (60 req/min/user)
* CSP headers + `helmet()` for Next.js
* File size & mime whitelist

</details>

<details>
<summary><strong>18&nbsp;·&nbsp;CI / CD</strong></summary>

* GitHub Actions: lint ➜ type-check ➜ unit tests ➜ preview deploy
* Chromatic visual regression for components
* Renovate bot config

</details>

<details>
<summary><strong>19&nbsp;· Docs & DX</strong></summary>

* `/docs/architecture.md` (updated diagram)
* `/docs/api.md` (wrapper usage & examples)
* One-line `npx convex dev && npm run dev`
* Issue templates & Contributing guide

</details>

---

### ✅ Shipping Criteria

| Milestone               | Feature Groups         |
| ----------------------- | ---------------------- |
| **MVP (Day 3)**         | 1 · 2 · 3 · 4 · 5 · 6  |
| **Prize-Ready (Day 6)** | + 7 · 8 · 9 · 10       |
| **Wow-Factor (Day 8)**  | + 11 · 12 · 13 · 14    |
| **Polish & Docs**       | 15 · 16 · 17 · 18 · 19 |

Implement every checkbox above and you’ll deliver a best-in-class, feature-complete T3 Chat clone with plenty of sparkle for the judges. 🏁
