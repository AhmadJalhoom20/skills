# nextjs-react-clean-code

A production-grade agent skill to refactor messy, AI-generated, or spaghetti React and Next.js code into clean, predictable, human-editable code written to a senior engineer (5+ years experience) standard.

<p align="left">
  <a href="#english">English</a> • <a href="README.ar.md">العربية (Arabic)</a>
</p>

[![License: MIT](https://img.shields.io/badge/License-MIT-black.svg?style=flat-square)](LICENSE)
[![Skills.sh](https://img.shields.io/badge/skills.sh-certified-black?style=flat-square)](https://skills.sh)
[![Compatible with Claude Code](https://img.shields.io/badge/Claude%20Code-Compatible-black?style=flat-square)](https://anthropic.com)
[![Compatible with Antigravity](https://img.shields.io/badge/Antigravity-Native-black?style=flat-square)](https://github.com)

```bash
# Install across all agents in one command
npx skills add AhmadJalhoom20/nextjs-react-clean-code -g -y
```

---

<a name="english"></a>
## The Problem: AI-Generated React Spaghetti

Language models and rapid prototyping tools frequently generate React and Next.js code that works initially, but becomes an unmaintainable nightmare to edit manually:

- **State Explosion:** 8+ independent `useState` hooks tracking related form inputs and modal states.
- **`useEffect` Chain Reactions:** Multiple cascading effects synchronizing state, causing race conditions and unnecessary re-renders.
- **Nested Ternary Pyramids:** Deeply nested `conditionA ? (conditionB ? <X /> : <Y />) : <Z />` expressions cluttering the JSX return tree.
- **Magic Values Buried in JSX:** Hardcoded URLs, status colors, pagination limits, and copy strings hidden hundreds of lines deep.
- **Boundary Confusion:** Slapping `'use client'` at the root of entire Next.js pages instead of isolating interactive leaves.

`nextjs-react-clean-code` enforces the **5-Year Senior Engineer Standard**: writing flat, obvious, and human-editable code governed by strict structural patterns.

---

## Code Comparison

### Before: Typical AI Spaghetti (Hard to Edit Manually)

```tsx
// ❌ AI-generated spaghetti: state explosion, useEffect chains, nested ternaries, inline handlers
"use client";

import { useState, useEffect } from "react";

export default function UserList() {
  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(true);
  const [search, setSearch] = useState("");
  const [filtered, setFiltered] = useState([]);
  const [selectedRole, setSelectedRole] = useState("all");
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch("/api/v1/users?limit=50")
      .then((res) => res.json())
      .then((d) => {
        setData(d.users);
        setLoading(false);
      })
      .catch((err) => {
        setError(err.message);
        setLoading(false);
      });
  }, []);

  // Anti-pattern: Syncing derived state via useEffect
  useEffect(() => {
    let result = data;
    if (search) {
      result = result.filter((u) => u.name.toLowerCase().includes(search.toLowerCase()));
    }
    if (selectedRole !== "all") {
      result = result.filter((u) => u.role === selectedRole);
    }
    setFiltered(result);
  }, [data, search, selectedRole]);

  return (
    <div className="p-8">
      {loading ? (
        <p>Loading...</p>
      ) : error ? (
        <p className="text-red-500">{error}</p>
      ) : filtered.length === 0 ? (
        <p>No users found</p>
      ) : (
        <div>
          <input
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            className="border p-2 mb-4"
            placeholder="Search users..."
          />
          {filtered.map((user) => (
            <div key={user.id} className="border-b py-2 flex justify-between">
              <span>{user.name}</span>
              <span className={user.role === "admin" ? "text-purple-600" : "text-gray-500"}>
                {user.role}
              </span>
            </div>
          ))}
        </div>
      )}
    </div>
  );
}
```

### After: Refactored with `nextjs-react-clean-code`

```tsx
// ✅ Senior Standard: Top config block, derived state, early return guards, zero useEffect chains
"use client";

import React, { useState, useMemo } from "react";

// ============================================================================
// 1. CONFIGURATION & CONSTANTS (EASY EDIT ZONE)
// ============================================================================
const CONFIG = {
  apiEndpoint: "/api/v1/users?limit=50",
  roles: ["all", "admin", "editor", "viewer"] as const,
  roleBadgeStyles: {
    admin: "bg-purple-50 text-purple-700 border-purple-200",
    editor: "bg-blue-50 text-blue-700 border-blue-200",
    viewer: "bg-slate-50 text-slate-700 border-slate-200",
  },
  copy: {
    searchPlaceholder: "Filter users by name...",
    emptyState: "No matching users found.",
    errorTitle: "Failed to load directory",
    retryButton: "Try again",
  },
} as const;

// ============================================================================
// 2. TYPES & INTERFACES
// ============================================================================
interface User {
  id: string;
  name: string;
  role: keyof typeof CONFIG.roleBadgeStyles;
}

// ============================================================================
// 3. SUB-COMPONENTS
// ============================================================================
function UserRow({ user }: { user: User }) {
  const badgeClass = CONFIG.roleBadgeStyles[user.role] ?? CONFIG.roleBadgeStyles.viewer;

  return (
    <div className="flex items-center justify-between border-b border-slate-100 py-3 text-sm">
      <span className="font-medium text-slate-900">{user.name}</span>
      <span className={`inline-flex items-center rounded-sm border px-2 py-0.5 text-xs font-medium ${badgeClass}`}>
        {user.role}
      </span>
    </div>
  );
}

// ============================================================================
// 4. MAIN COMPONENT
// ============================================================================
export default function UserList({ initialUsers = [] }: { initialUsers?: User[] }) {
  const [search, setSearch] = useState("");
  const [selectedRole, setSelectedRole] = useState<string>("all");

  // Derived state computed directly during render (No useEffect syncing)
  const filteredUsers = useMemo(() => {
    return initialUsers.filter((user) => {
      const matchesSearch = user.name.toLowerCase().includes(search.toLowerCase());
      const matchesRole = selectedRole === "all" || user.role === selectedRole;
      return matchesSearch && matchesRole;
    });
  }, [initialUsers, search, selectedRole]);

  return (
    <section className="mx-auto max-w-3xl p-6">
      <div className="mb-6 flex flex-col gap-4 sm:flex-row sm:items-center sm:justify-between">
        <input
          type="search"
          value={search}
          onChange={(e) => setSearch(e.target.value)}
          placeholder={CONFIG.copy.searchPlaceholder}
          className="h-10 w-full rounded-md border border-slate-200 px-3 text-sm focus-visible:outline-2 sm:max-w-xs"
        />
      </div>

      {filteredUsers.length === 0 ? (
        <div className="rounded-md border border-dashed border-slate-200 p-8 text-center text-sm text-slate-500">
          {CONFIG.copy.emptyState}
        </div>
      ) : (
        <div className="divide-y divide-slate-100">
          {filteredUsers.map((user) => (
            <UserRow key={user.id} user={user} />
          ))}
        </div>
      )}
    </section>
  );
}
```

---

## The Standard 6-Zone File Layout

Every refactored component follows a predictable top-to-bottom layout:

```
┌────────────────────────────────────────────────────────┐
│ 1. IMPORTS (grouped by core, third-party, local)       │
├────────────────────────────────────────────────────────┤
│ 2. CONFIGURATION & CONSTANTS (Easy edit zone)          │
│    - API URLs, labels, status color maps, options      │
├────────────────────────────────────────────────────────┤
│ 3. TYPES & INTERFACES                                  │
│    - Flat, clear TypeScript contracts                  │
├────────────────────────────────────────────────────────┤
│ 4. UTILITIES & CUSTOM HOOKS                            │
│    - Pure functions, data formatting, business logic   │
├────────────────────────────────────────────────────────┤
│ 5. SUB-COMPONENTS                                      │
│    - Focused, isolated UI building blocks              │
├────────────────────────────────────────────────────────┤
│ 6. MAIN COMPONENT                                      │
│    - Flat control flow, early return guard clauses     │
└────────────────────────────────────────────────────────┘
```

---

## The 8 Senior Refactoring Rules

1. **Extract Top Configuration:** Move all hardcoded text strings, theme styles, limits, and URLs into a `CONFIG` object at the top of the file so changes take under 10 seconds.
2. **Eliminate State Synchronization:** Replace `useEffect` calls that synchronize state with pure derived variables or `useMemo`.
3. **Flat Guard Clauses (Early Returns):** Never write 4-level nested ternaries inside JSX. Return early for Loading, Error, and Empty states.
4. **Group Related State:** Consolidate related `useState` variables into typed state objects or custom hooks.
5. **Extract Sub-Components:** Keep the main component under 150 lines by extracting focused items (`TableRow`, `FilterBar`, `SummaryCard`).
6. **Move Inline Anonymous Handlers:** Lift complex `onClick` or `onChange` logic into named handler functions.
7. **Isolate Next.js Client Boundaries:** Keep data fetching in Server Components (`async/await`); isolate `"use client"` to interactive leaves.
8. **Preserve 100% of Runtime Behavior:** Never alter existing props, external contracts, API payloads, or visual layouts without explicit instruction.

---

## Installation & Usage

### 1. Global Installation (Any Agent)
Compatible with Claude Code, Cursor, Copilot, Antigravity, Cline, and Codex:

```bash
npx skills add AhmadJalhoom20/nextjs-react-clean-code -g -y
```

### 2. Google Antigravity / Gemini CLI
Clone directly into your global plugins directory:

```bash
git clone https://github.com/AhmadJalhoom20/nextjs-react-clean-code.git ~/.gemini/config/plugins/nextjs-react-clean-code
```

### 3. Claude Code Marketplace
```bash
/plugin marketplace add AhmadJalhoom20/nextjs-react-clean-code
/plugin install nextjs-react-clean-code
```

### 4. How to Invoke in Chat
Simply paste your React or Next.js file and say:
```text
/nextjs-react-clean-code refactor this component to senior standard
```
Or:
```text
Clean up this React code and remove the useEffect chains
```

---

## Contributing

Contributions, test cases, and edge-case refactoring patterns are welcome. When opening a pull request, include both the messy input code and the refactored output conforming to the 6-zone standard.

## License

MIT © [Ahmad Jalhoom](https://github.com/AhmadJalhoom20)
