---
name: web-load-perf-playbook
description: Playbook for making a web app load fast and for replacing a slow frontend safely, distilled from X's 2026 rebuild of x.com (three servers into one, first post 22s -> 2.7s on Slow 4G). Use when diagnosing slow first load / long splash or skeleton screens, planning SSR or streaming, cutting startup JS/CSS/HTML bytes, deciding rewrite vs upgrade-in-place, planning a gradual traffic migration off a legacy frontend, or setting up trustworthy perf measurement. Triggers - 首屏慢, 白屏, 加载优化, 性能优化方案, 首屏优化, bundle 太大, chunk 太多, SSR, streaming, hydration, 重写还是重构, 灰度迁移, Lighthouse, Core Web Vitals, LCP, TTFB.
---

# Web load performance playbook

Source: X Engineering, "Rebuilding the global town square from scratch"
(2026-11-04). This is a method, not a stack recommendation: apply the
reasoning, then pick tools that fit the target repo.

Work through the phases in order. Stop after Phase 1 and report before
proposing changes; most wrong perf work comes from skipping the diagnosis.

## Phase 1 — Diagnose the chain before the first useful content

1. Pick the **metric that means "useful"**, not the first paint. A splash
   screen or skeleton makes FCP look good. Define a product metric
   (e.g. "time to first post / first row / editor ready").
2. Write down every step between navigation and that moment, in order:
   HTML -> CSS -> JS download -> JS execute -> data request -> render.
   Each serial step is a network round trip; on a slow link they add up.
3. Count, per step: round trips, compressed bytes, number of files, and what
   blocks what (render-blocking CSS, deferred scripts, serial API calls).
4. Classify the cause:
   - **Architectural** — content can only appear after JS runs and then
     fetches data (app-shell pattern). Tuning will not fix it -> Phase 2.
   - **Incremental** — content is in the first response but something delays
     it -> Phase 3 checklist.

## Phase 2 — Architecture decisions

**Rewrite vs improve-in-place.** List the options honestly:
improve in place / swap the toolchain but keep the architecture / new app.
A toolchain swap touches every file and leaves the architecture that causes
the problem. Choose a rewrite only when all of these hold:

- the problem is architectural (Phase 1), not build speed or code style;
- the old system keeps shipping during the rewrite;
- traffic can move gradually and move back (Phase 5);
- the old system can serve as an executable specification (Phase 5).

Write down **what to keep** first (URLs, redirects, aliases, edge-case
handling, SEO behaviour, product model). Rewrites throw away too much.

**One server, rendering chosen per audience.** Several renderers for the
same page drift; every fix must land N times. Serve all audiences from one
codebase and branch on who is asking:

| Audience | Response |
|---|---|
| People | Streamed HTML: shell first, each section as its data arrives |
| Verified search crawlers | Full buffered document, all boundaries resolved, no client bundle / hydration state / placeholders |
| Link-preview bots | Head tags only (title, description, image); skip body render and body-only data loading |
| Unverified "crawlers" | Same as a normal logged-out visitor |

Decide "is a crawler" from **edge-verified IP**, not User-Agent: a UA check
puts Lighthouse on the buffered path (so you measure the wrong thing) and
lets anyone request the expensive path with a header.

**Data.** Routes declare their data up front; loaders start requests while
the server is still rendering. Embed each API response in the page once, as
raw JSON, and replay it into the client cache — do not serialize the
normalized store (it was 4-5x larger). Keep unread responses briefly
(X: 30s) for soon-opened tabs/dialogs.

**Navigation.** Preload a route's code and data on intent (hover/focus;
for non-link items, pointer resting ~100 ms or press — not when content
scrolls under a still pointer).

**Isolation.** Split entry points by audience (logged-out pages download
only logged-out code). Run render workers that are replaced when their event
loop or memory stays over a limit, so one stuck render stays local.

## Phase 3 — Optimization checklist

Check each item against the target app; report which apply, with evidence.

### Server

- [ ] **Event-loop saturation.** Measure per-worker event-loop utilization
      and longest stall. A stalled loop fires timers before it processes
      responses that already arrived, producing fake upstream timeouts
      (X: 1s timeout firing on 1-9% of renders against a 25 ms service).
      Fix: more workers, less traffic per loop — not longer timeouts.
- [ ] **Serial requests with no dependency.** Any request waiting on another
      whose result it does not use -> start them together.
- [ ] **Payload embedded in HTML.** Raw responses once, not normalized
      store records plus incremental updates.
- [ ] **Document size for crawlers.** Googlebot reads only the first 2 MB.
      - Repeated inline SVG icons -> one hidden sprite, reference it.
      - Structured data repeating text/URLs -> keep one of each.
      - Strip anything that only matters after JS runs (timestamp scripts,
        router state, skeletons, sign-in buttons).

### Browser

- [ ] **Entry script timing with streaming.** A `defer`/deferred module
      runs only after the whole document parses, i.e. after the slowest
      streamed boundary. Use an async module so the shell hydrates at once
      and each boundary hydrates when its HTML lands.
- [ ] **Startup file count.** Hydration waits for the entire static import
      graph. ~100 concurrent HTTP/2 streams means hundreds of tiny chunks
      cost several round trips. Pack startup modules into a few chunks,
      grouped by change frequency: dependencies / own code / the manifest
      naming lazy chunks (changes almost every deploy).
- [ ] **Stylesheet first.** On slow links JS downloads steal bandwidth from
      render-blocking CSS. Inject the entry script and chunk preloads after
      the stylesheet loads. Trade-off: hydration may finish slightly later —
      measure and report it.
- [ ] **Bytes nobody reads.**
      - Default-locale catalog mapping keys to themselves -> render from
        the key, do not download the catalog.
      - Full ICU parser -> a parser covering only what the catalogs use.
      - Build plugins injecting per-chunk boilerplate (e.g. release IDs).
      - Images: request the modern format the CDN already serves (WebP).
- [ ] **Shared runtime across federated modules.** Lend the host's React
      and shared libraries to remotes instead of each shipping its own.

## Phase 4 — Measurement rules

A change without these is a claim, not a result.

- State **where** each number was measured: production build in a lab,
  dev server, or production fleet.
- Fixed throttling profile, stated in full (e.g. Slow 4G: 150 ms RTT,
  1.6 Mbps down, 750 Kbps up; CPU slowdown stated separately).
- Fresh browser per load, empty cache, service workers off.
- Several runs, **interleaved** between before/after; report the median.
  Call it a change only when the 95% bootstrap interval excludes zero.
- Add latency with a proxy to reproduce realistic round trips (use the p90
  RTT of real users).
- Report regressions next to gains, including TTFB cost of server work.
- Lighthouse CI on PRs against fixed, stable pages.
- Lab numbers need field numbers beside them: ship Core Web Vitals
  (LCP, INP, CLS) plus FCP and TTFB from real visits.

## Phase 5 — Migration

- **Old system = specification.** For each page type, diff old vs new
  output and close gaps one at a time. Pin each ported behaviour with a
  test that names where it came from.
- **Parity script.** Request the same path from both systems and compare
  what the consumer reads (status, redirect, title, description, canonical,
  robots, social tags, alternates count, structured-data types). Rerun it.
- **Order by risk.** Machine audiences and logged-out pages first, then
  signed-in users by cohort (X: one country at a time).
- **Edge routing.** Route table published from the new app's own route
  tree on every deploy; unknown paths fall through to the old app. A
  feature switch decides who is in, changeable without a deploy, and
  removal sends people straight back.
- **First cohort is an experiment**: split it, compare both apps on real
  traffic.
- **Delete as you go.** Retire each old path once its traffic is gone, and
  shrink the old fleet as load moves.
- **Shared features** built once and loaded by both apps at runtime
  (Module Federation) so they do not block the migration.

## Phase 6 — Agent-driven implementation guardrails

When agents do most of the porting:

- One root instructions file with code rules and "nothing is committed
  until lint, typecheck and relevant tests pass".
- Recurring workflows as repo skills, updated in one place.
- Conventions as lint rules, including rules against typical agent output
  (chained type assertions, `unknown` parameters, conditional spreads of
  empty objects). Pre-commit: format + lint + typecheck. Pre-push: lint
  dependants.
- Humans own architecture, scope and review, and verify in a real browser.

## Output

Report in this shape:

1. Useful-content metric and the measured chain (Phase 1 table).
2. Cause: architectural or incremental.
3. Applicable checklist items, each with evidence and expected gain.
4. Measurement plan (Phase 4) and, if traffic moves, rollout plan (Phase 5).

Do not quote X's numbers as expected results for another app; they
illustrate the size of an effect, measured on X's pages.
