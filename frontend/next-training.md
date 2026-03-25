# Next Training

## Overview
> **Tip:** In Next.js App Router, you can wrap a folder name in parentheses `( )` to create a **route group**.

---

### Folder → URL Mapping in Next.js App Router

```
src/app/
│
├─ layout.tsx          ← Root layout, wraps all pages
│
├─ (public)/           ← Route group, parentheses = folder name not in URL
│   └─ page.tsx        → URL: /
│
├─ (dashboard)/        ← Route group for dashboard
│   └─ page.tsx        → URL: /dashboard
│
├─ api/
│   ├─ health/
│   │   └─ route.ts    → API endpoint: /api/health
│   └─ auth/
│       └─ route.ts    → API endpoint: /api/auth
│
└─ public/             ← No parentheses → folder name appears in URL
    └─ page.tsx        → URL: /public
```

## Notes

* **Parentheses `( )`**: Folder is a route group; **ignored in URL**.
* **No parentheses**: Folder name appears in the URL.
* **Layouts**: Next.js automatically wraps the closest `layout.tsx` around its pages.
* **Root layout** (`src/app/layout.tsx`) is required to render pages.

---

