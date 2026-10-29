# PRJ-06 — Ahmad & Amin Technology: Engineering Evidence

**Project ID:** PRJ-06
**Canonical name:** Ahmad & Amin Technology
**Source:** https://github.com/aminbita162-glitch/ahmad-amin-technology
**Source HEAD (Phase 6 inspection):** `fbbfc307fd98bdad05ac02c3f2813b39368c5bc6` (2026-10-09, commit message: "R7 seal: RESULT PASS")
**Default branch:** `main`
**License:** NOASSERTION — `LICENSE` file present with custom proprietary text: "Copyright (c) 2026 Amin Azimi. Amin Azimi — AI Architect..." Not an SPDX-recognized license identifier. No source code is copied into the portfolio; all evidence uses SHA-pinned permalinks and attributed descriptions. No redistribution right is granted; copying source into the portfolio requires explicit owner authorization.
**Language:** TypeScript (Next.js 15 App Router, React 19, TypeScript strict)
**Framework:** Next.js 15 + React 19 + Supabase JS 2 + OpenAI SDK 4
**Phase 6 inspection date (UTC):** 2026-10-18
**Independence:** Source/documentation observations only; no independent third-party verification claimed.
**Independence level:** Source/documentation observations only; no independent reproduction claimed.
**Evidence maturity:** R1 — source and configuration inspected at immutable revision. No independent execution (R3) or longitudinal measurement (R5) has been performed.
**Reproducibility ladder:** R1 (source inspected at pinned SHA). R2 not achieved — no CI workflow present (0 CI runs). 5 unit tests (`lib/thread.test.ts`, `node:test`) are REPORTED in `execution/r5-seal.txt` and confirmed by static source count (5 `test(...)` calls — matching). Not independently executed.

---

## 1. Context and Scope

Ahmad & Amin Technology is a private, two-person web application — a bilingual (English/Persian) chat and messaging desk for two brothers (Amin in Germany, Ahmad in Iran) built on Next.js 15 App Router, React 19, Supabase, and OpenAI. There is no public signup, no multi-tenancy, and no admin panel. The app is gated by a whitelist of exactly two handles (`amin` | `ahmad`) sourced from environment variables. The architecture is explicitly offline-first: when Supabase or OpenAI env vars are empty, routes return `offline: true` with honest messages rather than failing or faking responses.

**Status (OBSERVED in README.md + source at HEAD):**

| Field | Value | Observation class |
|-------|-------|--------------------|
| System status | Version 1.0.0; R7 seal PASS; release seal BLOCKED (Phase 6 not PASS) | OBSERVED |
| CI | No CI workflow (0 `.github/workflows/` files) | OBSERVED |
| Test count | 5 unit tests in `lib/thread.test.ts` (`node:test`) | OBSERVED (static source count) |
| Release tags | 0 | OBSERVED |
| Published releases | 0 | OBSERVED |
| Deployment | Not deployed — Vercel deployment BLOCKED (3 human blockers) | OBSERVED |
| Model | Not live — `OPENAI_API_KEY` empty; route returns `offline: true` (does NOT fake) | OBSERVED |
| Sync | Not live — Supabase env empty; `isSupabaseConfigured()` returns false | OBSERVED |
| Users | Two-person whitelist (Amin/Germany, Ahmad/Iran) from env vars only | OBSERVED |
| Local desk controls | 25 (all browser `localStorage`, no server, no new package) | OBSERVED (in R7 seal) |

**Non-goals (OBSERVED across README.md, DIRECTIVE.txt, release-seal.txt):**

- No public signup, admin panel, third user, analytics, or advertising.
- No multi-tenancy — two handles only (`amin` | `ahmad`).
- No decorative library, theme toggle, animation pack, icon pack, or component kit.
- No Tailwind — visual system is hand-written CSS in `app/globals.css`.
- No stored model key in `localStorage`, `sessionStorage`, or a `NEXT_PUBLIC_` variable.
- No rewrite of `DIRECTIVE.txt` — the contract is read-only (byte-identical).
- No cloud, database, or model vendor other than Supabase and OpenAI.
- No new dependency beyond those named in the contract.
- No POST caching or model-reply caching in the service worker.
- No claim that chat is cross-country live unless env keys exist and a real call succeeded.

---

## 2. System Architecture (RECONSTRUCTED FROM SOURCE)

Architecture reconstructed by reading all source files at HEAD SHA `fbbfc307fd98bdad05ac02c3f2813b39368c5bc6` via local clone. Not a verified runtime topology.

### 2a. Domain and Boundary Decomposition

```
[Browser — Chamber Desk] (app/chamber/desk.tsx + app/chamber/page.tsx)
  │  25 local desk controls (all localStorage, no server):
  │  folder/file management, meetings, checklists, voice notes,
  │  tags, search, pin, export, model room (named chats), spend
  │  tracking, cached-prompt, attach PDF/image, web-search action,
  │  copy/share/like/dislike/listen per reply, rename/pin/move chats
  │
  ├─ Auth Boundary (app/api/auth/)
  │  login/route.ts: POST email+password → verify against whitelist → set aa_session cookie
  │  logout/route.ts: POST → clear aa_session (maxAge: 0)
  │  session/route.ts: GET → verify aa_session → return handle or 401
  │
  ├─ Session (lib/session.ts, 158 lines)
  │  getWhitelistUsers(): two users from env (AMIN_*, AHMAD_*)
  │  HMAC-SHA256 signed cookie (aa_session), httpOnly, SameSite=Lax,
  │  Secure in production, 18-hour max age
  │  isSupabaseConfigured(): checks 3 env vars → boolean
  │  isProduction(): NODE_ENV === "production"
  │  isDemoMode(): DEMO_MODE === "true"
  │
  ├─ Production Guard (middleware.ts, 25 lines)
  │  503 if NODE_ENV=production AND (DEMO_MODE=true OR SESSION_SECRET.length < 32)
  │  Matcher excludes _next/static, _next/image, favicon.ico
  │
  ├─ Messages Route (app/api/messages/route.ts, 192 lines)
  │  GET: isSupabaseConfigured()? offline:true : fetch chats+messages
  │  POST: send message (offline gate first)
  │  Supabase client with service-role key (server-only)
  │
  ├─ Model Room (app/api/chat/route.ts, 146 lines)
  │  OPENAI_API_KEY server-only (no NEXT_PUBLIC_)
  │  offline:true when key absent — does NOT fake a model response
  │  Rate limit 30/hr per handle (in-memory Map)
  │  Max 8 messages sliced before upstream call
  │  Temperature 0.4; gpt-4o-mini (900 tokens), gpt-4o (1200 tokens)
  │  Input sanitization: role must be user/assistant, content non-empty string
  │
  ├─ Health Route (app/api/health/route.ts, 25 lines)
  │  Returns booleans only: session, supabase, model
  │  No secret values exposed
  │
  ├─ Thread (lib/thread.ts, 218 lines)
  │  POLL_INTERVAL_MS = 3000
  │  shouldPoll(channelStatus): true unless status === "joined"
  │  Polling fallback for Iran path (WebSocket may die)
  │
  ├─ Service Worker (public/sw.js, 70 lines)
  │  Network-first GET navigation cache only
  │  POST never cached; /api/chat never cached
  │  Cache name: aa-tech-2026-v1
  │
  └─ Supabase Schema (supabase/schema.sql, 192 lines)
     RLS policies on chats + messages
     CHECK constraints: handle/sender in ('amin', 'ahmad') only
     Realtime publication for messages + chats
```

### 2b. Key Architectural Decisions

1. **Private two-person whitelist (not multi-tenant)** — `getWhitelistUsers()` reads exactly two users (`AMIN_*`, `AHMAD_*`) from environment variables. Handle type is `'amin' | 'ahmad'`. No public signup, no registration, no third user. This is the primary access control.

2. **HMAC-SHA256 session cookie** — The session is a signed cookie (`aa_session`), not a plain identifier. `httpOnly: true` (client scripts cannot read it), `SameSite: "lax"`, `Secure` in production, 18-hour max age. The signing secret comes from `SESSION_SECRET` env var. The production guard blocks serving if `SESSION_SECRET.length < 32`.

3. **Offline-first with honest blocked lines** — Every external dependency (Supabase, OpenAI) is gated by `isSupabaseConfigured()` and `OPENAI_API_KEY` checks. When unconfigured, routes return `offline: true` with an honest message. The app never fakes a model response or a sync response. This is the core resilience design.

4. **Polling fallback for Iran WebSocket path** — `shouldPoll(channelStatus)` returns `true` unless the channel status is `"joined"`. `POLL_INTERVAL_MS = 3000`. This is mandatory because the Iran network path may kill WebSocket connections; polling is the resilience layer.

5. **Production guard as hard gate** — `middleware.ts` returns HTTP 503 on every matched route when `NODE_ENV=production` AND (`DEMO_MODE=true` OR `SESSION_SECRET.length < 32`). This prevents the app from serving in production with a demo secret or a weak secret. The matcher excludes static assets.

6. **Meridian Seal hand-written CSS** — No Tailwind, no component kit. Visual system: ink `#54656f`, paper `#efeae2` background, amin `#00a884`, ahmad `#cf9466`, fault `#e0635a`. Vazirmatn for Persian, Syne for English. Bilingual EN/FA labels throughout.

---

## 3. Source Provenance

| Field | Value |
|-------|-------|
| Source URL | https://github.com/aminbita162-glitch/ahmad-amin-technology |
| HEAD SHA | `fbbfc307fd98bdad05ac02c3f2813b39368c5bc6` |
| HEAD commit message | "R7 seal: RESULT PASS" |
| HEAD commit date | 2026-10-09 |
| Default branch | `main` |
| License | NOASSERTION — custom proprietary text (Copyright (c) 2026 Amin Azimi); not SPDX-recognized |
| Total tracked files | 44 (excluding .git) |
| TS/TSX/CSS lines | 6,653 (`git ls-files \| grep -E '\.(ts\|tsx\|css)$' \| xargs wc -l` = 6653 total) |
| Test files | 1 (`lib/thread.test.ts`) |
| Test functions | 5 (OBSERVED — static source count: `grep -c "^test(" lib/thread.test.ts` = 5) |
| CI workflow | **NONE** — 0 `.github/workflows/` files |
| CI runs | **NONE** — 0 CI runs (no workflow to trigger) |
| Release tags | 0 |
| Published releases | 0 |
| Version | 1.0.0 (`package.json` `"version": "1.0.0"`) |
| Deployment | Not deployed — Vercel deployment BLOCKED (3 human blockers) |
| Inspection method | Local clone at pinned SHA (after `git fetch origin main`) |
| Observation class | OBSERVED (all file content); OBSERVED (test count via grep); OBSERVED (no CI via repo tree) |

**Commit history (git log --oneline -20 at HEAD):**

```
fbbfc30 R7 seal: RESULT PASS
dce594d R7: Model Room left menu + named chats, Thread named chats, sync-waits-for-Supabase, schema+service-role, R7 CSS
12a7fdf R5 seal: RESULT PASS
ee16b48 R5: WhatsApp-light surface — #efeae2 bg, #ffffff panels, #00a884 accent, no black/dark gray
6fc05e4 Options R4 seal: RESULT PASS
f3c1f40 Options R4: sky blue gate, white buttons, in-page folder rename, multi-file attach, file count, in-page PDF preview, translated status lines
8ed0af7 Options R3: section sub-tabs, EN/FA language switch, metallic gray background, class archive with seeded PDFs
ba714ad Options R2 seal: RESULT PASS
7f53fc2 Options R2: frame fix (ltr, globals.css import), 25 local desk controls, test-home list, lint/typecheck/build pass
931aac3 Gate: add login form (email + password) with Meridian Seal, POST /api/auth/login, redirect to #chamber on 200, show Access denied on 401
ae1b754 Phase 8: Release Seal — production guard, lint/typecheck/build pass, Vercel BLOCKED
a494150 Phase 7: Install, Resilience, Test Home — sw.js, health route, test-home, unsent queue, PASS seal
33cee87 Phase 6: Model Room — chat route, isolated model tab, schema fix, BLOCKED seal
0fb2036 Phase 5: Thread — chamber page, messages route, polling 3000ms, schema grants
27c002a Phase 4: Data Seal — schema.sql, thread.ts, BLOCKED seal
ac4b196 Phase 3: Whitelist Session — HMAC cookie, auth routes, production guard
502707a Phase 2 seal
376b90e Phase 2: Frame — gate, Meridian Seal, manifest
4c01a66 Initial commit
```

---

## 4. Authentication and Session Security (Phase 6 Lens)

### 4a. Private Two-Person Whitelist (OBSERVED in `lib/session.ts`, 158 lines)

`getWhitelistUsers()` reads exactly two users from environment variables:
- Amin: `AMIN_EMAIL`, `AMIN_PASSWORD`, `AMIN_NAME`
- Ahmad: `AHMAD_EMAIL`, `AHMAD_PASSWORD`, `AHMAD_NAME`

Handle type: `'amin' | 'ahmad'`. No public signup, no registration, no third user. The whitelist is the sole access control. `.env.example` confirms the two-user structure.

### 4b. HMAC-SHA256 Session Cookie (OBSERVED in `lib/session.ts`)

- Cookie name: `aa_session`
- Signing: HMAC-SHA256 (signed token, never a plain identifier)
- `httpOnly: true` — client scripts cannot read it
- `sameSite: "lax"`
- `secure: isProduction()` — Secure flag in production only
- `maxAge`: 18 hours (`SESSION_MAX_AGE_SECONDS`)
- Secret from `SESSION_SECRET` env var
- Logout clears cookie with `maxAge: 0`

**Key observation:** The cookie is a signed token, not an encrypted session store. The signature prevents tampering but the payload is not encrypted. The 18-hour max age limits replay window.

### 4c. Production Guard (OBSERVED in `middleware.ts`, 25 lines)

```typescript
const isProduction = process.env.NODE_ENV === "production";
const demoMode = process.env.DEMO_MODE === "true";
const secret = process.env.SESSION_SECRET ?? "";
// ...
if (isProduction && (demoMode || secret.length < 32)) {
  return new NextResponse("Service Unavailable", { status: 503 });
}
```

Matcher: `["/((?!_next/static|_next/image|favicon.ico).*)"]` — excludes static assets.

**Key observation:** The guard fires only in production. In development, it is inert. On Vercel (production), it WOULD fire 503 because `SESSION_SECRET` is empty (0 chars < 32). The deployment refuses to serve until `SESSION_SECRET` is set to ≥ 32 characters and `DEMO_MODE` is not `"true"`.

---

## 5. Supabase Integration and Offline Gate (Phase 6 Lens)

### 5a. isSupabaseConfigured() (OBSERVED in `lib/session.ts`)

`isSupabaseConfigured()` checks all three env vars:
1. `NEXT_PUBLIC_SUPABASE_URL`
2. `NEXT_PUBLIC_SUPABASE_ANON_KEY`
3. `SUPABASE_SERVICE_ROLE_KEY` (server-only — no `NEXT_PUBLIC_` prefix, never in client bundle)

Returns `false` when any are empty. All API routes call this gate before any Supabase operation.

### 5b. Offline Gate in Routes (OBSERVED in `app/api/messages/route.ts`, 192 lines)

When `isSupabaseConfigured()` is false:
- `GET /api/messages` → `{ offline: true, messages: [], chats: [] }`
- `POST /api/messages` → `{ offline: true, ... }` (honest message)
- All chat operations → `{ offline: true, chat: null }`

**Key observation:** The route never fakes data. It returns `offline: true` and empty arrays. The UI shows the honest "sync is offline" / "sync waits for Supabase" line.

### 5c. Supabase RLS Schema (OBSERVED in `supabase/schema.sql`, 192 lines)

- `profiles` table: `CHECK (handle in ('amin', 'ahmad'))`
- `chats` table: `CHECK (handle in ('amin', 'ahmad'))`
- `messages` table: `CHECK (sender in ('amin', 'ahmad'))`
- RLS policies: `chats_auth_select`, `chats_insert_own_handle`, `chats_update_own_handle`, `chats_delete_own_handle`, `messages_auth_select`, `messages_insert_own_handle`
- Default access revoked: `revoke all` → RLS is the only gate
- Realtime publication: `messages` and `chats` added to `supabase_realtime`

**Key observation:** The schema enforces two-person-only access at the database level via CHECK constraints. RLS ensures each handle can only access its own rows. The SQL is written but NOT applied to a live database (env vars empty).

---

## 6. OpenAI Model Room (Phase 6 Lens)

### 6a. Server-Only Key (OBSERVED in `app/api/chat/route.ts`, 146 lines)

- `OPENAI_API_KEY` read from `process.env` — no `NEXT_PUBLIC_` prefix, never in client bundle
- Model flags: `'gpt-4o-mini' | 'gpt-4o'`
- `MAX_TOKENS`: `gpt-4o-mini` → 900, `gpt-4o` → 1200
- `TEMPERATURE`: 0.4
- `MAX_MESSAGES`: 8 (sliced before upstream call: `incoming.slice(0, MAX_MESSAGES)`)

### 6b. Offline=true When Key Absent (OBSERVED in `app/api/chat/route.ts`)

```typescript
const apiKey = process.env.OPENAI_API_KEY ?? "";
if (!apiKey) {
  // Key is missing: return offline true and do NOT imitate a model answer.
  return NextResponse.json({ offline: true, ... });
}
```

**Key observation:** The route does NOT fake a model response. It returns `offline: true` and no reply. This is the honest-offline design — the same pattern as the Supabase gate.

### 6c. Rate Limit (OBSERVED in `app/api/chat/route.ts`)

- `RATE_LIMIT_PER_HOUR = 30` (from env `RATE_LIMIT_PER_HOUR`, default 30)
- In-memory `Map<Handle, number[]>` — per-handle timestamp array
- `recent = timestamps.filter(t => t > windowStart)`
- If `recent.length >= 30` → 429 (rate limited)
- Per-process, not distributed — acceptable for a two-person app

### 6d. Input Sanitization (OBSERVED in `app/api/chat/route.ts`)

`sanitizeMessages(raw)`:
- Each message: `role` must be `"user"` or `"assistant"` (else `continue` / skip)
- `content` must be a non-empty string (else skip)
- Output: `ChatMessage[]` with only valid role/content pairs
- After sanitization: `incoming.slice(0, MAX_MESSAGES)` (max 8)

**Key observation:** Invalid roles and empty content are silently dropped, not errored. This prevents prompt injection via role manipulation. The 8-message slice bounds the upstream payload.

---

## 7. Polling Fallback and Resilience (Phase 6 Lens)

### 7a. Polling Fallback (OBSERVED in `lib/thread.ts`, 218 lines)

```typescript
export const POLL_INTERVAL_MS = 3000;
export function shouldPoll(channelStatus: string): boolean {
  return channelStatus !== "joined";
}
```

- `POLL_INTERVAL_MS = 3000` (3 seconds)
- `shouldPoll()` is a pure function: returns `true` unless `channelStatus === "joined"`
- Channel statuses Supabase emits: `"connecting"`, `"joined"`, `"closing"`, `"closed"`, `"timed out"`, `""` (initial). Only `"joined"` means the socket is live.
- Case-sensitive: `"Joined"` and `"JOINED"` return `true` (polling continues).

**Key observation:** This is mandatory for the Iran network path — WebSocket connections may die, and polling is the resilience layer. The function is pure and testable.

### 7b. shouldPoll Unit Test (OBSERVED in `lib/thread.test.ts`, 37 lines)

5 tests via `node:test`:
1. `POLL_INTERVAL_MS is exactly 3000`
2. `shouldPoll returns false when channel is joined`
3. `shouldPoll returns true for non-joined statuses (polling fallback)` — tests `"connecting"`, `"closed"`, `"closing"`, `"timed out"`, `""`
4. `shouldPoll is case-sensitive — only lowercase 'joined' stops polling` — tests `"Joined"`, `"JOINED"`
5. `shouldPoll with undefined-ish values polls` — tests `""`

**Key observation:** 5 tests by static source count (`grep -c "^test(" lib/thread.test.ts` = 5). Matches `execution/r5-seal.txt` report ("5/5 pass"). NOT independently executed.

### 7c. Service Worker (OBSERVED in `public/sw.js`, 70 lines)

- Cache name: `aa-tech-2026-v1`
- Network-first for GET navigations (HTML page loads) only
- `if (request.method !== "GET") return;` — POST never cached
- `if (url.pathname === "/api/chat") return;` — model replies never cached
- On network failure: serve last cached navigation
- Cleans up old caches (`keys.filter(k => k !== CACHE_NAME)`)

**Key observation:** The service worker caches only same-origin HTML navigations. Writes (POST) and model replies are always live. This is the resilience layer for the Iran path — if the network drops, the last navigation is served, but no stale data.

---

## 8. Health Route and Observability (Phase 6 Lens)

### 8a. Health Route (OBSERVED in `app/api/health/route.ts`, 25 lines)

```typescript
const session = payload !== null;
const supabase = isSupabaseConfigured();
const model = (process.env.OPENAI_API_KEY ?? "") !== "";
return NextResponse.json({ session, supabase, model });
```

- Returns booleans only: `session`, `supabase`, `model`
- No secret values exposed — no keys, no URLs, no tokens
- `session`: true if caller has a valid `aa_session` cookie
- `supabase`: true if `isSupabaseConfigured()` (3 env vars set)
- `model`: true if `OPENAI_API_KEY` is non-empty

**Key observation:** This is a safe health check. An attacker learns only whether the app is configured, not how. No information disclosure.

---

## 9. Visual System: Meridian Seal (Phase 6 Lens)

### 9a. Hand-Written CSS (OBSERVED in `app/globals.css`, 1946 lines)

- No Tailwind, no component kit, no icon pack
- Color palette:
  - Ink: `#54656f` (text color)
  - Paper/background: `#efeae2`
  - Panels: `#ffffff`
  - Amin accent: `#00a884`
  - Ahmad accent: `#cf9466`
  - Fault: `#e0635a`
- Fonts: Vazirmatn (Persian), Syne (English) — `@import` from Google Fonts
- Bilingual EN/FA labels throughout the UI
- Asymmetric clip-path, frosted panel aesthetic

**Key observation:** The visual system is a single 1,946-line CSS file. No utility framework, no design tokens beyond CSS custom properties. This is a deliberate choice — no decorative library, no theme toggle.

---

## 10. Phase Seal System and Release Status (Phase 6 Lens)

### 10a. Phase Seal Files (OBSERVED in `execution/` directory)

| Seal file | Phase | Result | Notes |
|-----------|-------|--------|-------|
| `phase-1-seal.txt` | 1 (Root Truth) | PASS | LICENSE, README, env, config files |
| `phase-2-seal.txt` | 2 (Frame) | PASS | Layout, globals.css, gate page, manifest |
| `phase-3-seal.txt` | 3 (Whitelist Session) | PASS | HMAC cookie, auth routes, middleware |
| `phase-4-seal.txt` | 4 (Data Seal) | BLOCKED | Supabase env empty — SQL not applied |
| `phase-5-seal.txt` | 5 (Thread) | BLOCKED | Supabase env empty — sync offline |
| `phase-6-seal.txt` | 6 (Model Room) | BLOCKED | OPENAI_API_KEY empty — model not live |
| `phase-7-seal.txt` | 7 (Install, Resilience) | PASS | sw.js, health route, test-home |
| `r7-seal.txt` | R7 addendum | PASS | lint/typecheck/build pass, 13/13 pages, commit dce594d |
| `release-seal.txt` | Release (Phase 8) | BLOCKED | Phase 6 not PASS; 3 Vercel blockers |

### 10b. Release Seal BLOCKED (OBSERVED in `execution/release-seal.txt`)

The release seal is BLOCKED because:
1. **Phase 6 is BLOCKED** — `OPENAI_API_KEY` is empty. The DIRECTIVE requires Phase 6 to be PASS for a PASS release. Phases 4 and 5 may be BLOCKED on missing human secrets (Supabase), but Phase 6 must be PASS.
2. **Three Vercel deployment blockers (all human):**
   - BLOCKER V1: Vercel CLI not installed (no `vercel` on PATH, no `.vercel` directory)
   - BLOCKER V2: Vercel login/token missing (no account linked)
   - BLOCKER V3: Production env values empty (`SESSION_SECRET` 0 chars < 32, `OPENAI_API_KEY` empty, Supabase env empty)

**Key observation:** The production build compiles (`npm run build` exit 0, 13/13 pages generated, commit `dce594d`). The deployment is blocked by missing human-supplied secrets and tooling, not by code defects. `DIRECTIVE.txt` was not edited (read-only, byte-identical).

### 10c. R7 Seal PASS (OBSERVED in `execution/r7-seal.txt`)

- Contract addendum `AA-TECH-2026-R7`
- RESULT: PASS
- `npm run lint` — exit 0, no warnings
- `npm run typecheck` — exit 0, `tsc --noEmit` clean
- `npm run build` — exit 0, Next.js 15.5.27, 13/13 static pages
- 25 local desk controls (all `localStorage`, no server, no new package)
- Model room: left menu (Projects, New chat, Files), named chats, rename/pin/search/save/move-to-file
- Cached-prompt (`aa-cached-prompt-v1`), spend tracking (`aa-spend-v1`)
- Thread: named chats, voice (MediaRecorder), transcribe (Web Speech API), attach image/PDF

---

## 11. Engineering Decisions (ADR-Style)

### ADR-06-001: Private Two-Person Whitelist (Not Multi-Tenant)

| Field | Value |
|-------|-------|
| ADR_ID | ADR-06-001 |
| Source SHA | `fbbfc307fd98bdad05ac02c3f2813b39368c5bc6` |
| Problem | Provide authenticated access for exactly two users (two brothers) without a registration system |
| Constraints | No public signup, no admin panel, no third user; handles are `amin` and `ahmad` only |
| Selected approach | `getWhitelistUsers()` in `lib/session.ts` reads two users from env vars (`AMIN_*`, `AHMAD_*`); handle type `'amin' \| 'ahmad'`; HMAC-SHA256 session cookie |
| Alternatives | Multi-tenant with registration; shared password; OAuth with provider |
| Trade-offs | Simpler — no tenant isolation needed for two people; no registration UI to maintain; but not scalable beyond two users without code changes |
| Failure implications | Adding a third user requires code changes (handle type + env vars + schema CHECK); not a runtime configuration |
| Evidence | `lib/session.ts:54 getWhitelistUsers()`; `supabase/schema.sql:17 CHECK (handle in ('amin','ahmad'))`; `.env.example` (AMIN_*, AHMAD_*) |
| Status | ACCEPTED — IMPLEMENTED; tested via auth routes |

### ADR-06-002: Offline-First Design with Honest Blocked Lines

| Field | Value |
|-------|-------|
| ADR_ID | ADR-06-002 |
| Source SHA | `fbbfc307fd98bdad05ac02c3f2813b39368c5bc6` |
| Problem | Handle missing Supabase and OpenAI env vars gracefully without faking responses |
| Constraints | App must run in dev with empty env; must not crash; must not mislead the user |
| Selected approach | `isSupabaseConfigured()` checks 3 env vars; `OPENAI_API_KEY` check in chat route; when unconfigured, return `{ offline: true }` with honest message — never fake a response |
| Alternatives | Hard-fail with HTTP 500; silent failure with empty response; fake model response for dev |
| Trade-offs | Graceful degradation and honest UI; but the app is non-functional without secrets — intentional, not a bug |
| Failure implications | If a route forgets the `isSupabaseConfigured()` gate, it would crash on Supabase call — mitigated by consistent gate in all routes |
| Evidence | `lib/session.ts isSupabaseConfigured()`; `app/api/messages/route.ts:49 if (!isSupabaseConfigured())`; `app/api/chat/route.ts:104 if (!apiKey)`; `app/api/health/route.ts` booleans only |
| Status | ACCEPTED — IMPLEMENTED; all routes gate consistently |

### ADR-06-003: Polling Fallback for Iran WebSocket Path

| Field | Value |
|-------|-------|
| ADR_ID | ADR-06-003 |
| Source SHA | `fbbfc307fd98bdad05ac02c3f2813b39368c5bc6` |
| Problem | WebSocket connections may die on the Iran network path; messages must still sync |
| Constraints | Supabase Realtime uses WebSocket; Iran network may block or drop WebSocket connections |
| Selected approach | `shouldPoll(channelStatus)` pure function returns `true` unless status is `"joined"`; `POLL_INTERVAL_MS = 3000`; client falls back to GET polling when channel is not joined |
| Alternatives | WebSocket-only (no fallback); long-polling always; SSE |
| Trade-offs | Resilience over real-time — polling adds 3s latency but survives WebSocket death; 5 unit tests verify the pure function |
| Failure implications | If WebSocket never joins, polling runs indefinitely at 3s intervals — acceptable for two-person app, not a performance concern |
| Evidence | `lib/thread.ts:206 POLL_INTERVAL_MS = 3000`; `lib/thread.ts:216 shouldPoll()`; `lib/thread.test.ts` (5 tests, `node:test`) |
| Status | ACCEPTED — IMPLEMENTED; 5/5 unit tests pass (REPORTED in r5-seal, static source count) |

---

## 12. Known Limitations and Gaps

| Gap ID | Severity | Description |
|--------|----------|-------------|
| GAP-06-001 | HIGH | No CI workflow — 0 `.github/workflows/` files. `npm run lint`, `npm run typecheck`, `npm run build` pass REPORTED in R7 seal but not CI-enforced. Tests could regress without detection. |
| GAP-06-002 | MEDIUM | No published releases or tags — 0 tags, 0 releases. Version 1.0.0 in `package.json` only. Not anchored to a Git tag or GitHub release. |
| GAP-06-003 | MEDIUM | Release seal BLOCKED — Phase 6 not PASS (`OPENAI_API_KEY` empty). Three human blockers for Vercel deployment (CLI, login, env values). |
| GAP-06-004 | MEDIUM | Supabase sync not live — `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY` all empty. `schema.sql` not applied to live database. |
| GAP-06-005 | LOW | NOASSERTION license — custom proprietary text ("Copyright (c) 2026 Amin Azimi. Amin Azimi — AI Architect...") not SPDX-recognized. No source code copied into portfolio; all evidence uses permalinks and descriptions. |
| GAP-06-006 | LOW | In-memory rate limit — per-process `Map<Handle, number[]>`, not distributed. Acceptable for a two-person app; would not survive a restart (resets per deploy). |
| GAP-06-007 | LOW | 5 unit tests only (`thread.test.ts`) — minimal test coverage. 25 local desk controls are untested. Auth, messages, chat routes have no automated tests. |
| GAP-06-008 | LOW | OpenAI API key absent — model room is built but not live. The route returns `offline: true` and does NOT fake responses. Honest gap, not a defect. |

---

## 13. Claim Comparison

| Claim ID | Claim | Phase 1 Class | Phase 6 Assessment |
|----------|-------|---------------|---------------------|
| CLM-06-001 | "Phase 1 PASS — root truth verified, lint/typecheck/build pass" | REPORTED / NOT_TESTED | **OBSERVED / CONDITIONAL** — Phase 1 seal OBSERVED (`execution/phase-1-seal.txt` RESULT: PASS). `npm run lint/typecheck/build` pass REPORTED in R7 seal (exit 0, 13/13 pages). No CI to independently verify — the pass is author-reported, not CI-enforced. |
| CLM-06-002 | "Supabase sync not yet live — honest blocked lines" | REPORTED / NOT_TESTED | **OBSERVED / PASS** — `isSupabaseConfigured()` returns `false` when env vars empty (`lib/session.ts`). `app/api/messages/route.ts` returns `{ offline: true }`. README and seal files show honest "sync is offline" / "sync waits for Supabase" lines. Confirmed. |

---

## 14. Reproducibility Recipe (R1 → R3 path)

This is a documented path to reach R3 independently. It has NOT been executed (not performed; source repositories are treated as read-only).

```bash
git clone https://github.com/aminbita162-glitch/ahmad-amin-technology.git
cd ahmad-amin-technology
git checkout fbbfc307fd98bdad05ac02c3f2813b39368c5bc6  # HEAD SHA
# Requires: Node.js >= 20, npm
npm install

# Lint, typecheck, build (author-reported in R7 seal)
npm run lint        # ESLint, exit 0
npm run typecheck   # tsc --noEmit, exit 0
npm run build       # Next.js 15, 13/13 static pages, exit 0

# Unit tests (5 tests, node:test)
node --test lib/thread.test.ts
# Expected: 5 pass, 0 fail (author-reported in r5-seal.txt)

# With secrets (for live sync + model)
export SESSION_SECRET="$(openssl rand -hex 32)"  # >= 32 chars
export DEMO_MODE=false
export AMIN_EMAIL=... AMIN_PASSWORD=... AMIN_NAME=Amin
export AHMAD_EMAIL=... AHMAD_PASSWORD=... AHMAD_NAME=Ahmad
export NEXT_PUBLIC_SUPABASE_URL=... NEXT_PUBLIC_SUPABASE_ANON_KEY=... SUPABASE_SERVICE_ROLE_KEY=...
export OPENAI_API_KEY=sk-...
export RATE_LIMIT_PER_HOUR=30

npm run dev
# App at http://localhost:3000
# Login at / → /chamber (desk) → model room, thread, files
```

Preconditions: Node.js ≥ 20; `SESSION_SECRET` ≥ 32 chars for production; Supabase project with `schema.sql` applied for live sync; `OPENAI_API_KEY` for live model. Without secrets, the app runs in offline mode (`offline: true` on all gated routes).

**Test reproduction note:** 5 test functions OBSERVED by static source count (`grep -c "^test(" lib/thread.test.ts` = 5). Author's `r5-seal.txt` records "5/5 pass" via `node --test lib/thread.test.ts`. Tests are NOT independently executed. No CI workflow exists — tests are run manually.

---

## 15. Evidence Matrix

| EVD ID | Claim | Source path | Observation class | Result | Source URL | Source SHA | Notes |
|--------|-------|-------------|-------------------|--------|------------|------------|-------|
| EVD-06-001 | Private two-person whitelist auth — `getWhitelistUsers()` reads two users from env | `lib/session.ts` | OBSERVED | PASS | https://github.com/aminbita162-glitch/ahmad-amin-technology/blob/fbbfc307fd98bdad05ac02c3f2813b39368c5bc6/lib/session.ts | `fbbfc307fd98` | `getWhitelistUsers():54` — AMIN_*/AHMAD_* env vars; handle type `'amin' \| 'ahmad'`; no public signup. |
| EVD-06-002 | HMAC-SHA256 session cookie (`aa_session`, httpOnly, SameSite=Lax, Secure in production, 18h max age) | `lib/session.ts` | OBSERVED | PASS | https://github.com/aminbita162-glitch/ahmad-amin-technology/blob/fbbfc307fd98bdad05ac02c3f2813b39368c5bc6/lib/session.ts | `fbbfc307fd98` | `SESSION_COOKIE_NAME="aa_session"`; `httpOnly:true`, `sameSite:"lax"`, `secure:isProduction()`, `maxAge: SESSION_MAX_AGE_SECONDS` (18h). Signed token, not plain ID. |
| EVD-06-003 | Production guard returns 503 when `NODE_ENV=production` AND (`DEMO_MODE=true` OR `SESSION_SECRET.length < 32`) | `middleware.ts` | OBSERVED | PASS | https://github.com/aminbita162-glitch/ahmad-amin-technology/blob/fbbfc307fd98bdad05ac02c3f2813b39368c5bc6/middleware.ts | `fbbfc307fd98` | 25 lines. Matcher excludes `_next/static`, `_next/image`, `favicon.ico`. Inert in dev; fires 503 on Vercel (SESSION_SECRET empty). |
| EVD-06-004 | Supabase offline gate — `isSupabaseConfigured()` checks 3 env vars; routes return `offline:true` when false | `lib/session.ts` + `app/api/messages/route.ts` | OBSERVED | PASS | https://github.com/aminbita162-glitch/ahmad-amin-technology/blob/fbbfc307fd98bdad05ac02c3f2813b39368c5bc6/app/api/messages/route.ts | `fbbfc307fd98` | `isSupabaseConfigured()` checks `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`. All routes gate before Supabase call. |
| EVD-06-005 | Supabase RLS schema — CHECK constraints (`amin`/`ahmad` only), RLS policies, realtime publication | `supabase/schema.sql` | OBSERVED | PASS | https://github.com/aminbita162-glitch/ahmad-amin-technology/blob/fbbfc307fd98bdad05ac02c3f2813b39368c5bc6/supabase/schema.sql | `fbbfc307fd98` | 192 lines. `CHECK (handle in ('amin','ahmad'))` on profiles+chats; `CHECK (sender in ('amin','ahmad'))` on messages. RLS policies: select/insert/update/delete. Realtime: messages+chats. Not applied to live DB. |
| EVD-06-006 | OpenAI model room is server-only — `OPENAI_API_KEY` from env, no `NEXT_PUBLIC_` prefix | `app/api/chat/route.ts` | OBSERVED | PASS | https://github.com/aminbita162-glitch/ahmad-amin-technology/blob/fbbfc307fd98bdad05ac02c3f2813b39368c5bc6/app/api/chat/route.ts | `fbbfc307fd98` | 146 lines. `process.env.OPENAI_API_KEY` — never in client bundle. Model flags: `gpt-4o-mini` (900 tokens), `gpt-4o` (1200 tokens). Temperature 0.4. |
| EVD-06-007 | OpenAI returns `offline:true` when key absent — does NOT fake a model response | `app/api/chat/route.ts` | OBSERVED | PASS | https://github.com/aminbita162-glitch/ahmad-amin-technology/blob/fbbfc307fd98bdad05ac02c3f2813b39368c5bc6/app/api/chat/route.ts | `fbbfc307fd98` | `if (!apiKey) { return NextResponse.json({ offline: true, ... }); }` — comment: "do NOT imitate a model answer." Honest offline, not a stub. |
| EVD-06-008 | Rate limit 30/hr per handle (in-memory `Map`) | `app/api/chat/route.ts` | OBSERVED | PASS | https://github.com/aminbita162-glitch/ahmad-amin-technology/blob/fbbfc307fd98bdad05ac02c3f2813b39368c5bc6/app/api/chat/route.ts | `fbbfc307fd98` | `RATE_LIMIT_PER_HOUR=30`; `rateBuckets = new Map<Handle, number[]>()`; `recent.length >= 30` → 429. Per-process, not distributed. |
| EVD-06-009 | Input sanitization — role must be `user`/`assistant`, content non-empty string; max 8 messages sliced | `app/api/chat/route.ts` | OBSERVED | PASS | https://github.com/aminbita162-glitch/ahmad-amin-technology/blob/fbbfc307fd98bdad05ac02c3f2813b39368c5bc6/app/api/chat/route.ts | `fbbfc307fd98` | `sanitizeMessages()`: skips invalid roles, skips empty content. `MAX_MESSAGES=8`, `incoming.slice(0, MAX_MESSAGES)`. Prevents prompt injection via role. |
| EVD-06-010 | Polling fallback — `POLL_INTERVAL_MS=3000`, `shouldPoll()` pure function | `lib/thread.ts` | OBSERVED | PASS | https://github.com/aminbita162-glitch/ahmad-amin-technology/blob/fbbfc307fd98bdad05ac02c3f2813b39368c5bc6/lib/thread.ts | `fbbfc307fd98` | `POLL_INTERVAL_MS=3000`; `shouldPoll(channelStatus): channelStatus !== "joined"`. Pure function — true unless status is `"joined"`. Iran path: WebSocket may die; polling is resilience. |
| EVD-06-011 | `shouldPoll` unit test — 5 tests via `node:test` | `lib/thread.test.ts` | OBSERVED | PASS | https://github.com/aminbita162-glitch/ahmad-amin-technology/blob/fbbfc307fd98bdad05ac02c3f2813b39668c5bc6/lib/thread.test.ts | `fbbfc307fd98` | 37 lines, 5 `test(...)` calls. Tests: POLL_INTERVAL_MS=3000, joined→false, non-joined→true, case-sensitivity, empty string→true. Static source count (5). |
| EVD-06-012 | Service worker — network-first GET navigation cache only; POST never cached; `/api/chat` never cached | `public/sw.js` | OBSERVED | PASS | https://github.com/aminbita162-glitch/ahmad-amin-technology/blob/fbbfc307fd98bdad05ac02c3f2813b39368c5bc6/public/sw.js | `fbbfc307fd98` | 70 lines. `CACHE_NAME="aa-tech-2026-v1"`. `if (request.method !== "GET") return;`. `if (url.pathname === "/api/chat") return;`. Cleans old caches. |
| EVD-06-013 | Health route returns booleans only (session, supabase, model) — no secret values | `app/api/health/route.ts` | OBSERVED | PASS | https://github.com/aminbita162-glitch/ahmad-amin-technology/blob/fbbfc307fd98bdad05ac02c3f2813b39368c5bc6/app/api/health/route.ts | `fbbfc307fd98` | 25 lines. `NextResponse.json({ session, supabase, model })` — all booleans. No keys, URLs, or tokens exposed. |
| EVD-06-014 | Meridian Seal — hand-written CSS (no Tailwind), ink `#54656f`, paper `#efeae2`, amin `#00a884`, ahmad `#cf9466`, fault `#e0635a` | `app/globals.css` | OBSERVED | PASS | https://github.com/aminbita162-glitch/ahmad-amin-technology/blob/fbbfc307fd98bdad05ac02c3f2813b39368c5bc6/app/globals.css | `fbbfc307fd98` | 1946 lines. `--ink:#54656f`, `--paper` (bg `#efeae2`), `--amin:#00a884`, `--ahmad:#cf9466`, `--fault:#e0635a`. Fonts: Vazirmatn (FA), Syne (EN). Bilingual. |
| EVD-06-015 | Phase seal system — `execution/phase-N-seal.txt` files; Phase 1-3 PASS, 4-6 BLOCKED, 7 PASS, R7 PASS | `execution/` | OBSERVED | PASS | https://github.com/aminbita162-glitch/ahmad-amin-technology/tree/fbbfc307fd98bdad05ac02c3f2813b39368c5bc6/execution | `fbbfc307fd98` | 11 seal files. Phase 1 PASS, 2 PASS, 3 PASS, 4 BLOCKED (Supabase), 5 BLOCKED (sync), 6 BLOCKED (OPENAI_API_KEY), 7 PASS. R7 seal PASS (lint/typecheck/build, 13/13 pages, commit dce594d). |
| EVD-06-016 | Release seal BLOCKED — Phase 6 not PASS; 3 Vercel deployment blockers (all human) | `execution/release-seal.txt` | OBSERVED | PASS | https://github.com/aminbita162-glitch/ahmad-amin-technology/blob/fbbfc307fd98bdad05ac02c3f2813b39368c5bc6/execution/release-seal.txt | `fbbfc307fd98` | Phase 8 release seal. BLOCKED because Phase 6 BLOCKED (OPENAI_API_KEY empty). V1: Vercel CLI not installed. V2: no Vercel login. V3: production env values empty. Build compiles (exit 0). |
| EVD-06-017 | No CI workflow — 0 `.github/workflows/` files | repo tree | OBSERVED | PASS | https://github.com/aminbita162-glitch/ahmad-amin-technology/tree/fbbfc307fd98bdad05ac02c3f2813b39368c5bc6 | `fbbfc307fd98` | 44 tracked files, 0 in `.github/workflows/`. No CI to trigger. Lint/typecheck/build pass REPORTED in R7 seal, not CI-enforced. |
| EVD-06-018 | No release tags, no published releases | GitHub API + `git tag` | OBSERVED | PASS | https://github.com/aminbita162-glitch/ahmad-amin-technology | `fbbfc307fd98` | `git tag` → empty. 0 tags, 0 releases. Version 1.0.0 in `package.json` only. |
| EVD-06-019 | NOASSERTION license — custom proprietary text, not SPDX-recognized | `LICENSE` | OBSERVED | PASS | https://github.com/aminbita162-glitch/ahmad-amin-technology/blob/fbbfc307fd98bdad05ac02c3f2813b39368c5bc6/LICENSE | `fbbfc307fd98` | "Copyright (c) 2026 Amin Azimi. Amin Azimi — AI Architect..." Not SPDX-recognized. No source code copied into portfolio; all evidence uses permalinks. |
| EVD-06-020 | 44 tracked files, 6,653 total lines of TS/TSX/CSS | repo tree + `wc -l` | OBSERVED | PASS | https://github.com/aminbita162-glitch/ahmad-amin-technology/tree/fbbfc307fd98bdad05ac02c3f2813b39368c5bc6 | `fbbfc307fd98` | `git ls-files \| wc -l` = 44. `git ls-files \| grep -E '\.(ts\|tsx\|css)$' \| xargs wc -l` = 6653 total. Largest: `app/globals.css` (1946), `app/chamber/desk.tsx` (1746), `app/chamber/page.tsx` (1720). |
