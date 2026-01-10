# Failure Modes Reference

Common failure patterns to include in enriched plans. Copy relevant patterns to each task's "Failure Modes" section.

## Build Failures

### TypeScript/JavaScript

```markdown
**Failure Modes:**
- **If "Cannot find module 'X'":** Missing import statement. Add `import { X } from './path'` at file top.
- **If "Property 'X' does not exist on type 'Y'":** Type mismatch after refactoring. Check interface definition matches usage.
- **If "Type 'X' is not assignable to type 'Y'":** Return type changed. Update function signature or cast appropriately.
- **If "Cannot use import statement outside a module":** Missing `"type": "module"` in package.json or wrong tsconfig target.
```

### Angular-Specific

```markdown
**Failure Modes:**
- **If "NullInjectorError: No provider for X":** Service not provided. Add to module's `providers` array or use `providedIn: 'root'`.
- **If "Can't bind to 'X' since it isn't a known property":** Missing module import. Import the module containing directive/component.
- **If "Expression has changed after it was checked":** Change detection issue. Use `ChangeDetectorRef.detectChanges()` or restructure async flow.
- **If template compilation fails:** Check for typos in property bindings, missing closing tags, or incorrect pipe syntax.
```

### NestJS-Specific

```markdown
**Failure Modes:**
- **If "Nest can't resolve dependencies":** Circular dependency or missing provider. Check module imports and use `forwardRef()` if needed.
- **If "Cannot read property 'X' of undefined" in service:** Service not properly injected. Verify constructor injection and module configuration.
- **If "EntityMetadataNotFoundError":** Entity not registered. Add entity to TypeORM `entities` array in module.
- **If validation pipe fails silently:** DTO class missing `class-validator` decorators or not using `ValidationPipe`.
```

## Test Failures

### Unit Tests

```markdown
**Failure Modes:**
- **If test times out:** Async operation not awaited. Add `await` or use `fakeAsync`/`done` callback.
- **If "Cannot spy on X":** Method doesn't exist on mock or is already spied. Check mock setup.
- **If mock not called:** Wrong import path or mock hoisting issue. Verify mock is set up before import.
- **If "X is not a function":** Mock returning wrong type. Check mock implementation returns callable.
```

### Integration Tests

```markdown
**Failure Modes:**
- **If database connection fails:** Test database not running or wrong connection string. Check docker-compose or test config.
- **If "relation X does not exist":** Migrations not run on test database. Run `npm run migration:run:test`.
- **If flaky pass/fail:** Race condition or shared state. Use test isolation, fresh database per test.
- **If HTTP 401 in tests:** Auth token expired or not set. Check test setup provides valid auth context.
```

### E2E Tests

```markdown
**Failure Modes:**
- **If element not found:** Selector changed or element not rendered. Check component renders, update selector.
- **If click doesn't trigger:** Element obscured or animation in progress. Add wait or scroll into view.
- **If assertion fails intermittently:** Timing issue. Add explicit waits for state changes.
- **If test passes locally but fails in CI:** Environment difference. Check headless mode, viewport size, network timing.
```

## Runtime Failures

### API/Backend

```markdown
**Failure Modes:**
- **If 500 error with no details:** Unhandled exception. Check logs, add try/catch with proper error handling.
- **If 404 on new endpoint:** Route not registered. Verify controller decorator and module imports.
- **If request body empty:** Missing `@Body()` decorator or content-type header mismatch.
- **If CORS error:** Backend CORS config missing origin. Add origin to allowed list.
```

### Database

```markdown
**Failure Modes:**
- **If "duplicate key" error:** Unique constraint violation. Check for existing record before insert.
- **If "foreign key constraint fails":** Referenced record doesn't exist. Create parent record first or use cascade.
- **If query returns empty:** Wrong WHERE clause or data not seeded. Verify query parameters and test data.
- **If connection pool exhausted:** Connections not released. Check for missing `await` on queries or unclosed transactions.
```

### Authentication

```markdown
**Failure Modes:**
- **If token validation fails:** Token expired, wrong secret, or clock skew. Check expiry and server time sync.
- **If refresh token rejected:** Token already used (rotation) or revoked. Issue new login flow.
- **If "invalid signature":** Secret mismatch between services. Verify all services use same AUTH_SECRET.
- **If user context undefined:** Middleware not applied or wrong order. Check route guards and middleware chain.
```

## Environment/Configuration

```markdown
**Failure Modes:**
- **If "X is undefined" for env var:** Variable not loaded. Check .env file exists and is loaded before access.
- **If works locally but fails in CI:** Environment variable not set in CI config. Add to GitHub secrets/variables.
- **If config validation fails:** New required field added. Update .env.example and CI config.
- **If feature flag not working:** Flag not enabled for environment. Check feature flag service/config.
```

## State Management

### Angular/RxJS

```markdown
**Failure Modes:**
- **If observable never emits:** Missing subscription or source not triggered. Check subscription lifecycle.
- **If "ObjectUnsubscribedError":** Subject completed before emission. Check subject lifecycle, use BehaviorSubject if late subscribers expected.
- **If memory leak suspected:** Subscription not unsubscribed. Add takeUntil pattern or async pipe.
- **If state stale after navigation:** Store not reset. Implement store reset on component destroy or route change.
```

### General

```markdown
**Failure Modes:**
- **If state update not reflected:** Mutating state directly instead of creating new reference. Use spread operator or immer.
- **If action dispatched but no effect:** Effect not registered or selector not matching. Check effect registration and action type.
- **If circular update detected:** State change triggering effect that changes same state. Break cycle with distinctUntilChanged or debounce.
```

## Usage in Plans

When enriching a plan, select 2-4 relevant failure modes per task based on:

1. **Task complexity** - More modes for 🟡/🔴 tasks
2. **Code area** - Pick modes matching the domain (API, UI, DB, etc.)
3. **Historical issues** - Include modes for problems you've seen in this codebase
4. **Non-obvious prerequisites** - Modes for things easy to forget

**Example for a NestJS service task:**
```markdown
**Failure Modes:**
- **If "Nest can't resolve dependencies":** Circular dependency. Use `forwardRef()` wrapper.
- **If tests fail with "Cannot spy on refreshToken":** Method is private. Make method protected or test via public interface.
- **If integration test times out:** Database transaction not committed. Check `await` on repository calls.
```
