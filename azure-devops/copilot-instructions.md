# Copilot code review instructions — Angular applications

Review modern Angular (15+) code. Report only issues that matter in production. Skip style, formatting, and naming unless they cause an actual bug — ESLint already covers those.

## Impact analysis

- When a component's `@Input()` / `@Output()` contract, a service method signature, or an observable's shape changes, search the repository for consumers and check each still works. Name the consumer's file and line in the comment.
- When a shared interface or model changes, check the components and services that read it.
- Always review a component's `.html` template alongside its `.ts` file — template bugs are invisible otherwise.

## Subscriptions and memory leaks

- Every `.subscribe()` must be torn down. Require the `async` pipe, `takeUntilDestroyed()`, `takeUntil(destroy$)`, or an explicit unsubscribe in `ngOnDestroy`. This is the highest-value check in an Angular review.
- Flag nested `subscribe()` inside `subscribe()` — flatten with `switchMap`, `mergeMap`, `concatMap`, or `exhaustMap`.
- Flag the wrong higher-order operator: `switchMap` for typeahead, `exhaustMap` for form submit, `concatMap` where order matters.

## Templates and performance

- Flag `@for` or `*ngFor` with no `track` / `trackBy` on lists that change.
- Flag presentational components without `ChangeDetectionStrategy.OnPush`.
- Flag function calls in template bindings — they re-run on every change detection cycle.
- Flag heavy computation not memoized via `computed()` or a pure pipe.

## State and data flow

- Flag shared state mutated directly from a component instead of through a service.
- Flag mixing signals and RxJS for the same piece of state without a clear reason.

## HTTP and error handling

- Flag `HttpClient` calls with no error handling, where a failure leaves the UI stuck loading.
- Flag missing loading and error states in templates that fetch data.
- Flag untyped `HttpClient` calls — use generics.

## Forms

- Flag template-driven forms used for non-trivial input where reactive forms belong.
- Flag async validators with no debounce.

## Security

- Flag `[innerHTML]` bound to anything not sanitized through `DomSanitizer`.
- Flag `bypassSecurityTrust*` calls on values derived from user input.
- Flag auth tokens in `localStorage` where the app has XSS exposure.

## Accessibility

- Flag `<div>` or `<span>` with `(click)` where a `<button>` belongs.
- Flag custom interactive widgets with no ARIA attributes or keyboard handling.

## Comment style

- One issue per comment, anchored to the line it affects.
- Say what breaks and give the concrete fix. Two or three sentences maximum.
- Start every comment with its severity in bold: **Critical**, **High**, **Medium**, or **Low**.
- Critical = security hole, memory leak on a hot path, or broken auth. High = correctness bug or major performance regression. Medium = real quality issue. Low = polish.
- Do not explain what the code does or teach the underlying concept.
