# Qualys QScanner + Azure DevOps

Container image vulnerability scanning in Azure Pipelines with [QScanner](https://docs.qualys.com/en/qscanner/latest/).

No Qualys sensor or agent required — QScanner is a standalone binary. Works on Microsoft-hosted agents (`ubuntu-latest`) and self-hosted ones.

## Requirements

- Qualys subscription with Container Security.
- Access Token (Container Security → Configurations → Access Token).
- Linux amd64 agent with Docker and outbound internet access.

## Setup

1. Copy `azure-pipelines-qscanner.yml` to your repo root.

2. Adjust the variables:

   ```yaml
   QUALYS_POD: 'US2'               # your platform
   POLICY_TAGS: 'poc_image_policy' # your policy tag (gate mode only)
   ```

3. In the pipeline, add `QUALYS_ACCESS_TOKEN` as a **secret** variable.

> The token starts with `eyJ`. Paste it without quotes, without `Bearer `, and without the `QUALYS_ACCESS_TOKEN=` prefix, or it fails with `JWT token parsing failed`.

## Usage

Run the pipeline. The image to scan is picked at queue time (`image` parameter).

Output: results table in the log, plus JSON/SARIF reports in the `qualys-report` artifact.

## Modes

| Mode | Behavior |
|---|---|
| `--mode get-report` | Scan and report only. Never fails the build. **Default.** |
| `--mode evaluate-policy` | Evaluates the policy. Fails the build on violation (exit 42/43). |

To enable the gate, change the scan step to:

```yaml
  --mode evaluate-policy \
  --policy-tags "$(POLICY_TAGS)" \
```

## Quieting console output

| Flag | Effect |
|---|---|
| `--report-format json,sarif` | Drops the results table from the log (still in the artifact) |
| `--log-level error` | Errors only |
| `--quiet` | Fully silent |

Recommended: the first two. `--quiet` also hides authentication errors.

## Notes

- Don't comment out lines inside the QScanner command. The `\` line continuations join everything before bash parses `#`, so the comment swallows the rest of the command.
- The Access Token is platform-specific — it must match `--pod`.
- On self-hosted agents, the agent user needs to be in the `docker` group, and the agent service must be restarted afterwards.

## Files

| File | Description |
|---|---|
| `azure-pipelines-qscanner.yml` | Pipeline |
| `Dockerfile`, `package.json`, `app.js` | Test app with intentionally outdated dependencies |
