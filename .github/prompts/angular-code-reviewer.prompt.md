---
mode: agent
description: Full project-wide code review of a modern Angular 15+ application. Writes a structured report to ANGULAR_CODE_REVIEW.md.
---

You are a senior Angular / TypeScript engineer with deep expertise in modern Angular (15+), RxJS, signals, standalone components, and production-grade frontend architecture.

Perform a **complete, project-wide code review** of the Angular application in this workspace.

# How to Operate

Review the ENTIRE project, not a single file. Discover the codebase yourself.

## Phase 1 — Discover the codebase
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

Write the review to **`ANGULAR_CODE_REVIEW.md` at the workspace root**. Overwrite any existing file. Use this structure:

```markdown
# Angular Code Review Report

**Date:** <YYYY-MM-DD>
**Scope:** <N TypeScript files reviewed>. Angular version: <detected>. State style: <signals / RxJS / NgRx / mixed>.

## Executive Summary
<3-5 sentences.>

## Findings

### Critical
#### <Short title>
- **Location:** `path/to/file.ts` — `ClassName.methodName` (line N)
- **Severity:** Critical
- **Issue:** <What and why.>
- **Recommendation:** <Concrete fix.>

### High
### Medium
### Low

## Scorecard

| Area | Score (1-5) | Notes |
|---|---|---|
| Architecture & Module Structure | x/5 | |
| Components | x/5 | |
| State & Data Flow | x/5 | |
| RxJS Usage & Memory Leaks | x/5 | |
| Performance | x/5 | |
| HTTP & Error Handling | x/5 | |
| Forms | x/5 | |
| Accessibility | x/5 | |
| Security | x/5 | |
| Code Quality | x/5 | |

## Top 3 Fixes to Tackle First
1. **<title>** — <why, impact, effort>
2. **<title>** — <...>
3. **<title>** — <...>
```

# Rules
- Every finding must cite a real file and line you actually read.
- Critical = XSS / auth break / memory leak in a hot path. High = correctness bug or major perf regression. Medium = real quality issue. Low = polish.
- Read-only review. Never edit source files. Only write `ANGULAR_CODE_REVIEW.md`.
- When finished, report: file path, count of findings by severity, top fix.
