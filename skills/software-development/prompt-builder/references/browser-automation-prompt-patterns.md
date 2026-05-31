# Browser Automation Prompt Patterns

Reference patterns for building prompts that integrate with browser automation APIs (Camofox, Playwright, Puppeteer).

## Camofox REST API Quick Reference

### Endpoints (VERIFIED)
```
GET  /health                    → {ok, enabled, running, engine, browserConnected}
GET  /tabs?userId=X             → {tabs: [{tabId, url, title}]}
POST /tabs                      → {userId, sessionKey, url} → {tabId}
DELETE /tabs/{tabId}?userId=X
GET  /tabs/{tabId}/snapshot?userId=X → {url, snapshot, refsCount, truncated}
POST /tabs/{tabId}/evaluate     → {userId, expression} → {result}
POST /tabs/{tabId}/click        → {userId, ref}
POST /tabs/{tabId}/type         → {userId, ref, text, mode, submit, pressEnter}
POST /tabs/{tabId}/upload       → {userId, ref, filePath}
POST /tabs/{tabId}/press        → {userId, key}
GET  /tabs/{tabId}/screenshot?userId=X → PNG binary
```

### Auth
Bearer token via `Authorization: Bearer <CAMOFOX_API_KEY>` header.
Required in production mode (`NODE_ENV=production`).

### Snapshot Format
Accessibility tree text with refs:
```
- button "Utas baru" [e3]:
- textbox "Kolom teks kosong..." [e22]:
- searchbox "Cari" [e21]
```

Parse regex: `/- (button|link|textbox|searchbox|checkbox|menuitem|input) "([^"]{0,220})" \[(e\d+)\]/`

### Critical Pitfalls
1. NEVER use `mode="fill"` for text input — causes double-insert on contenteditable
2. NEVER do `Ctrl+A+Backspace` on Layer 2+ in thread-post — crashes Camofox
3. ALWAYS close stale tabs before opening new ones
4. Snapshot refs change after every action — always re-snapshot
5. Camofox session timeout: 86400000ms (1 day), configurable

## Pattern: Bilingual Selectors (Threads)

Threads UI labels are bilingual (Indonesian + English):

| Function | ID | EN | Role |
|----------|----|----|------|
| New thread | `^Utas baru$` | `^New thread$` | button |
| Textbox | `Kolom teks kosong` | `Empty text field` | textbox |
| Submit | `^Kirim$` | `^Post$` | button |
| Reply | `^Balas(\s+\d+)?$` | `^Reply(\s+\d+)?$` | button |
| Attach | `^Lampirkan$` | `^Attach media$` | button |
| Add to thread | `Tambah ke utas` | `Add to thread` | button |
| Topic search | `Community or topic` | `Community or topic` | searchbox |

## Pattern: Image Upload (3 Strategies)

1. Click "Attach media" → find file input → `upload_file()`
2. JS expose hidden input → re-snapshot → upload
3. Base64 + DataTransfer API via `evaluate()` ← MOST RELIABLE

## Pattern: Post Flow

```
Open profile page → Snapshot → Find "New thread" button → Click
→ Snapshot → Find textbox → Click → Clear (Ctrl+A + Backspace)
→ Type text (mode="keyboard") → Snapshot → Find submit → Click
```

## Pattern: Chain Post Flow

```
Post Layer 1 → Find latest URL via own-search → Open URL
→ Click Reply → Type text → Submit → Repeat for each layer
```

## Pattern: Thread Post Flow

```
Open profile → Click "New thread" → Fill Layer 1
→ Click "Add to thread" → Fill Layer 2 (NO clear!)
→ Click "Add to thread" → Fill Layer 3 (NO clear!)
→ Click Post (publishes all layers at once)
```
