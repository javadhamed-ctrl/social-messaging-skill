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

---

## ظپط§ط±ط³غŒ (Persian)

### ط§غŒظ† ع†غŒط³طھطں

غŒع© **ظ…ظ‡ط§ط±طھ opencode** ع©ظ‡ غŒع© playbook ط¢ط²ظ…ظˆط¯ظ‡â€Œط´ط¯ظ‡ظ” ط¹ظ…ظ„غŒ ط¨ط±ط§غŒ **ط§ط±ط³ط§ظ„ ظ¾غŒط§ظ… ظˆ ظ…ط¯غŒط±غŒطھ ظ…ط®ط§ط·ط¨غŒظ† ط¯ط± ظ¾غŒط§ظ…â€Œط±ط³ط§ظ†â€Œظ‡ط§غŒ ط§غŒط±ط§ظ†غŒ ط±ظˆغŒ ظˆغŒظ†ط¯ظˆط²** ط§ط³طھ â€” ط±ظˆط¨غŒع©ط§طŒ ط¨ظ„ظ‡طŒ ط§غŒطھط§ ظˆ ط³ط±ظˆط´ ظ¾ظ„ط§ط³.

ط§غŒظ† غŒع© ظ…ط¬ظ…ظˆط¹ظ‡ظ” **ظ‚ظˆط§ظ†غŒظ† ط³ط®طھع¯غŒط±ط§ظ†ظ‡** ط§ط³طھطŒ ظ†ظ‡ ظ¾غŒط´ظ†ظ‡ط§ط¯. ظ‡ط± ط¯ط³طھظˆط±ط§ظ„ط¹ظ…ظ„ ط¢ظ† ط¨ط§ ط¯ط±ط§غŒظˆ ظˆط§ظ‚ط¹غŒ ط¨ط±ظ†ط§ظ…ظ‡â€Œظ‡ط§غŒ ظˆط§ظ‚ط¹غŒ ط±ظˆغŒ غŒع© ظ…ط§ط´غŒظ† ظˆغŒظ†ط¯ظˆط² ظˆط§ظ‚ط¹غŒ ظˆ طھط£غŒغŒط¯ ظ†طھغŒط¬ظ‡ ع©ط´ظپ ط´ط¯ظ‡ â€” ط§ط² ط¬ظ…ظ„ظ‡ ط§ط´طھط¨ط§ظ‡ط§طھطŒ ع©ظ‡ ظ…ط³طھظ†ط¯ ط´ط¯ظ‡â€Œط§ظ†ط¯ طھط§ طھع©ط±ط§ط± ظ†ط´ظˆظ†ط¯.

### ع†ط±ط§ ط¨ظ‡ ظˆط¬ظˆط¯ ط¢ظ…ط¯طں

ظ¾غŒط§ظ… ط¯ط§ط¯ظ† ط¨ظ‡ غŒع© ظپط±ط¯ ظˆط§ظ‚ط¹غŒ ط؛غŒط±ظ‚ط§ط¨ظ„ ط¨ط§ط²ع¯ط´طھ ط§ط³طھ. ط¨غŒط´طھط± ط´ع©ط³طھâ€Œظ‡ط§غŒ ط§طھظˆظ…ط§ط³غŒظˆظ† آ«ع©ظ„غŒع© ظ†ط®ظˆط±ط¯آ» ظ†غŒط³طھ â€” ط¨ظ„ع©ظ‡ آ«ظ¾غŒط§ظ… ط¨ظ‡ ظپط±ط¯ ط§ط´طھط¨ط§ظ‡ ط±ظپطھآ» غŒط§ آ«ظ‡ظ…ط§ظ† ظ¾غŒط§ظ… ط³ظ‡ ط¨ط§ط± ط§ط±ط³ط§ظ„ ط´ط¯آ». ط§غŒظ† ظ…ظ‡ط§ط±طھ ط¨ط±ط§غŒ ط؛غŒط±ظ…ظ…ع©ظ† ع©ط±ط¯ظ† ط³ط§ط®طھط§ط±غŒ ط§غŒظ† ط´ع©ط³طھâ€Œظ‡ط§ ظˆط¬ظˆط¯ ط¯ط§ط±ط¯:

- **ظ‚ط§ظ†ظˆظ† طµظپط± طھط£غŒغŒط¯** â€” ظ…طھظ† ط¯ظ‚غŒظ‚ ظˆ ع¯غŒط±ظ†ط¯ظ‡ظ” ط¯ظ‚غŒظ‚ ظ‚ط¨ظ„ ط§ط² ظ‡ط± ع©ط§ط± ط؛غŒط±ظ‚ط§ط¨ظ„ ط¨ط§ط²ع¯ط´طھ ط¨ظ‡ ع©ط§ط±ط¨ط± ظ†ط´ط§ظ† ط¯ط§ط¯ظ‡ ظ…غŒâ€Œط´ظˆط¯.
- **ط³ظ‡ ظ…ظ†ط¨ط¹ ط­ظ‚غŒظ‚طھ** â€” ظ…طھظ† ع¯ظپطھظ‡â€Œط´ط¯ظ‡طŒ ظ…طھظ† ظˆط§ظ‚ط¹ط§ظ‹ ط¯ط± ظپط§غŒظ„طŒ ظˆ ظ…طھظ† ط¨ط±ع¯ط´طھغŒ ط§ط² ط³ط±ظˆط± ط¬ط¯ط§ع¯ط§ظ†ظ‡ ط´ظ†ط§ط³ط§غŒغŒ ظ…غŒâ€Œط´ظˆظ†ط¯.
- **Idempotency by design** â€” harness ظ‡ط§غŒ ط§طھظˆظ…ط§ط³غŒظˆظ† ظ…ط±ظˆط±ع¯ط± ط§ط³ع©ط±غŒظ¾طھâ€Œظ‡ط§ ط±ط§ ط¯ظˆط¨ط§ط±ظ‡ ط§ط¬ط±ط§ ظ…غŒâ€Œع©ظ†ظ†ط¯ط› ظ‡ط± ع©ط§ط± ط؛غŒط±ظ‚ط§ط¨ظ„ ط¨ط§ط²ع¯ط´طھ ط¨ط§ غŒع© ظپط§غŒظ„ ظ†ط´ط§ظ†ع¯ط± ظˆ ع†ع© ظˆط¶ط¹غŒطھ ط²ظ†ط¯ظ‡ ظ…ط­ط§ظپط¸طھ ظ…غŒâ€Œط´ظˆط¯.
- **ظ‡ط±ع¯ط² ع¯غŒط±ظ†ط¯ظ‡ ط±ط§ ط­ط¯ط³ ظ†ط²ظ†** â€” ظ…ط®ط§ط·ط¨غŒظ† ط¨ط§ ط§ظ†ظˆط§ط¹ ظ†ط§ظ… ط¬ط³طھط¬ظˆ ظ…غŒâ€Œط´ظˆظ†ط¯طŒ ط­ط³ط§ط¨â€Œظ‡ط§غŒ ط´ط®طµغŒ ط§ط² ع©ط§ظ†ط§ظ„â€Œظ‡ط§ طھظ…غŒط² ط¯ط§ط¯ظ‡ ظ…غŒâ€Œط´ظˆظ†ط¯.
- **طھط£غŒغŒط¯غŒ ع©ظ‡ ط§ط¹طھظ…ط§ط¯ ع©ط§ط°ط¨ طھظˆظ„غŒط¯ ظ†ظ…غŒâ€Œع©ظ†ط¯** â€” ظ…ظ‚ط§غŒط³ظ‡ظ” ط®ط§ظ… ط§ط¨طھط¯ط§طŒ ط§ط®طھظ„ط§ظپط§طھ طµط±غŒط­ط§ظ‹ ع¯ط²ط§ط±ط´ ظ…غŒâ€Œط´ظˆظ†ط¯.

â­گ **ط§ع¯ط± ط§غŒظ† ظ¾ط±ظˆعکظ‡ ط¨ط±ط§غŒطھط§ظ† ظ…ظپغŒط¯ ط¨ظˆط¯طŒ ظ„ط·ظپط§ظ‹ ط³طھط§ط±ظ‡ ط¨ط¯ظ‡غŒط¯!** / **If you found this useful, please star the repo!** â­گ
