# SaaS Starter Kit Prompt Template

Known-good template for generating a comprehensive SaaS starter kit prompt.
Covers: Go/Echo backend, React/TanStack frontend, dual DB, plugin architecture.

---

## Customization Variables

Replace these before using:

- `<PROJECT_NAME>` — Name of the project
- `<BACKEND_FRAMEWORK>` — e.g., Echo v4, Gin, Fiber
- `<FRONTEND_ROUTER>` — e.g., TanStack Router, React Router
- `<UI_LIBRARY>` — e.g., shadcn/ui, Headless UI
- `<AUTH_MECHANISM>` — e.g., PASETO v4, JWT, session-based
- `<DATABASE_PRIMARY>` — e.g., PostgreSQL, MySQL
- `<DATABASE_SECONDARY>` — e.g., SQLite3 WAL, none
- `<PLAN_TIERS>` — e.g., Free/Pro/Enterprise, Free/Team/Enterprise

---

## Key Architectural Patterns

### Plugin Modular System
Every feature is an independent plugin:
```
internal/plugins/
├── auth/        # PASETO auth
├── roles/       # RBAC
├── users/       # User management
├── plans/       # Subscription plans
└── docs/        # CMS/Docs

ui/src/plugins/
├── auth/
├── admin/
├── dashboard/
└── docs/
```

Plugin removal = delete dir + remove registry entry. No other files change.

### Dual Database Support
Auto-detect from DATABASE_URL scheme:
- `postgres://` → pgx driver
- `sqlite://` → modernc.org/sqlite (pure Go)

### go:embed SPA Serving
Single binary serves embedded frontend:
```go
//go:embed ui/dist/*
var distFS embed.FS
```

### File-Based Routing (TanStack Router)
```
ui/src/routes/
├── __root.tsx
├── _guest.tsx          # Public layout
│   ├── index.tsx       # Landing
│   ├── pricing.tsx     # Pricing
│   ├── login.tsx
│   ├── register.tsx
│   └── docs/
├── _admin.tsx          # Admin layout (RequireRole)
│   ├── index.tsx       # Overview
│   ├── users.tsx
│   ├── plans.tsx
│   └── content.tsx
└── _dashboard.tsx      # User layout (RequireAuth)
    ├── index.tsx
    ├── profile.tsx
    └── subscription.tsx
```

---

## System Constraints Checklist

When generating a SaaS starter kit prompt, always include:

- [ ] Rule 1-9: Standard anti-hallucination + verification rules
- [ ] Rule 10: Stack-specific verification commands (go test + npm run build)
- [ ] Rule 11: Cross-stack contract consistency
- [ ] Rule 12: Plugin independence
- [ ] Rule 13: Database driver auto-detection
- [ ] Rule 14: go:embed SPA serving
- [ ] Completion gate with ALL verification commands listed

---

## Completion Gate Template

```xml
<completion_gate>
  Task status may ONLY transition to COMPLETE when ALL conditions are true:
    ✓ go build ./... → exit 0
    ✓ go vet ./... → exit 0
    ✓ go test ./... → exit 0, all tests pass
    ✓ npx tsc --noEmit → exit 0
    ✓ npm run build → exit 0
    ✓ Cross-stack API contracts verified
    ✓ Plugin independence verified
    ✓ go:embed works: single binary serves SPA
    ✓ DATABASE_URL=sqlite://... starts correctly
    ✓ DATABASE_URL=postgres://... starts correctly
    ✓ Full verbatim output displayed
  Failure of ANY condition = INCOMPLETE.
</completion_gate>
```

---

## Verification Commands by Stack

| Stack | Commands |
|-------|----------|
| Go | `go build ./...`, `go vet ./...`, `go test ./...` |
| React/TypeScript | `npx tsc --noEmit`, `npm run build` |
| Go + React combined | All of above, run both stacks |

---

## Notes

- Plugin modular architecture is KEY for starter kits
- Always include dual DB support for flexibility
- PASETO preferred over JWT for security
- shadcn/ui preferred for customizability
- File-based routing for cleaner code organization
- Single binary deployment via go:embed

---

*Template generated from session 2026-05-30. Verified working prompt structure.*
