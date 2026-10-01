---
name: social-messaging
description: Use when the user asks to send messages on social messengers (Bale, Rubika, Eitaa, Soroush Plus) or when dealing with contacts/search/find/verify recipient before sending. Provides a strict, verified methodology for messaging and contact management on Iranian messengers from Windows — built from hands-on practice and platform-specific quirks.
---

# Skill: Social Messaging — Verified Windows Methodology

This skill encodes a **methodology**, not a script for one person's contacts. Every rule below was discovered by actually driving the real applications on a real Windows machine and verifying the result — including the mistakes, which are documented so they are not repeated. The concrete identifiers (guids, phone numbers, names) in earlier drafts were **training examples**; they are replaced here with placeholders. The methodology is what remains.

> **Meta-rule:** Any concrete identifier (guid, phone, username, display name) that appears in this file is a **placeholder**. Substitute your own target's data. The methodology — the *how* — is invariant.

## 1) Core principles (must never violate)

- **Zero-approval rule:** Do NOT send or delete without explicit user approval shown before the action. Display the exact final text and the exact recipient (name + network + identifier + window title) and wait for an explicit "yes".
- **Three-source truth:** Before sending, separately identify (1) the user's dictated text, (2) the file/script text (what actually gets sent), (3) the in-chat bubble (what the server returned). If (1) != (2) -> abort. If (2) != (3) -> report the mismatch explicitly, never fabricate success.
- **Idempotency:** Sending/deleting are async and browser-automation harnesses may retry the whole script twice. Always guard with a live state check and write a marker file **immediately before** the irreversible action; if the goal is already achieved (message present / contact deleted), treat a retry as verification only.
- **Never guess the recipient:** use search variants (full name, surname alone, first name alone, alternate spellings). If there is no exact match OR multiple exact matches -> present numbered results and ask. Distinguish personal accounts from channels/groups.
- **Search-variant protocol:** saved names differ (aliases) across networks — a confirmed alias is person-specific. If a target is not found under one form, try the others; never conclude "no account exists" after a single query.
- **Tool-state trap (assistant overlay):** after every interactive prompt/question tool call, minimize the assistant window and verify the target app is foreground (full-screen OCR). The assistant window restores on top and clicks/keys go to it instead.
- **Windows/console safety:** never print non-ASCII text to a legacy PowerShell console (CP1256 corruption). Write UTF-8 files and use `python -X utf8`. Build embedded strings from `\uXXXX`/`chr()` or read them from UTF-8 files.
- **Surgical verification:** prefer reading the focused control value (UI Automation) over OCR; OCR only when text is not exposed. Never strip ZWNJ (U+200C) during verification; compare raw first, then report differences (timestamp appended, emoji rendered as `<img>`, yeh/kaf/alef variants) explicitly — do not silently reconcile.

## 2) Rubika — API-first (recommended), UI fallback

### API approach (`rubpy`) — verified
- **Platform override (critical):** the library defaults to a PWA platform that the server rejects. Before any API call:
  ```python
  Client.DEFAULT_PLATFORM.update({'app_version':'4.4.34','platform':'Web',
                                  'package':'web.rubika.ir','lang_code':'fa'})
  ```
  Otherwise the server replies `INVALID_INPUT`.
- **Two-phase login:** `rubika-sendcode.py <phone_e164>` -> writes `phone_code_hash` to a state file; then `rubika-signin.py <code>` -> signs in and saves the session (`rub_session.rp`).
- **Per-process restore:** after loading the session file, do all of:
  ```python
  app.decode_auth = Crypto.decode_auth(app.auth)
  app.import_key = pkcs1_15.new(RSA.import_key(app.private_key.encode()))
  app.key = Crypto.passphrase(app.auth)
  await app.connection.get_dcs()      # BEFORE any builder()/connection call
  ```
- **registerDevice (Web):** the library helper may take the Android path under `platform='Web'` and fail. Call it raw instead:
  ```python
  await app.builder(name='registerDevice', input={
      'token':'', 'lang_code':'fa', 'token_type':'Web',
      'app_version':'WB_4.4.34', 'system_version':'Windows 10',
      'device_model':'Windows',
      'device_hash':'2'+''.join(re.findall(r'\d+', app.user_agent))}, dict=True)
  ```
- **Method names:** `getUserInfo` — NOT `getMe` (that returns `INVALID_INPUT`). Raw responses hide status inside `data_enc`; decrypt with `Crypto.decrypt(data_enc, key=app.key)`.
- **Pagination:** `getContacts` page cap is 100. Page 1 -> `{'start_id': None}`; next page -> `{'start_id': <last user_guid>}`. Never pass the string `'0'`.
- **Contacts / chats:** `delete_contact(user_guid)` works — use the three-stage guard (live check -> marker before action -> delete -> re-pull to verify absence). For the chat list use `getChats` + `next_start_id` pagination and resolve user titles with `getUserInfo`.
- **Search:** `searchGlobalObjects(search_text=...)`. For personal accounts that already have a chat, prefer `getChats` + `getUserInfo` — global search often returns only channels and bots.
- **Send:** use the client method `await app.send_message(object_guid, text=text)`. A raw `builder(name='sendMessage', {...})` call may not match the installed version — if you get `INVALID_INPUT`, read the signature in the installed package instead of guessing.
- **Contacts list is not proof of identity:** a stored label can be wrong. Verify the phone number you are about to act on against the profile, and never reuse a guid recorded under a different person's name.

### UI automation (fallback)
- Chrome app window; synthetic key events do **not** reach Chrome — use low-level `SendInput` helpers for typing and keys.
- The search box is an editable control in the chat-list header; the results dropdown renders below it. Re-derive coordinates every session (the window resizes/moves).
- Prefer reading the composer draft through the accessibility tree over OCR.
- Verify the opened chat by its in-pane header, not by the app window title (which stays fixed).

## 3) Eitaa — native Qt app, UIA preferred
- Native Qt desktop app; the window title is `<ChatName> @ <Username> (N)` and it **changes** when you open a chat — that title change is the primary recipient verification. Never match a literal app name.
- Fields are `QTextEdit` and the **focused composer's value is readable via UIA** (full draft, including newlines) — verify drafts from the value, not from OCR.
- The search field may report an empty value in UIA even when it contains text — verify search state with full-screen OCR.
- Send with a key-injection helper that does **not** restore/resize the window (an `SW_RESTORE` variant un-maximizes it and invalidates every coordinate).

## 4) Bale — UI automation; Bot API only for bot chats
- Chrome app window; synthetic keys do not reach it — `SendInput` required.
- **Enter does not send in this app.** Click the real send control (an element with `aria-label="send-button"`), reading its rect **after** the draft is typed, because the composer grows and moves the button. Identify the control by its aria-label and geometry before clicking, and hit-test the point first.
- **Sender direction is determined only by bubble background colour** (outgoing = mint, incoming = light grey). Never infer direction from message wording, x-offset, or an ancestor's text content (an ancestor may include the chat header even for your own messages).
- **Text extraction caveats:** `innerText` drops emoji (they render as `<img>`); `textContent` includes the timestamp. **Never compare a bubble with `===`** — use `indexOf(FULL) >= 0`. Compare raw first and report ZWNJ / yeh / kaf / alef / timestamp differences explicitly instead of normalizing them away.
- **Delete:** the "delete for everyone" checkbox may be **checked by default**; unchecking it silently downgrades the action to "delete for me only". Verify the checkbox state before clicking, and verify again after. The modal title can share its text with the confirm button — distinguish them by geometry (the confirm button sits on the same row as cancel). Open and confirm within a single run, because a modal left open gets clicked through by the next run.
- **Search:** the magnifier sits at the top of the chat-list column and its coordinates are not stable. If a search returns nothing but the person definitely exists, check "recent searches" in the contacts page.
- **Idempotency is critical:** dispatch exactly one event. A retry must be a verification pass, never a second send.

## 5) Soroush Plus — UIA-exposed inputs
- Chrome app window, fixed title (does not change per chat).
- The composer and the search field are real inputs exposed through UI Automation, so drafts and queries can be read from control values — no OCR needed. Region OCR is unreliable here; use full-screen OCR when you must.
- Send with a key-injection helper. Verify the recipient from the in-pane header (there is no title change).

## 6) Browser automation harness (CDP over real Chrome)
- Some harnesses auto-click the "allow remote debugging" consent dialog and retry the command once — a first failure followed by success is the normal path, not a bug. **This also means every script runs twice**, so non-idempotent actions duplicate unless guarded.
- **`switch_tab(t)`, not `activate_tab(t)`** — reading functions act on the harness's active page, so you will silently read the wrong tab otherwise.
- Read text from the accessibility tree, not from `document.body.innerText` (which returns CSS and misses content).
- Parametrised CDP calls may fail in this harness; get geometry from JS `getBoundingClientRect()` instead. Clear fields with JS input events (`ctrl+a` + `Delete` do nothing). Send a key by dispatching exactly **one** bubbling `KeyboardEvent('keydown', {key:'Enter', bubbles:true})`.
- `elementFromPoint` returning `null` means the point is off-screen — scroll it into view and re-read the rect instead of clicking blindly.
- Never put an irreversible click in a script whose retry you have not made idempotent.

## 7) Implementation checklist (use before every send)
1. [ ] Read the recipient identity (name / network / guid / chat title) and confirm it with the user; present numbered options if ambiguous.
2. [ ] Write the final message to a UTF-8 file; compute SHA-256 over the bytes and over the text. Record ZWNJ count and indices.
3. [ ] Show the exact final text and the exact recipient to the user; get explicit approval.
4. [ ] Write a marker file **immediately before** the action (if it already exists -> verify only, do not act).
5. [ ] Perform the send via the correct path (API where a personal API exists, otherwise the proven UI path).
6. [ ] Post-send verification against live state: raw comparison with `indexOf(FULL) >= 0`; compare the SHA when the full text is readable.
7. [ ] Record the result in your persistent message store with SHA, destination and status.
8. [ ] On any mismatch, do NOT claim success — report the character-level difference.

## 8) File conventions
- Helper scripts live in a temp working directory (per-machine, e.g. `%TEMP%\<agent>\`).
- Session/auth files are secrets: keep them out of version control.
- Markers (`action-*.done`) are ephemeral verification guards.
- Store every dictated message in your persistent memory bank so the user never has to retype it.

## 9) Golden rules (from hard-earned lessons)
- **The recipient is a field, not a guess.** It must be explicitly confirmed and written into the action record.
- **Approval before every irreversible action.** No exceptions.
- **Raw comparison first; silent normalization never.** False confidence is worse than no verification.
- **Idempotent by design.** A retry is a verification pass, not a duplicate action.
- **Record corrections as new lines (append), never overwrite history.** Preserve the error trail — a stored-but-wrong fact is the most expensive kind of memory.