# VimCrew Review Gate

A cross-vendor review quorum for pull requests. Two models from **different vendors** (Claude, GPT,
GLM, Kimi, MiniMax, DeepSeek, Qwen) independently review the PR diff; the aggregated verdict is the
job's exit code, so the gate works as a required status check.

- **Bring your own keys.** Requests go from your runner straight to the providers you configure,
  billed to your accounts. VimCrew has no keys, no billing and no server in the loop.
- **Independent opinions.** Two reviewers from the same vendor are a config error: approval
  requires two different model families.
- **Few false blocks.** A merge is held only by a finding the reviewer explicitly marked
  `blocking` in a blocking category (security, correctness, bug, regression, tests, …), with an
  adjustable threshold.

## Quick start (2 steps)

1. **Add API keys of two vendors as repository secrets** (*Settings → Secrets and variables →
   Actions*), e.g. `ANTHROPIC_API_KEY` and `OPENAI_API_KEY`.
2. **Add the workflow** `.github/workflows/review-gate.yml`
   (full copy: [`examples/review-gate.yml`](examples/review-gate.yml)):

   ```yaml
   name: Review Gate
   on:
     pull_request:
       types: [opened, synchronize, reopened, ready_for_review, labeled, unlabeled]
   permissions:
     contents: read
     pull-requests: write
   jobs:
     review:
       if: ${{ !github.event.pull_request.draft }}
       runs-on: ubuntu-latest
       timeout-minutes: 30
       steps:
         - uses: actions/checkout@v4
           with:
             ref: ${{ github.event.pull_request.base.sha }}   # config from the BASE commit
             persist-credentials: false
         - uses: vimcrew/review-gate@v1   # pin a full commit SHA in production
           with:
             anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
             openai-api-key: ${{ secrets.OPENAI_API_KEY }}
   ```

That's it — open a PR. The gate posts a summary comment, inline comments for blocking findings with a
line, and a job summary.

**The config is optional.** Without `.vimcrew/review-gate.yml` (it is read from the base commit, so
the PR that adds the workflow is reviewed too) the gate uses defaults: `comment` mode, one reviewer
per vendor whose key you passed, with the vendor's default model, and a quorum of 2 (two
different vendors must approve; an explicit `quorum` above 4 or above the number of reviewers is a
config error, exit `3`). The
summary says so explicitly ("Config not found — using defaults") and
lists the models. With a key of only one vendor the gate does not review and tells you a second
vendor is needed (exit `0` in `comment` mode). A config that exists but is invalid is still exit `3`.
After a week in `comment` mode add a config with `mode: block` and make the job a required check.

| vendor | default model |
|---|---|
| `anthropic` | `claude-opus-5` |
| `openai` | `gpt-5.6-sol` |
| `zai` | `glm-5.3` |
| `moonshot` | `k3` |
| `minimax` | `MiniMax-M3` |
| `deepseek` | `deepseek-v4-pro` |
| `qwen` | `qwen3.8-max` |

Use `pull_request`, **not** `pull_request_target`. The gate never needs the PR's code: the diff is
read through the GitHub API, and the checkout above is of the base commit, only for the config.

### Permissions

| Permission | Why |
|---|---|
| `contents: read` | checkout of the base commit (config file) |
| `pull-requests: write` | summary comment and inline findings |

The job runs on a Linux runner with Docker (`ubuntu-latest` has it): the action pulls the gate
image by an immutable digest and runs it with the workspace mounted read-only.

## Inputs

| Input | Default | Meaning |
|---|---|---|
| `mode` | from config | `block` or `comment` |
| `language` | `en` | `en` or `ru`; config takes priority, input is a fallback |
| `config` | `.vimcrew/review-gate.yml` | config path relative to the workspace |
| `anthropic-api-key` | — | `ANTHROPIC_API_KEY` |
| `openai-api-key` | — | `OPENAI_API_KEY` |
| `zai-api-key` | — | `ZAI_API_KEY` (GLM) |
| `moonshot-api-key` | — | `MOONSHOT_API_KEY` (Kimi) |
| `minimax-api-key` | — | `MINIMAX_API_KEY` |
| `deepseek-api-key` | — | `DEEPSEEK_API_KEY` |
| `dashscope-api-key` | — | `DASHSCOPE_API_KEY` (Qwen) |
| `github-token` | `github.token` | reads the PR and posts comments |
| `rules-token` | `github-token` | contents:read access to external rules repositories |

| Output | Meaning |
|---|---|
| `report` | path to `review-gate-report.json` (run metrics, schema 1) in `RUNNER_TEMP`; save it with `actions/upload-artifact` |

Keys enter the container only as these environment variables. A custom `api_key_env` in the config
is not supported by the action.

## Output language

English is the default for summaries (PR and `GITHUB_STEP_SUMMARY`), reviewer decisions, reasons,
warnings, inline comments and costs. Set `language: ru` in `.vimcrew/review-gate.yml` for Russian.
Allowed values are `en` and `ru`; other values are config errors (exit `3`). The trusted base config
wins over the optional Action `language` input / CLI `--language en|ru`; the input is a fallback
when the config omits `language` or is absent. This keeps repository policy consistent across callers.
For local `--diff`, the local config has the same priority. Without a readable base, only the input
or the English default is available. Invalid YAML uses that fallback for its error message.

Models are instructed to write findings in the output language. JSON report keys and wire values
(status, decision, outcome, error kinds) stay unchanged; gate-generated reason text follows the output
language. Provider diagnostics and developer logs retain their original text. Publication keys
(`vimcrew:pub:…`), JSON parsing and the `REDACTED` redaction marker are language-independent; the
fixed author-marker replacement is also unchanged. Prompts are maintained separately from the UI
message catalog, which provides English and Russian translations.

## Modes

- **`block`** — the exit code is the verdict: a blocking finding fails the check and holds the
  merge (once the check is required).
- **`comment`** — the gate reviews and comments but always exits `0`, except for a config error
  (`3`). Recommended for the first week to calibrate the threshold.

## Exit codes

| Code | Meaning |
|---|---|
| `0` | approved (or `comment` mode) |
| `1` | blocking findings |
| `2` | no quorum: a reviewer failed (401/403/404, read timeout, unreadable, contradictory or truncated answer, cost-cap abstention), too few valid votes left after unavailable reviewers, the diff was truncated (`max_diff_bytes`), or the review was skipped (fork PR without secrets, a key of only one vendor in `block` mode) |
| `3` | config or input error |

A partial review never approves: a truncated diff or a truncated answer gives `2`, because the part
nobody read could hold a blocking issue.

## Unavailable reviewers

A provider failure that does not depend on the PR content marks the reviewer as *unavailable*. This is
an allowlist of structured "account exhausted" signals per vendor — Z.ai code `1113`, Kimi's plan
usage limit ("You've reached your … usage limit"), MiniMax `insufficient_balance_error` / `1008`,
OpenAI `insufficient_quota` — plus network and 5xx/529 errors after the retries. The gate then decides on the remaining votes if they still reach `quorum`, come from at
least two vendors, and `quorum` is below the number of reviewers (with `quorum` equal to it, every
reviewer is required and any failure gives `2`). Unavailable reviewers are listed in the summary and
in the metrics record (`reviewers[].status: unavailable`). Failures a PR author could trigger with
the diff — a truncated or unreadable answer, a read timeout, a cost-cap abstention, any other 429
(TPM/RPM, "request too large", Qwen `Throttling.AllocationQuota`, OpenAI `rate_limit_exceeded`) — and
401/403/404 still give `2`. A 504 (the gateway did not wait for a long answer, which a heavy diff
can cause) and a 408 are not "unavailable" either. Residual risk: 500/502/503/529 after retries do
count as unavailable, so an input that deterministically crashes one provider would remove its vote;
require every reviewer (`quorum` equal to the number of reviewers) if that is unacceptable.

## Config is read from base

For PRs and Actions events (including `--diff`), the entire config is read from base SHA git
objects, regardless of checkout. Its path must be inside the repository. A missing config in an
available base uses defaults. An unavailable base/config blob produces a warning without using
the checkout: exit `2` in `block`, or when the mode is still unknown. Explicit `--mode comment`
uses defaults and reviews without organization sources.

## Rules from another repository

Keep product, team or organization rules in a separate GitHub repository, without copying them
into each project. Add sources to the config **from the base commit**:

```yaml
rules_sources:
  - repo: acme/engineering-rules
    ref: v1.2.0                 # prefer a full commit SHA or tag over a moving branch
    paths: [AGENTS.md, "product/*.md"]
rules: [AGENTS.md, ".vimcrew/review-rules.md"]
max_rules_bytes: 24000          # shared external + local budget, including names and headers
```

The loader uses `gh api`: it resolves the ref once, then reads the tree and contents at that
full SHA. A moving branch produces a summary warning. Sources and paths follow config order;
glob matches are sorted by filename. Organization rules come first, then project rules from
base, which can refine the general requirements. `rules: []` disables only local rules.
Truncation is visible; contents of subsequent files are not fetched after the shared budget runs out.
The summary and local JSON report identify each external file as `repo@resolved-SHA:path`.
Contents and paths pass through `redactSecrets`. The JSON report stays on your runner; it is not
uploaded to VimCrew.

The default is `GITHUB_TOKEN` (CLI also accepts `GH_TOKEN`). For a private external repository,
pass a separate token with **contents:read** and access to that repository:

```yaml
with:
  rules-token: ${{ secrets.RULES_READ_TOKEN }}
```

The CLI equivalent is `RULES_TOKEN`. It is passed through the environment, never logged, and
does not change the PR publication token. API errors, 403/404, incomplete trees and unmatched
paths/globs cause `block` mode to exit **2** with “organization rules not loaded”, before
calling models. In `comment` mode review continues without external rules, with a prominent
warning in the log, summary and JSON report. Partial external results are discarded.

Planned: rules served by an MCP server. No MCP configuration key is available yet.

## Project rules

Reviewers also see your project's rules, read from the **base commit**: by default `AGENTS.md`,
`CLAUDE.md`, `.github/copilot-instructions.md` and `.vimcrew/review-rules.md` (whichever exist).
Set your own list with `rules:` in the config (paths or globs from the repository root, e.g.
`rules: [AGENTS.md, "docs/rules/*.md"]`); `rules: []` turns it off. PR rules come from git objects at
the base SHA, regardless of the checkout. If the base objects are unavailable, no rules are loaded;
the log and summary warn explicitly. Losing local rules after loading the trusted config keeps the
review verdict; losing the config itself follows the policy above. A change to `AGENTS.md` in the PR stays untrusted diff data. Rules go into the
prompt as a separate trusted section before the PR data, within a byte budget (`max_rules_bytes`,
default 24000, including file headings and truncation markers; the fixed section instruction is
excluded). Overflow is truncated and marked; subsequent files are not read. File names (capped at
240 bytes) and contents pass through the secret redactor. Local `--diff` alone supports `--rules-root`
and labels the directory source explicitly. The summary lists the files considered and truncation. Rules add to every
reviewer's input, and so to the cost.

## Lifting a block: the `review-gate:override` label

Add the label `review-gate:override` to a PR to lift the block (a false block, an urgent hotfix, a fork
PR without secrets). The gate checks who added it — the last `labeled`/`unlabeled` event of that label
(removed last means no override) and the user's permission in the repository, and checks both again
right before exiting `0`. A human with write, maintain or admin access gets exit `0`; the
models' verdict is still computed and published, and the summary names who lifted the block. A label
added by a bot or by a user without write access (including the PR author) is ignored with an
explanation. The current head is always reviewed (and paid for, like a manual re-run): a summary
marker in the comments is not trusted, since anyone can type one. Removing the label restores the block on the next run. Do not filter the label name in the job's
`if:`: a skipped job counts as success for a required check.

## Retries

A reviewer call is retried up to 2 times (2 s and 5 s pauses) only on a failure provably before the
response — connect timeout or refusal, DNS, a connection reset before any status — and on a received
5xx/529 status. A connection dropped after a 200 or 4xx status is not retried.
A read timeout (the model may have worked for minutes), 400/401/403/404, 429 and an exhausted balance
are not retried. Each attempt reserves its own share of `max_usd_per_pr`: no room — no retry.

## Config `.vimcrew/review-gate.yml`

```yaml
mode: block            # block | comment
quorum: 2
max_diff_bytes: 200000
timeout_minutes: 10
# Blocking threshold (both keys optional):
# block_on:
#   severity: blocking
#   categories: [security, correctness, bug, regression]   # must include security
# min_reviewers_flagging: 1                                  # 1 | 2 (2 = both must flag)
reviewers:
  - id: claude
    vendor: anthropic
    model: claude-opus-5-5
    max_tokens: 16000
  - id: gpt
    vendor: openai
    model: gpt-6.1-sol
    max_tokens: 32000
    # protocol: openai          # default: the vendor's protocol
    # base_url: https://…       # default: the vendor's endpoint
```

An unsafe threshold is a config error (`3`), not a silent approve: empty `categories`, an unknown
category, `categories` without `security`, `min_reviewers_flagging` other than 1 or 2.

## Rule templates

Choose a base, a corporate or individual profile, and a stack from the
[ready-to-use rules library](rules/README.md). It includes 13 stacks and four
gate configs with local `rules:` paths. Project-rules support in the engine is
required; the library explains base-commit loading and the 24000-byte budget.

## Vendors

`vendor` is who trained the model (quorum independence is checked on it); `protocol` is how the
gate talks to it.

| vendor (aliases) | protocol | default endpoint | key (env) |
|---|---|---|---|
| `anthropic` | anthropic | `https://api.anthropic.com` | `ANTHROPIC_API_KEY` |
| `openai` | openai | `https://api.openai.com/v1` | `OPENAI_API_KEY` |
| `zai` (`zhipu`, `glm`) | openai | `https://api.z.ai/api/paas/v4` | `ZAI_API_KEY` |
| `moonshot` (`kimi`) | anthropic | `https://api.kimi.com/coding/` | `MOONSHOT_API_KEY` |
| `minimax` | anthropic | `https://api.minimax.io/anthropic` | `MINIMAX_API_KEY` |
| `deepseek` | openai | `https://api.deepseek.com` | `DEEPSEEK_API_KEY` |
| `qwen` (`dashscope`, `alibaba`) | openai | `https://dashscope-intl.aliyuncs.com/compatible-mode/v1` | `DASHSCOPE_API_KEY` |

### Key type → vendor and endpoint

A subscription key does not work on the pay-as-you-go endpoint and vice versa. A preset changes the
protocol, endpoint and billing, but not the vendor used for quorum independence (`zai` + `zai-coding`
in one config is a config error).

| Your key | `vendor` | protocol, endpoint | cost in the summary |
|---|---|---|---|
| Z.ai pay-as-you-go | `zai` (`glm`) | openai, `https://api.z.ai/api/paas/v4` | per token |
| Z.ai **GLM Coding Plan** | `zai-coding` (`glm-coding`) | openai, `https://api.z.ai/api/coding/paas/v4` | subscription |
| MiniMax pay-as-you-go | `minimax` | anthropic, `https://api.minimax.io/anthropic` | per token |
| MiniMax **Token Plan** | `minimax` + `billing: subscription` | same endpoint, its own key | subscription |
| Kimi **Kimi Code** | `moonshot` (`kimi`) | anthropic, `https://api.kimi.com/coding/`, model `k3` | per token (set `billing: subscription`) |
| Kimi platform (platform.kimi.ai) | `moonshot-platform` (`kimi-platform`) | anthropic, `https://api.moonshot.ai/anthropic`, model `kimi-k3` | per token |

Z.ai accepts a Coding Plan key on `api/anthropic` only from Claude Code and Goose; other tools,
this gate included, must use `coding/paas/v4`, otherwise the API answers `1113 Insufficient
Balance` (Z.ai Coding Plan FAQ). For a pay-as-you-go balance Z.ai documents the general OpenAI
endpoint `api/paas/v4`; its Anthropic route bills the balance only for allowlisted accounts that
never bought a Coding Plan. MiniMax uses one endpoint for both plans, with different keys.
`billing: subscription` sets the marginal actual cost to $0 and shows its token-priced PAYG
estimate separately. It does **not** relax `max_usd_per_pr`: the gate cannot verify the key type, so the cap still
plans and trims `max_tokens` by the model's token prices.

The summary shows actual spend, the PAYG forecast for **all** reviewers as if none had subscriptions,
and subscription savings (PAYG minus actual). Fixed subscription fees are not allocated to a run.
Missing metering is shown as a lower bound (`≥`); missing cache detail can give a range. Each total
preserves its own bounds, so actual subscription spend remains exactly $0 even when PAYG is unknown.

Metrics keep `schema: 1`: reviewer `cost_usd` is actual spend (`0` for subscriptions, previously
`null`), and `cost_usd_payg` is token-priced cost. `cost_usd_estimate` remains a subscription-only
alias of `cost_usd_payg`. Numeric cost sums remain compatible; identify subscriptions by `billing`,
not by a null cost. Gate totals are `total_cost_usd`, `total_cost_usd_payg`, and
`subscription_savings_usd`. For reviewers and totals, `_min` holds a range's lower endpoint (the
unsuffixed field is the upper endpoint). Incomplete metering without `_min` means only a lower
bound is known; a null reviewer cost means unknown. Flags are `metering_incomplete` for actual,
`metering_payg_incomplete` for PAYG, and gate `subscription_savings_incomplete` for savings.

### Presets

```yaml
# claude+gpt — two frontier vendors
reviewers:
  - { id: claude, vendor: anthropic, model: claude-opus-5-5, max_tokens: 16000 }
  - { id: gpt, vendor: openai, model: gpt-6.1-sol, max_tokens: 32000 }
```

```yaml
# glm+minimax — two Chinese vendors, low cost
reviewers:
  - { id: glm, vendor: zai, model: glm-5.3, max_tokens: 32000 }
  - { id: minimax, vendor: minimax, model: MiniMax-M3, max_tokens: 16000 }
```

```yaml
# deepseek+claude — a cheap first voice and a strong second one
reviewers:
  - { id: deepseek, vendor: deepseek, model: deepseek-v4-pro, max_tokens: 32000 }
  - { id: claude, vendor: anthropic, model: claude-sonnet-5-5, max_tokens: 16000 }
```

Reasoning models spend the same token budget on thinking: if a reviewer's answer is cut off, the
summary says so and names `max_tokens` — raise it.

## Security

- **Fork PRs get no secrets** under `pull_request`. The gate detects a fork and does not call any
  model: in `block` mode it exits `2` ("PR from a fork"), so the merge stays held until a maintainer
  runs the gate with their keys; in `comment` mode it exits `0` and writes SKIPPED to the job
  summary.
- **Prompt injection.** The PR title, description and diff are placed between randomized
  untrusted-data markers, and the model is told that instructions inside are data and that an
  attempt to steer the review is a blocking security finding. This lowers the risk; the main
  defense is the quorum of two independent vendors — approval needs both.
- **Secrets in output.** Everything the gate publishes (summary, comments, finding paths and text)
  goes through a secret redactor first.

## Privacy

- **Your code goes only to the providers you configure.** The diff, title and description are sent
  from your runner directly to the endpoints of the two reviewers in your config, under your API
  keys and your agreements with those providers. Choose vendors whose data terms fit your code.
- No telemetry. Opt-in anonymous statistics are planned; report_upload is reserved and currently sends nothing.
  Setting `report_upload: true` only prints a warning; it does not send a report to VimCrew.
- Before the diff leaves the runner, likely secrets in it are redacted (`sk-…`, `ghp_…`, `AKIA…`, PEM,
  literal passwords). The redaction is conservative and may also mask references such as
  `${{ github.token }}`; the reviewers are told that `«REDACTED»` is the gate's marker, not code, and
  not a finding.

## License

- This action (the files in this repository) — [Apache License 2.0](LICENSE).
- The container image `ghcr.io/vimcrew/review-gate` that the action runs is proprietary and is
  provided under the [VimCrew Review Gate EULA](EULA.md). Running the action means running that
  image under the EULA.
- Open-source components inside the image (Java/Kotlin libraries, Eclipse Temurin JRE, GitHub CLI,
  base OS packages) keep their own licenses; their notices are in the image at
  `/licenses/THIRD-PARTY-NOTICES.txt`, e.g.
  `docker run --rm --entrypoint cat ghcr.io/vimcrew/review-gate@<digest> /licenses/THIRD-PARTY-NOTICES.txt`.
