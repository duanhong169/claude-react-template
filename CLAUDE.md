# Project Instructions

> React + TypeScript + Tailwind CSS project template.

## Tech Stack

- **Framework**: React 18+ with TypeScript (strict mode)
- **Styling**: Tailwind CSS — no inline styles, no CSS modules unless justified
- **Build**: Vite
- **Linting**: ESLint + Prettier
- **Testing**: Vitest + React Testing Library
- **State management**: React context + hooks first; only add Zustand/Jotai if context gets painful
- **Routing**: React Router v6+
- **Markdown rendering**: `react-markdown` + `remark-gfm`
- **Code highlighting**: Shiki (preferred) — never render a code block without syntax highlighting

## File & Folder Structure

```
src/
├── components/       # Reusable UI components
│   └── Button/
│       ├── Button.tsx
│       ├── Button.test.tsx
│       └── index.ts
├── features/         # Feature modules (business logic + UI)
│   └── auth/
│       ├── components/
│       ├── hooks/
│       ├── utils/
│       └── index.ts
├── hooks/            # Shared custom hooks
├── utils/            # Pure utility functions
├── types/            # Shared TypeScript types/interfaces
├── lib/              # Third-party wrappers & configs
├── services/         # API layer
├── constants/        # App-wide constants
├── App.tsx
└── main.tsx
```

Rules:
- One component per file
- Colocate component + test + styles in a folder when the component is non-trivial
- Use barrel exports (`index.ts`) for public APIs of each folder
- Keep `components/` for truly reusable pieces; feature-specific components live in `features/`

## Naming Conventions

| Thing              | Convention           | Example                         |
|--------------------|----------------------|---------------------------------|
| Component files    | PascalCase           | `UserProfile.tsx`               |
| Component names    | PascalCase           | `export function UserProfile()` |
| Hook files         | camelCase            | `useAuth.ts`                    |
| Hook names         | `use` prefix         | `useAuth`, `useFetchData`       |
| Utility files      | camelCase            | `formatDate.ts`                 |
| Utility functions  | camelCase            | `formatDate()`, `parseToken()`  |
| Constants          | UPPER_SNAKE_CASE     | `MAX_RETRY_COUNT`               |
| Types/Interfaces   | PascalCase           | `UserProfile`, `ApiResponse`    |
| Type files         | camelCase            | `user.ts`, `api.ts`             |
| Enum values        | PascalCase           | `Status.Active`                 |
| Boolean vars       | `is/has/should`      | `isLoading`, `hasPermission`    |
| Event handlers     | `handle` prefix      | `handleClick`, `handleSubmit`   |
| Props callbacks    | `on` prefix          | `onClick`, `onSubmit`           |
| CSS classes        | Tailwind utilities   | `className="flex items-center"` |
| Test files         | `.test.tsx`          | `Button.test.tsx`               |

## TypeScript Patterns

- Enable `strict: true` — never turn it off
- Prefer `interface` for component props, `type` for unions/intersections
- Export prop interfaces: `export interface ButtonProps { ... }`
- Avoid `any` — use `unknown` and narrow. If truly unavoidable, add `// eslint-disable-next-line` with a comment explaining why
- Use discriminated unions for complex state:
  ```ts
  type State =
    | { status: 'idle' }
    | { status: 'loading' }
    | { status: 'success'; data: User }
    | { status: 'error'; error: string };
  ```
- Prefer `as const` over enums for simple cases
- Use generics when a function/component works with multiple types, not to show off

## React Component Structure

Every component should follow this order:

```tsx
// 1. Imports (see import order below)
import { useState } from 'react';
import { cn } from '@/lib/utils';
import type { ButtonProps } from './types';

// 2. Props interface (if not in separate types file)
export interface ButtonProps {
  variant?: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
  children: React.ReactNode;
  onClick?: () => void;
}

// 3. Component (named export, function declaration)
export function Button({ variant = 'primary', size = 'md', children, onClick }: ButtonProps) {
  // 3a. Hooks first
  const [isHovered, setIsHovered] = useState(false);

  // 3b. Derived state / computations
  const classes = cn(
    'rounded font-medium transition-colors',
    variant === 'primary' && 'bg-blue-600 text-white hover:bg-blue-700',
    variant === 'secondary' && 'bg-gray-200 text-gray-800 hover:bg-gray-300',
    size === 'sm' && 'px-3 py-1.5 text-sm',
    size === 'md' && 'px-4 py-2 text-base',
    size === 'lg' && 'px-6 py-3 text-lg',
  );

  // 3c. Event handlers
  const handleClick = () => {
    onClick?.();
  };

  // 3d. Return JSX
  return (
    <button className={classes} onClick={handleClick}>
      {children}
    </button>
  );
}
```

Rules:
- **Named exports only** — no `export default`
- Function declarations (`function Foo()`) not arrow functions for components
- Destructure props in the parameter list
- Hooks at the top of the function body, always
- Keep components < 150 lines. If longer, extract sub-components or hooks

## Import Order

Group and sort imports in this order (separated by blank lines):

```ts
// 1. React
import { useState, useEffect } from 'react';

// 2. Third-party libraries
import { clsx } from 'clsx';
import { z } from 'zod';

// 3. Internal aliases (@/ paths)
import { Button } from '@/components/Button';
import { useAuth } from '@/hooks/useAuth';
import { formatDate } from '@/utils/formatDate';

// 4. Relative imports
import { UserAvatar } from './UserAvatar';
import type { UserCardProps } from './types';

// 5. Type-only imports (at end of each group, using `import type`)
import type { User } from '@/types/user';
```

## Tailwind CSS Guidelines

- **No inline styles** — use Tailwind classes for everything
- **Use `cn()` utility** (clsx + tailwind-merge) for conditional classes:
  ```ts
  import { clsx, type ClassValue } from 'clsx';
  import { twMerge } from 'tailwind-merge';
  export function cn(...inputs: ClassValue[]) { return twMerge(clsx(inputs)); }
  ```
- **Responsive design**: mobile-first, use breakpoints `sm:`, `md:`, `lg:`, `xl:`
- **Design tokens**: define colors, spacing, fonts in `tailwind.config.ts` — don't use arbitrary values like `text-[#1a2b3c]` unless truly one-off
- **Dark mode**: use `dark:` variant, design for both modes
- **Avoid `@apply`** in most cases — prefer utility classes directly in JSX
- **Component patterns**:
  - Accept a `className` prop and merge it with `cn()` for flexible overrides
  - Use Tailwind's group/peer modifiers for interactive states

## Error Handling

- **API calls**: always wrap in try/catch, show user-friendly messages
- **React**: use Error Boundaries for component-level crashes
- **Forms**: validate with Zod schemas, show inline errors
- **Async**: handle loading, success, and error states explicitly — never assume success
- Pattern:
  ```ts
  try {
    const data = await fetchUser(id);
    setState({ status: 'success', data });
  } catch (error) {
    const message = error instanceof Error ? error.message : 'An unexpected error occurred';
    setState({ status: 'error', error: message });
  }
  ```

## Custom Hooks Patterns

```ts
// Good hook: single responsibility, clean return
export function useFetchData<T>(url: string) {
  const [state, setState] = useState<State<T>>({ status: 'idle' });

  useEffect(() => {
    const controller = new AbortController();
    // ... fetch logic with cleanup
    return () => controller.abort();
  }, [url]);

  return state;
}
```

- Every hook with side effects must clean up (AbortController, event listeners, timers)
- Return objects for 3+ values, tuples for 1-2 values
- Prefix with `use`, keep single responsibility

## Git Workflow

### Commit Messages (Conventional Commits)

```
feat: add user profile page
fix: resolve token refresh race condition
docs: update API integration guide
chore: upgrade tailwind to v4
refactor: extract auth logic into useAuth hook
test: add unit tests for formatDate utility
style: fix import ordering in components
```

- Subject line: imperative mood, lowercase, < 72 chars, no period
- Body (optional): explain *why*, not *what*

### Branch Naming

```
feat/user-profile
fix/token-refresh
chore/upgrade-deps
refactor/auth-hooks
```

## Comment Guidelines

- **Don't comment what** — the code should be self-explanatory
- **Do comment why** — explain non-obvious decisions, workarounds, or business rules
- **TODO format**: `// TODO: description (link to issue if exists)`
- **JSDoc**: use for exported utility functions and complex hook parameters
  ```ts
  /** Formats a date relative to now (e.g., "2 hours ago"). */
  export function formatRelativeDate(date: Date): string { ... }
  ```
- No commented-out code in commits — delete it, git has history

## Testing Guidelines

- Test behavior, not implementation
- Name tests descriptively: `it('shows error message when API call fails')`
- Arrange-Act-Assert pattern
- Mock API calls, not internal functions
- Aim for testing user-visible outcomes (what the user sees/interacts with)

---

# UI Design System — GitHub Style

> All UI must follow this design system. Do not deviate unless explicitly asked.

## Color Palette — GitHub Duo (Light + Dark)

Colors are defined as CSS variables in `src/index.css` and mapped to Tailwind custom theme.

### Light Mode (default, follows system)

| Token | Value | Usage |
|-------|-------|-------|
| `--color-primary` | `#0969DA` | Links, primary actions |
| `--color-success` | `#1A7F37` | Success state, primary submit buttons |
| `--color-danger` | `#CF222E` | Destructive actions, errors |
| `--color-warning` | `#9A6700` | Warning messages |
| `--color-bg` | `#F6F8FA` | Page background |
| `--color-surface` | `#FFFFFF` | Card / panel backgrounds |
| `--color-border` | `#D0D7DE` | Borders, dividers |
| `--color-text` | `#1F2328` | Primary text |
| `--color-text-muted` | `#656D76` | Secondary text |
| `--color-nav-bg` | `#24292F` | Top navigation bar |

### Dark Mode

| Token | Value | Usage |
|-------|-------|-------|
| `--color-primary` | `#58A6FF` | Links, primary actions |
| `--color-success` | `#3FB950` | Success state |
| `--color-danger` | `#F85149` | Destructive actions |
| `--color-warning` | `#D29922` | Warning messages |
| `--color-bg` | `#0D1117` | Page background |
| `--color-surface` | `#161B22` | Card / panel backgrounds |
| `--color-border` | `#30363D` | Borders |
| `--color-text` | `#E6EDF3` | Primary text |
| `--color-text-muted` | `#8B949E` | Secondary text |
| `--color-nav-bg` | `#161B22` | Top navigation bar |

**Implementation**: CSS variables on `:root` / `.dark`, `prefers-color-scheme` auto-detect + manual toggle override.

## Border Radius — 6px Standard

| Element | Radius | Tailwind |
|---------|--------|----------|
| Buttons | 6px | `rounded-md` |
| Cards / Panels | 6px | `rounded-md` |
| Inputs | 6px | `rounded-md` |
| Avatars | 50% | `rounded-full` |
| Badges / Tags | 9999px | `rounded-full` |
| Modals / Dialogs | 12px | `rounded-xl` |

## Spacing & Density — Compact

| Parameter | Value |
|-----------|-------|
| Base font size | 16px |
| Line height | 1.5 |
| Card padding | 16px (`p-4`) |
| Section gap | 16px (`gap-4`) |
| Page max-width | 1280px (`max-w-7xl`) |
| Table row height | 40px |
| List item spacing | 0 (separated by borders) |

## Layout — Top Nav

```
┌─────────────────────────────────────────┐
│ Logo  Tab1  Tab2  Tab3        [Avatar]  │ ← fixed, h-12, bg-[--color-nav-bg]
├─────────────────────────────────────────┤
│                                         │
│   ┌──── max-w-7xl mx-auto px-4 ────┐   │ ← bg-[--color-bg]
│   │                                 │   │
│   │     Content (bg-[--color-surface│   │
│   │     cards with border)          │   │
│   │                                 │   │
│   └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

- Top nav: `fixed top-0`, dark bg, height 48px
- Content: centered `max-w-7xl mx-auto`, `pt-14` offset for fixed nav
- Sub-layouts (settings sidebar etc.) added per-page inside content area

## Typography — System Font Stack

```
Sans:  -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Noto Sans', Helvetica, Arial, sans-serif
Mono:  ui-monospace, SFMono-Regular, 'SF Mono', Menlo, Consolas, monospace
```

## Animation — Minimal

- Button hover: `background-color` change, 80ms
- Button active: darken (instant)
- Focus ring: `box-shadow: 0 0 0 3px rgba(9,105,218,.3)` (instant)
- Link hover: color + underline (instant)
- **No** page transitions, scroll animations, transform/scale effects
- Feedback through **color change only**

## Button System — GitHub Border Gradient

| Variant | Background | Border | Text |
|---------|-----------|--------|------|
| Primary | `#1F883D` | `rgba(27,31,36,.15)` | white |
| Secondary | `var(--color-bg)` | `var(--color-border)` | `var(--color-text)` |
| Danger | `#CF222E` | `rgba(27,31,36,.15)` | white |
| Outline | transparent | `var(--color-border)` | `var(--color-text)` |
| Ghost | transparent | none | `var(--color-primary)` |

Common styles: `px-4 py-[5px]`, `text-sm font-medium`, `rounded-md`, `shadow-[inset_0_1px_0_rgba(255,255,255,.25)]`

## Prebuilt UI Components

Available in `src/components/ui/`:

- **Button** — GitHub-style with variants: primary, secondary, danger, outline, ghost. Use `<Button>` component, not raw `<button>`.
- **Skeleton** — Content loading placeholder with pulse animation
- **Toast** — Top-right notification, auto-dismiss 3s, supports success/error/warning
- **EmptyState** — Centered icon + description + action button for empty lists
- **CodeBlock** — Syntax-highlighted code block (see "Code Block Rendering" below). Use `<CodeBlock>` whenever you need to display code — never hand-roll `<pre><code>`.

---

# Code Block Rendering & Syntax Highlighting

> **Hard rule**: any time the UI displays code — Markdown output, AI chat responses, docs pages, README viewers, snippet cards, diff views, inline help — it **must** be syntax-highlighted from the very first render. Never ship a plain `<pre><code>` with raw text, "we'll add highlighting later" is not acceptable.

## When this rule applies

Trigger this setup on the **first** task that introduces any of:

- Rendering Markdown (`react-markdown`, MDX, a docs viewer, a README page)
- Showing LLM / AI / chat responses that may contain fenced code blocks
- Displaying user-submitted or server-returned source code (snippets, gists, logs with code)
- Any component named like `CodeBlock`, `Snippet`, `Pre`, `Highlight`, `MarkdownRenderer`, `MessageBubble`, `DocsContent`

If the task mentions any of the above, **install highlighting in the same change** as the renderer — not in a follow-up.

## Library choice

**Default: [Shiki](https://shiki.style/)** (uses VS Code TextMate grammars, ships accurate highlighting, supports dual themes).

```bash
npm i shiki
# If rendering markdown:
npm i react-markdown remark-gfm
```

Fallbacks (only if Shiki is explicitly undesirable, e.g. hard bundle-size constraint):

- `react-syntax-highlighter` (Prism-based, easier API, larger bundle)
- `highlight.js` + `rehype-highlight` (lightest, less accurate for edge languages)

Do **not** mix multiple highlighters in the same project.

## Required implementation

### 1. `CodeBlock` component — the single entry point

Put it at `src/components/ui/CodeBlock/CodeBlock.tsx`. Every code-rendering surface routes through it.

```tsx
// src/components/ui/CodeBlock/CodeBlock.tsx
import { useEffect, useState } from 'react';
import { codeToHtml } from 'shiki';
import { cn } from '@/lib/utils';

export interface CodeBlockProps {
  code: string;
  lang?: string;              // e.g. 'ts', 'tsx', 'bash', 'json'. Defaults to 'text'.
  className?: string;
  showCopy?: boolean;         // default true
}

export function CodeBlock({ code, lang = 'text', className, showCopy = true }: CodeBlockProps) {
  const [html, setHtml] = useState<string>('');

  useEffect(() => {
    let cancelled = false;
    codeToHtml(code, {
      lang: normalizeLang(lang),
      themes: { light: 'github-light', dark: 'github-dark' },
      defaultColor: false,    // emit both themes, toggle via CSS vars
    }).then((out) => {
      if (!cancelled) setHtml(out);
    });
    return () => {
      cancelled = true;
    };
  }, [code, lang]);

  return (
    <div className={cn('relative group rounded-md border border-[--color-border] overflow-hidden', className)}>
      {showCopy && <CopyButton text={code} />}
      {/* Shiki returns a <pre> with inline styles; we only add layout */}
      <div
        className="text-sm [&>pre]:!bg-transparent [&>pre]:p-4 [&>pre]:overflow-x-auto font-mono"
        // html is produced by Shiki from a known string; safe.
        dangerouslySetInnerHTML={{ __html: html || fallbackPre(code) }}
      />
    </div>
  );
}

function fallbackPre(code: string) {
  // Rendered only for the first paint before Shiki resolves.
  // Still uses <pre> + mono font so layout doesn't shift.
  const escaped = code
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;');
  return `<pre class="p-4 overflow-x-auto"><code>${escaped}</code></pre>`;
}

function normalizeLang(lang: string) {
  const map: Record<string, string> = {
    js: 'javascript', ts: 'typescript', jsx: 'javascript', tsx: 'tsx',
    sh: 'bash', shell: 'bash', zsh: 'bash',
    yml: 'yaml', md: 'markdown',
  };
  return map[lang.toLowerCase()] ?? lang.toLowerCase();
}
```

Rules baked into this component:

- **Dual-theme output** (`github-light` + `github-dark`), driven by the same `.dark` class the rest of the app uses. Never hardcode a single theme.
- **Language normalization** — accept common aliases (`ts`, `sh`, `yml`) without forcing callers to know Shiki's internal names.
- **Fallback `<pre>`** renders synchronously with the correct font/padding so there's no layout shift while Shiki loads.
- **Copy button** is on by default; users expect it.
- **Export a barrel**: `src/components/ui/CodeBlock/index.ts` → `export * from './CodeBlock';`.

### 2. Markdown integration — wire it up the first time you render Markdown

```tsx
// src/components/ui/Markdown/Markdown.tsx
import ReactMarkdown from 'react-markdown';
import remarkGfm from 'remark-gfm';
import { CodeBlock } from '@/components/ui/CodeBlock';

export function Markdown({ children }: { children: string }) {
  return (
    <ReactMarkdown
      remarkPlugins={[remarkGfm]}
      components={{
        code({ inline, className, children, ...props }) {
          const match = /language-(\w+)/.exec(className ?? '');
          const code = String(children).replace(/\n$/, '');
          if (inline) {
            return (
              <code className="px-1 py-0.5 rounded bg-[--color-bg] font-mono text-[0.9em]" {...props}>
                {children}
              </code>
            );
          }
          return <CodeBlock code={code} lang={match?.[1] ?? 'text'} />;
        },
      }}
    >
      {children}
    </ReactMarkdown>
  );
}
```

Every new Markdown-rendering feature (chat message, docs page, post body, changelog) **must** use `<Markdown>` — do not call `react-markdown` directly elsewhere, or the `code` override is easy to forget.

### 3. Streaming / AI chat responses

If code blocks stream in token-by-token:

- Render the partial text through the same `<CodeBlock>` — Shiki re-highlights on each `code` change; for long blocks debounce re-highlighting at ~60ms.
- Do **not** wait for the stream to finish before highlighting — half-highlighted is still better than none.

### 4. Styling contract

- `CodeBlock` respects the design tokens in "UI Design System" — border uses `--color-border`, surface blends into `--color-surface`.
- Font: `font-mono` (system mono stack from the typography section).
- Rounded: `rounded-md` (matches cards/buttons).
- No box shadow, no gradient — stay consistent with the GitHub-style flat look.

## Anti-patterns — reject these on sight

- Shipping `<pre><code>{code}</code></pre>` with a TODO comment.
- Rendering `react-markdown` without overriding the `code` component.
- Hardcoding a single theme (e.g. only dark) when the app has light/dark toggle.
- Importing Shiki's full bundle on the server for every request without caching — use `createHighlighter` with a scoped language list if SSR bundle size matters.
- Using `dangerouslySetInnerHTML` with **unsanitized** HTML from user input — only Shiki's output (from a known string) is safe. For user-provided Markdown, rely on `react-markdown`'s sanitization, never bypass it.

## Checklist before calling a code-rendering task "done"

- [ ] A `CodeBlock` (or equivalent) component exists and is the single entry point.
- [ ] Light **and** dark themes both render correctly.
- [ ] At least `ts`, `tsx`, `js`, `jsx`, `json`, `bash`, `css`, `html`, `md` highlight correctly.
- [ ] Unknown / missing language falls back to plain mono text **without crashing**.
- [ ] Copy button works and shows feedback.
- [ ] Markdown renderer routes fenced code through `CodeBlock`.
- [ ] No layout shift between first paint and highlighted paint.
