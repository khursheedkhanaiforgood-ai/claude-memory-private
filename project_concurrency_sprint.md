---
name: Concurrency & Streaming Sprint — cisco-en-cli-agent
description: Fix Streamlit single-session blocking — background threading for queries + streaming Claude responses
type: project
---

# Concurrency & Streaming Sprint (cisco-en-cli-agent)

**Status:** Deferred — design captured, not started
**Why:** When AI Query is running (Claude API call), the entire Streamlit session is blocked. User cannot interact with any other tab until the query completes (~2-8s). Other users (separate sessions) are unaffected.

---

## Root Cause

Streamlit runs one script execution at a time per browser session. A synchronous Claude API call in `_run_query()` (`query_box.py`) holds the execution lock until it returns. Streamlit cannot process tab switches or button clicks during this time.

Affected operations (all synchronous today):
- AI Query → `src/agents/orchestrator.py` → Claude API (~2-8s)
- Config Translator → `translate_config()` → multiple Claude API calls (~30-120s)
- E2E Design Wizard → Lead Agent → Claude API (~5-15s)

---

## Option 1 — Streaming Responses (lower risk, quick win)

Use Claude's streaming API so partial tokens appear immediately.

**Files to change:**
- `src/agents/orchestrator.py`: switch to `client.messages.stream()` context manager
- `src/ui/components/query_box.py`: use `st.write_stream()` to render token-by-token

**How:**
```python
# orchestrator.py
with client.messages.stream(model=CLAUDE_MODEL, ...) as stream:
    for text in stream.text_stream:
        yield text   # generator

# query_box.py
response_placeholder = st.empty()
with response_placeholder:
    st.write_stream(agent_query_stream(...))
```

**Result:** Response appears word-by-word (~0.3s to first token). Does NOT unblock other tabs but feels ~4x faster. Low complexity.

---

## Option 2 — Background Threading (proper fix, sprint item)

Launch Claude calls in a background thread. Session is unblocked immediately. User can use other tabs while query runs.

**Pattern:**
```python
import threading
import uuid

# On button click:
job_id = str(uuid.uuid4())
st.session_state["_query_job"] = {"id": job_id, "status": "running", "result": None}

def _run_in_bg(job_id, query, ...):
    result = agent_query(query, ...)
    st.session_state["_query_job"]["result"] = result
    st.session_state["_query_job"]["status"] = "done"

t = threading.Thread(target=_run_in_bg, args=(job_id, query, ...), daemon=True)
t.start()
st.rerun()  # return control immediately

# On subsequent reruns (auto-refresh every 1s while pending):
job = st.session_state.get("_query_job")
if job and job["status"] == "running":
    st.spinner("Query running in background — you can use other tabs...")
    time.sleep(1)
    st.rerun()
elif job and job["status"] == "done":
    _render_result(job["result"])
```

**Auto-refresh:** Use `st_autorefresh` package or `time.sleep(1); st.rerun()` while pending.

**Applies to:** AI Query, Config Translator (shows live section-by-section progress), E2E Wizard.

**Result:** User can switch to Config Translator, DB Browser, etc. while AI Query runs. Query result appears when they return to that tab.

**Risk:** Thread safety — `st.session_state` is not thread-safe by default. Need to use a separate dict/queue outside session_state for the thread to write into, then read it back on rerun.

**Safer pattern (thread-safe):**
```python
# Use a module-level dict keyed by job_id (not session_state)
_JOBS: dict[str, dict] = {}

def _run_in_bg(job_id, ...):
    _JOBS[job_id]["result"] = agent_query(...)
    _JOBS[job_id]["status"] = "done"

# In render:
job_id = st.session_state.get("_active_job_id")
if job_id and _JOBS.get(job_id, {}).get("status") == "done":
    result = _JOBS.pop(job_id)["result"]
    # render result
```

---

## Recommendation

1. **Sprint A (quick win, ~1 day):** Implement streaming for AI Query only
   - `st.write_stream()` for the AI response section
   - Feels dramatically faster, no architecture change

2. **Sprint B (proper fix, ~3 days):** Background threading for all 3 query tabs
   - Thread-safe job store
   - Auto-refresh while pending
   - Live progress for Config Translator (section-by-section as they complete)

**How to apply:** When Khursheed asks about slow queries or concurrent tab usage, refer to this sprint design. Recommend Option 1 (streaming) as first step.
