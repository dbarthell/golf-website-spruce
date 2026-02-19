---
display:
  color: blue
  icon: flask
command: ARTIFACT_ID={{artifact_id}} ./tests/e2e/run-e2e-background.sh
---
# Regression Test

Runs **all** E2E tests to verify nothing is broken.

Use this before merging to main or after major changes to ensure all features still work.

**How it works:**
1. Copies source files to temp dir (~5 seconds)
2. Symlinks `node_modules` and `target` (no rebuild needed)
3. Launches `tauri dev` server in the temp dir
4. Runs all E2E tests
5. Sends notification when done

**Logs:** `/tmp/spruce-e2e/{artifact-id}/`