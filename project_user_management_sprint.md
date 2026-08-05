---
name: User Management Sprint — cisco-en-cli-agent
description: Future sprint design for user self-service password change, first-time login flow, logout UX fix, and admin user management UI
type: project
---

# User Management Sprint (cisco-en-cli-agent)

**Status:** Deferred — design captured, not started
**Why:** Users need to receive a temp password from admin and change it on first login. Currently no self-service and no admin UI for user management.

---

## Current State (as of 2026-04-03)

### Auth architecture
- Users stored in Railway env var `APP_USERS` as JSON — **not in DB**
- Format: `{"username": {"password": "plaintext", "role": "admin|superadmin|user"}}`
- No bcrypt (security risk T-002 — plaintext comparison at `auth.py:87`)
- No session expiry (security risk T-008)
- No brute-force lockout (security risk T-003)

### Logout button
- **EXISTS** but hidden — `Sign out` button is at the very bottom of the sidebar (`render_user_badge()` in `auth.py:110`)
- Users miss it because it's below RAG Tuning + Translation Quality panels which push it off-screen
- **Fix needed:** Move or pin logout to top of sidebar, or make it a floating button

### Concurrent sessions
- Streamlit gives each browser tab its own independent session (server-side)
- No session limit in the code
- Railway hobby plan: ~20-30 concurrent users before memory pressure

### Adding users today (manual process)
1. Go to Railway → cisco-en-cli-agent service → Variables
2. Edit `APP_USERS` JSON to add new user entry
3. Trigger redeploy (Railway may auto-redeploy on env var save)
4. User can log in immediately with the password you set

---

## Sprint Design: User Management

### Phase 1 — Quick wins (no DB migration needed)

**Fix 1: Logout button visibility**
- Move `render_user_badge()` call to TOP of sidebar (before stats, before RAG panel)
- Or: add a floating top-right "👤 username · Sign out" badge using `st.markdown` with fixed positioning

**Fix 2: First-time password change via session state**
- Add `must_change_password` field to `APP_USERS` JSON: `{"password":"TempPass1!", "role":"user", "must_change_password": true}`
- After successful login, check this flag
- If true: show a "Set your password" page (block access to app until done)
- New password stored back in... env var? → Not possible without Railway API. Need DB for this.

### Phase 2 — DB-backed user management (proper sprint)

**New DB table: `app_users`**
```sql
CREATE TABLE app_users (
    id              SERIAL PRIMARY KEY,
    username        VARCHAR(50) UNIQUE NOT NULL,
    password_hash   VARCHAR(128) NOT NULL,   -- bcrypt
    role            VARCHAR(20) DEFAULT 'user',
    must_change_pw  BOOLEAN DEFAULT TRUE,
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    last_login      TIMESTAMPTZ,
    created_by      VARCHAR(50)
);
```

**First-time login flow**
1. Admin creates user via admin UI (or Railway env bootstrap on first run)
2. User logs in with temp password
3. `must_change_pw = true` → redirect to "Set your password" page before app access
4. User sets new password → bcrypt hash stored → `must_change_pw = false`
5. User accesses app normally

**Self-service password change**
- Settings page (accessible from sidebar user badge)
- Fields: Current password | New password | Confirm new password
- Validates current password before allowing change

**Admin user management UI (superadmin only)**
- New section in DB Browser tab or separate "⚙️ Admin" tab
- Table: all users, roles, last login, must_change_pw status
- Actions: Add user (set temp password + role), Deactivate user, Reset password, Change role
- Superadmin cannot delete themselves

**Role system (keep existing)**
- `user` — read-only access, can query and translate
- `admin` — can add data, view query logs
- `superadmin` — full access including user management

**Migration plan**
1. On first startup with new code: read `APP_USERS` env var → seed into `app_users` table → mark all as `must_change_pw = false` (existing users keep passwords)
2. New users added via admin UI → bcrypt hashed → stored in DB
3. `APP_USERS` env var becomes bootstrap-only (used only if DB table is empty)

### Security improvements (align with T-002, T-003, T-008)
- T-002: bcrypt with cost factor 12 (`pip install bcrypt`)
- T-003: brute-force lockout: 5 failed attempts → account locked 15 min (track in DB)
- T-008: session expiry: 30-min idle + 8-hour absolute (store login timestamp in session state)

---

## How to Give Access to a New User Right Now (until sprint is built)

1. Railway Dashboard → cisco-en-cli-agent → Variables
2. Find `APP_USERS`
3. Edit the JSON to add: `"newname": {"password": "TempPassword1!", "role": "user"}`
4. Save → Railway auto-redeploys in ~2 min
5. Share username + temp password with the user
6. User logs in — they cannot change their own password yet (future sprint)

**How to apply:** When Khursheed asks about adding users or user management, refer to this sprint design and the current manual process above.
