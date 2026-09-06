# Prototype TODO — Travel Agent Assistant

> Tactical working list for prototyping. The durable design lives in [`plan.md`](./plan.md);
> promote things here into `plan.md` once they prove out.
>
> `[ ]` open · `[~]` in progress · `[x]` done · **(you)** = needs Sean, not Claude

---

## Open decisions (blocking, quick)

- [ ] **Build order** — vertical slice first (intake → 2 specialists → synthesize → review,
  real LLM, fixture tools; add `coherence_check` + locks after) vs. whole §7 at once.
  _Leaning: vertical slice._
- [ ] **Framing** for `plan.md` §1/§4 — pattern demo, traveller-facing tool, or both.
- [ ] **Model split** — `claude-sonnet-5` everywhere, or Sonnet for intake/synthesize/
  coherence + `claude-haiku-4-5` for the two specialists.

---

## Slice 0 — toolchain **(you)**

- [ ] Install Python 3.11+ (`brew install python@3.12`, or `uv python install 3.12`)
- [ ] Install `uv` (`curl -LsSf https://astral.sh/uv/install.sh | sh`)
- [ ] Create `tavel-agent-assistant-python/.env` with `ANTHROPIC_API_KEY=...`
- [ ] `git init` in `tavel-agent-assistant-python/`

## Slice 1 — first end-to-end plan (no coherence, no locks)

- [ ] `pyproject.toml` + `uv` deps: `langgraph`, `langchain`, `langchain-anthropic`,
  `python-dotenv`, `pydantic`; dev: `pytest`
- [ ] `src/travel_agent/state.py` — `State` TypedDict per `plan.md` §7.1 (start with the
  fields Slice 1 needs: `messages`, `budget`, `trip_length_days`, `brief`,
  `specialist_results`, `draft_plan`, `feedback`, `approved`, `revision_count`)
- [ ] `src/travel_agent/tools/fixtures.py` + `tests/fixtures/{travel,accommodation}.json`
- [ ] `travel_search` / `accommodation_search` tools reading the fixtures
- [ ] `src/travel_agent/nodes/intake.py` — `ChatAnthropic` structured-output call →
  `budget`, `trip_length_days`, `brief`
- [ ] `src/travel_agent/nodes/specialists.py` — `travel_agent` / `accommodation_agent`
  via `create_agent(model, tools=[...])`, append to `specialist_results`
- [ ] `src/travel_agent/nodes/synthesize.py` — LLM → `draft_plan`
- [ ] `src/travel_agent/nodes/review.py` — `interrupt(...)`, `route_feedback`
  (approve / feedback only for this slice)
- [ ] `src/travel_agent/graph.py` — wire `START → intake → (travel ∥ accommodation) →
  synthesize → review`; `InMemorySaver`
- [ ] `src/travel_agent/prompts/` — one file per LLM node
- [ ] `scripts/run_once.py` — start a session, print plan, feed one scripted feedback,
  print revised plan
- [ ] **Milestone:** `run_once.py` with "£800, 5 days in Barcelona, art and food"
  produces a plan, then a revised plan after "somewhere cheaper to stay"

## Slice 2 — coherence check + risks

- [ ] Add `risks` to state; `nodes/coherence_check.py`
- [ ] Re-wire: `specialists → coherence_check → synthesize`
- [ ] `route_feedback` handles `adjust` / `keep` on a risk
- [ ] Milestone: an arrival-change feedback raises a risk instead of cascading

## Slice 3 — section locks

- [ ] Add `section_locks` to state; specialists gated on loose; auto-unlock on direct edit
- [ ] `route_feedback` handles lock / unlock payloads
- [ ] Milestone: locked section stays put; its risk offers unlock-and-adjust

## Slice 4 — graph service + wire the console

- [ ] Stand up `langgraph dev` (or thin FastAPI) per `plan.md` §8
- [ ] Replace `travel-agent-console/src/mock/useTravelAgent.ts` internals with real calls
- [ ] Milestone: the React console drives a real graph run

---

## Promote-to-plan.md queue

_Things learned while building that should land in `plan.md` once confirmed._

- (none yet)
