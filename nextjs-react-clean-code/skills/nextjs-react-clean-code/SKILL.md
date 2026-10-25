---
name: nextjs-react-clean-code
description: Refactor messy, AI-generated, or spaghetti React and Next.js code into clean, maintainable, human-editable code written at a senior (5+ years experience) standard. Use whenever the user asks to clean up, refactor, de-spaghettify, simplify, modularize, or make React/Next.js (.jsx, .tsx, .js, .ts) code easier to read and edit manually without relying on AI for future changes.
---

# Next.js and React Clean Code Refactoring

Transform tangled, hard-to-maintain React and Next.js spaghetti code into flat, predictable, senior-level code (5+ years experience standard) that a developer can read, understand, and edit directly in seconds without asking AI for help.

## When to Use

- The user shares a messy, long, or AI-generated React component or Next.js page/route and asks to clean it up, refactor it, or turn it into clean code.
- The user complains that their code is "spaghetti code", fragile, hard to read, or breaks whenever they try to edit it manually.
- The user wants code written like an experienced senior React/Next.js engineer (clear structure, derived state, guard clauses, no nested ternary soup, and zero over-engineered abstractions).
- The user asks to reorganize a monolithic `.jsx` or `.tsx` file so they can easily tweak UI copy, styles, API endpoints, or business logic on their own.

## Core Philosophy: The 5-Year Senior Engineer Standard

Junior code is messy because everything is mixed together in one giant component. Over-engineered code is messy because simple logic is hidden behind ten layers of clever abstractions and unreadable TypeScript generics.

A pragmatic senior React/Next.js engineer writes **flat, boring, obvious code** governed by five rules:

1. **Editability Over Cleverness**: A human should be able to change a button label, API URL, table column, validation rule, or Tailwind class in under 10 seconds without tracing through five callbacks.
2. **Flat Control Flow**: Use early returns (guard clauses) instead of deeply nested `if/else` blocks or multi-level ternary operators inside JSX.
3. **Single Responsibility Sections**: Every file follows a strict top-to-bottom layout with clear section comments so the developer always knows where to look.
4. **Derived State Over Synced State**: Never use `useEffect` + `useState` to compute values that can be calculated directly during render.
5. **Preserve 100% of Behavior**: Refactoring changes internal structure and readability while keeping all existing props, UI behavior, API payloads, and visual styling intact unless asked otherwise.

## Step 1: Diagnose the Spaghetti Patterns

Before rewriting the file, scan the input code for these common React and Next.js anti-patterns:

- **Hardcoded Magic Values in JSX**: Text strings, navigation links, status colors, pagination limits, and API routes buried at line 300 inside JSX tags.
- **State Explosion (`useState` Soup)**: Eight or more independent `useState` calls tracking related form fields or modal states that belong in a single object, custom hook, or derived variable.
- **`useEffect` Chain Reactions**: Multiple `useEffect` hooks watching state variables and triggering further state updates, causing infinite loops or race conditions.
- **Nested Ternary Pyramids**: Three or more levels of `conditionA ? (conditionB ? <X /> : <Y />) : <Z />` inside the `return` block.
- **Inline Anonymous Handler Bloat**: Multi-line `onClick={() => { ... }}` or `onChange={(e) => { ... }}` functions written directly inside JSX props.
- **Tailwind Class Soup**: 250-character class strings repeated across ten buttons or cards without a shared style map or small reusable component.
- **Next.js Boundary Confusion**: Adding `'use client'` to an entire page just for one interactive button, or fetching initial page data inside `useEffect` when a Next.js Server Component or clean data hook should handle it.

## Step 2: Apply the Standard 6-Zone File Layout

Reorganize every refactored React/Next.js file into six predictable zones from top to bottom. Add clear banner comments so the user can jump straight to the part they want to edit:

```tsx
// ============================================================================
// 1. IMPORTS
// ============================================================================
import React, { useState, useMemo } from 'react';

// ============================================================================
// 2. CONFIGURATION & CONSTANTS (EASY EDIT ZONE)
// ============================================================================
// Put all tweakable text, URLs, limits, status badges, and Tailwind theme maps
// right here so you never have to hunt through JSX to make quick edits.

// ============================================================================
// 3. TYPES & INTERFACES
// ============================================================================
// Simple, flat TypeScript interfaces. Avoid unreadable nested generics.

// ============================================================================
// 4. HELPER FUNCTIONS & CUSTOM HOOKS
// ============================================================================
// Pure utility functions (formatting, filtering, validation) and state hooks.

// ============================================================================
// 5. SUB-COMPONENTS
// ============================================================================
// Small, focused UI pieces (e.g., UserRow, EmptyState, FilterBar).

// ============================================================================
// 6. MAIN COMPONENT
// ============================================================================
// High-level layout, guard clauses (loading/error), and clean composition.
```

## Step 3: Execute the 8 Senior Refactoring Rules

### Rule 1: Extract a Top-of-File Config & Copy Block

Pull every magic number, API path, dropdown option list, table header, and reusable Tailwind variant map into Zone 2 at the top of the file.

**Spaghetti Pattern (Hard to Edit):**

```tsx
export default function PricingCard({ plan }: { plan: string }) {
  return (
    <div className={plan === 'pro' ? 'border-blue-600 bg-blue-50/40 p-6 rounded-xl' : plan === 'team' ? 'border-purple-600 bg-purple-50/40 p-6 rounded-xl' : 'border-gray-200 bg-white p-6 rounded-xl'}>
      <h3>{plan === 'pro' ? 'Professional Plan' : plan === 'team' ? 'Enterprise Team' : 'Starter Free'}</h3>
      <p>{plan === 'pro' ? '$29/mo' : plan === 'team' ? '$99/mo' : '$0/mo'}</p>
    </div>
  );
}
```

**Clean Senior Pattern (Easy to Edit in 5 Seconds):**

```tsx
// === CONFIGURATION & CONSTANTS (EDIT PLANS HERE) ===
const PLAN_CONFIG = {
  starter: {
    title: 'Starter Free',
    price: '$0/mo',
    cardClass: 'border-gray-200 bg-white',
  },
  pro: {
    title: 'Professional Plan',
    price: '$29/mo',
    cardClass: 'border-blue-600 bg-blue-50/40',
  },
  team: {
    title: 'Enterprise Team',
    price: '$99/mo',
    cardClass: 'border-purple-600 bg-purple-50/40',
  },
} as const;

type PlanKey = keyof typeof PLAN_CONFIG;

export default function PricingCard({ plan = 'starter' }: { plan: PlanKey }) {
  const currentPlan = PLAN_CONFIG[plan] ?? PLAN_CONFIG.starter;

  return (
    <div className={`p-6 rounded-xl border ${currentPlan.cardClass}`}>
      <h3>{currentPlan.title}</h3>
      <p>{currentPlan.price}</p>
    </div>
  );
}
```

Why this works: To add a new plan or change a price/color, the user only edits `PLAN_CONFIG` at the top of the file without touching JSX.

### Rule 2: Replace Synced `useEffect` State with Derived State

Junior and AI-generated code often uses `useState` + `useEffect` to filter lists, compute totals, or check form validity. This adds extra renders and bugs. Compute derived values directly during render (or with `useMemo` if the list is large).

**Spaghetti Pattern:**

```tsx
const [users, setUsers] = useState<User[]>([]);
const [search, setSearch] = useState('');
const [filteredUsers, setFilteredUsers] = useState<User[]>([]);
const [activeCount, setActiveCount] = useState(0);

useEffect(() => {
  const result = users.filter((u) =>
    u.name.toLowerCase().includes(search.toLowerCase())
  );
  setFilteredUsers(result);
  setActiveCount(result.filter((u) => u.isActive).length);
}, [users, search]);
```

**Clean Senior Pattern:**

```tsx
const [users, setUsers] = useState<User[]>([]);
const [search, setSearch] = useState('');

// Derived directly during render - zero useEffect needed
const filteredUsers = useMemo(() => {
  const query = search.trim().toLowerCase();
  if (!query) return users;
  return users.filter((user) => user.name.toLowerCase().includes(query));
}, [users, search]);

const activeCount = filteredUsers.filter((user) => user.isActive).length;
```

### Rule 3: Collapse State Explosion into Logical Groups

When a component has more than 4 related `useState` hooks (such as form fields or filter settings), group them into a single state object with a clean `updateField` helper, or extract a custom hook in the same file.

**Spaghetti Pattern:**

```tsx
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const [email, setEmail] = useState('');
const [role, setRole] = useState('member');
const [isSubmitting, setIsSubmitting] = useState(false);
const [errorMsg, setErrorMsg] = useState<string | null>(null);
```

**Clean Senior Pattern:**

```tsx
const INITIAL_FORM = {
  firstName: '',
  lastName: '',
  email: '',
  role: 'member',
};

const [form, setForm] = useState(INITIAL_FORM);
const [status, setStatus] = useState<'idle' | 'submitting' | 'error'>('idle');
const [errorMessage, setErrorMessage] = useState<string | null>(null);

function updateFormField<K extends keyof typeof INITIAL_FORM>(
  field: K,
  value: (typeof INITIAL_FORM)[K]
) {
  setForm((prev) => ({ ...prev, [field]: value }));
}
```

### Rule 4: Eliminate Nested Ternaries with Guard Clauses and Sub-Components

Never nest ternary operators (`? :`) inside JSX. Handle loading, error, and empty states with early `if` returns at the top of the component or inside a dedicated content renderer.

**Spaghetti Pattern:**

```tsx
return (
  <div className="p-6">
    {isLoading ? (
      <div className="animate-pulse">Loading...</div>
    ) : error ? (
      <div className="text-red-500">{error}</div>
    ) : items.length === 0 ? (
      <div className="text-gray-400">No items found.</div>
    ) : (
      <ul>
        {items.map((item) => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    )}
  </div>
);
```

**Clean Senior Pattern:**

```tsx
if (isLoading) {
  return <div className="p-6 animate-pulse">Loading...</div>;
}

if (error) {
  return <div className="p-6 text-red-500">{error}</div>;
}

if (items.length === 0) {
  return <div className="p-6 text-gray-400">No items found.</div>;
}

return (
  <div className="p-6">
    <ul className="space-y-2">
      {items.map((item) => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  </div>
);
```

### Rule 5: Extract Named Event Handlers Above the `return` Statement

Any event handler longer than a single state setter call (`onClick={() => setIsOpen(true)}`) must be extracted into a named function prefixed with `handle` (`handleSearchChange`, `handleDeleteUser`, `handleFormSubmit`).

**Spaghetti Pattern:**

```tsx
<button
  onClick={async () => {
    if (!confirm('Delete item?')) return;
    setDeletingId(item.id);
    try {
      const res = await fetch(`/api/items/${item.id}`, { method: 'DELETE' });
      if (!res.ok) throw new Error('Failed');
      setItems((prev) => prev.filter((i) => i.id !== item.id));
    } catch (err) {
      alert('Could not delete item');
    } finally {
      setDeletingId(null);
    }
  }}
>
  Delete
</button>
```

**Clean Senior Pattern:**

```tsx
async function handleDeleteItem(itemId: string) {
  if (!window.confirm('Delete item?')) return;

  setDeletingId(itemId);
  try {
    const response = await fetch(`${API_ENDPOINTS.items}/${itemId}`, {
      method: 'DELETE',
    });
    if (!response.ok) throw new Error('Delete request failed');

    setItems((prev) => prev.filter((item) => item.id !== itemId));
  } catch {
    window.alert('Could not delete item. Please try again.');
  } finally {
    setDeletingId(null);
  }
}

// Inside JSX:
<button onClick={() => handleDeleteItem(item.id)}>Delete</button>
```

### Rule 6: Clean Up Tailwind CSS Without Over-Abstracting

Long Tailwind strings make JSX noisy and hard to scan. Organize styles using a `STYLES` dictionary in Zone 2 or small wrapper components in Zone 5 so changing the look of every button or input only requires editing one line.

```tsx
// === 2. CONFIGURATION & STYLES (EDIT UI THEME HERE) ===
const STYLES = {
  card: 'rounded-xl border border-slate-200 bg-white p-6 shadow-sm',
  primaryButton:
    'inline-flex items-center justify-center rounded-lg bg-blue-600 px-4 py-2 text-sm font-medium text-white hover:bg-blue-700 disabled:opacity-50',
  secondaryButton:
    'inline-flex items-center justify-center rounded-lg border border-slate-300 bg-white px-4 py-2 text-sm font-medium text-slate-700 hover:bg-slate-50',
  input:
    'w-full rounded-lg border border-slate-300 px-3 py-2 text-sm focus:border-blue-500 focus:outline-none',
};
```

### Rule 7: Next.js App Router Best Practices

When refactoring Next.js files (`app/` or `pages/`):

- **Keep Server Components Clean**: If a Next.js App Router page doesn't use hooks (`useState`, `useEffect`) or browser events (`onClick`), keep it as an `async` Server Component that fetches data directly and passes plain props to small Client Components.
- **Isolate `'use client'` Boundaries**: Do not slap `'use client'` at the top of an entire layout or page if only a search bar or modal needs interactivity.
- **Centralize Fetch Logic**: Move raw `fetch('/api/...')` calls into clearly named async helper functions (`fetchOrders()`, `updateOrderStatus()`) in Zone 4 so changing an endpoint or headers happens in one place.
- **Always Handle Loading, Empty, and Error States**: Ensure every data request has explicit, user-friendly UI states.

### Rule 8: Write Human-Friendly TypeScript (No Type Gymnastics)

Senior engineers write simple, readable TypeScript interfaces that help autocomplete work instead of fighting the developer:

- Prefer `interface UserProfile { ... }` with explicit field names and comments over convoluted utility type chains (`Omit<Pick<Partial<T>, K>, V>`).
- Use string union types (`type OrderStatus = 'pending' | 'paid' | 'shipped'`) instead of numeric enums so values are self-explanatory in logs and JSX.

## Step 4: Deliver the Refactored File and "Human Edit Guide"

Whenever you finish refactoring a file using this skill:

1. **Output the Complete Refactored File**: Never leave `// ... rest of code` placeholders unless the user explicitly asks for a partial snippet. Deliver the complete, working file (or save/update the source file directly using the workspace file tools).
2. **Include a Concise "How to Edit This Yourself" Cheat Sheet**: Right after presenting the refactored file, include a short 4-to-6 bullet guide pointing to the exact sections the user can edit on their own:
   - **To change text, labels, or API URLs**: Edit `CONFIG` in Zone 2.
   - **To change colors, spacing, or button styles**: Edit `STYLES` in Zone 2.
   - **To add or remove a form field / table column**: Add the field to `INITIAL_FORM` or `COLUMNS` in Zone 2 and the interface in Zone 3.
   - **To change what happens on submit / click**: Edit `handleSubmit` in Zone 4.

## Gotchas to Avoid

- **Do not break existing props or exports**: Keep the same component name and `default` vs. named export signature so the user can drop the refactored file straight into their project without breaking imports elsewhere.
- **Do not split into 10 tiny files unless asked**: By default, keep the refactored code in the **same single file** organized into the 6 clean zones (unless the user explicitly asks to split into multiple files). Single-file modularity is much easier for the user to copy, paste, and edit than jumping across eight folders.
- **Do not introduce external libraries the user hasn't installed**: Do not replace standard React state with Zustand, Redux, TanStack Query, or Zod unless those packages are already imported in the user's code or explicitly requested.
- **Do not over-comment obvious lines**: Skip noisy comments like `// increment count by 1`. Instead, comment the **why** and mark **Edit Zones** (`// EDIT HERE: Add new navigation links to this array`).

# Next.js & React Spaghetti-to-Clean Refactoring Playbook

This reference contains complete before-and-after refactoring examples demonstrating how to convert typical AI-generated or junior spaghetti code into senior-level (5+ years experience), human-editable Next.js and React code.

## Example 1: Full Dashboard Page Refactoring (Before vs. After)

### Before: Typical Spaghetti React/Next.js Component (Hard to Edit)

Problems with this code:
- 9 separate `useState` hooks and chained `useEffect` calls.
- Magic strings, status colors, and API URLs scattered inside JSX.
- Deeply nested ternaries inside the `return` statement.
- Inline `fetch` logic inside button `onClick` handlers.
- Impossible to add a new status filter or table column without editing 5 different places.

```tsx
'use client';
import React, { useState, useEffect } from 'react';

export default function OrdersDashboard() {
  const [orders, setOrders] = useState<any[]>([]);
  const [filtered, setFiltered] = useState<any[]>([]);
  const [loading, setLoading] = useState(true);
  const [err, setErr] = useState('');
  const [q, setQ] = useState('');
  const [status, setStatus] = useState('all');
  const [sortAsc, setSortAsc] = useState(false);
  const [updatingId, setUpdatingId] = useState<string | null>(null);
  const [totalRev, setTotalRev] = useState(0);

  useEffect(() => {
    setLoading(true);
    fetch('/api/v1/orders')
      .then((r) => r.json())
      .then((d) => {
        setOrders(d.orders || []);
        setLoading(false);
      })
      .catch((e) => {
        setErr('Failed to load orders');
        setLoading(false);
      });
  }, []);

  useEffect(() => {
    let temp = [...orders];
    if (status !== 'all') {
      temp = temp.filter((o) => o.status === status);
    }
    if (q.trim() !== '') {
      temp = temp.filter(
        (o) =>
          o.customerName.toLowerCase().includes(q.toLowerCase()) ||
          o.id.toLowerCase().includes(q.toLowerCase())
      );
    }
    temp.sort((a, b) =>
      sortAsc
        ? new Date(a.createdAt).getTime() - new Date(b.createdAt).getTime()
        : new Date(b.createdAt).getTime() - new Date(a.createdAt).getTime()
    );
    setFiltered(temp);
    setTotalRev(temp.reduce((acc, cur) => acc + (cur.amount || 0), 0));
  }, [orders, status, q, sortAsc]);

  return (
    <div className="max-w-6xl mx-auto p-8">
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-2xl font-bold text-gray-900">Customer Orders</h1>
        <div className="bg-emerald-50 text-emerald-700 px-4 py-2 rounded-lg font-semibold">
          Revenue: ${totalRev.toFixed(2)}
        </div>
      </div>
      <div className="flex gap-4 mb-6">
        <input
          value={q}
          onChange={(e) => setQ(e.target.value)}
          placeholder="Search customer or ID..."
          className="border border-gray-300 rounded-lg px-3 py-2 w-64"
        />
        <select
          value={status}
          onChange={(e) => setStatus(e.target.value)}
          className="border border-gray-300 rounded-lg px-3 py-2"
        >
          <option value="all">All Statuses</option>
          <option value="pending">Pending</option>
          <option value="paid">Paid</option>
          <option value="shipped">Shipped</option>
        </select>
        <button
          onClick={() => setSortAsc(!sortAsc)}
          className="border border-gray-300 rounded-lg px-4 py-2 bg-white hover:bg-gray-50"
        >
          Sort Date: {sortAsc ? 'Oldest First' : 'Newest First'}
        </button>
      </div>
      {loading ? (
        <div className="p-12 text-center text-gray-500">Loading orders...</div>
      ) : err ? (
        <div className="p-6 bg-red-50 text-red-600 rounded-lg">{err}</div>
      ) : filtered.length === 0 ? (
        <div className="p-12 text-center text-gray-400 border rounded-lg">
          No matching orders found.
        </div>
      ) : (
        <div className="border rounded-xl overflow-hidden">
          <table className="w-full text-left border-collapse">
            <thead className="bg-gray-50 border-b">
              <tr>
                <th className="p-4">Order ID</th>
                <th className="p-4">Customer</th>
                <th className="p-4">Amount</th>
                <th className="p-4">Status</th>
                <th className="p-4">Actions</th>
              </tr>
            </thead>
            <tbody>
              {filtered.map((o) => (
                <tr key={o.id} className="border-b hover:bg-gray-50">
                  <td className="p-4 font-mono text-sm">{o.id}</td>
                  <td className="p-4">{o.customerName}</td>
                  <td className="p-4">${Number(o.amount).toFixed(2)}</td>
                  <td className="p-4">
                    <span
                      className={
                        o.status === 'paid'
                          ? 'bg-green-100 text-green-800 px-2 py-1 rounded text-xs'
                          : o.status === 'shipped'
                          ? 'bg-blue-100 text-blue-800 px-2 py-1 rounded text-xs'
                          : 'bg-yellow-100 text-yellow-800 px-2 py-1 rounded text-xs'
                      }
                    >
                      {o.status === 'paid'
                        ? 'Paid'
                        : o.status === 'shipped'
                        ? 'Shipped'
                        : 'Pending'}
                    </span>
                  </td>
                  <td className="p-4">
                    {o.status !== 'shipped' && (
                      <button
                        disabled={updatingId === o.id}
                        onClick={async () => {
                          setUpdatingId(o.id);
                          try {
                            const res = await fetch(`/api/v1/orders/${o.id}`, {
                              method: 'PATCH',
                              headers: { 'Content-Type': 'application/json' },
                              body: JSON.stringify({ status: 'shipped' }),
                            });
                            if (res.ok) {
                              setOrders(
                                orders.map((item) =>
                                  item.id === o.id
                                    ? { ...item, status: 'shipped' }
                                    : item
                                )
                              );
                            }
                          } finally {
                            setUpdatingId(null);
                          }
                        }}
                        className="text-sm bg-blue-600 text-white px-3 py-1 rounded hover:bg-blue-700 disabled:opacity-50"
                      >
                        {updatingId === o.id ? 'Saving...' : 'Mark Shipped'}
                      </button>
                    )}
                  </td>
                </tr>
              ))}
            </tbody>
          </table>
        </div>
      )}
    </div>
  );
}
```

### After: Refactored Senior-Level React/Next.js Component (Easy to Edit)

```tsx
'use client';

// ============================================================================
// 1. IMPORTS
// ============================================================================
import React, { useState, useEffect, useMemo } from 'react';

// ============================================================================
// 2. CONFIGURATION, CONSTANTS & STYLES (EASY EDIT ZONE)
// ============================================================================
const CONFIG = {
  pageTitle: 'Customer Orders',
  apiEndpoint: '/api/v1/orders',
  searchPlaceholder: 'Search customer or ID...',
};

const ORDER_STATUS_CONFIG = {
  pending: {
    label: 'Pending',
    badgeClass: 'bg-yellow-100 text-yellow-800',
  },
  paid: {
    label: 'Paid',
    badgeClass: 'bg-green-100 text-green-800',
  },
  shipped: {
    label: 'Shipped',
    badgeClass: 'bg-blue-100 text-blue-800',
  },
} as const;

const STATUS_FILTER_OPTIONS = [
  { value: 'all', label: 'All Statuses' },
  { value: 'pending', label: 'Pending' },
  { value: 'paid', label: 'Paid' },
  { value: 'shipped', label: 'Shipped' },
] as const;

const STYLES = {
  container: 'max-w-6xl mx-auto p-8',
  input: 'border border-gray-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:border-blue-500',
  secondaryButton: 'border border-gray-300 rounded-lg px-4 py-2 text-sm bg-white hover:bg-gray-50',
  primaryActionButton: 'text-sm bg-blue-600 text-white px-3 py-1 rounded hover:bg-blue-700 disabled:opacity-50',
  tableHeaderCell: 'p-4 text-sm font-semibold text-gray-700',
  tableBodyCell: 'p-4 text-sm text-gray-800',
};

// ============================================================================
// 3. TYPES & INTERFACES
// ============================================================================
type OrderStatus = keyof typeof ORDER_STATUS_CONFIG;
type StatusFilter = OrderStatus | 'all';

interface Order {
  id: string;
  customerName: string;
  amount: number;
  status: OrderStatus;
  createdAt: string;
}

interface FilterState {
  query: string;
  status: StatusFilter;
  sortAscending: boolean;
}

// ============================================================================
// 4. HELPER FUNCTIONS & API CALLS
// ============================================================================
function formatCurrency(amount: number): string {
  return `$${Number(amount || 0).toFixed(2)}`;
}

function filterAndSortOrders(orders: Order[], filters: FilterState): Order[] {
  const normalizedQuery = filters.query.trim().toLowerCase();

  return orders
    .filter((order) => {
      const matchesStatus =
        filters.status === 'all' || order.status === filters.status;
      const matchesQuery =
        !normalizedQuery ||
        order.customerName.toLowerCase().includes(normalizedQuery) ||
        order.id.toLowerCase().includes(normalizedQuery);

      return matchesStatus && matchesQuery;
    })
    .sort((a, b) => {
      const timeA = new Date(a.createdAt).getTime();
      const timeB = new Date(b.createdAt).getTime();
      return filters.sortAscending ? timeA - timeB : timeB - timeA;
    });
}

// ============================================================================
// 5. SUB-COMPONENTS
// ============================================================================
function StatusBadge({ status }: { status: OrderStatus }) {
  const config = ORDER_STATUS_CONFIG[status] ?? ORDER_STATUS_CONFIG.pending;
  return (
    <span className={`px-2 py-1 rounded text-xs font-medium ${config.badgeClass}`}>
      {config.label}
    </span>
  );
}

interface OrderRowProps {
  order: Order;
  isUpdating: boolean;
  onMarkShipped: (orderId: string) => void;
}

function OrderRow({ order, isUpdating, onMarkShipped }: OrderRowProps) {
  return (
    <tr className="border-b hover:bg-gray-50">
      <td className={`${STYLES.tableBodyCell} font-mono`}>{order.id}</td>
      <td className={STYLES.tableBodyCell}>{order.customerName}</td>
      <td className={STYLES.tableBodyCell}>{formatCurrency(order.amount)}</td>
      <td className={STYLES.tableBodyCell}>
        <StatusBadge status={order.status} />
      </td>
      <td className={STYLES.tableBodyCell}>
        {order.status !== 'shipped' && (
          <button
            disabled={isUpdating}
            onClick={() => onMarkShipped(order.id)}
            className={STYLES.primaryActionButton}
          >
            {isUpdating ? 'Saving...' : 'Mark Shipped'}
          </button>
        )}
      </td>
    </tr>
  );
}

// ============================================================================
// 6. MAIN COMPONENT
// ============================================================================
export default function OrdersDashboard() {
  const [orders, setOrders] = useState<Order[]>([]);
  const [isLoading, setIsLoading] = useState(true);
  const [errorMessage, setErrorMessage] = useState<string | null>(null);
  const [updatingOrderId, setUpdatingOrderId] = useState<string | null>(null);

  const [filters, setFilters] = useState<FilterState>({
    query: '',
    status: 'all',
    sortAscending: false,
  });

  // --- Data Fetching ---
  useEffect(() => {
    async function loadOrders() {
      setIsLoading(true);
      try {
        const response = await fetch(CONFIG.apiEndpoint);
        const data = await response.json();
        setOrders(data.orders ?? []);
      } catch {
        setErrorMessage('Failed to load orders');
      } finally {
        setIsLoading(false);
      }
    }
    loadOrders();
  }, []);

  // --- Derived State (No extra useEffect needed) ---
  const visibleOrders = useMemo(
    () => filterAndSortOrders(orders, filters),
    [orders, filters]
  );

  const totalRevenue = useMemo(
    () => visibleOrders.reduce((sum, order) => sum + (order.amount || 0), 0),
    [visibleOrders]
  );

  // --- Event Handlers ---
  function updateFilter<K extends keyof FilterState>(
    key: K,
    value: FilterState[K]
  ) {
    setFilters((prev) => ({ ...prev, [key]: value }));
  }

  async function handleMarkShipped(orderId: string) {
    setUpdatingOrderId(orderId);
    try {
      const response = await fetch(`${CONFIG.apiEndpoint}/${orderId}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ status: 'shipped' }),
      });

      if (response.ok) {
        setOrders((prev) =>
          prev.map((order) =>
            order.id === orderId ? { ...order, status: 'shipped' } : order
          )
        );
      }
    } finally {
      setUpdatingOrderId(null);
    }
  }

  // --- Content Renderer with Guard Clauses ---
  function renderOrdersContent() {
    if (isLoading) {
      return (
        <div className="p-12 text-center text-gray-500">Loading orders...</div>
      );
    }

    if (errorMessage) {
      return (
        <div className="p-6 bg-red-50 text-red-600 rounded-lg">
          {errorMessage}
        </div>
      );
    }

    if (visibleOrders.length === 0) {
      return (
        <div className="p-12 text-center text-gray-400 border rounded-lg">
          No matching orders found.
        </div>
      );
    }

    return (
      <div className="border rounded-xl overflow-hidden">
        <table className="w-full text-left border-collapse">
          <thead className="bg-gray-50 border-b">
            <tr>
              <th className={STYLES.tableHeaderCell}>Order ID</th>
              <th className={STYLES.tableHeaderCell}>Customer</th>
              <th className={STYLES.tableHeaderCell}>Amount</th>
              <th className={STYLES.tableHeaderCell}>Status</th>
              <th className={STYLES.tableHeaderCell}>Actions</th>
            </tr>
          </thead>
          <tbody>
            {visibleOrders.map((order) => (
              <OrderRow
                key={order.id}
                order={order}
                isUpdating={updatingOrderId === order.id}
                onMarkShipped={handleMarkShipped}
              />
            ))}
          </tbody>
        </table>
      </div>
    );
  }

  // --- Main Render ---
  return (
    <div className={STYLES.container}>
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-2xl font-bold text-gray-900">{CONFIG.pageTitle}</h1>
        <div className="bg-emerald-50 text-emerald-700 px-4 py-2 rounded-lg font-semibold">
          Revenue: {formatCurrency(totalRevenue)}
        </div>
      </div>

      <div className="flex gap-4 mb-6">
        <input
          value={filters.query}
          onChange={(e) => updateFilter('query', e.target.value)}
          placeholder={CONFIG.searchPlaceholder}
          className={`${STYLES.input} w-64`}
        />
        <select
          value={filters.status}
          onChange={(e) =>
            updateFilter('status', e.target.value as StatusFilter)
          }
          className={STYLES.input}
        >
          {STATUS_FILTER_OPTIONS.map((option) => (
            <option key={option.value} value={option.value}>
              {option.label}
            </option>
          ))}
        </select>
        <button
          onClick={() => updateFilter('sortAscending', !filters.sortAscending)}
          className={STYLES.secondaryButton}
        >
          Sort Date: {filters.sortAscending ? 'Oldest First' : 'Newest First'}
        </button>
      </div>

      {renderOrdersContent()}
    </div>
  );
}
```

## Example 2: Next.js App Router Server + Client Split Pattern

When a Next.js `app/` page mixes server data fetching with interactive client state, split the responsibilities cleanly so you never have to debug `useEffect` race conditions on initial page load.

### Pattern Overview

1. **Server Page (`app/products/page.tsx`)**: Fetches initial data directly using `async/await` and passes plain serializable props to the interactive view.
2. **Client View (`components/ProductsClientView.tsx`)**: Handles search, modals, and optimistic UI updates using the 6-Zone Clean Code layout.

```tsx
// ============================================================================
// FILE 1: app/products/page.tsx (Next.js Server Component)
// ============================================================================
import ProductsClientView from './ProductsClientView';

const API_URL = process.env.INTERNAL_API_URL || 'https://api.example.com';

async function getInitialProducts() {
  const response = await fetch(`${API_URL}/products`, {
    cache: 'no-store',
  });

  if (!response.ok) {
    return [];
  }

  return response.json();
}

export default async function ProductsPage() {
  const initialProducts = await getInitialProducts();

  return (
    <main className="min-h-screen bg-slate-50 py-10">
      <ProductsClientView initialProducts={initialProducts} />
    </main>
  );
}
```

## Quick Self-Edit Checklist for Every Refactored File

Use this lookup table whenever you want to make manual edits to a file refactored by this skill:

| What You Want to Change | Where to Go in the File | What to Edit |
| :--- | :--- | :--- |
| Change page titles, button text, or API endpoints | **Zone 2: `CONFIG`** | Update the string value in the `CONFIG` object at the top of the file. |
| Change badge colors, button styles, or input borders | **Zone 2: `STYLES` / `*_CONFIG`** | Update the Tailwind class string in `STYLES` or the status map. |
| Add a new dropdown option or tab | **Zone 2: `*_OPTIONS`** | Add `{ value: 'new', label: 'New Option' }` to the options array. |
| Add a new field to a data item or form | **Zone 2 & Zone 3** | Add the default value in `INITIAL_FORM` (Zone 2) and field type in `interface` (Zone 3). |
| Change how search, filtering, or sorting works | **Zone 4: Helpers** | Edit the pure `filterAndSort*` helper function. |
| Change what happens when a button or form is clicked | **Zone 6: `handle*` functions** | Edit the named `handleSave` / `handleDelete` function above `return`. |
| Rearrange the visual layout or columns | **Zone 5 & Zone 6 JSX** | Move the clean sub-components (`<OrderRow />`, `<StatusBadge />`) inside the `return` block. |
