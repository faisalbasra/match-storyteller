# Match Storyteller — Product Requirements and Build Contract

**Status:** Version 1.0, implementation-ready plan (2026-10-08)

**Owner:** Solo hackathon entrant

**Target effort:** 10–15 focused hours
**Event:** [Inside the Game: Developer Hackathon](https://events.microsoft.com/flow/microsoft/plmshackathon/registration/page/landing)

## 1. Product in one sentence

Match Storyteller replays **original, synthetic football events** as a simulated live match, computes trustworthy statistics, and uses an AI narrator to explain selected moments differently to a casual fan and an analyst.

This is a compact, locally runnable demonstration, not a production broadcast service. "Live" means replay of synthetic events at an accelerated pace, **not** ingestion of a real match, video analysis, or a claim of production latency.

## 2. Non-negotiable constraints

1. Build a **new** project for this hackathon; no Premier League footage, official logos, real player identities, scraped match data, or unlicensed media. Synthetic teams, players, events, and graphics only.
2. Python is the application language. The app runs locally without Azure or a paid deployment. The entrant uses Copilot CLI or Codex CLI **to write the application**; the application's runtime calls the entrant's existing LiteLLM proxy **only for AI inference**. Do not put a coding agent in the runtime request path.
3. Every displayed statistic is computed deterministically from replayed events. The model explains supplied facts; it does not invent, calculate, or alter match state.
4. Two genuinely different audience modes are required: **Fan** (plain language and excitement) and **Analyst** (tactical context and evidence). Switching modes changes generated insight, not the underlying facts.
5. A key moment must answer **what happened, why it matters, and what evidence supports that explanation**. The UI must show evidence alongside prose.
6. No Azure spend is required or assumed. The proxy may incur upstream costs: restrict runtime requests and configure a spend cap at the proxy/provider separately. Never present an Azure budget alert as a spending cutoff.
7. An AI failure must be visible as an error; do not silently display canned narration as if it came from the model. Facts, statistics, and replay remain usable when inference is unavailable.
8. The public repository contains no API keys, personal proxy credentials, private match data, or private service configuration.

## 3. Hackathon fit and known uncertainty

The [official rules](https://github.com/microsoft/insidethegamehackathon/blob/main/OFFICIAL%20RULES.md) describe an ingest → interpret → explain → render → personalize pipeline and require synthetic, football-realistic data. Stage-one judging checks reasonable use of featured technologies; later judging considers technological implementation, agentic design, impact, UX, and category fit. Use **Microsoft Agent Framework for the narrator agent** if the compatibility gate in section 9 passes. This is a real runtime integration, not just a claim that a coding assistant was used.

The rules do **not** promise entrant Azure credits or supplied footage/data. They require a public GitHub repository, a publicly accessible demo video **under two minutes**, a description naming Microsoft/Azure technologies used, and access to a functioning demo or test build through judging. A local test build with instructions is the planned access path; judges may choose to judge solely from the submission materials. **Do not claim an Azure deployment, Foundry usage, multi-agent system, or eligibility for an Azure-specific prize unless actually demonstrated.** Check any additional requirements shown in the registration/submission portal before submitting.

Registration closes **October 20, 2026, noon Pacific**; submission closes **October 27, 2026, 11:59 p.m. Pacific**. The rules list judging through **November 10, 2026**. Verify portal deadlines before submission.

## 4. User, problem, and success

**Primary user:** A viewer of a fictional match who wants to understand its key moments without reading a wall of statistics. **Secondary user:** A technically curious judge who needs to verify that the AI output is grounded in a working application.

**Problem:** An event feed says "shot" or "possession changed" but not why the moment mattered, and a single explanation does not serve both novice fans and analysts.

**Success criteria (demonstrable, not aspirational):**

- One command starts the web app after configuration; a documented test mode can run without a live proxy.
- A browser can start, pause, reset, and step through a **60–100-event** synthetic match covering minutes 0–90 in roughly **60–120 seconds** at normal replay speed.
- Score and basic statistics are correct at **every replay prefix**, including after reset and stepping; seeded fixture generation produces byte-for-byte equivalent event content across runs.
- **At least five distinct key moments** receive AI narration during a complete replay when the proxy is healthy; fan and analyst outputs differ in vocabulary and emphasis on a fixed evaluation set of at least three moments.
- Each insight displays a headline, explanation, evidence references, and match minute; every displayed evidence reference resolves to a fact derived from the replay prefix.
- No more than **12 uncached model calls per server process**, at most **one active call at a time**, and no automatic retry. Requests for the same event/audience/preference combination use an in-process cache. A process restart resets this demo-only limit.
- When the proxy fails, is misconfigured, times out, or returns invalid content, the UI states that narration is unavailable and still shows the deterministic match state. No secrets appear in logs, responses, or the public repo.

## 5. MVP journeys and UI

1. Open `http://127.0.0.1:8000`. See "Fictional match / synthetic data" prominently, team names, 0–0 score, event timeline, stats, mode selector, and Play/Pause/Step/Reset controls.
2. Choose Fan or Analyst, optionally select a favorite of the two fictional teams, and press Play. Events advance in order; score and metrics update instantly. Favorite team changes *framing*, never source facts.
3. When a qualifying moment occurs, the app requests an explanation in the background. The timeline continues even if AI is slow. A pending state and then a grounded insight card or explicit failure state appear at the appropriate event.
4. Pause, inspect an insight's evidence (event IDs and computed values), switch mode, and regenerate that moment's explanation **on demand** within the same call limit.
5. Reset returns match, timeline, and metrics to the beginning. The model-call counter does **not** reset; it reflects calls already made by this server process. Repeat prompts may be served from the in-process cache.

**One-page layout:** header with synthetic-data label and score/minute; controls and mode/favorite selectors; two-column area with event timeline + insight cards and a small stats panel. On mobile, stack columns. Use text and simple CSS/SVG shapes created for this project, with accessible controls, keyboard focus, and visible loading/error states. No external assets needed.

## 6. Scope boundaries

**Must ship:** Synthetic fixture generator, event replay, deterministic stats and score, key-moment selection, grounded AI narration, two modes, evidence display, basic tests, runnable README, demo video, public repo.

**Only after must-ship is complete:** A second fixture; language selection (English plus one other language only if checked for correctness); a download of machine-readable insight JSON; an optional deployment funded and controlled by the entrant. These are not necessary for the core submission.

**Explicitly out of scope:** Real feeds, video ingestion/computer vision, actual live-stream overlays, real player/team data, xG or possession percentages without defensible source data, authentication, accounts, persistent database, semantic search/RAG, tool-using multi-agent theater, AKS, Foundry, paid Azure infrastructure, social sharing, and guaranteed real-time broadcast latency.

## 7. Tech stack and architecture

| Layer | Choice | Why |
|---|---|---|
| Language/runtime | Python 3.12 | Preferred stack; straightforward deterministic domain logic |
| HTTP/UI | FastAPI + Jinja2 HTML template + small vanilla JavaScript/CSS | One local server; no npm, SPA build, or external hosting |
| Types/contracts | Pydantic models and Python type hints | Input/output validation at HTTP and model boundaries |
| AI orchestration | Microsoft Agent Framework, **single narrator agent** | Meaningful Microsoft runtime technology without unnecessary multi-agent complexity |
| AI transport | OpenAI-compatible **chat completions** to user-controlled LiteLLM proxy, configured via environment | Existing inference access; broad compatibility; no Azure account dependency |
| Dependency management | `uv` with `pyproject.toml` and committed lockfile | Reproducible local installation |
| Tests | pytest + FastAPI TestClient/httpx; fake AI client | Fast and fully offline deterministic tests |
| Storage | Immutable fixture JSON/generated in memory; ephemeral process-local narration cache | No database or paid service |
| Development | Copilot CLI or Codex CLI, human-reviewed changes | Development tools only; not runtime dependencies |

**Data flow:** `seeded fixture → ordered events → pure stat reducer → key-moment selector → fact packet → Agent Framework narrator → validated insight → browser card`. Browser replay is client-timed and invokes narration asynchronously: it does not send raw browser-authored "facts" to the model. The server owns the fixture, derives facts through the requested event index, and validates mode/preference inputs.

**Files (target layout, adjust names only if needed):**

```text
app/
  main.py                  # FastAPI routes, app wiring, health
  config.py                # validated environment settings
  models.py                # event, snapshot, fact, insight contracts
  fixture.py               # seeded synthetic generator and fixture validation
  reducer.py               # score/stats/event-prefix facts; pure functions
  moments.py               # deterministic key-moment ranking and fact packets
  narrator.py              # Agent Framework adapter, prompt, response validation
  templates/index.html
  static/app.js
  static/styles.css
tests/
  test_fixture.py
  test_reducer.py
  test_moments.py
  test_api.py
  test_narrator.py
pyproject.toml
uv.lock
.env.example
.gitignore
README.md
PRD.md
```

Keep domain functions independent of FastAPI and the model SDK. No global model calls during import or application startup. No raw credentials in fixtures or snapshots.

## 8. Data contract and match rules

**Fixture:** Exactly two fictional teams, each with 11 unique fictional players; fixed seed (e.g. `20261008`); 60–100 events with strictly increasing `(minute, second, sequence)` order. Generate and validate at startup once. Include an intentionally designed, believable arc with **at least two goals, three shots on target, two tackles, and six possession changes**. Names, colors, IDs, and timestamps are synthetic. A seeded generator may fill connective passes/tackles, but required anchor moments must be guaranteed; do not hope random events produce them.

**Event schema** (JSON-compatible; no coordinates or advanced metrics in MVP):

```json
{
  "id": "e042",
  "sequence": 42,
  "minute": 57,
  "second": 18,
  "team_id": "harbor",
  "player_id": "harbor-07",
  "type": "shot",
  "outcome": "goal",
  "possession_team_id": "harbor",
  "metadata": {}
}
```

Allowed `type` values: `pass`, `shot`, `tackle`, `possession_change`. Allowed outcomes: pass `complete|incomplete`; shot `off_target|saved|goal`; tackle `won|lost`; possession change `turnover|interception|restart`. Every event has a canonical actor and team. `possession_team_id` is the team in possession **after** this event; validate transition consistency (e.g. a goal followed by restart for the opposing team). Event IDs and sequence numbers are unique; timestamps are bounded to 0:00–90:59; player IDs must belong to their event team.

**Reducer:** Apply events in sequence and expose at each prefix: goals per team, total shots per team, shots on target (`saved|goal`), completed/attempted passes, successful tackles, current possession team, and a chronological event list. A goal increments both shots and shots on target. A pass counted as complete must have outcome `complete`; changing possession is represented explicitly via `possession_change` or restart, not guessed from adjacent events. Do **not** display a percentage of possession or claim a team is "pressing" solely from this limited event data.

**Key moments:** Deterministic priority: all goals; selected saved shots; qualifying changes in possession after at least three consecutive completed passes by one team; optionally a late equalizer if fixture contains one. Select **5–8 moments maximum** per fixture, spacing non-goal moments by at least five simulated minutes; goals are never suppressed. At each moment create a `FactPacket` with `event_id`, match time, score before/after, triggering event type/outcome, current team/player, and up to three **evidence items** with stable IDs, labels, values, and their source event IDs. Example: "Harbor completed 4 consecutive passes before this shot" must be calculated from the event prefix and link to those four passes. Avoid causal/tactical claims the events cannot substantiate.

**Example narration result shape:**

```json
{
  "event_id": "e042",
  "mode": "fan",
  "headline": "Harbor takes the lead",
  "explanation": "A short move ended with a goal; Harbor now leads.",
  "why_it_matters": "The score changed at this moment.",
  "evidence_ids": ["score-after-e042", "shot-e042"]
}
```

The endpoint returns the server-computed evidence values separately with the insight. Length limits: headline ≤90 characters, each body field ≤240 characters, 1–3 evidence IDs. `mode` is `fan|analyst`; `favorite_team_id` is one of the fixture's two team IDs or `null`. Enforce these contracts server-side. Do not accept client-provided stat totals or evidence text.

## 9. AI design, compatibility gate, and cost controls

**The single narrator agent** receives a compact `FactPacket`, a mode, and optional favorite team. System instructions: use only supplied facts; do not claim actual Premier League association, unseen video, estimated probabilities, or unprovided tactical causes; produce only the required JSON shape; cite evidence IDs; distinguish observation from interpretation. Fan tone is accessible without hype unsupported by data; analyst tone names patterns only when measured. Favor low temperature and short responses.

**Compatibility gate, time-boxed to 60 minutes before building the UI:**

1. Configure a disposable/limited-scope proxy key in environment variables and list/select **one** known available model that supports OpenAI-style chat completions; do not assume all proxy backends have identical behavior.
2. Use the **documented** Python Agent Framework OpenAI chat-completion client with an OpenAI-compatible LiteLLM base URL, using a tiny local script to send one structured narration request. Check the installed package version's actual constructor/API rather than copying unverified snippets. Send no secret to a public endpoint.
3. Confirm request success, JSON parseability, latency, and an approximate cost/token estimate **without** disclosing the key. Record supported model and package versions in the README (not private endpoint credentials).
4. If incompatible, try one other available chat-completion model or client configuration within the 60-minute box. If still blocked, **stop and record a decision**: either direct OpenAI-compatible client behind the same `Narrator` interface (loses the planned Microsoft runtime integration and may weaken hackathon fit) or a supported Microsoft provider after confirming cost. Do not silently swap technologies or pretend the fallback satisfies the same criterion.

**Environment contract:** `LLM_BASE_URL` (proxy's OpenAI-compatible API base, typically ending in `/v1`, verified by the spike), `LLM_API_KEY`, `LLM_MODEL`, `MAX_NARRATION_CALLS=12`, `LLM_TIMEOUT_SECONDS=15`. `.env.example` has placeholders only. Missing/incomplete AI configuration starts the app in an explicitly labeled **replay-only mode**; invoking narration returns a configuration error, not fabricated text. Never put the proxy URL/key in browser JS, query parameters, logs, GitHub Actions, demo screenshots, or public sample requests. Server-side connection only; no wildcard browser-origin access.

**Cost/safety controls:** One active request at a time per app instance; at most 12 uncached model requests **per server process**, including failed calls. Automatic key moments are capped at eight; manual regeneration and all browser sessions share the same limit. Check the in-process cache by `(fixture version, event_id, mode, favorite_team_id, model)` **before** checking the limit; reserve a call slot atomically before invoking the model. No automatic retries; a 15-second timeout and bounded output tokens (aim 250, subject to provider support). Display the process-wide call counter and a clear "narration limit reached" state. Configure a per-key spend limit on the proxy/upstream **if supported** and verify it separately; an application request cap is not a guaranteed money cap, especially across server restarts. No model call for every pass.

**Validation:** Strictly parse the model response into the insight contract, require exact requested event/mode, nonempty fields, and evidence IDs that exist in the server fact packet. Reject references to real clubs/players or unsupported numerical claims in the final displayed text; for numbers, either match a value explicitly supplied in the fact packet or reject and show an error. Treat a timeout, non-JSON response, invalid evidence, and budget exhaustion as different explicit errors. Log minimal error type and event ID, never prompt bodies or credentials. Tests use a fake adapter; live proxy smoke check is a separate manual step.

## 10. HTTP contract and playback implementation

| Route | Method | Behavior |
|---|---|---|
| `/` | GET | Server-rendered single-page UI |
| `/health` | GET | Readiness, `ai_available` boolean, and process-wide call count, **no configuration values** |
| `/api/match` | GET | Fixture metadata, public ordered events, key-moment IDs; never sends secret or precomputed AI output |
| `/api/snapshot?through=e042` | GET | Server-computed stats and evidence through event ID; reject unknown ID with 404 |
| `/api/insights` | POST | `{event_id, mode, favorite_team_id}` → validated insight + evidence; standard explicit 4xx/5xx error JSON on failure |

Use a frontend `setTimeout` loop that advances one event about every second; maintain one current event index. UI must not await a narration response to advance. Track pending insight requests by `(event_id, mode, favorite_team_id)` so late replies attach to their own moment and never overwrite the newest card. Cancel or ignore stale mode-specific requests when mode changes. Keep the replay state in the browser for this single-user prototype and compute authoritative snapshots on the server. Do not offer shared multi-user session semantics.

Set an HTTP timeout on model requests; implement status distinctions: 400 invalid mode/team, 404 unknown event, 429 replay call cap, 503 missing/unavailable inference, 502 invalid upstream output, 504 timeout. A user can always replay the deterministic fixture even when narration fails. Consider in-process limits **demo safeguards, not a secure rate limit**; run the app locally and do not expose it on a public host without authentication and stronger controls.

## 11. Testing and acceptance matrix

| Case | Expected result |
|---|---|
| Same seed twice; generator constraints | Identical fixture JSON, valid references/order, required anchor moments present |
| Replay every event prefix | Correct cumulative score, shots, shots on target, pass counts, tackles, and possession |
| Goal, missed shot, saved shot | Correct score/shot/on-target combination with no double count |
| Pass streak interrupted by turnover | Streak resets; no unsupported streak insight |
| Key-moment selector | 5–8 moments, all goals retained, spacing for non-goals, stable IDs |
| Fan vs Analyst on same fact packet | Different prompts/persona and same factual fields/evidence |
| Unknown event/invalid team or mode | Explicit 4xx, no model call |
| Malformed JSON, alien evidence ID, unsupported number | No false insight card; clear upstream error |
| Missing key, proxy timeout, call cap | Clear configuration/timeout/limit states; stats and replay unaffected |
| Repeat request for same moment/mode/team | Cache hit; no additional model call |
| Pause/step/reset and late AI reply | Correct UI state; no stale reply shown as current event |
| Security/privacy | No API key in `/`, `/health`, `/api/match`, error payloads, committed files, or browser network responses |

Add one offline end-to-end TestClient run with a fake narrator; run `uv run pytest` and manually inspect a full browser replay. For a real proxy smoke test, use a low-cost configured model, one fact packet, and one request; confirm compatible behavior without publishing the response if it contains operational details.

## 12. Delivery sequence and time budget

| Stage | Hours | Exit criterion |
|---|---:|---|
| Setup + compatibility gate | 1.5 | Local project runs; proxy/Agent Framework decision explicitly recorded |
| Fixture + reducer | 2.5 | Seeded fixture, every-prefix stats, invariant tests pass |
| Moments + evidence packets | 1.5 | Five or more defensible moments with source event references |
| Narrator + validation/limits | 2.5 | Grounded two-mode JSON, fake-client tests, explicit failures |
| Single-page replay UI | 2.5 | Controls, stats, evidence cards, asynchronous moment narration |
| Polish, tests, README, demo/submission | 2.5 | Complete replay, <2-minute video, public code and testing steps |
| **Total** | **13** | Leaves up to two hours contingency |

**Stop rule:** If a stage runs >1 hour over budget, cut only optional work in section 6. Do not remove grounded evidence, two modes, explicit errors, or submission artifacts to save time. Avoid pursuing cloud deployment when the local test-build route works.

## 13. Submission and operating checklist

- Register before the registration deadline; read portal-specific submission fields and eligibility conditions.
- Publish public GitHub repo with this PRD, install/run/test README, `.env.example`, lockfile, original synthetic data description, and no secrets.
- Ensure an evaluator can install the **test build** and run a replay without personal proxy access: provide a clearly labeled replay-only mode and representative screenshots/video; **for evaluating live AI behavior**, provide controlled temporary access or a safe test configuration if the portal's testing instructions require it. Do not publish proxy credentials in the repo or public submission. If reviewers cannot run live narration, disclose that limitation plainly; it may reduce scoring.
- Record a **<2-minute** screen demo showing score/stats evolving, a key moment, Fan vs Analyst grounded insight, and evidence. Use no copyrighted match footage, logos, or music.
- In the description, state exactly which Microsoft technology actually runs in the project; show the Agent Framework integration if the gate passed. Mention Copilot CLI only as a development tool if used, not as the runtime AI model. State that inference is routed through a private LiteLLM proxy and cloud deployment is not required for local testing.
- Keep the project/test-build instructions and any promised judge access available through the judging period; remove temporary credentials/access afterward. Do not leave a paid endpoint open-ended.

## 14. Change control for another coding agent

**This document is the source of truth for scope, contracts, budget, and acceptance.** Before any implementation, the coding agent must read it and report conflicts, not quietly reinterpret requirements. Implement in the order in section 12. A change to stack, inference path, public hosting, synthetic-data policy, event schema, narrator framework, call cap, or judged access requires an explicit owner decision and a PRD version update **before** coding that change. Minor file layout or CSS choices may vary without changing observable behavior.

Copyable handoff instruction:

> Read `PRD.md` fully. Implement the next unfinished stage only; preserve all non-negotiable constraints, contracts, and acceptance criteria. Use Copilot CLI/Codex CLI for development, not runtime narration. Do not add Azure resources or assume hackathon-provided credits. Use my LiteLLM proxy only through server-side environment configuration; never expose credentials. Run stage-specific tests and report completed criteria, deviations, remaining work, and any decision the owner must make. If Agent Framework and the proxy cannot interoperate within the documented compatibility gate, stop and request the explicit fallback choice.

**Primary references:** [Official rules](https://github.com/microsoft/insidethegamehackathon/blob/main/OFFICIAL%20RULES.md), [Microsoft Agent Framework overview](https://learn.microsoft.com/en-us/agent-framework/overview/), [Agent Framework OpenAI provider](https://learn.microsoft.com/en-us/agent-framework/integrations/by-component/model-providers/openai), [event learning playlists](https://github.com/microsoft/insidethegamehackathon/blob/main/RESOURCE%20PLAYLISTS.md).
