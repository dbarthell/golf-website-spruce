---
display:
  color: blue
  icon: server
command: ./scripts/test-vm.sh
---
# Regression Test (VM)

Runs **all** E2E tests headlessly in the Lima VM.

Test suites run in **parallel** — each gets its own isolated environment via Linux network namespaces. Use this before merging to main or after major changes to ensure all features still work.

The VM is bootstrapped automatically on first run (~5 min) and starts in ~22s on subsequent runs.

**How it works:**
1. Bootstraps VM if needed (installs Lima, creates VM, configures git + settings)
2. Syncs project to VM via rsync
3. Installs dependencies + builds Tauri binary
4. Runs all test suites in parallel (one namespace per suite)
5. Collects results summary

**Logs:** `test-results/{suite-name}.log`, `test-results/tauri-{suite-name}.log`
