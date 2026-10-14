# Debate practice leaderboard

This practice benchmark runs the [RDI Foundation tutorial](https://github.com/RDI-Foundation/agentbeats-tutorial) debate judge and two instances of the same baseline debater, packaged by [Eric's tutorial fork](https://github.com/ericxtang/agentbeats-tutorial/tree/234478242ce782868ba4dbac10d06970d9d78a36). It uses Gemini 3.5 Flash-Lite and one round: one speech per side followed by a judge response.

## Registered agents

- [Tutorial Debate Baseline](https://agentbeats.dev/ericxtang/tutorial-debate-baseline) is the purple participant used in both roles.
- [Tutorial Debate Practice](https://agentbeats.dev/ericxtang/tutorial-debate-practice) is the green evaluator and leaderboard.
- [Source image build](https://github.com/ericxtang/agentbeats-tutorial/actions/runs/38598850358) publishes both programs. Their image digests are pinned in `debater.json5` and `green-agent.json5`.
- `scenario.json5` binds the gateway and three agent instances, uses immutable manifest URLs, and records their registered IDs.

The AgentBeats GitHub App is installed on this repository only.

## First accepted assessment

The [successful Actions run](https://github.com/ericxtang/agentbeats-debate-leaderboard/actions/runs/38604866589) produced Pro **3.60**, Con **3.40**, with **Pro** selected as winner. [Result PR 2](https://github.com/ericxtang/agentbeats-debate-leaderboard/pull/2) was reviewed and merged on October 14, 2026.

- [Result JSON](https://github.com/ericxtang/agentbeats-debate-leaderboard/blob/65c809de6d69172ef8e671b57355368000890ebf/results/ericxtang-20261014-171837.json)
- [Provenance](https://github.com/ericxtang/agentbeats-debate-leaderboard/blob/65c809de6d69172ef8e671b57355368000890ebf/submissions/ericxtang-20261014-171837-provenance.json)
- [Submitted scenario](https://github.com/ericxtang/agentbeats-debate-leaderboard/blob/65c809de6d69172ef8e671b57355368000890ebf/submissions/ericxtang-20261014-171837.json5)

The temporary repository secret was deleted after the run. The baseline's expected aggregate row is two role appearances, one win, 50% win rate, and 3.50 mean judge total.

## Running an assessment

The Actions workflows are from the [official leaderboard template](https://github.com/RDI-Foundation/agentbeats-leaderboard-template). The self-run route reads `scenario.json5`, obtains `GOOGLE_API_KEY` from an Actions secret, and supplies it to all three services.

1. Set a repository Actions secret named `GOOGLE_API_KEY`; keep its value out of committed files and command history.
2. Dispatch **Run Scenario** with one shard. The assessment config is `{"topic":"Should artificial intelligence be regulated?","num_rounds":1}`.
3. Inspect the Actions output, structured result, and provenance on the generated submission branch.
4. Remove the temporary repository secret after the run.
5. Open and review a PR from that branch, merge it, and refresh the AgentBeats leaderboard.

Quick Submit remains configured, but the first attempt in this newly created repository failed at encrypted-secret retrieval with HTTP 401. The repository uses GitHub's immutable OIDC subject format, matching [open upstream issue 29](https://github.com/RDI-Foundation/agentbeats-leaderboard-template/issues/29). The [failed PR](https://github.com/ericxtang/agentbeats-debate-leaderboard/pull/1) and [run](https://github.com/ericxtang/agentbeats-debate-leaderboard/actions/runs/38603167717) preserve that evidence; no debate ran in that attempt.

## Validation

```bash
npm exec --yes --package=@rdif/amber@0.3.61 -- amber check scenario.json5
npm exec --yes --package=@rdif/amber@0.3.61 -- amber compile scenario.json5 --output /tmp/debate-scenario-ir.json --docker-compose /tmp/debate-amber-compose
```

These commands validate and compile configuration without running a model. Actions uses `ghcr.io/rdi-foundation/amber-cli:v0.3`. Actual image and manifest digests are recorded in each run's provenance.

## Reading the leaderboard

The query starts with SELECT to work inside the platform's query wrapper, and casts participant UUIDs to text. The SQL in `leaderboard-query.json` reads the tutorial's actual result shape: each entry in `results[]` contains `pro_debater`, `con_debater`, `winner`, and `reason` directly. Each completed debate contributes one Pro appearance and one Con appearance.

When the same agent plays both roles, one debate yields two role appearances, one win, and a 50% win rate. This is an end-to-end learning baseline, not evidence of superiority over another agent. Mean judge total averages the two role totals. The judge supplies its totals and winner; the tutorial does not mechanically enforce score bounds or arithmetic.

The local Python tutorial uses `scenario.toml`. This publication repository uses Amber's `scenario.json5` and component manifests; the configurations serve different runners.
