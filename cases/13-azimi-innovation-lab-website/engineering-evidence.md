# PRJ-13 — Azimi Innovation Lab Website: Engineering Evidence

**Project ID:** PRJ-13
**Canonical name:** Azimi Innovation Lab Website
**Source:** https://azimi-innovation-lab-website.vercel.app/ (live URL only — no source repository disclosed)
**Source HEAD:** NOT_AVAILABLE — no source repository known or disclosed
**License:** UNKNOWN — no source repository to inspect
**Language:** JavaScript/TypeScript (Vite SPA — inferred from HTML structure and JS bundle)
**Phase 6 inspection date (UTC):** 2026-10-18
**Independence:** Source/documentation observations only; no independent third-party verification claimed.
**Independence level:** Source/documentation observations only; no independent reproduction claimed.
**Evidence maturity:** R0 — no source artifact available. Only live URL content and JS bundle inspected. No source code repository, no tests, no CI, no releases.
**Reproducibility ladder:** R0 (live URL content captured at timestamp 2026-10-18T00:00:00Z). No source revision to pin. URL content is volatile — may change without notice.

---

## 1. Context and Scope

The Azimi Innovation Lab Website is the public identity surface for Amin Azimi as an AI Systems Architect, hosted under the Azimi Innovation Lab brand on Vercel. It is a single-page application (SPA) built with Vite, serving as the professional profile and brand presence. It is **not a SaaS product, not a client-facing application with user accounts, and not a portfolio management system.**

**Critical constraint:** No source repository has been disclosed or found for this website. The website repository is unknown. All evidence is based on the live URL HTML content and the served JS bundle. No source code, tests, CI, or releases are available for inspection.

**Status (OBSERVED via HTTP inspection 2026-10-18):**

| Field | Value | Observation class |
|-------|-------|--------------------|
| Live URL | https://azimi-innovation-lab-website.vercel.app/ | OBSERVED (HTTP 200) |
| HTML title | "Amin Azimi \| AI Systems Architect \| Azimi Innovation Lab" | OBSERVED |
| Meta description | "Amin Azimi designs end-to-end AI systems for complex business challenges under the personal brand Azimi Innovation Lab." | OBSERVED |
| Language | `lang="fa" dir="rtl"` (Persian, right-to-left) | OBSERVED |
| Hosting | Vercel (inferred from URL) | OBSERVED |
| JS framework | Vite (inferred from `/assets/index-*.js` bundle pattern) | OBSERVED |
| JS bundle size | 453,536 chars (~443 KB) | OBSERVED |
| Source repository | UNKNOWN — not disclosed, not found | BLOCKED |
| CI | UNKNOWN — no source repo | BLOCKED |
| Tests | UNKNOWN — no source repo | BLOCKED |
| Releases | UNKNOWN — no source repo | BLOCKED |
| License | UNKNOWN — no source repo | BLOCKED |

**Non-goals (OBSERVED):**

- Not a SaaS product.
- Not a client-facing application with user accounts.
- Not a portfolio management system.

---

## 2. System Architecture (OBSERVED FROM LIVE URL — NOT RECONSTRUCTED FROM SOURCE)

Architecture is inferred from the live deployed HTML and JS bundle content. **No source code is available.** This is NOT a source-level architecture reconstruction — it is a live-URL and bundle content observation.

### 2a. Page Structure (OBSERVED)

The root HTML document (`index.html`, 1,615 chars) is a minimal SPA shell:

- `<html lang="fa" dir="rtl">` — Persian language, right-to-left direction
- `<title>Amin Azimi | AI Systems Architect | Azimi Innovation Lab</title>`
- Meta tags: description, keywords, author, Open Graph (og:title, og:description, og:type, og:image)
- Favicon: `/azimi-logo.PNG`
- Font: Vazirmatn (300-800 weights) via Google Fonts preconnect
- JS entry: `/assets/index-sfx2RhJi.js` (module, crossorigin)
- CSS: `/assets/index-lXImfDC6.css`
- `<div id="root"></div>` — React/Vite mount point

### 2b. JS Bundle Content Analysis (OBSERVED)

The JS bundle (453,536 chars / ~443 KB) contains string literals revealing the application's content and structure:

**Identity strings (OBSERVED in bundle):**
- "Amin Azimi · AI Systems Architect · Azimi Innovation Lab"
- "AI Systems Architect | End-to-End AI Systems for Complex Business Challenges"
- "I design and build end-to-end AI systems that turn complex operational problems into governed, testable, deployable architectures."
- "Systems — Selected end-to-end AI systems for operational problems."

**Multilingual content (OBSERVED in bundle):**
- Persian: "امین عظیمی شما را به وب‌سایت Azimi Innovation Lab خوشامد می‌گوید"
- German: "Amin Azimi heißt Sie auf der Website des Azimi Innovation Lab willkommen"
- Ukrainian: "Амін Азімі вітає вас на вебсайті Azimi Innovation Lab"
- Spanish: "Amin Azimi le da la bienvenida al sitio web de Azimi Innovation Lab"

**Project links (OBSERVED in bundle):**
- AILORA: https://github.com/azimilab2025-ai/ailora (with links to README, releases, actions, qualification docs)
- IBM Challenge: https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026 (with links to README, verification.md, evaluate_ai.py, commits, tags, releases)
- Aegis Agent: https://github.com/aminazimi42-coder/aegis-agent
- "Not an IBM product. Not a claim that IBM deployed this in enterprise production. It is Amin Azimi's challenge system with live candidate endpoints and reproducible checks."
- AI Builders Challenge projects page: https://aibuilderschallenge-bobhub.bemyapp.com/#/projects

**UI components (INFERRED from bundle):**
- Framer Motion animations (`whileHover`, `whileTap`, `scale` transitions)
- Project cards with links to GitHub repositories
- Contact button
- "View selected systems" navigation
- Projects button

### 2c. SEO and Accessibility (OBSERVED)

| Aspect | Value | Status |
|--------|-------|--------|
| Title tag | "Amin Azimi \| AI Systems Architect \| Azimi Innovation Lab" | PASS — descriptive, keyword-rich |
| Meta description | Present, accurate | PASS |
| Meta keywords | "AI Systems Architect, Amin Azimi, Azimi Innovation Lab, end-to-end AI systems" | PASS |
| Open Graph | og:title, og:description, og:type, og:image present | PASS |
| Twitter card | summary, title, description | PASS |
| Language attribute | `lang="fa"` | PASS — correct for Persian content |
| Direction | `dir="rtl"` | PASS — correct for Persian |
| Viewport | `width=device-width, initial-scale=1.0` | PASS — mobile responsive |
| Favicon | `/azimi-logo.PNG` | PASS |
| Apple touch icon | `/azimi-logo.PNG` | PASS |
| Font loading | Google Fonts preconnect + Vazirmatn | PASS — but external dependency |
| Alt text | Logo image has `alt="Azimi Innovation Lab Logo"` (in JS bundle) | PASS |

---

## 3. Source Provenance

| Field | Value |
|-------|-------|
| Live URL | https://azimi-innovation-lab-website.vercel.app/ |
| HTTP status | 200 (2026-10-18T00:00:00Z) |
| HTML content length | 1,615 chars |
| JS bundle | /assets/index-sfx2RhJi.js (453,536 chars) |
| CSS bundle | /assets/index-lXImfDC6.css |
| Source repository | UNKNOWN — not disclosed, not found |
| License | UNKNOWN — no source to inspect |
| CI | UNKNOWN — no source |
| Tests | UNKNOWN — no source |
| Releases | UNKNOWN — no source |
| Inspection method | HTTP GET + HTML parsing + JS bundle string extraction |

---

## 4. Published Claims Assessment

The website makes several public claims that the portfolio must assess for consistency:

| Claim | Source | Observation class | Assessment |
|-------|--------|-------------------|------------|
| "AI Systems Architect" | HTML title, meta description | OBSERVED | Consistent with portfolio identity |
| "End-to-End AI Systems for Complex Business Challenges" | HTML title | OBSERVED | Consistent with portfolio thesis |
| "Azimi Innovation Lab" | HTML title, meta | OBSERVED | Consistent with portfolio brand |
| "Amin Azimi designs end-to-end AI systems" | Meta description | OBSERVED | Consistent |
| Project links to AILORA, IBM Challenge, Aegis | JS bundle | OBSERVED | Links point to real GitHub repos verified in earlier phases |
| "Not an IBM product" disclaimer | JS bundle | OBSERVED | Consistent with portfolio case (CLM-03-004, EVD-03-004) — correct IBM non-endorsement qualification |
| AI Builders Challenge link | JS bundle | OBSERVED | External link to bemyapp.com — challenge affiliation, not endorsement |

**No unsupported claims detected.** The website's public claims are consistent with the portfolio's evidence-first positioning. The IBM non-endorsement disclaimer is present and correct. No SaaS, production-ready, or revenue claims are made on the website.

---

## 5. Engineering Decisions

### ADR-13-001: Vite SPA for static identity site

| Field | Value |
|-------|-------|
| ADR_ID | ADR-13-001 |
| Source SHA | NOT_AVAILABLE — no source repo |
| Problem | Public identity surface for an AI Systems Architect |
| Constraints | Fast load, SEO-friendly, multilingual, zero infrastructure |
| Selected approach | Vite SPA with minimal HTML shell + JS bundle; Vercel hosting |
| Alternatives | Static HTML, Next.js SSR, WordPress |
| Trade-offs | SPA means content is in JS bundle (not server-rendered HTML) — SEO depends on meta tags, not body content |
| Failure implications | If JS fails, page shows empty `#root` div; search engines see meta tags but no body content |
| Evidence | HTML has `<div id="root"></div>`; all content in JS bundle; Framer Motion animations |
| Status | INFERRED — consistent with observed HTML structure |

### ADR-13-002: Persian (RTL) primary language with multilingual greeting

| Field | Value |
|-------|-------|
| ADR_ID | ADR-13-002 |
| Source SHA | NOT_AVAILABLE — no source repo |
| Problem | Personal brand site for a Persian-speaking AI architect |
| Constraints | Persian is primary audience; English is secondary |
| Selected approach | `lang="fa" dir="rtl"`; Vazirmatn font; multilingual greeting in JS bundle (FA, EN, DE, UK, ES) |
| Alternatives | English-primary with Persian toggle |
| Trade-offs | Correct for primary audience; RTL layout requires careful CSS |
| Failure implications | RTL may break if CSS not properly direction-aware |
| Evidence | HTML `lang="fa" dir="rtl"`; Vazirmatn font; multilingual strings in bundle |
| Status | INFERRED — consistent with observed content |

---

## 6. Known Limitations and Gaps

| Gap ID | Severity | Description |
|--------|----------|-------------|
| GAP-13-001 | HIGH | No source repository — all claims are E0/REPORTED at source level. No source code, tests, CI, or releases can be inspected. |
| GAP-13-002 | MEDIUM | SPA content is in JS bundle, not server-rendered HTML — SEO depends on meta tags. Search engines that don't execute JS see only the empty `#root` div. |
| GAP-13-003 | MEDIUM | External font dependency (Google Fonts) — if Google Fonts is unavailable, Vazirmatn fallback to sans-serif. Not a hard dependency but a UX degradation. |
| GAP-13-004 | LOW | JS bundle is 443 KB — moderately large for an identity site. Could be optimized with code splitting or static generation. |
| GAP-13-005 | LOW | Framer Motion dependency — adds bundle weight for animations on a static identity site. |
| GAP-13-006 | LOW | No `robots.txt` or `sitemap.xml` observed — may affect search engine crawl efficiency. Not verified (not fetched). |
| GAP-13-007 | LOW | Website repository unknown; consistent with the README statement. |

---

## 7. Claim Comparison

| Claim ID | Claim | Phase 1 status | Phase 6 assessment |
|----------|-------|----------------|---------------------|
| CLM-13-001 | Azimi Innovation Lab Website is live on Vercel | REPORTED/NOT_TESTED | **Upgraded to OBSERVED/PASS.** HTTP GET to https://azimi-innovation-lab-website.vercel.app/ returns 200 on 2026-10-18. HTML content 1,615 chars with title "Amin Azimi \| AI Systems Architect \| Azimi Innovation Lab". Site is live and accessible. |
| CLM-13-002 | Website source repository is unknown | REPORTED/BLOCKED | **Confirmed OBSERVED/BLOCKED.** No source repository has been disclosed or found. The README states the same; source remains UNKNOWN. All evidence is from live URL content only. |

---

## 8. Reproducibility Recipe

To independently inspect the Azimi Innovation Lab website at the observed state:

1. Fetch the root page: `curl -s https://azimi-innovation-lab-website.vercel.app/`
2. Verify HTTP 200 and HTML title "Amin Azimi | AI Systems Architect | Azimi Innovation Lab"
3. Verify `lang="fa" dir="rtl"` in HTML
4. Fetch the JS bundle: `curl -s https://azimi-innovation-lab-website.vercel.app/assets/index-sfx2RhJi.js`
5. Search bundle for identity strings: "Amin Azimi", "AI Systems Architect", "Azimi Innovation Lab"
6. Search bundle for project links: AILORA, IBM Challenge, Aegis Agent GitHub URLs
7. Verify "Not an IBM product" disclaimer is present in bundle
8. Verify meta tags: description, keywords, author, Open Graph, Twitter card
9. Note: URL content is volatile; capture retrieval timestamp for any claim
10. Note: JS bundle hash may change on redeployment; content is not version-pinned

---

## 9. Evidence Matrix

| EVD ID | Claim | Source | Observation class | Result | Notes |
|--------|-------|--------|-------------------|--------|-------|
| EVD-13-001 | Website live on Vercel (HTTP 200) | https://azimi-innovation-lab-website.vercel.app/ | OBSERVED | PASS | HTTP 200 on 2026-10-18. 1,615 chars HTML. Title present |
| EVD-13-002 | HTML title: "Amin Azimi \| AI Systems Architect \| Azimi Innovation Lab" | HTML `<title>` | OBSERVED | PASS | Exact title tag matches |
| EVD-13-003 | Meta description present and accurate | HTML `<meta name="description">` | OBSERVED | PASS | "Amin Azimi designs end-to-end AI systems for complex business challenges under the personal brand Azimi Innovation Lab." |
| EVD-13-004 | Persian primary language (lang="fa" dir="rtl") | HTML `<html>` | OBSERVED | PASS | `lang="fa" dir="rtl"` — correct for Persian content |
| EVD-13-005 | Vazirmatn font via Google Fonts | HTML `<link>` | OBSERVED | PASS | Preconnect + stylesheet link for Vazirmatn (300-800). External dependency (GAP-13-003) |
| EVD-13-006 | Vite SPA architecture | HTML + JS bundle | OBSERVED | PASS | `<div id="root">` + `/assets/index-*.js` module script. Vite bundle pattern |
| EVD-13-007 | JS bundle 453,536 chars (~443 KB) | /assets/index-sfx2RhJi.js | OBSERVED | PASS | Fetched and sized. Moderately large (GAP-13-004) |
| EVD-13-008 | Identity string: "Amin Azimi · AI Systems Architect · Azimi Innovation Lab" | JS bundle | OBSERVED | PASS | String literal found in bundle |
| EVD-13-009 | "I design and build end-to-end AI systems" statement | JS bundle | OBSERVED | PASS | String literal found: "I design and build end-to-end AI systems that turn complex operational problems into governed, testable, deployable architectures." |
| EVD-13-010 | Project links to AILORA, IBM Challenge, Aegis | JS bundle | OBSERVED | PASS | GitHub URLs for ailora, IBM-Bob-Challenge-2026, aegis-agent found in bundle. Links verified in earlier phases |
| EVD-13-011 | IBM non-endorsement disclaimer present | JS bundle | OBSERVED | PASS | "Not an IBM product. Not a claim that IBM deployed this in enterprise production." Consistent with CLM-03-004 |
| EVD-13-012 | Multilingual greeting (FA, EN, DE, UK, ES) | JS bundle | OBSERVED | PASS | 5 language greeting strings found in bundle |
| EVD-13-013 | Open Graph + Twitter card meta tags | HTML `<meta>` | OBSERVED | PASS | og:title, og:description, og:type, og:image, twitter:card, twitter:title, twitter:description |
| EVD-13-014 | No source repository found | GitHub search | OBSERVED | PASS | No repo disclosed. |
| EVD-13-015 | Framer Motion animations in bundle | JS bundle | OBSERVED | CONDITIONAL | `whileHover`, `whileTap`, `scale` transition strings found. Adds bundle weight (GAP-13-005) |
