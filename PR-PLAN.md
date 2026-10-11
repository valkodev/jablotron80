# Local branches and upstream PR plan

Origin: https://github.com/valkodev/jablotron80.git
Upstream: https://github.com/tahvane1/jablotron80.git
Base reviewed: d497cf17 (upstream version 0.48).

| Branch | Commit | Proposed upstream base |
| --- | --- | --- |
| codex/optional-detector-queries | d0113d5 | upstream main |
| codex/account-scoped-codes | 6027293 | upstream main |
| codex/encrypted-codes | 08aa1dc | upstream main after account-scoped codes merge |
| codex/deployment-0.48.3 | combined 0.48.4 deployment | deployment only, not an upstream PR |

The first two branches are independent children of the original main. The third
builds on the second because it encrypts the personal-code Store as well as the
original panel credentials. Open the first two upstream PRs separately. Wait for
account-scoped codes to merge before opening the encryption PR against upstream
main; otherwise that PR also contains the whole personal-code feature.
The deployment branch combines the three features and bumps the local version;
release/version changes are intentionally absent from the upstream feature PRs.
The optional-detector-queries and deployment-0.48.3 branches have been pushed
to the valkodev fork over SSH. Account-scoped-codes and encrypted-codes remain
local. No upstream GitHub PR has been opened by this task.

## Push when ready

```bash
git push -u origin codex/optional-detector-queries
git push -u origin codex/account-scoped-codes
git push -u origin codex/encrypted-codes
```

After the personal-code PR is merged, fetch upstream and inspect how it landed.
If the local base commit is not in upstream history (for example squash merge),
rebase only the encryption commit onto current upstream main:

```bash
git fetch upstream
git switch codex/encrypted-codes
git rebase --onto upstream/main codex/account-scoped-codes
```

If that branch was already published, rewriting it needs `--force-with-lease`.
Review upstream changes and resolve/test conflicts before pushing. The deployment
branch is a frozen candidate and should not be used as an upstream PR branch.

## PR 1 draft

Title: Allow disabling background detector activity queries

When a detector becomes active, background detail queries emulate keypad keys and
can cause keypad beeps/activity displays. Add an option, enabled by default, to
suppress those queries for alarm-only installations. Alarm status and A/AB/ABC
commands keep working; detector states may be incomplete or stale when disabled.
The manual Query button remains usable when automatic queries are disabled.
Documentation explains that inactive detector entities are not verified closed/clear
states and that manual queries can also cause keypad sounds.
Validation: 14 targeted tests for detector activity, cumulative zone states,
live option changes, manual button requests (including pending-query handling),
and explicit arm/disarm/detail commands; Black check.

## PR 2 draft

Title: Add account-scoped disarm codes with configurable common/personal mode

Add a common/personal code-mode setting and authenticated account-scoped PIN
storage. Common mode keeps the existing manual-PIN/master-PIN behavior by default.
Personal mode uses only the logged-in account's existing panel PIN, never a master
fallback. The optional card saves/replaces/removes the current account's code,
shows confirmation near Save, and offers native manual entry when no code is saved.
Stored-code disarm requests do not place PINs in service events or frontend state;
they serialize requests, reject stale/disarmed/service states, and disable serial
retries. PINs are plaintext in a private atomic HA Store in this PR; encryption is
separate and the documentation says so.
Validation: 41 backend/API tests and a Node frontend test for input clearing,
secret-free requests, delayed responses after account changes, and manual fallback.

## PR 3 draft

Title: Add optional encryption and migration of panel and account codes

Add authenticated Fernet encryption independently of code mode. Protect master
credentials, imported panel code1/code2 records, and all stored personal PINs.
Accept a configurable absolute key-file path with optional exclusive key creation;
the feature works without Docker-specific APIs. Validate old credentials before
migration, bind ciphertext to panel/account scope, reject missing/wrong keys, and
support changing key paths and explicitly turning encryption off. Raw bus logs
honor the existing sensitive-output flags. Document backup separation, memory
limits, interrupted migration, and downgrade recovery.
Validation: real encryption, scope binding, tampering, key permissions/generation,
rotation, migration and failed-write tests, options-flow tests, full pytest suite,
frontend test, JSON parsing and Black check. HA interfaces are stubbed in tests;
Version 0.48.3 was deployed with encryption and personal mode; user-operated
arming/disarming passed. Plaintext panel fields were confirmed absent from the
config entry. The private personal Store is not readable by the SSH account.

## Combined candidate checks

On `codex/deployment-0.48.3`: 152 pytest tests passed, Node card test passed,
all integration Python files passed Black at line length 110, JSON and syntax
validated. Tests require permission to start worker processes in this environment;
sandboxed runs blocked in unchanged upstream retry tests, while the full run
outside the sandbox passed. No production PIN or encryption key was accessed.
Hassfest has not been run locally; its GitHub workflow will run after publishing.
