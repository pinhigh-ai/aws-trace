# aws-trace

An **agent skill** that renders a full AWS transaction trace as an interactive HTML timeline
from a single X-Ray trace ID.

Give your agent a trace ID —

> trace 1-68b0c2f4-3a7e91d24c0f8a6b5e1d9f02

— and it gathers the X-Ray segments, pulls the CloudWatch logs for every hop, and produces
one self-contained HTML document in `./.traces/`, opened in your default browser.

## What the output looks like

Header, summary Gantt timeline, and chronological hop cards:

![Trace overview — Gantt timeline and first hop](docs/example-collapsed.png)

Every hop expands: HTTP requests/responses (headers + body), per-hop CloudWatch logs with
level coloring, DynamoDB items, message payloads, and audit records:

![Fully expanded trace with logs and DynamoDB detail](docs/example-expanded.png)

A fully rendered mock document is in [examples/example-trace.html](examples/example-trace.html)
— open it in a browser to try the interactivity.

## What it shows

- **Chronological hop flow** — e.g. API Gateway → Lambda → external HTTP → DynamoDB →
  SQS/SNS → downstream consumers, ordered by start time (async hops included).
- **Logs at each stage** — CloudWatch log lines attached to their hop by Lambda request ID,
  with timestamps and level coloring; unmatched events are kept in a trailing section.
- **Any AWS component** — hops are derived from the X-Ray segments themselves, so segment
  types beyond the common ones (Step Functions, EventBridge, S3, …) are rendered with the
  same card pattern rather than dropped ([rules/hop-types.md](rules/hop-types.md)).
- **Safety** — read-only AWS calls, all data HTML-escaped, credential-looking values redacted.

## Repository layout

```
SKILL.md                        the skill definition (entry point agents read)
templates/trace-template.html   canonical CSS/markup/JS the output must use
rules/hop-types.md              per-hop-kind content rules + extension rules
examples/example-trace.html     rendered gold standard
.traces/                        generated output (gitignored)
```

## Installation

Install with the [`skills` CLI](https://github.com/vercel-labs/skills) — no clone needed:

```sh
npx skills add pinhigh-ai/aws-trace
```

It installs the skill for your agent harness(es) — Claude Code, Cursor, Codex, opencode, and
other `.agents/skills`-style locations (which Warp reads too). Target a specific harness with
`-a`, e.g.:

```sh
npx skills add pinhigh-ai/aws-trace -a claude-code
```

Manual alternative: copy (or clone and symlink) this folder into your harness's skills
directory — e.g. `~/.claude/skills/aws-trace` (Claude Code) or `~/.warp/skills/aws-trace`
(Warp). Each skill is just a folder containing `SKILL.md`.

## Requirements

- **AWS CLI** configured with read access to X-Ray (`xray:BatchGetTraces`) and CloudWatch Logs
  (`logs:FilterLogEvents`, `logs:DescribeLogGroups`) in the account/region of the trace.
  The skill uses your default credential chain (`AWS_PROFILE`, SSO, env vars).
- X-Ray tracing enabled on the services involved.

## Usage

Ask your agent, in a directory where you want `.traces/` created:

- `trace 1-68b0c2f4-3a7e91d24c0f8a6b5e1d9f02`
- `show me the full trace for 1-… using profile staging`
- `trace 1-… and include the /aws/lambda/acme-notify-service log group`

Options are conversational — profile, region, log-search window, extra log groups, or
"don't open the browser" can all be stated in the request.
