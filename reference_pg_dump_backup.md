---
name: pg_dump Backup & Restore — Railway PostgreSQL
description: Full commands for backing up and restoring the cisco-en-cli-agent Railway PostgreSQL database
type: reference
---

# pg_dump Backup & Restore — cisco-en-cli-agent

## Tool Location
libpq installed via Homebrew on Mac:
```
/opt/homebrew/opt/libpq/bin/pg_dump
/opt/homebrew/opt/libpq/bin/pg_restore
```
Add to PATH permanently: `echo 'export PATH="/opt/homebrew/opt/libpq/bin:$PATH"' >> ~/.zshrc`

## Backup Command
```bash
/opt/homebrew/opt/libpq/bin/pg_dump \
  "postgresql://postgres:<PASSWORD>@<HOST>:<PORT>/railway" \
  --no-owner --no-acl -Fc \
  -f /Users/khukhan/Projects/cisco-en-cli-agent/data/backups/railway_$(date +%Y%m%d_%H%M).dump
```
- `-Fc` = custom binary format (smaller, faster restore than plain SQL)
- `--no-owner` = don't restore ownership (avoids permission errors)
- `--no-acl` = skip GRANT/REVOKE (Railway manages its own permissions)
- Output dir `data/backups/` is gitignored — local only

## Restore Command
```bash
/opt/homebrew/opt/libpq/bin/pg_restore \
  -d "postgresql://postgres:<NEW_PASSWORD>@<NEW_HOST>:<PORT>/railway" \
  --no-owner -Fc \
  /Users/khukhan/Projects/cisco-en-cli-agent/data/backups/railway_20260402_1340.dump
```
Use `--clean` flag to drop existing objects before restoring (full replace):
```bash
... --clean --if-exists -Fc railway_backup.dump
```

## Known Backups
| File | Date | Size | Contents |
|------|------|------|----------|
| `data/backups/railway_20260402_1340.dump` | 2026-04-02 13:40 | 7.7 MB | 515 seed rows + 3,629 bulk-seeded rows (7,550 extracted, 3,921 merged) |

**NOTE:** Password was rotated on 2026-04-02 via Railway → PostgreSQL → Config → Regenerate.
A fresh pg_dump with the NEW URL is still pending (app was redeploying when session ended).
Get new public URL from: Railway → PostgreSQL → Connect tab → Public URL.

## Railway PostgreSQL Public URL Format
```
postgresql://postgres:<PASSWORD>@interchange.proxy.rlwy.net:<PORT>/railway
```
Find current URL: Railway Dashboard → PostgreSQL plugin → Connect tab → "Public URL"

## Why This Matters
- Avoids re-running embedding generation (which downloads all-MiniLM-L6-v2 ~90MB and re-encodes all rows)
- On new Railway projects: pg_restore is ~10s vs ~10 min for full seed
- Always pg_dump BEFORE rotating PostgreSQL password

## Security Reminder
- Rotate Railway PostgreSQL password after every pg_dump if the URL was shared in a conversation
- Backup files contain all data — store locally only, never commit to Git
