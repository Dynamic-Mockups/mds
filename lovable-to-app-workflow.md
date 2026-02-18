# Prototype-to-Production Workflow: Lovable → GitHub → Claude Code → Your App

## 1. Overview

Your colleague builds a prototype or new page/feature in Lovable. You need to implement it in your existing React app — without involving a designer or recreating anything in Figma.

Lovable outputs real React code (Vite + TypeScript + Tailwind + shadcn/ui). This means the prototype IS the design spec. Claude Code can read the Lovable codebase directly and translate the layout, interactions, and component structure into your app's architecture — matching your conventions, tokens, and patterns.

---

## 2. Tech Stack & Prerequisites

| Tool | Purpose | Install / Access |
|------|---------|-----------------|
| **Lovable** | AI prototype builder (your colleague uses this) | [lovable.dev](https://lovable.dev) |
| **GitHub** | Lovable syncs prototype code here | Lovable has built-in GitHub sync |
| **Claude Code** | AI coding agent that reads prototype + builds in your app | `curl -fsSL https://claude.ai/install.sh \| bash` |
| **Git** | Clone the Lovable repo locally | Already installed |
| **Node.js 18+** | Runtime | `nvm install 18` |

---

## 3. Setup: Getting the Lovable Code Locally

### 3.1 Colleague Syncs Lovable to GitHub

In Lovable editor:

1. Click the **GitHub icon** (top-right of editor)
2. Click **Connect to GitHub** → authorize Lovable
3. Lovable creates a repo and auto-syncs every change

> The colleague shares the GitHub repo URL with you.

### 3.2 Clone the Prototype Locally

```bash
# Clone next to your app — NOT inside it
cd ~/projects
git clone https://github.com/colleague/lovable-prototype.git
```

Your folder structure should look like this:

```
~/projects/
  ├── your-react-app/          ← your production codebase
  └── lovable-prototype/       ← reference code from Lovable
```

### 3.3 Explore What Lovable Generated

Lovable outputs a standard React project:

```
lovable-prototype/
  ├── src/
  │   ├── components/        ← UI components (shadcn/ui based)
  │   │   └── ui/            ← base shadcn components
  │   ├── pages/             ← route pages
  │   ├── hooks/             ← custom React hooks
  │   ├── lib/               ← utilities
  │   └── integrations/      ← Supabase config (if used)
  ├── tailwind.config.ts
  ├── vite.config.ts
  └── package.json
```

> **Key point:** You don't install or run the Lovable project. You only read its source files as reference.

---

## 4. CLAUDE.md Configuration

Add rules to your app's `CLAUDE.md` so Claude Code translates Lovable code into your architecture — not a copy-paste.

### 4.1 Required Rules

```markdown
# Lovable Prototype Translation Rules

## Source Reference
- The Lovable prototype at `~/projects/lovable-prototype/` is a DESIGN REFERENCE ONLY
- NEVER copy Lovable code directly — translate it into our architecture
- NEVER import from or depend on the Lovable project

## What To Extract From Lovable
- Layout structure (component hierarchy, flex/grid arrangements)
- Responsive breakpoints and behavior
- Interactive states (hover, active, disabled, loading, empty, error)
- Conditional rendering logic
- Component composition patterns (which parts are separate components)

## What To Ignore From Lovable
- Tailwind classes — translate to our design token system
- shadcn/ui components — use our Radix UI components instead
- Inline styles or hardcoded values
- Lovable's file structure — follow our project conventions
- Supabase integration code — use our backend/API layer

## Our Architecture Rules
- ALWAYS use Radix UI components — never raw HTML elements
- ALWAYS use our design tokens for colors, spacing, typography
- Components go in `src/components/` following our naming conventions
- State management: [your approach — e.g., Zustand, RTK Query, Context]
- All props must have explicit TypeScript interfaces

```

### 4.2 Optional but Recommended

```markdown
## Styling Translation Guide

| Lovable Uses | We Use Instead |
|-------------|----------------|
| `className="flex gap-4 p-6"` | Radix `<Flex gap="4" p="6">` |
| `className="text-lg font-bold"` | Radix `<Text size="4" weight="bold">` |
| `className="bg-blue-500"` | Token: `var(--color-primary)` |
| `className="rounded-lg"` | Token: `var(--radius-md)` |
| `<Button variant="outline">` (shadcn) | `<Button variant="outline">` (our Radix-based Button) |
| `<Card>` (shadcn) | Our `<Card>` component or Radix `<Box>` |
| `<Dialog>` (shadcn) | `<Modal>` (our compound component) |

## What NOT To Generate
- No Tailwind classes — use Radix props and our tokens
- No shadcn/ui imports
- No `any` types
- No inline `style={{}}` with hardcoded values
- No raw `<div>` for layout — use Radix `<Box>`, `<Flex>`, `<Grid>`
```

---

## 5. Workflow in Action

### Step-by-step Process

```
┌───────────┐   GitHub    ┌───────────┐   reads both   ┌──────────────────┐
│  Lovable  │ ──────────▶ │   Clone   │ ─────────────▶ │   Claude Code    │
│ Prototype │   sync      │  (local)  │                │                  │
└───────────┘             └───────────┘                │  translates to   │
                                                       │  your app arch   │
                          ┌───────────┐                │                  │
                          │ Your App  │ ◀───────────── │                  │
                          │ Codebase  │   generates    └──────────────────┘
                          └───────────┘
```

### 5.1 Point Claude Code at the Lovable Component

In Claude Code, reference the Lovable file using `@` to provide it as context:

```bash
cd ~/projects/your-react-app
claude
```

### 5.2 Example Prompts

**Implement a full page:**

```
Read @~/projects/lovable-prototype/src/pages/PricingPage.tsx
and all components it imports from @~/projects/lovable-prototype/src/components/

Implement the same page in our app at src/pages/Pricing/
following our CLAUDE.md rules. Match the visual layout and behavior
but use our component library and design tokens.
```

**Implement a single component:**

```
Read @~/projects/lovable-prototype/src/components/FeatureCard.tsx

Build the equivalent in our app at src/components/FeatureCard/
using our Radix UI components and design tokens.
Include all states visible in the Lovable code (hover, disabled, loading).
```

**Implement with screenshot for visual reference:**

```
[paste screenshot of Lovable prototype]

Here's the source code for this design:
@~/projects/lovable-prototype/src/pages/Dashboard.tsx

Build this in our app. The screenshot shows the target visual result.
The code shows the layout logic. Use our architecture.
```

### 5.3 What Claude Code Does

| Step | Action |
|------|--------|
| 1 | Reads Lovable component files — extracts layout structure, component hierarchy, responsive behavior, state logic |
| 2 | Reads your app's existing codebase — learns your patterns, tokens, component library |
| 3 | Reads `CLAUDE.md` — applies translation rules (Tailwind → tokens, shadcn → Radix, etc.) |
| 4 | Generates components in your app's architecture using your components |
| 5 | Generates TypeScript interfaces matching your conventions |
| 6 | Generates Storybook stories (if applicable) |

---

## 6. Example: Translating a Lovable Component

### Lovable Source (what your colleague built)

```tsx
// lovable-prototype/src/components/StatsCard.tsx
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { TrendingUp, TrendingDown } from "lucide-react";

export function StatsCard({ title, value, change, trend }) {
  return (
    <Card>
      <CardHeader className="flex flex-row items-center justify-between pb-2">
        <CardTitle className="text-sm font-medium text-muted-foreground">
          {title}
        </CardTitle>
        <Badge variant={trend === "up" ? "default" : "destructive"}>
          {trend === "up" ? <TrendingUp className="h-3 w-3" /> : <TrendingDown className="h-3 w-3" />}
          {change}
        </Badge>
      </CardHeader>
      <CardContent>
        <div className="text-2xl font-bold">{value}</div>
      </CardContent>
    </Card>
  );
}
```

### Your App Output (what Claude Code generates)

```tsx
// your-react-app/src/components/StatsCard/StatsCard.tsx
import { Box, Flex, Text } from "@radix-ui/themes";
import { Badge } from "../Badge";
import { TrendingUp, TrendingDown } from "lucide-react";
import type { StatsCardProps } from "./StatsCard.types";

export function StatsCard({ title, value, change, trend }: StatsCardProps) {
  return (
    <Box className="ds-stats-card" p="4">
      <Flex align="center" justify="between" mb="2">
        <Text size="2" weight="medium" color="gray">
          {title}
        </Text>
        <Badge color={trend === "up" ? "green" : "red"}>
          {trend === "up" ? <TrendingUp size={12} /> : <TrendingDown size={12} />}
          {change}
        </Badge>
      </Flex>
      <Text size="6" weight="bold">
        {value}
      </Text>
    </Box>
  );
}
```

```tsx
// your-react-app/src/components/StatsCard/StatsCard.types.ts
export interface StatsCardProps {
  title: string;
  value: string | number;
  change: string;
  trend: "up" | "down";
}
```

**What changed:** shadcn `Card` → Radix `Box`, Tailwind classes → Radix props, untyped props → explicit TypeScript interface, `className="text-2xl font-bold"` → `<Text size="6" weight="bold">`.

---

## 7. Tips & Gotchas

| Issue | Solution |
|-------|----------|
| Claude Code copies Lovable code verbatim instead of translating | Make `CLAUDE.md` rules explicit: "NEVER import from lovable-prototype, NEVER use Tailwind classes" |
| Lovable components use shadcn — your app uses Radix | Add the translation table (Section 4.2) to your `CLAUDE.md` so Claude maps components correctly |
| Layout doesn't look the same after translation | Paste a screenshot alongside the code reference — Claude Code uses both visual + code context |
| Lovable uses Supabase, your app has a different backend | Tell Claude Code to skip all data-fetching logic: "Only translate the UI layer, leave API calls as TODO comments" |
| Colleague updates the prototype after you started | `cd lovable-prototype && git pull` — then reference the updated files |
| Lovable code has oversized "god components" (500+ lines) | Ask Claude Code to decompose: "Split this into smaller components following our conventions before implementing" |
| Some Lovable components have no equivalent in your system | Claude Code should create new components first, then compose — add this rule to `CLAUDE.md` |

---