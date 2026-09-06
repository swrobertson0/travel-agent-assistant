# Prototype TODO — Travel Agent Assistant

> Tactical working list for prototyping. The durable design lives in [`plan.md`](./plan.md);
> promote things here into `plan.md` once they prove out.
>
> `[ ]` open · `[~]` in progress · `[x]` done · **(you)** = needs Sean, not Claude

Python package: `travel_agent_assistant` (`src/travel_agent_assistant/`).

---

## Open decisions (blocking, quick)

- [ ] **Build order** — vertical slice first (intake → 2 specialists → synthesize → review,
  real LLM, fixture tools; add `coherence_check` + locks after) vs. whole §7 at once.
  _Leaning: vertical slice._
- [ ] **Framing** for `plan.md` §1/§4 — pattern demo, traveller-facing tool, or both.
- [ ] **Provider to build on** — Gemini (`google_genai:gemini-2.5-flash`, free key, no
  card) / Groq / local Ollama. Design is provider-agnostic (`TAA_MODEL`), so this is only
  "what do we run against now". _Leaning: Gemini._
- [ ] **Per-role models** — one `TAA_MODEL` everywhere for the slice; revisit
  `TAA_MODEL_SPECIALIST` etc. only if cost/latency/quality warrants.

---

## Slice 0 — toolchain **(you)**

- [x] Python 3.12 (`.python-version`) + `uv` + `uv init` scaffold
- [x] `git init`, remote → `github.com/swrobertson0/travel-agent-assistant`
- [x] `.gitignore` + `.env.example`
- [ ] Get a provider key (Gemini: https://aistudio.google.com/apikey → `GOOGLE_API_KEY`)
- [ ] `.env` ← `TAA_MODEL=google_genai:gemini-2.5-flash` + `GOOGLE_API_KEY=...`
- [ ] `git push -u --force origin main` (overwrites the old Gradle "Initial commit")

## Slice 1 — first end-to-end plan (no coherence, no locks)

- [ ] `pyproject.toml` deps: `langchain>=1.0,<2.0`, `langchain-core>=1.0,<2.0`,
  `langgraph>=1.0,<2.0`, `langsmith>=0.3.0`, `python-dotenv`, `pydantic`;
  `[project.optional-dependencies]`: `anthropic`/`openai`/`google`/`groq`/`ollama`
  → the matching `langchain-*` package; dev: `pytest`. Then `uv sync --extra google`.
- [ ] `src/travel_agent_assistant/models.py` — `get_model(role)` → `init_chat_model`
  reading `TAA_MODEL` + `TAA_MODEL_<ROLE>`; per-role temperature. Nodes import this only.
- [ ] `src/travel_agent_assistant/state.py` — `State` TypedDict per `plan.md` §7.1
  (Slice 1 fields: `messages`, `budget`, `trip_length_days`, `brief`,
  `specialist_results`, `draft_plan`, `feedback`, `approved`, `revision_count`)
- [ ] `src/travel_agent_assistant/tools/fixtures.py` +
  `tests/fixtures/{travel,accommodation}.json`
- [ ] `travel_search` / `accommodation_search` tools reading the fixtures
- [ ] `src/travel_agent_assistant/nodes/intake.py` — `get_model("intake")`
  structured-output call → `budget`, `trip_length_days`, `brief`
- [ ] `src/travel_agent_assistant/nodes/specialists.py` — `travel_agent` /
  `accommodation_agent` via `create_agent(get_model("specialist"), tools=[...])`,
  append to `specialist_results`
- [ ] `src/travel_agent_assistant/nodes/synthesize.py` — `get_model("synthesize")` →
  `draft_plan`
- [ ] `src/travel_agent_assistant/nodes/review.py` — `interrupt(...)`, `route_feedback`
  (approve / feedback only for this slice)
- [ ] `src/travel_agent_assistant/graph.py` — wire `START → intake →
  (travel ∥ accommodation) → synthesize → review`; `InMemorySaver`
- [ ] `src/travel_agent_assistant/prompts/` — one module per LLM node
- [ ] `scripts/run_once.py` — start a session, print plan, feed one scripted feedback,
  print revised plan
- [ ] **Milestone:** `uv run scripts/run_once.py` with "£800, 5 days in Barcelona, art
  and food" produces a plan, then a revised plan after "somewhere cheaper to stay"
  (on whatever `TAA_MODEL` is set)

## Slice 2 — coherence check + risks

- [ ] Add `risks` to state; `nodes/coherence_check.py` (`get_model("coherence")`)
- [ ] Re-wire: `specialists → coherence_check → synthesize`
- [ ] `route_feedback` handles `adjust` / `keep` on a risk
- [ ] Milestone: an arrival-change feedback raises a risk instead of cascading

## Slice 3 — section locks

- [ ] Add `section_locks` to state; specialists gated on loose; auto-unlock on direct edit
- [ ] `route_feedback` handles lock / unlock payloads
- [ ] Milestone: locked section stays put; its risk offers unlock-and-adjust

## Slice 4 — graph service + wire the console

- [ ] Stand up `langgraph dev` (or thin FastAPI) per `plan.md` §8
- [ ] Replace `travel-agent-ui/src/mock/useTravelAgent.ts` internals with real calls
- [ ] Milestone: the React console drives a real graph run

---

## Promote-to-plan.md queue

_Things learned while building that should land in `plan.md` once confirmed._

- (none yet)
