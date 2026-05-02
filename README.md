# Vending-Bench v0

Multi-agent AI benchmark platform comparing **4 brain families across 5 dispatch shapes** on identical prompts. Methodology designed for honest first-shot reasoning comparison, not infrastructure-disparity measurement.

## Evidentiary limits (read first)

This is a v0 portfolio benchmark with deliberately scoped claims. Before drawing inferences from the results:

- **Sample size is small.** C1 produced N=3 substantive results (anismin, meridian-codex, meridian-opus self-include) + N=2 INCOMPLETE (kcs-classic, kcs-swarm — see Tab 1 KCS finding for the rail-bug story). Single-challenge data; do not extrapolate.
- **Scoring rubric is not inter-rater-validated.** The 4-axis subjective scoring (accessibility / responsive / inline-discipline / brief-fidelity) is Trident-consensus across two independent reviewer brains (Codex GPT-5.4 + Gemini-flash). Absolute scores drift between Trident invocations on the same content (rubric calibration noise) — trust the **ranking** more than the absolute numbers. No human inter-rater reliability study.
- **Brain-family overlap.** `meridian-opus` (clawfish-opus-4-7 on Discord) and `meridian-codex` (gpt-5.4 via openclaw-gateway) share Meridian's persona + memory files but are distinct inference brains. The `5 dispatch shapes` framing is honest; calling it `5 brains` would inflate diversity. Anismin Opus 4.7 + KimiClawSwarm K2.6 + Meridian-Codex gpt-5.4 + Meridian-Opus claude-opus-4-7 = 4 brain families.
- **KCS over-decomposition is a known v0 finding.** Both KCS dispatch shapes timed out without producing deliverables on this simple-task benchmark. The bench-fanout rail bug that initially masked this as "OK" was caught + patched mid-C1 (commit 462ce5b). KCS cadence on small tasks is documented as a separate operational signal, not a benchmark failure of the rail.
- **C2 is in-flight as of v0 publication.** The functional-code challenge (stdlib Python CSV reconciliation) is being drafted by Meridian. Tab 2 shows placeholder until results land via the same bench-fanout rail.

## Methodology

### Fairness contract (forwarded verbatim to every agent)

Every benchmark dispatch carries this contract at the top of the prompt body, identical text, no paraphrase:

> Blind generation. No tool use. No web access. Single-shot. No iterative refinement. No follow-up turns. No clarifying questions. Identical prompt forwarded verbatim to multiple independent agents. Output: one self-contained file per the challenge's output_files spec.

The same contract applies to the Meridian-Opus self-include path. The operator/Meridian generates the cold-prompt one-shot reply against the exact prompt body in a clean buffer with no Discord context bleed, no tool use during generation, no iterative refinement.

### Agent roster

| Slug | Brain | Transport | Notes |
|---|---|---|---|
| `anismin` | Anismin Opus 4.7 | `anismin-chat` (POST :18889/api/chat) | Strategist brain, direct chat path |
| `meridian-codex` | Meridian gpt-5.4 | `meridian-chat` (`node Meridian/scripts/meridian-chat.js`) | Codex-family, full tool surface in her openclaw container |
| `meridian-opus` | Clawfish-opus-4-7 | self-include via omission OR `starfish-reason` shim | Discord-facing brain; reply manually saved to subdir to avoid double-counting with meridian-codex |
| `kcs-classic` | KimiClawSwarm classic mode | `kcs-swarm` mode=classic, max_agents=1, max_steps=6 | Single-shot KCS for parity with the others |
| `kcs-swarm` | KimiClawSwarm crew mode | `kcs-swarm` mode=crew, max_agents=4, max_steps=12 | Multi-agent debate — honest swarm-vs-single-shot comparison |

### Dispatch infrastructure

Fanout via `bench-fanout.py` rail (AnisminOS commit 42cb921, watcher 9f8f110). Operator/Meridian writes a YAML spec to `~/.openclaw/workspace/bench-specs/inbox/<challenge-slug>.yaml`; the host-side watcher (schtask `AnisminOS-BenchFanoutWatcher`, every 1 min) claims atomically, runs the fanout, writes per-agent results to `~/.openclaw/workspace/bench-results/<challenge-slug>/<agent-slug>/`. Each agent reply lands as the spec's `output_files[0]` plus an `_meta.json` sidecar with timing + status.

The script forwards prompts verbatim — no inference, no paraphrase, no summarization. Fairness-contract preservation is verified by inspecting each agent's reply for evidence the contract was honored, then noted in the postback summary.

### Scoring

Per-challenge:

- **Subjective challenges** (e.g. design, copy): scored via Trident `ask all --qc` consensus across multiple independent reviewers (Codex, Gemini, Grok, Claude). Rubric is named in the challenge's per-row scoring sheet (e.g. accessibility, responsive design, inline-discipline, brief-fidelity for Challenge 1).
- **Deterministic challenges** (e.g. code with verifiable output): scored by diff-against-reference. Operator/Meridian pre-computes the expected output BEFORE dispatch, saves to `reference/<challenge>/expected.json`. Each agent's output is diffed; pass requires byte-exact or schema-equivalent match per the challenge's tolerance rules.

### Brain-state header convention (required for Historical tab)

Every row ported from a prior benchmark carries a brain-state header:

- `model_version` (e.g. `claude-opus-4-7`, `gpt-5.4`, `K2.6`)
- `sprint_level` (e.g. `pre-Sprint-1`, `Sprint-3`)
- `timeout_config_at_run_time` (e.g. `chat=180s, kcs=600s`)

Without this header, comparison across eras is dishonest — older runs pre-date sprint tooling and session-bloat fixes that materially changed brain behavior. Silent overlay of pre-Sprint vs current-Sprint results in the same chart is forbidden.

## Repo layout

```
Vending-Bench-v0/
├── README.md (this file)
├── index.html (3-tab dashboard, GitHub Pages)
├── challenge-1/<agent-slug>/output.html  (Challenge 1 per-agent submissions)
├── challenge-2/<agent-slug>/solution.py + report.json  (Challenge 2 submissions)
├── reference/challenge-2/expected.json  (operator/Meridian-precomputed reference output)
└── historical/  (Kiln-Tide-Bench port; see Tab 3)
```

Slugs reserved: `anismin`, `meridian-codex`, `meridian-opus`, `kcs-classic`, `kcs-swarm`.

## Tabs

- **Challenge 1: Website Design** — single-file HTML benchmark (system-fonts-only typographic specimen as the discriminator vs template recall). 4-axis scoring: accessibility, responsive design, inline-discipline, brief-fidelity.
- **Challenge 2: Functional Code** — stdlib-only Python deterministic data-processing benchmark (CSV reconciliation with conflicting-refund-records edge case as the load-bearing discriminator). Diff-against-reference scoring.
- **Historical: Kiln-Tide-Bench** — live port of [ademczuk/Kiln-Tide-Bench](https://github.com/ademczuk/Kiln-Tide-Bench). 7 agent attempts at port-time. **Tagged "pre-Sprint-1/2/3, session-bloat era"** — comparison to current Vending-Bench results requires explicit brain-state header acknowledgment per the convention above.

## Operating contract

This repo is generated and curated under the AnisminOS bench-fanout rail. Specs and results are the source of truth; the dashboard is a convenience renderer over them. Any analysis or claim derived from this benchmark must cite both the specific spec.yaml dispatched AND the per-agent `_meta.json` sidecar so reviewers can audit timing, transport, and any contract-violation flags without trusting prose summaries.
