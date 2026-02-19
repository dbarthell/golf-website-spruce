---
display:
  color: purple
  icon: flask
command: ./tests/e2e/run-feature-test.sh {{artifact_id}}
---
# Feature Test

Runs E2E tests **only for this feature**.

Tests are matched by the `@artifact` annotation in the test file header:

```typescript
/**
 * @artifact SPR-2gg5ne
 * @feature Git Integration
 */
describe('Git Integration', () => { ... })
```

Use this to quickly validate your feature without running the full regression suite.

**No tests found?** Tag your test file with `@artifact {{artifact_id}}` to link it to this feature.