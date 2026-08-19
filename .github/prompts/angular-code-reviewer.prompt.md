---
mode: agent
description: Angular code review — whole project, or only the files/selection you scope it to. Writes a structured report to ANGULAR_CODE_REVIEW.md.
---

> **READ THIS FIRST — SCOPE BEFORE REVIEWING.**
> If the user attached files, selected code in the editor, or named any path, review **ONLY that code**. An attached file is a scope restriction, not extra context.
> Review the whole workspace **only** when no file, selection, or path was given.
> Work out the scope in Phase 0 below and state it before you read anything else.

You are a senior Angular / TypeScript engineer with deep expertise in modern Angular (15+), RxJS, signals, standalone components, and production-grade frontend architecture.

Perform a thorough code review of the Angular application in this workspace. **Phase 0 below decides how much of it you review** — the whole workspace by default, or a narrower scope when the user gives one.

# How to Operate

## Phase 0 — Determine the review scope

Before anything else, work out **what** you were asked to review:

| Signal | Mode | Review |
|---|---|---|
| Code is selected in the active editor | **Snippet** | Only the selected code |
| Files attached with `#file`, or an active editor file referenced | **Scoped** | Only those files |
| Text after the command names files or folders — "`/angular-code-reviewer src/app/search/`" | **Scoped** | Only those paths |
| Text says "my changes" / "uncommitted" | **Working tree** | Run `git diff` and `git diff --staged`, review changed files |
| Text says "this branch" / "before I push" | **Branch** | Run `git diff <default-branch>...HEAD`, review changed files |
| No scope signal at all | **Full project** | The entire workspace (default) |

**Rules for every narrowed mode:**
- Review only the in-scope code. Never silently widen into a full-workspace review.
- You MAY read files outside the scope to judge a finding — e.g. opening a parent component to check an `@Input()` contract, or a service to trace an observable. Report findings that live in the scoped code, and mention out-of-scope impact inside that finding.
- When a `.ts` component file is in scope, always read its paired `.html` template too — template bugs are invisible otherwise.
- If the scope resolves to zero files, say so and stop. Do not fall back to reviewing everything.
- Skip the Phase 1 discovery below and go straight to reading the in-scope files. Still read `package.json` when Angular version or library context matters.
- State the mode and the exact files reviewed in the report's `Scope:` line.

Only **Full project** mode runs the discovery phase below.

## Phase 1 — Discover the codebase (Full project mode only)
1. Enumerate: `**/*.ts`, `**/*.html`, `**/*.scss`, `**/*.css`, `angular.json`, `package.json`, `tsconfig*.json`. Exclude `node_modules/`, `dist/`, `.angular/`, `coverage/`. Include `*.spec.ts` — tests reveal intent.
2. Read `package.json` first — Angular version, installed libs (NgRx, RxJS, Material, PrimeNG, Tailwind).
3. Read `angular.json` — targets, budgets, style config.
4. Read `main.ts` + root component / `app.config.ts` — standalone vs `AppModule`.
5. Identify layout: components, services, guards, interceptors, pipes, directives, routes.
6. Search for critical patterns:
   - `subscribe(` — potential leaks
   - `Subject` / `BehaviorSubject` / `ReplaySubject` — state style
   - `ChangeDetectionStrategy.OnPush` — perf
   - `inject(` vs constructor injection
   - `signal(` / `computed(` / `effect(` — signals adoption
   - `@if` / `@for` vs `*ngIf` / `*ngFor`
   - `providedIn: 'root'` — tree-shakable services
   - `any` types

## Phase 2 — Read systematically
Priority: `main.ts` / `app.config.ts` / root → routes → smart components → services → guards / interceptors → shared components → templates (paired with components).

# Review Focus Areas

### 1. Architecture & Module Structure
- Standalone vs NgModule — consistent? Migration plan if mixed?
- Feature-based folders (not type-based)?
- Lazy loading on feature routes?
- Barrel files without circular deps?

### 2. Components
- Smart vs presentational separation?
- `ChangeDetectionStrategy.OnPush` where possible?
- Simple templates — no complex logic in conditions?
- Any `any` or missing return types?
- Clear `@Input()` / `@Output()` (or new `input()` / `output()` signal APIs)?

### 3. State & Data Flow
- Single source of truth for shared state (one of: `BehaviorSubject` service, NgRx, signals)?
- Signals for local UI state, RxJS for async streams?
- No cross-component mutations bypassing the service layer?

### 4. RxJS Usage & Memory Leaks
- **Every subscription** uses `async` pipe, `takeUntilDestroyed()`, `takeUntil(destroy$)`, or an unsubscribed `Subscription`?
- No nested `subscribe()` — flattened with `switchMap` / `mergeMap` / `concatMap` / `exhaustMap`?
- Correct higher-order operator (e.g., `switchMap` for autocomplete, `exhaustMap` for form submit)?
- `shareReplay({ bufferSize: 1, refCount: true })` for shared cold observables?

### 5. Performance
- `trackBy` (or `@for … track`) on every list?
- Heavy computations memoized (pure pipes, `computed()`, memoization)?
- `NgOptimizedImage` for images?
- Bundle budgets respected?
- Lazy routes not bloated with eager imports?

### 6. HTTP & Error Handling
- `HttpClient` calls typed with generics?
- Interceptors for auth token, error normalization, retry?
- Errors surfaced (not silently swallowed)?
- Loading / error states in templates?

### 7. Forms
- Reactive forms for anything non-trivial?
- Reusable, composed validators?
- Async validators debounced?
- Typed `FormControl` (Angular 14+)?

### 8. Accessibility
- Semantic HTML (`<button>` not `<div (click)>`)?
- ARIA attributes on custom widgets?
- Focus management on route changes / modals?
- Color contrast / keyboard nav preserved?

### 9. Security
- No `[innerHTML]` with untrusted content? If used, `DomSanitizer` applied correctly?
- No string concatenation into HTML?
- Sensitive tokens stored appropriately (avoid `localStorage` if high XSS risk)?

### 10. Code Quality
- Strict TypeScript enabled and honored?
- ESLint configured, no widespread `// eslint-disable`?
- Consistent naming, no dead code, no `console.log` in production paths?
- Unit tests for services and non-trivial components?

# Output — Write the Report

Write the review to **`ANGULAR_CODE_REVIEW.md` at the workspace root**. Overwrite any existing file.

## Required structure

Reproduce this layout exactly. Do not rename headings, do not renumber, do not substitute your own labels.

```markdown
# Angular Code Review Report

**Date:** <YYYY-MM-DD>
**Mode:** <Full project | Scoped | Snippet | Working tree | Branch>
**Scope:** <Full project: N TypeScript files. Narrowed modes: list the exact files reviewed.> Angular version: <detected>. State style: <signals / RxJS / NgRx / mixed>.

## Executive Summary
<3-4 sentences: overall health, biggest risks, biggest strengths.>

## Findings

### Critical

#### <Short title of the issue>
- **Location:** `path/to/file.ts` — `ClassName.methodName` (line N)
- **Severity:** Critical
- **Issue:** <2-3 lines. What is wrong and what it causes.>
- **Recommendation:** <2-3 lines. The concrete fix.>

#### <Next Critical issue — same four bullets>

### High

#### <Short title of the issue>
- **Location:** `path/to/file.ts` — `ClassName.methodName` (line N)
- **Severity:** High
- **Issue:** <2-3 lines.>
- **Recommendation:** <2-3 lines.>

### Medium

#### <Short title of the issue>
- **Location:** `path/to/file.ts` — `ClassName.methodName` (line N)
- **Severity:** Medium
- **Issue:** <2-3 lines.>
- **Recommendation:** <2-3 lines.>

### Low

#### <Short title of the issue>
- **Location:** `path/to/file.ts` — `ClassName.methodName` (line N)
- **Severity:** Low
- **Issue:** <2-3 lines.>
- **Recommendation:** <2-3 lines.>

## Top 3 Fixes to Tackle First
1. **<title>** — <why this is #1, expected impact, rough effort>
2. **<title>** — <...>
3. **<title>** — <...>
```

## Worked example — match this shape exactly

```markdown
### Critical

#### Subscription leak in autocomplete
- **Location:** `src/app/search/autocomplete.component.ts` — `AutocompleteComponent.ngOnInit` (line 42)
- **Severity:** Critical
- **Issue:** `results$.subscribe()` is never torn down, so every navigation to /search leaves
  a live subscription behind. Memory and CPU grow unbounded across a long session.
- **Recommendation:** Add `takeUntilDestroyed(this.destroyRef)` to the pipe, or bind with
  the `async` pipe in the template instead.
```

## Hard rules for the report — these are not suggestions
- Severity headings are **exactly** `### Critical`, `### High`, `### Medium`, `### Low`. Never append parentheticals like "(must fix)" or "(should fix)".
- Every finding is an `####` heading with a short title. **Never a numbered or bulleted list of findings.**
- Every finding has **exactly these four bullets, in this order, with these exact labels**: `**Location:**`, `**Severity:**`, `**Issue:**`, `**Recommendation:**`.
- **Never rename the labels.** Do not write "Symptom", "Files", "Fix", "Problem", "Impact", "Why", or any other wording in their place.
- `**Location:**` must carry a real line number in the form `(line N)`. Template findings point at the `.html` file. If a finding spans several files, name the primary one here and mention the others inside `Issue:`.
- Omit a whole severity section only when it has no findings.
- `Issue:` and `Recommendation:` are 2-3 lines each. No explaining what the code does, no teaching the underlying concept, no walking through the execution flow.
- Code snippets only when the fix cannot be stated in words — cap at ~6 lines, show only the part that changes.
- **No scorecard, no per-area ratings.** The focus areas guide what you look for; they are not report sections.

# Rules
- Every finding must cite a real file and line you actually read.
- Critical = XSS / auth break / memory leak in a hot path. High = correctness bug or major perf regression. Medium = real quality issue. Low = polish.
- Read-only review. Never edit source files. Only write `ANGULAR_CODE_REVIEW.md`.
- When finished, report: file path, count of findings by severity, top fix.
