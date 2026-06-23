# gh-demo-action

A demo GitHub Action for testing the Origin supply-chain-security platform.

## What it does

1. Logs `Hello, <name>!` (default name: `world`).
2. **Unconditionally** writes credential-shaped strings to the job log so the Origin credential scanner has something to detect.

The credential-leak behavior is tied to the action SHA: pre-leak SHAs run cleanly; this SHA and any newer ones emit the leak. This lets a demo distinguish "clean SHA → job allowed" from "leaky SHA → post-job scanner findings → job blocked" without runtime configuration.

## Usage

### Clean SHA (pre-leak)

Reference an earlier SHA (before the credential-leak step was added) to demo the happy path:

```yaml
jobs:
  demo:
    runs-on: self-hosted-with-origin
    steps:
      - uses: resilientscale/gh-demo-action@<pre-leak-sha>
        with:
          name: 'GRunS'
```

### Leaky SHA (this SHA and newer)

Reference the current SHA to demo the credential-scanner path:

```yaml
jobs:
  demo-leak:
    runs-on: self-hosted-with-origin
    steps:
      - uses: resilientscale/gh-demo-action@<leak-sha>
```

The leak step emits randomly-generated values matching common secret patterns: GitHub PAT (`ghp_…`), AWS access key ID (`AKIA…`), AWS secret access key, Slack bot token (`xoxb-…`), Stripe live key (`sk_live_…`). They are not real credentials.

## Inputs

| Name | Required | Default | Description |
|---|---|---|---|
| `name` | no | `world` | Name to greet in the hello-world step |

## Safety

This action is intended for demo and testing of Origin only.

- The credential-leak step deliberately writes secret-shaped strings to logs.
- Do NOT use any SHA of this action in production workflows.
- The repository should NOT be granted access to real secrets.

## How this maps to the Origin demo thread

See [`gh-runner-sentinel/setup/demo-thread.md`](../gh-runner-sentinel/setup/demo-thread.md) — the three scenarios this action exercises:

1. **Happy path** — workflow uses a clean (pre-leak) SHA of this action; agent reports the action; GRunS allows it; job runs to completion.
2. **Immutable-action enforcement** — workflow references this action by tag (e.g. `@v1`) instead of a SHA, with `GRUNS_REQUIRE_IMMUTABLE_ACTIONS=true` on the agent; agent submits the mutable ref and GRunS blocks the job with reason "mutable action reference".
3. **Post-run credential audit** — workflow uses the leak SHA of this action with `GRUNS_SCAN_LOGS_FOR_CREDENTIALS=true` + `GRUNS_FAIL_JOB_ON_LEAK=true` on the agent; agent runs the post-job credential scanner, finds the synthesized secrets, reports findings, fails the job.
