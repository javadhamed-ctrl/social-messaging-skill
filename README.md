# social-messaging-skill

An [opencode](https://opencode.ai) skill that encodes a hard-won, hands-on-tested playbook for **sending messages and managing contacts on Iranian social messengers on Windows** — Rubika, Bale, Eitaa and Soroush Plus.

It is a set of **strict rules**, not suggestions. Every recipe in it was discovered by actually driving the real applications on a real Windows machine and verifying the result — including the mistakes, which are documented so they are not repeated.

## Why this exists

Messaging a real person is irreversible. Most automation failures are not "the click missed" — they are **"the wrong person got the message"** or **"the same message was sent three times"**. This skill exists to make those failures structurally impossible:

- **Zero-approval rule** — the exact text and the exact recipient are shown to the user before anything irreversible happens.
- **Three-source truth** — the dictated text, the text actually in the file, and the text that came back from the server are identified separately. A verification that compares a bubble against the very file it was written from proves nothing, and this skill refuses to report that as success.
- **Idempotency by design** — browser automation harnesses re-run scripts; every irreversible action is guarded by a marker file and a live state check, so a retry becomes a verification pass instead of a duplicate send.
- **Never guess the recipient** — contacts are searched by name variants, personal accounts are distinguished from channels, and ambiguous matches are presented as a numbered list instead of being resolved by assumption.
- **Verification that does not manufacture confidence** — raw comparison first, differences reported explicitly, never silently normalized away.

## What is inside

| Network | Approach | Key hard-won details |
|---|---|---|
| **Rubika** | unofficial API (`rubpy`), UI fallback | Platform override required or the server returns `INVALID_INPUT`; `getUserInfo` is the real method (`getMe` is not); 100-row pagination; `SendInput` for the UI path |
| **Bale** | UI automation; Bot API for bot chats only | Enter does **not** send — the real button must be clicked, and its position must be read *after* typing; sender direction is only reliable from bubble colour |
| **Eitaa** | native Qt app + UI Automation | Window title changes per chat (primary recipient check); the focused composer's value is readable, so drafts need no OCR |
| **Soroush Plus** | UI Automation | Composer and search are real exposed inputs; region OCR is unreliable |

Plus: the browser-automation (CDP) harness pitfalls, and an implementation checklist to run before every single send.

## Install

Copy the skill folder into your opencode skills directory:

```powershell
Copy-Item -Recurse .\skills\social-messaging "$env:USERPROFILE\.config\opencode\skills\social-messaging"
```

opencode auto-discovers skills — no restart-free reload assumed, so a new skill is picked up on the next session.

## Requirements

- Windows (the automation is built on Win32 `SendInput`, UI Automation and PowerShell).
- For the Rubika API path: Python with `rubpy` installed.
- For the UI paths: the messenger's own desktop/web app installed and signed in.

## Usage

Once installed, the skill activates when you ask to send a message or manage a contact on one of these messengers. The workflow it enforces:

1. Resolve and **confirm the recipient** with you (numbered list if ambiguous).
2. Write the final text to a UTF-8 file and compute its **SHA-256**.
3. Show you the exact text and recipient, and **wait for your approval**.
4. Write a **marker file** immediately before the irreversible action.
5. Send via the correct path (API where a personal API exists, else the proven UI path).
6. **Verify against live state** by raw comparison, never `===`.
7. Report any character-level difference honestly instead of claiming success.

## Author & how this project was built

**Author:** Javad Hamed — experienced software developer (30 years), Windows / PowerShell.

This project was written with **AI-assisted coding on the [opencode](https://opencode.ai) platform** using the `opencode/big-pickle` model.

Every workflow here was genuinely exercised and verified on the author's own Windows machine against the real messenger applications — messages were sent, contacts were deleted and confirmed absent, and the documented pitfalls (Enter not sending in Bale, `getMe` being invalid in Rubika, the CP1256 console corruption, the "verification that proves nothing" trap) were each hit for real and then recorded. Nothing here is theoretical.

## License

MIT — see [LICENSE](LICENSE).---

â­گ **If you found this project useful, please give it a star!** It helps others discover it and motivates continued development.

â­گ **ط§ع¯ط± ط§غŒظ† ظ¾ط±ظˆعکظ‡ ط¨ط±ط§غŒطھط§ظ† ظ…ظپغŒط¯ ط¨ظˆط¯طŒ ظ„ط·ظپط§ظ‹ ط¨ظ‡ ط¢ظ† ط³طھط§ط±ظ‡ (Star) ط¨ط¯ظ‡غŒط¯!** ط§غŒظ† ع©ط§ط± ط¨ظ‡ ط¯غŒع¯ط±ط§ظ† ط¯ط± ع©ط´ظپ ظ¾ط±ظˆعکظ‡ ع©ظ…ع© ظ…غŒâ€Œع©ظ†ط¯ ظˆ ط§ظ†ع¯غŒط²ظ‡ظ” طھظˆط³ط¹ظ‡ظ” ط¨غŒط´طھط± ط±ط§ ظپط±ط§ظ‡ظ… ظ…غŒâ€Œط¢ظˆط±ط¯.

