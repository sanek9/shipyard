# CVE Fix Review: ${REPO} ${BRANCH}

You are reviewing CVE fix results for ${REPO} on ${BRANCH}.
The deterministic phase has already attempted to fix all CVEs.
All evidence has been pre-fetched below.

## Fix Results

${FIX_SUMMARY}

## Current Scan

${CURRENT_SCAN}

## Unfixed CVE Details

${LOCATE_OUTPUT}

## Build Environment

Go version in Shipyard build image: ${SHIPYARD_GO_VERSION}

## Task

Review ALL CVE outcomes:

1. **Verify fixes**: Do the committed fixes look correct? Any concerns?

2. **Handle unfixed CVEs**: For each CVE that was not fixed:
   - Try a different approach if possible (different version, drop replace directive)
   - If **no fix is available**: use `--no-fix` so the entry auto-expires when a fix is published.
     Run: `bash ${CVE_SCRIPTS}/ignore.sh ${STATE_FILE} PACKAGE SEVERITY "reason" --no-fix CVE_ID [CVE_ID...]`
   - If fix requires a **Go or K8s minor version upgrade** on a stable branch: report as **UNRESOLVED**.
     Do NOT ignore — this needs team approval.
     **Exception**: Go directive bumps that are already committed were validated by the deterministic phase
     against the build image (Go ${SHIPYARD_GO_VERSION}). Do NOT roll back or flag these.
   - If fix exists but would break API compatibility (not a version upgrade): omit `--no-fix` (permanent ignore).
     Run: `bash ${CVE_SCRIPTS}/ignore.sh ${STATE_FILE} PACKAGE SEVERITY "reason" CVE_ID [CVE_ID...]`
   - **One ignore.sh call per package.** Combine ALL CVEs regardless of severity.
     Use the highest severity for the SEVERITY arg (it is only a log label).

3. **Check for regressions**: Did any fix introduce new CVEs?

## Available Actions

ONLY use these scripts. Do NOT run go/git/sed commands directly.

- `bash ${CVE_SCRIPTS}/fix-package.sh ${STATE_FILE} PACKAGE VERSION CVE_IDS...`
- `bash ${CVE_SCRIPTS}/fix-stdlib.sh ${STATE_FILE} GO_VERSION CVE_IDS...`
- `bash ${CVE_SCRIPTS}/ignore.sh ${STATE_FILE} PACKAGE SEVERITY "reason" [--no-fix] CVE_ID [CVE_ID...]`
- `bash ${CVE_SCRIPTS}/scan.sh ${STATE_FILE}`

If a script exits with NEEDS_REVIEW, do NOT attempt the same fix manually.
If the reason indicates a policy decision (version upgrade), report as UNRESOLVED.
Otherwise use ignore.sh.

## Output

End with a summary:

```text
FIXED: N packages (list)
IGNORED: M packages (list with reasons)
UNRESOLVED: P packages (list — need team input)
```
