<div dir="rtl">

# nextjs-react-clean-code

مهارة هندسية احترافية لوكلاء ونماذج الذكاء الاصطناعي، متخصصة في إعادة هيكلة وتنظيف كود React و Next.js المتشابك (Spaghetti Code) أو المولد عشوائياً، وتحويله إلى كود معياري نظيف، مسطح، وسهل التعديل اليدوي بمستوى مهندس أول (Senior Engineer +5 سنوات خبرة).

<p align="right">
  <a href="README.md">English (الإنجليزية)</a> • <b>العربية</b>
</p>

[![License: MIT](https://img.shields.io/badge/الرخصة-MIT-black.svg?style=flat-square)](LICENSE)
[![Skills.sh](https://img.shields.io/badge/skills.sh-معتمد-black?style=flat-square)](https://skills.sh)
[![متوافق مع Claude Code](https://img.shields.io/badge/Claude%20Code-متوافق-black?style=flat-square)](https://anthropic.com)
[![متوافق مع Antigravity](https://img.shields.io/badge/Antigravity-أصلي-black?style=flat-square)](https://github.com)

```bash
# تثبيت المهارة عبر الـ CLI لجميع الوكلاء بأمر واحد
npx skills add AhmadJalhoom20/nextjs-react-clean-code -g -y
```

---

## المشكلة: فوضى كود React المولد بالذكاء الاصطناعي

تميل نماذج الذكاء الاصطناعي عند توليد كود React أو صفحات Next.js إلى إنتاج كود يعمل في الظاهر، ولكنه يتحول إلى كابوس صيانة يصعب تعديله يدوياً:

1. **انفجار الـ State (`useState` Soup):** وجود 8 أو 10 متغيرات `useState` مستقلة لحقول نماذج أو نوافذ منبثقة مترابطة.
2. **سلاسل تفاعلات `useEffect`:** استخدام `useEffect` لمزامنة الحالات المحسوبة، مما يسبب حلقات لا نهائية وبطئاً في إعادة التصيير (Re-renders).
3. **أهرامات الشروط الثلاثية (Ternary Pyramids):** حشو شجرة الـ JSX بـ 3 مستويات أو أكثر من الشروط المتداخلة `a ? (b ? <X/> : <Y/>) : <Z/>`.
4. **القيم الثابتة المدفونة:** نصوص الروابط وعناوين الـ API وألوان الحالات مدفونة في السطر 300 داخل وسوم الـ JSX.
5. **خلط حدود Next.js:** وضع `'use client'` في رأس صفحات كاملة بدلاً من عزل المكونات التفاعلية كأوراق نهائية في الشجرة.

تفرض مهارة **`nextjs-react-clean-code`** معايير المهندس الأول: كتابة كود مسطح، واضح، يسهل على أي مطور بشري قراءته وتعديله في ثوانٍ دون الحاجة لمساعدة الذكاء الاصطناعي.

---

## مقارنة برمجية واقعية

### قبل: كود سباغيتي مولد بالذكاء الاصطناعي (يصعب تعديله يدوياً)

```tsx
// ❌ كود مشوش: انفجار في الـ useState، سلاسل useEffect، وشروط ثلاثية متداخلة
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

  // خطأ شائع: استخدام useEffect لحساب بيانات مشتقة
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
        <p>جاري التحميل...</p>
      ) : error ? (
        <p className="text-red-500">{error}</p>
      ) : filtered.length === 0 ? (
        <p>لا يوجد مستخدمين</p>
      ) : (
        <div>
          <input
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            className="border p-2 mb-4"
            placeholder="البحث عن مستخدم..."
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

### بعد: كود نظيف بمعايير المهندس الأول (Senior Standard)

```tsx
// ✅ كود احترافي: كتلة ثوابت عليا سهلة التعديل، بيانات مشتقة مباشرة، وحراسة الخروج المبكر
"use client";

import React, { useState, useMemo } from "react";

// ============================================================================
// 1. الإعدادات والثوابت (منطقة التعديل السريع)
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
    searchPlaceholder: "البحث في دليل المستخدمين...",
    emptyState: "لم يتم العثور على أي مستخدم مطابق.",
    errorTitle: "فشل في تحميل الدليل",
    retryButton: "إعادة المحاولة",
  },
} as const;

// ============================================================================
// 2. الواجهات والأنواع (Types & Interfaces)
// ============================================================================
interface User {
  id: string;
  name: string;
  role: keyof typeof CONFIG.roleBadgeStyles;
}

// ============================================================================
// 3. المكونات الفرعية (Sub-Components)
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
// 4. المكون الرئيسي (Main Component)
// ============================================================================
export default function UserList({ initialUsers = [] }: { initialUsers?: User[] }) {
  const [search, setSearch] = useState("");
  const [selectedRole, setSelectedRole] = useState<string>("all");

  // بيانات مشتقة مباشرة أثناء التصيير دون الحاجة لـ useEffect
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

## هيكلية المناطق الـ 6 القياسية (The 6-Zone File Layout)

كل ملف يتم إعادة هيكلته يتبع تسلسلاً ثابتاً من الأعلى إلى الأسفل:

```
┌────────────────────────────────────────────────────────┐
│ 1. الاستيرادات (مجمعة: المكتبات الأساسية، ثم الخارجية، ثم المحلية) │
├────────────────────────────────────────────────────────┤
│ 2. الإعدادات والثوابت (منطقة التعديل السريع في ثوانٍ)   │
│    - الروابط، النصوص، خرائط ألوان الحالات، الخيارات     │
├────────────────────────────────────────────────────────┤
│ 3. الواجهات والأنواع (عقود TypeScript مسطحة وواضحة)     │
├────────────────────────────────────────────────────────┤
│ 4. الدوال المساعدة والخطافات المخصصة (Custom Hooks)    │
│    - دوال التنسيق، التصفية، ومنطق الأعمال               │
├────────────────────────────────────────────────────────┤
│ 5. المكونات الفرعية المعزولة (Sub-Components)          │
│    - كتل الواجهة المصغرة المركزة                        │
├────────────────────────────────────────────────────────┤
│ 6. المكون الرئيسي (Main Component)                     │
│    - تدفق تحكم مسطح، وحراسة الخروج المبكر              │
└────────────────────────────────────────────────────────┘
```

---

## قواعد إعادة الهيكلة الـ 8 للمهندسين الأوائل

1. **استخراج كتلة إعدادات عليا (Top Config):** وضع جميع النصوص الثابتة، وألوان الحالات، وعناوين الـ API في كائن `CONFIG` في أعلى الملف لتعديلها في أقل من 10 ثوانٍ.
2. **إلغاء مزامنة الحالات عبر `useEffect`:** استبدال خطافات `useEffect` المخصصة لحساب القيم بحساب مباشر أو `useMemo`.
3. **حراسة الخروج المبكر (Early Returns):** التخلص من أهرامات الشروط الثلاثية، واستخدام حراسة مبكرة لحالات التحميل والأخطاء والحالة الفارغة.
4. **تجميع الحالات المترابطة:** دمج متغيرات `useState` المتفرقة داخل كائن حالة واحد أو خطاف مخصص.
5. **تفكيك المكونات الضخمة:** الحفاظ على المكون الرئيسي أقل من 150 سطراً عبر استخراج مكونات فرعية محددة (`TableRow`, `FilterBar`).
6. **رفع الدوال المجهولة من داخل الـ JSX:** نقل دوال `onClick` و `onChange` المعقدة إلى دوال مسماة واضحة ومستقلة.
7. **عزل حدود العميل في Next.js:** إبقاء جلب البيانات في Server Components، وعزل استخدام `'use client'` في أصغر مكون تفاعلي ممكن.
8. **الحفاظ التام على السلوك الوظيفي (100% Behavioral Preservation):** عدم تغيير أي خصائص (Props) خارجية أو واجهات API دون طلب صريح.

---

## التثبيت والاستخدام

### 1. تثبيت عام عبر أداة Skills CLI
```bash
npx skills add AhmadJalhoom20/nextjs-react-clean-code -g -y
```

### 2. في Google Antigravity / Gemini CLI
```bash
git clone https://github.com/AhmadJalhoom20/nextjs-react-clean-code.git ~/.gemini/config/plugins/nextjs-react-clean-code
```

### 3. في متجر إضافات Claude Code
```bash
/plugin marketplace add AhmadJalhoom20/nextjs-react-clean-code
/plugin install nextjs-react-clean-code
```

### 4. طريقة الاستخدام في المحادثة
فقط شارك ملف الـ React أو Next.js واكتب:
```text
/nextjs-react-clean-code نظف وهيكل هذا الكود لمعايير المهندس الأول
```

---

## المساهمة في المشروع

نرحب بمساهمات المجتمع البرمجي، وحالات الاختبار المعقدة، وأنماط إعادة الهيكلة الإضافية.

## الرخصة

مرخص تحت رخصة **MIT**. راجع ملف [`LICENSE`](LICENSE) للمزيد من التفاصيل.

<br/>

<div align="center">
  <sub>طُوّر بعناية بواسطة <a href="https://github.com/AhmadJalhoom20">أحمد جلهوم (Ahmad Jalhoom)</a>. كود React نظيف ومقروء يدوم طويلاً.</sub>
</div>

</div>
