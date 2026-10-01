---
name: social-messaging
description: Use when the user asks to send messages on social messengers (Bale, Rubika, Eitaa, Soroush Plus) or when dealing with contacts/search/find/verify recipient before sending. Provides the proven, tested workflows learned from hands-on practice (SendInput for Chrome apps, UIA for Qt, OCR safety, idempotency, three-source truth, and "never send without explicit user approval").
---

# Skill: Social Messaging (Verified Windows Workflows)

> **PRIVATE REPO.** This file contains real account guids, phone numbers and contact names for one machine. Never publish it, never paste it into a public issue, never send it to a model as context.

This skill collects everything learned and verified on this machine (end-to-end tests) into a reusable, concrete playbook. Treat it as a set of strict rules, not suggestions.

## 1) Core principles (must never violate)

- **Zero-approval rule:** Do NOT send or delete without explicit user approval shown before the action. Display the exact final text and the exact recipient (name+network+guid/phone/window title) and wait for "yes"/"بله".
- **Three-source truth:** Before sending, separately identify (1) User's dictated text, (2) File/script text (what actually gets sent), (3) In-chat bubble (what server returned). If (1)!= (2) → abort. If (2)!= (3) → report mismatch explicitly, never fabricate success.
- **Idempotency:** Sending/deleting are async and `bu-real.ps1` can retry the whole script twice. Always guard with a live state check + write a marker file **immediately before** the irreversible action; if the goal is already achieved (message present/deleted), treat retry as verification only.
- **Never guess recipient:** Use search variants (name, surname alone, first name alone, alternate spellings). If no exact match OR multiple exact matches → present numbered results and ask. Distinguish personal accounts vs channels/groups.
- **Search-variant protocol:** Saved names differ (alias) across networks — treat confirmed alias as person-specific. If not found under one form, try others; never conclude "no account" after a single query.
- **Tool-state trap (opencode overlay):** After every `question` tool call, minimize opencode and verify the target app is foreground (full-screen OCR). Opencode restores on top and clicks go to it.
- **Windows/console safety:** Never print Persian to PowerShell 5.1 console (cp1256 corruption). Use UTF-8 for files (`io.open(..., encoding="utf-8")`) and when piping Python use `-X utf8`. Build Persian strings in scripts via `\uXXXX`/`chr()` when embedding, or read from UTF-8 files.
- **Surgical verification:** Prefer reading the focused control value (UIA) over OCR when available. OCR only when text not exposed. Never strip ZWNJ during verification; compare raw first, then report differences (timestamp appended, emoji as <img>, yeh/kaf/alef variants) explicitly — do not silently reconcile.

## 2) Rubika (روبیکا) — API-first (recommended), UI fallback

### API approach (rubpy) — verified
- **Platform override (critical):** `Client.DEFAULT_PLATFORM.update({'app_version':'4.4.34','platform':'Web','package':'web.rubika.ir','lang_code':'fa'})` before any API call.
- **Two-phase login:** `rubika-sendcode.py <phone98>` → writes `phone_code_hash` (state). `rubika-signin.py <code>` → signs in and saves `rub_session.rp`.
- **Per-process restore:** After loading `rub_session.rp`, do: `app.decode_auth = Crypto.decode_auth(app.auth)`, `app.import_key = pkcs1_15.new(RSA.import_key(app.private_key.encode()))`, `app.key = Crypto.passphrase(app.auth)`, `await app.connection.get_dcs()` **before** any builder/connection calls.
- **registerDevice (Web):** rubpy's helper may use Android path under platform Web → call raw `builder(name='registerDevice', input={'token':'','lang_code':'fa','token_type':'Web','app_version':'WB_4.4.34','system_version':'Windows 10','device_model':'Windows','device_hash':'2'+digits(UA)})` → OK.
- **Methods:** `getUserInfo` (NOT `getMe` — that gives INVALID_INPUT). Decrypt raw `data_enc` with `Crypto.decrypt(data_enc, key=app.key)` to read status.
- **Pagination:** `getContacts` page cap 100; page 1 `{'start_id': None}`, next `{'start_id': <last user_guid>}`; never `'0'`.
- **Contacts/Chats:** `delete_contact(user_guid)` works with three-stage guard (live check → marker BEFORE action → delete → re-pull to verify absence). `getChats` + `next_start_id` for pagination; fetch `getUserInfo` for user guids to resolve chat titles.
- **Search:** `searchGlobalObjects(search_text=...)` — returns objects (users/channels). For personal accounts already in chats, prefer `getChats`+`getUserInfo` (global search often returns channels/bots).
- **Send:** use `app.send_message(object_guid, text=text)` (method on client). `builder(name='sendMessage', {...})` form can vary by version — prefer client method; if INVALID_INPUT, inspect method signature in installed rubpy.
- **Own accounts (verified):** primary `u0BgYZ80af99f760d2fc652550efe6ba` (جواد حامد / EngJavad / 989358568801). Second `u0BzfbH09ce3c093eb381055c7d0e750` (Eng. Hamed / Iontherapy). Third `u0EwNp30601adbf35b4931b78e1e67bb` (Javad Hamed / EngJHamed). **Never send to "مهدی" guid by mistake** — the old guid belonged to `مهناز حبیبی` (corrected 2026-09-29).
- **Contacts list is not proof of identity:** a stored label can be wrong. Verify the phone number you are about to act on against the profile before deleting or messaging.

### UI automation (fallback)
- Chrome app `Rubika - پیام رسان وب روبیکا`. Keys don't reach Chrome → use SendInput helpers (`type-unicode.ps1`, `activate-send-enter.ps1`/`send-key.ps1`).
- Search at top-right (EditControl «جستجو»), dropdown appears under it; OCR on Rubika can be flaky → prefer full-screen OCR when needed. Compose field exposed; send via Enter injection with geometry verification.
- **Recipient verification #1:** window title becomes `<ChatName> @ <Username>` after opening chat (Qt/Eitaa case differs; Rubika window title is fixed app title — verify via header). Use in-pane header + three-source truth.

## 3) Eitaa (ایتا) — native Qt, UIA preferred
- Native Qt desktop app; window title `<ChatName> @ <Username> (N)` changes when opening a chat. Do not match literal «ایتا».
- Qt fields are `QTextEdit` and **focused compose's VALUE is readable via UIA** (full draft incl. `\n`) — use `uia_get_focused` for draft verification, not OCR.
- Search `QTextEdit` value may appear empty in UIA even with text → verify search state via full-screen OCR for results.
- Send: use `send-key.ps1 -KeyName Enter` (no SW_RESTORE) to avoid un-maximizing. Never use window-restore variant here.
- Layout known (maximized). After `question` call, re-minimize opencode + full-screen OCR before continuing (overlay trap).

## 4) Bale (بله) — UI automation only for arbitrary users; Bot API for bot-chats
- Chrome app window (`Bale Web` variants). Keys don't reach Chrome → SendInput required.
- **Send = real button click, NOT Enter.** Enter does not send in Bale. Click `[aria-label="send-button"]` at rect read **after** typing the draft (composer grows/moves button). Identify button by aria-label + border-radius/cursor; verify with elementFromPoint.
- **Sender direction:** ONLY reliable via bubble background colour: outgoing `rgb(220,250,245)` (mint), incoming `rgb(244,245,247)` (grey). Do NOT infer from ancestor `innerText` (includes chat header for own messages).
- **Text extraction caveats:** `innerText` drops emoji (<img>), `textContent` includes timestamp. **Never compare with `===` against full bubble; use `indexOf(FULL) >= 0`**. Never strip ZWNJ silently — compare raw first, report differences explicitly. Treat `U+200C`/yeh/kaf/alef/timestamp as explicit warnings, never absorbed.
- **Delete:** checkbox is checked by default; unchecking deletes for me only (irreversible loss of "delete for both"). Modal title «حذف پیام» matches confirm button text — distinguish by Y (confirm vs cancel row). Open+confirm in same run; verify count before/after.
- **Search:** magnifier top of chat column; coords unstable → derive per session. If empty results, check «جستجوهای اخیر» (bottom nav «مخاطبین»).
- **Idempotency critical:** dispatch exactly one `keydown`/click event. Never triple-dispatch. A retry must be verification-only (marker before action).

## 5) Soroush Plus (سروش پلاس) — UIA available for inputs
- Chrome app `سروش پلاس`. Composer and search are actual inputs exposed via UIA (`automation_id` `editable-message-text`, search `search-input`) — verify drafts via VALUE (no OCR needed). Region OCR unreliable → use full-screen OCR. Send via `send-key.ps1 -KeyName Enter`. Recipient verified via pane header (no title change).

## 6) Browser Use (bu-real.ps1) — for complex web UI
- Use `bu-real.ps1` for real Chrome (auto-clicks consent dialog; first-attempt fail+retry is normal). **`switch_tab(t)` NOT `activate_tab(t)`** (page_info acts on active page). Read via AX tree (`Accessibility.getFullAXTree`) not `document.body.innerText` (drops Persian structure). **Limitations:** param'd `cdp()` (DOM.getBoxModel) fails in this harness → get geometry via JS `getBoundingClientRect()`. Typing → click+`type_text`, clearing → JS events (not `ctrl+a+Delete`). Sending keys → single bubbling `KeyboardEvent('keydown',{key:'Enter',bubbles:true})` via JS (never multi-dispatch). `elementFromPoint` null = off-screen → scrollIntoView first. **Retry safety:** script runs twice → non-idempotent actions duplicate. Marker file before irreversible action mandatory.

## 7) Implementation checklist (use before every send)
1. [ ] Read recipient identity (name/network/guid/chat title) and verify with user (numbered if ambiguous).
2. [ ] Write final message to UTF-8 temp file; compute SHA-256 (bytes+text). Record ZWNJ count/indices.
3. [ ] Show exact final text + recipient to user; get explicit approval («بله»).
4. [ ] Write marker `action-*.done` **immediately before** action (if not exists → act, else verify-only).
5. [ ] Perform send via correct path (API preferred for Rubika/BotAPI where applicable, else proven UI path).
6. [ ] Post-send verification: read live bubble/value (raw). Check `indexOf(FULL)>=0` (never `===`). Compare SHA if reading full text. Report differences explicitly.
7. [ ] Record result in Memory Bank (`MESSAGES.json`) with SHA, destination, status (`sent`/`failed`), note.
8. [ ] If mismatch → do NOT claim success; report character-level diff (timestamp/emoji/ZWNJ/variants).

## 8) File conventions
- Temp helpers/scripts: `%TEMP%\opencode\` (`rubika-*.py`, `bu-real.ps1`, `sendkeys.ps1`, etc.)
- Session/auth: `rub_session.rp` (keep; never commit). Markers `action-*.done` (ephemeral verification).
- Memory Bank: `~\.config\opencode\memory\` (`INDEX.json`, `MESSAGES.json`, `PROJECTS/<slug>/MEMORY.md`)

## 9) Golden rules (from hard-earned lessons)
- **Recipient is a field, not a guess.** Must be explicitly confirmed and written to the action doc.
- **Approval before irreversible.** No exception.
- **Raw comparison first, silent normalization never.** False confidence is worse than no verification.
- **Idempotent by design.** Retries are verification passes, not duplicates.
- **Document corrections as new lines (append), never overwrite history.** Preserve the error trail.