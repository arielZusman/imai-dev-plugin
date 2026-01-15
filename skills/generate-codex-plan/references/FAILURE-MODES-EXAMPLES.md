# Failure Modes Reference (Codex Plans)

Common failure patterns to include in enriched Codex plans. Copy relevant patterns to each task's "Failure Modes" section.

**Note:** All code fixes should be done using Codex CLI, not manually.

## Build Failures

### TypeScript/JavaScript

```markdown
**Failure Modes:**
- **If "Cannot find module 'X'":** Missing import statement. Use Codex to add `import { X } from './path'` at file top.
- **If "Property 'X' does not exist on type 'Y'":** Type mismatch after refactoring. Use Codex to check interface definition matches usage.
- **If "Type 'X' is not assignable to type 'Y'":** Return type changed. Use Codex to update function signature or cast appropriately.
- **If "Cannot use import statement outside a module":** Missing `"type": "module"` in package.json or wrong tsconfig target. Use Codex to update config.
- **Rollback:** `git checkout HEAD -- [files modified]`
```

### Angular-Specific

```markdown
**Failure Modes:**
- **If "NullInjectorError: No provider for X":** Service not provided. Use Codex to add to module's `providers` array or use `providedIn: 'root'`.
- **If "Can't bind to 'X' since it isn't a known property":** Missing module import. Use Codex to import the module containing directive/component.
- **If "Expression has changed after it was checked":** Change detection issue. Use Codex to add `ChangeDetectorRef.detectChanges()` or restructure async flow.
- **If template compilation fails:** Check for typos in property bindings, missing closing tags, or incorrect pipe syntax. Use Codex to fix.
- **Rollback:** `git checkout HEAD -- [files modified]`
```

### NestJS-Specific

```markdown
**Failure Modes:**
- **If "Nest can't resolve dependencies":** Circular dependency or missing provider. Use Codex to check module imports and add `forwardRef()` if needed.
- **If "Cannot read property 'X' of undefined" in service:** Service not properly injected. Use Codex to verify constructor injection and module configuration.
- **If "EntityMetadataNotFoundError":** Entity not registered. Use Codex to add entity to TypeORM `entities` array in module.
- **If validation pipe fails silently:** DTO class missing `class-validator` decorators or not using `ValidationPipe`. Use Codex to add decorators.
- **Rollback:** `git checkout HEAD -- [files modified]`
```

## Test Failures

### Unit Tests

```markdown
**Failure Modes:**
- **If test times out:** Async operation not awaited. Use Codex to add `await` or use `fakeAsync`/`done` callback.
- **If "Cannot spy on X":** Method doesn't exist on mock or is already spied. Use Codex to check mock setup.
- **If mock not called:** Wrong import path or mock hoisting issue. Use Codex to verify mock is set up before import.
- **If "X is not a function":** Mock returning wrong type. Use Codex to check mock implementation returns callable.
- **Rollback:** `git checkout HEAD -- [files modified]`
```

### Integration Tests

```markdown
**Failure Modes:**
- **If database connection fails:** Test database not running or wrong connection string. Check docker-compose or test config.
- **If "relation X does not exist":** Migrations not run on test database. Run `npm run migration:run:test`.
- **If flaky pass/fail:** Race condition or shared state. Use Codex to add test isolation, fresh database per test.
- **If HTTP 401 in tests:** Auth token expired or not set. Use Codex to check test setup provides valid auth context.
- **Rollback:** `git checkout HEAD -- [files modified]`
```

### E2E Tests

```markdown
**Failure Modes:**
- **If element not found:** Selector changed or element not rendered. Use Codex to check component renders, update selector.
- **If click doesn't trigger:** Element obscured or animation in progress. Use Codex to add wait or scroll into view.
- **If assertion fails intermittently:** Timing issue. Use Codex to add explicit waits for state changes.
- **If test passes locally but fails in CI:** Environment difference. Check headless mode, viewport size, network timing.
- **Rollback:** `git checkout HEAD -- [files modified]`
```

## Runtime Failures

### API/Backend

```markdown
**Failure Modes:**
- **If 500 error with no details:** Unhandled exception. Check logs, use Codex to add try/catch with proper error handling.
- **If 404 on new endpoint:** Route not registered. Use Codex to verify controller decorator and module imports.
- **If request body empty:** Missing `@Body()` decorator or content-type header mismatch. Use Codex to add decorator.
- **If CORS error:** Backend CORS config missing origin. Use Codex to add origin to allowed list.
- **Rollback:** `git checkout HEAD -- [files modified]`
```

### Frontend/UI

```markdown
**Failure Modes:**
- **If component doesn't render:** Check browser console for errors. Use Codex to fix import paths or missing dependencies.
- **If event handler not firing:** Wrong event name or binding syntax. Use Codex to verify event binding syntax.
- **If state not updating:** Immutability issue or change detection not triggered. Use Codex to ensure proper state updates.
- **If XSS warning:** Unsafe HTML or URL binding. Use Codex to add proper sanitization.
- **Rollback:** `git checkout HEAD -- [files modified]`
```

## Dependency Issues

### Package Installation

```markdown
**Failure Modes:**
- **If peer dependency conflict:** Check `npm info <pkg> peerDependencies` for required versions. Use Codex to update package.json.
- **If "ERESOLVE unable to resolve dependency tree":** Version conflict. Try `npm install --legacy-peer-deps` or resolve conflict manually with Codex.
- **If postinstall script fails:** Build dependencies missing or platform mismatch. Check error logs, install native build tools.
- **If "Cannot find module" after install:** Package not in dependencies. Use Codex to add to package.json.
- **Rollback:** `git checkout HEAD -- package.json package-lock.json && npm install`
```

## Git Issues

### Merge Conflicts

```markdown
**Failure Modes:**
- **If merge conflict on pull:** Use Codex to resolve conflicts, keeping both changes where appropriate.
- **If "Your local changes would be overwritten":** Commit or stash changes before pulling.
- **If conflict markers in code:** Use Codex to remove `<<<<<<<`, `=======`, `>>>>>>>` markers after resolving.
- **Rollback:** `git merge --abort` or `git rebase --abort`
```

## Usage in Plans

When enriching Codex plans:

1. **Identify likely failure modes** - Based on task type and changes
2. **Copy relevant patterns** - From sections above
3. **Add "Use Codex to fix"** - Emphasize that Codex handles code changes
4. **Include rollback command** - For each task
5. **Be specific** - Match failure modes to exact task operations

**Format for task files:**
```markdown
## Failure Modes

- **If [symptom]:** Likely cause is [X]. Use Codex to fix by [Y].
- **If [symptom]:** [Cause]. Use Codex to [specific fix].
- **Rollback:** `git checkout HEAD -- [files modified in this task]`
```
