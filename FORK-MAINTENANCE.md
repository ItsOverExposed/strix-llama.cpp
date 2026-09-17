# Personal Strix Halo fork

GitHub repository: `ItsOverExposed/strix-llama.cpp`.
Source: `halo-box/strix-llama.cpp`, initially inspected at `8c1c282ecb194e8f02613defcc4a07c22b6d1c08`.

## Branches

- `master`: unchanged Halo Box history. Advance only by fast-forwarding from `upstream/master`.
- `integration`: personal changes and selected upstream updates. No automatic deployment.
- `experiment/<name>`: an isolated change based on integration.
- `tested-YYYY-MM-DD`: tags for revisions actually validated on the target machine. Do not create a tested tag until validation passes.

## Remotes

- `origin`: your personal GitHub fork.
- `upstream`: `https://github.com/halo-box/strix-llama.cpp.git`.
- `llama-main`: `https://github.com/ggml-org/llama.cpp.git`.

The prepared local checkout disables pushing to upstream and llama-main and defaults pushes to origin. The GitHub connector's login does not automatically authenticate terminal Git pushes. Use an authenticated Git client or the connector for publishing changes.

## Updating the maintained branch

Start with a clean working tree. Run from your clone:

```bash
git fetch upstream
git switch master
git merge --ff-only upstream/master
git push origin master
git switch integration
git switch -c experiment/halo-update-YYYY-MM-DD
git merge master
```

Replace the date with today's date. If the fast-forward fails, inspect the divergence; do not reset or force-push. If the integration merge conflicts, resolve and test it or use `git merge --abort`. The existing integration branch is preserved while the experiment is evaluated.

## Testing and promotion

Build the current integration revision and candidate in separate worktrees and build directories. Use the same compiler, backend, model, quantization, prompts, KV settings, batch sizes, context depths and power profile. Record full commit IDs, build commands, driver versions and model hashes.

Check correctness before timing: relevant backend tests and deterministic output/logit comparisons, including long context. Then compare repeated prompt-processing and generation measurements with variance. Record untested paths explicitly. Read the source repository's AGENTS.md and CONTRIBUTING.md for its full validation matrix.

After validation succeeds:

```bash
git switch integration
git merge --ff-only experiment/halo-update-YYYY-MM-DD
git tag -a tested-YYYY-MM-DD -m "Validated on Strix Halo; see recorded test results"
git push origin integration
git push origin tested-YYYY-MM-DD
```

If integration has advanced meanwhile, merge it into the experiment and repeat affected checks before promotion. Keep the running installation pinned to a tested tag until its replacement passes.

## Selecting an upstream PR

```bash
git fetch llama-main refs/pull/NUMBER/head:refs/remotes/llama-main/pr-NUMBER
git show --stat llama-main/pr-NUMBER
git diff llama-main/pr-NUMBER^ llama-main/pr-NUMBER
```

Inspect all commits for multi-commit PRs. Check for equivalent changes already present before cherry-picking; a PR's parent may differ significantly from Halo Box. Work only on an experiment branch and avoid merging the entire upstream PR branch blindly.

## Flash-Next direct reads

The inspected Halo Box source already implements parallel, sorted, deduplicated PLE row reads for Qwen3.8-Flash-Next with `--lazy-mode on-direct`. PR #29030 is a generalized, portable implementation of the same approach, not an additional proven speed multiplier. The inspected Halo Box reader does not support native Windows. No inference modification or speedup has been validated as part of this repository setup.

## Maintenance policy

A daily Codex task named Sync Halo Box master checks at 09:00 local time and fast-forwards the GitHub master branch from halo-box/strix-llama.cpp master. It never force-pushes and reports divergence or access failures. It verifies each update and stays quiet when nothing changes. This schedule is managed in Codex, not GitHub Actions. The integration branch is our separate custom version and receives no automatic merges. Builds, deployment and upstream contributions are not automated. Local branches update when explicitly fetched and fast-forwarded; the schedule does not change the working checkout.
