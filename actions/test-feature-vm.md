---
display:
  color: purple
  icon: server
command: ./scripts/test-vm.sh --feature {{artifact_id}}
---
# Feature Test (VM)

Runs E2E tests **only for this feature** headlessly in the Lima VM.

Tests are matched by the `@artifact` annotation in the test file header:

```typescript
/**
 * @artifact SPR-2gg5ne
 * @feature Git Integration
 */
describe('Git Integration', () => { ... })
```

Matching test suites run in **parallel** — each gets its own isolated environment via Linux network namespaces.

The VM is bootstrapped automatically on first run (~5 min) and starts in ~22s on subsequent runs.

**No tests found?** Tag your test file with `@artifact {{artifact_id}}` to link it to this feature.

**Logs:** `test-results/{suite-name}.log`
