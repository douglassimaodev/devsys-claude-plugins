# Version notes — Angular 22 (active until Jun 2027, LTS until Jun 2028)

Read after confirming `@angular/core` is `^22` in `package.json`. From Angular 22 on, majors ship once a year (next: Angular 23, around June 2027); new features arrive in minors (22.x).
These notes cover what models tend to get wrong because they default to older Angular. When unsure about an API's status (stable / developer preview / experimental), check angular.dev or the Angular CLI's MCP server instead of guessing, and add the finding here.

## Use this, not that

| Use | Not | Why |
|---|---|---|
| Standalone components without writing `standalone: true` | `standalone: true` boilerplate, NgModules | Standalone is the default |
| Built-in control flow `@if`, `@for (…; track …)`, `@switch`, `@defer` | `*ngIf`, `*ngFor`, `*ngSwitch`, `ngTemplateOutlet` tricks | Structural directives are deprecated; `@for` requires `track` |
| `input()`, `input.required()`, `output()`, `model()` | `@Input()`, `@Output() new EventEmitter()` | Signal-based component API |
| `viewChild()`, `viewChildren()`, `contentChild()` | `@ViewChild`, `@ContentChild` decorators | Signal queries |
| `computed()` for derived state; `linkedSignal()` for state reset by a source | `effect()` that writes to another signal | Effects are for side effects only |
| `inject()` | Constructor injection | Project convention |
| Signals + zoneless change detection | Adding `zone.js`, manual `ChangeDetectorRef.detectChanges()` workarounds | New apps are zoneless by default since v21 |
| Vitest (`ng test`) | Karma/Jasmine setup | Vitest is the default runner since v21 |
| `ApiService` + Reactive Forms (project convention) | Experimental APIs (e.g. signal forms, `httpResource`) without an ADR | Only adopt non-stable APIs after an explicit decision |
| The repository's existing file naming (`*.service.ts`, `*.model.ts`, pages) | The newer suffix-less `ng generate` naming | Consistency with the codebase wins |

## Checks
- Syncfusion `@syncfusion/ej2-angular-*` packages must be the release line that supports Angular 22.
- TypeScript must be **6.0.x** (`@angular/compiler-cli@22` requires `>=6.0 <6.1`). Don't let a dependency update pull TypeScript 7. TypeScript 6 also requires an explicit `rootDir` when the source root isn't obvious (error TS5011).
- Subscriptions that remain (router events, third-party observables) use `takeUntilDestroyed()`.

## Add findings here
When the senior reviewer or QA finds an Angular-22-specific correction that keeps coming back, add it to the table above.
