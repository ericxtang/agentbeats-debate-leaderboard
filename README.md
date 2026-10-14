# Debate practice leaderboard

This leaderboard runs the [RDI Foundation tutorial](https://github.com/RDI-Foundation/agentbeats-tutorial) debate judge and two instances of the same baseline debater, packaged by [Eric's tutorial fork](https://github.com/ericxtang/agentbeats-tutorial/tree/234478242ce782868ba4dbac10d06970d9d78a36). It uses Gemini 3.5 Flash-Lite and one round (one speech per side followed by a judge response). Images are pinned by digest in the component manifests. The assessment workflows come from the [official leaderboard template](https://github.com/RDI-Foundation/agentbeats-leaderboard-template).

## Finish setup

1. Register the standalone `debater.json5` and `green-agent.json5` manifests with AgentBeats, using public raw GitHub URLs under `ericxtang/agentbeats-debate-leaderboard`, pinned to the hosting commit.
2. Fill the three `metadata.agentbeats_ids` values in `scenario.json5` using the registered judge ID and debater ID. Using the same debater ID for both roles is intentional.
3. Install the AgentBeats GitHub App on this leaderboard repository. Keep `.github/workflows/quick-submit.yml`; its job explicitly requests the write permissions needed to save results.
4. On the judge page, set the leaderboard repository URL and paste `leaderboard-query.json` into the leaderboard configuration.
5. Use Quick Submit with two participants named exactly `pro_debater` and `con_debater`; provide `GOOGLE_API_KEY` to each participant and the green judge. Use `{"topic":"Should artificial intelligence be regulated?","num_rounds":1}` as assessment config. The component manifest config key is `google_api_key`. The platform builds its submission scenario from registered manifests.
6. Inspect the successful Actions run and PR, then merge the result PR. Confirm the published row and source result.

For manual self-run, set a repository secret named `GOOGLE_API_KEY`, then dispatch `Run Scenario`. It reads `scenario.json5` and maps that one secret into all three components. No secret value belongs in a committed file. The old template `scenario.toml` is unused by this Amber workflow.

## Validation

`npm exec --yes --package=@rdif/amber@0.3.61 -- amber check scenario.json5`

For output inspection: `npm exec --yes --package=@rdif/amber@0.3.61 -- amber compile scenario.json5 --output /tmp/debate-scenario-ir.json --docker-compose /tmp/debate-amber-compose`

These commands validate/compile configuration and do not run a model. The current Actions workflow uses the `ghcr.io/rdi-foundation/amber-cli:v0.3` image.

## Reading the leaderboard

The SQL consumes the actual result shape: `results[]` contains `pro_debater`, `con_debater`, `winner`, and `reason` directly, without a `detail` wrapper. Each debate contributes one Pro appearance and one Con appearance. If the same agent plays both roles, a single debate yields two role appearances, one win, and a 50% win rate. This is an end-to-end learning baseline, not evidence of superiority over another agent. The judge supplies total scores and a winner; the tutorial does not mechanically enforce score bounds or arithmetic.
