# Local copy

This repo is upstream pstack from [`cursor/plugins`](https://github.com/cursor/plugins) (see [`UPSTREAM`](./UPSTREAM)), unedited except for the patches below. It runs on Claude Code, omp, and agy. An instructions file, `global/AGENTS.md`, adapts the Cursor-specific steps instead of edits to the skills, so upstream updates apply without conflicts.

## Branches

- `upstream` holds upstream's `pstack/` verbatim, one commit per update, without `.cursor-plugin/`, `assets/`, and `docs/`.
- `main` is `upstream` plus the local files and patches.

## Local files

- `global/AGENTS.md` sets the push and merge policy, runs pstack's delegation through Orca, maps Cursor names to each harness, and lists the model aliases that `/setup-pstack` chooses from. `bin/sync-agents --apply` links it as each harness's global instructions.
- `bin/sync-skills` links `skills/*` into each harness. It is a copy of the script in the agents repo.
- `vendor/rro-maryta` is a submodule with the Checkbox skills (`redmine`, `sentry`, `gitlab-mr`, and others). `bin/sync-vendors`, also a copy from the agents repo, updates it and links each of its skills into `skills/`.
- `vendor/agent-browser` is a submodule with the `agent-browser` stub skill, which loads its guide from the installed CLI. Its `skills = skills` key in `.gitmodules` keeps `bin/sync-vendors` from linking the CLI-served `skill-data/` skills.
- `vendor/vuejs-ai` is a submodule with the Vue 3 skills (`vue-best-practices` and seven more). Its `skills = skills` key in `.gitmodules` is required for discovery: upstream keeps each skill one directory down, under `skills/`.
- `bin/sync-agents` renders `agents/*.md` per harness and links `global/AGENTS.md`. It derives a role's name from its file name, so upstream's role files stay unedited. A role runs on the parent's model.
- `skills/deslop/`, `skills/control-ui/`, and `skills/control-cli/` are Cursor's skills from `cursor-team-kit`, copied verbatim (see `UPSTREAM`). pstack invokes them by name.

## Patches to upstream files

- `skills/make-bot-ui/` is deleted. It drives Cursor-hosted bot routines.
- `.gitignore`: `agents/.build/`, the output of `bin/sync-agents`, and `tmp/` for local artifacts.

Keep this list short. Fix a Cursor-ism in `global/AGENTS.md` unless only code can fix it.

## Update from upstream

```sh
bin/sync-upstream            # new commits, changed files, Cursor terms in added lines
bin/sync-upstream --diff     # plus the full diff
bin/sync-upstream --apply    # vendor onto `upstream`, merge into main, refresh the cursor-team-kit skills, bump UPSTREAM
```

It keeps a sparse clone of cursor/plugins in `~/.cache/pstack-upstream`. After `--apply`, check the reported Cursor terms against the map in `global/AGENTS.md`, then deploy.

## Live sync with agentbox

Mutagen keeps `~/Repos/pstack` on the Mac and `agentbox:~/Repos/pstack` identical in both directions, without a commit. Git runs on the Mac only: `.git` is not synced, so the agentbox `.git` is stale.

```sh
mutagen sync create --name=pstack --mode=two-way-safe \
  --ignore=.git --ignore=/agents/.build --ignore=.DS_Store --ignore=/plans --ignore=/tmp \
  ~/Repos/pstack agentbox:~/Repos/pstack
mutagen sync list pstack     # expect "Watching for changes" and no conflicts
```

After a change to `agents/*.md`, run `ssh agentbox ~/Repos/pstack/bin/sync-agents --apply`. If both hosts change one file between syncs, Mutagen keeps both versions and lists a conflict. Resolve it by hand.

## Deploy

On the target machine, from a clone of this repo:

```sh
git submodule update --init
bin/sync-skills          # interactive
bin/sync-agents          # dry run
bin/sync-agents --apply
```

Then, in a Claude Code, omp, or agy session, run `/setup-pstack`. It writes the role lines to `~/.cursor/rules/pstack-models.mdc`, with the aliases from `global/AGENTS.md` as values. When the alias table in `global/AGENTS.md` changes, run `/setup-pstack` again.

omp uses its own default models. `sync-agents` does not set `modelRoles` or `task.agentModelOverrides`.
