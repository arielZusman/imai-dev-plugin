# Migration Patterns Reference

For plans involving syntax migrations or API changes, include before/after examples. This helps executors understand the transformation pattern without guessing.

## Angular Control Flow Migration

### @if (replaces *ngIf)

```markdown
### Migration Pattern: *ngIf to @if

**Before:**
```html
<div *ngIf="user">{{ user.name }}</div>
<div *ngIf="loading; else content">Loading...</div>
<ng-template #content>Content here</ng-template>
```

**After:**
```html
@if (user) {
  <div>{{ user.name }}</div>
}
@if (loading) {
  <div>Loading...</div>
} @else {
  <div>Content here</div>
}
```

**Notes:**
- Remove `<ng-template>` wrappers when using @else
- @if block requires curly braces, even for single elements
- Condition parentheses are required
```

### @for (replaces *ngFor)

```markdown
### Migration Pattern: *ngFor to @for

**Before:**
```html
<li *ngFor="let item of items; let i = index; trackBy: trackById">
  {{ i }}: {{ item.name }}
</li>
```

**After:**
```html
@for (item of items; track item.id; let i = $index) {
  <li>{{ i }}: {{ item.name }}</li>
}
```

**Notes:**
- `track` is mandatory (no default trackBy)
- Built-in variables: $index, $first, $last, $even, $odd, $count
- trackBy function replaced with `track <expression>`
```

### @switch (replaces ngSwitch)

```markdown
### Migration Pattern: ngSwitch to @switch

**Before:**
```html
<div [ngSwitch]="status">
  <span *ngSwitchCase="'active'">Active</span>
  <span *ngSwitchCase="'pending'">Pending</span>
  <span *ngSwitchDefault>Unknown</span>
</div>
```

**After:**
```html
@switch (status) {
  @case ('active') {
    <span>Active</span>
  }
  @case ('pending') {
    <span>Pending</span>
  }
  @default {
    <span>Unknown</span>
  }
}
```

**Notes:**
- No host element needed (no more div wrapper)
- Each @case requires curly braces
- String literals need quotes in @case
```

## RxJS Operator Changes

### Creation Operators

```markdown
### Migration Pattern: RxJS 6→7 Creation

**Before:**
```typescript
import { of, from } from 'rxjs';
import { pluck } from 'rxjs/operators';

this.http.get('/api').pipe(pluck('data'))
```

**After:**
```typescript
import { of, from, map } from 'rxjs';

this.http.get('/api').pipe(map(res => res.data))
```

**Notes:**
- `pluck` deprecated, use `map` with property access
- Many operators moved to rxjs root export in v7
```

### Subscription Patterns

```markdown
### Migration Pattern: takeUntil to takeUntilDestroyed

**Before:**
```typescript
private destroy$ = new Subject<void>();

ngOnInit() {
  this.data$.pipe(takeUntil(this.destroy$)).subscribe();
}

ngOnDestroy() {
  this.destroy$.next();
  this.destroy$.complete();
}
```

**After:**
```typescript
private destroyRef = inject(DestroyRef);

ngOnInit() {
  this.data$.pipe(takeUntilDestroyed(this.destroyRef)).subscribe();
}
```

**Notes:**
- Requires Angular 16+
- Import from `@angular/core/rxjs-interop`
- No manual cleanup needed
```

## TypeScript Version Upgrades

### Satisfies Operator (TS 4.9+)

```markdown
### Migration Pattern: Type Assertion to Satisfies

**Before:**
```typescript
const config = {
  endpoint: '/api',
  timeout: 5000
} as Config;  // loses literal types
```

**After:**
```typescript
const config = {
  endpoint: '/api',
  timeout: 5000
} satisfies Config;  // preserves literal types
```

**Notes:**
- Use `satisfies` when you want type checking without widening
- Still use `as` for actual type assertions
```

### Using Declarations (TS 5.2+)

```markdown
### Migration Pattern: try/finally to using

**Before:**
```typescript
const file = await openFile(path);
try {
  await file.write(data);
} finally {
  await file.close();
}
```

**After:**
```typescript
await using file = await openFile(path);
await file.write(data);
// automatically closed when scope exits
```

**Notes:**
- Resource must implement `Symbol.dispose` or `Symbol.asyncDispose`
- Use `using` for sync, `await using` for async disposal
```

## Usage in Plans

When enriching upgrade/migration plans:

1. **Identify patterns** - What syntax or APIs are changing?
2. **Find examples in codebase** - Search for usage of old patterns
3. **Document before/after** - Copy relevant patterns from this file
4. **Add codebase-specific notes** - Project conventions, edge cases

Include patterns directly in the plan's "Relevant Code Context" section or per-task as needed.
