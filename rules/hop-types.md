# Hop types

One hop = one chronological stage of the transaction. Every hop renders as the template's
hop-card pattern: timeline dot (emoji icon) + card with clickable header (title, badge,
subtitle, start → end, duration) + expandable sections.

## Known kinds

| Kind | Icon | Gantt color | Title pattern | Sections (in order) |
|------|------|-------------|---------------|---------------------|
| `apigw` | 🌐 | `var(--http)` | `API Gateway` + method/path pill + status badge | HTTP Request (open) · HTTP Response |
| `lambda` | λ | `var(--lambda)` | `Lambda` + function-name pill | Logs (open) · anything segment metadata adds (cold start, memory, billed ms in subtitle) |
| `http` | ⇄ | `var(--http)` | `HTTP (external)` + method/host pill + status badge | HTTP Request · HTTP Response |
| `dynamodb` | 🗄️ | `var(--dynamo)` | `DynamoDB` + operation/table pill | Item / key detail (from logs if X-Ray lacks payloads) · consumed capacity in subtitle |
| `sqs` | 📨 | `var(--sqs)` | `SQS` + operation/queue pill | Message body (from logs when available) |
| `sns` | 📣 | `var(--sqs)` | `SNS` + topic pill | Message detail |

## Section content rules

- **HTTP Request:** key/value table (method/path, source IP, user-agent, notable headers —
  redacted per SKILL.md) then a `<pre>` body when one is known from logs.
- **HTTP Response:** status + headers table, `<pre>` body when known. Show latency in the
  section title, e.g. `HTTP Response — 200 (92 ms)`.
- **Logs:** template log-line grid — `SS.mmm` timestamp, colored level (INFO blue, WARN amber,
  ERROR red, DEBUG muted), message. Structured JSON log lines: show `msg` first, then the
  remaining fields compactly. Open this section by default when the hop's story is in its logs.
- **JSON payloads** (envelopes, DynamoDB items): pretty-printed 2-space `<pre>`.

## Badge / status mapping

- ok (green): success, 2xx/3xx
- warn (amber): 4xx, throttled, retries, WARN-level log presence worth surfacing
- error (red): segment `error`/`fault`, 5xx, dead-lettered — also add `err-card` to the card

## Unknown components — extension rule

X-Ray will surface services this table doesn't list (Step Functions, EventBridge, S3, Secrets
Manager, Kinesis, Bedrock, …). Never drop them:

1. Create a hop with the same card pattern.
2. Pick one fitting emoji icon; keep it stable within the document.
3. Reuse the closest existing palette variable (storage → `--dynamo`, messaging → `--sqs`,
   compute → `--lambda`, network/API → `--http`); do not invent new colors.
4. Subtitle = operation + resource name from the segment's `aws` block.
5. Render the segment's `aws`/`http`/`sql` metadata as a key/value section, payloads from logs
   as `<pre>` sections, and attach any correlated log lines as a Logs section.
