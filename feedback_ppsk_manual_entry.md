---
name: PPSK must be typed manually — never paste from email/SMS
description: Copying a PPSK from email or SMS introduces invisible characters that fail iOS/macOS password validation silently. Join button stays grayed out with no error.
type: feedback
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---

Never paste a PPSK from email or SMS into the Wi-Fi password field. Email clients add smart quotes, trailing spaces, or zero-width characters that fail WPA2 password validation. The client gives no error — the Join button simply stays grayed out.

**Why:** Discovered May 12 when PPSK_Demo Join button was grayed out on both iPhone and MacBook. Typing the PPSK character by character resolved it.

**How to apply:** When testing PPSK with a new user:
1. Open the email/SMS side-by-side with the password field
2. Type each character manually with the password visible (tap the eye icon)
3. If Join is still grayed out after 8+ chars → check character count in the email (count them — if more than the expected PPSK length, there are hidden characters)
4. Alternative: create a test user with a known simple password (e.g., `Extreme01`) directly in XIQ to eliminate the variable entirely
