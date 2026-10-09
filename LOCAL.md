# Local copy

This repo is upstream pstack from [`cursor/plugins`](https://github.com/cursor/plugins) (see [`UPSTREAM`](./UPSTREAM)), unedited except for the patches below. It runs on Claude Code, omp, and agy. An instructions file, `global/AGENTS.md`, adapts the Cursor-specific steps instead of edits to the skills, so upstream updates apply without conflicts.

## Branches

- `upstream` holds upstream's `pstack/` verbatim, one commit per update, without `.cursor-plugin/`, `assets/`, and `docs/`.
- `main` is `upstream` plus the local files and patches.

## Local files

- `global/AGENTS.md` sets the push and merge policy, runs pstack's delegation through Orca, and lists the model aliases that `/setup-pstack` chooses from. `bin/sync-agents --apply` links it as each harness's global instructions.
- `bin/sync-skills` links `skills/*` into each harness. It is a copy of the script in the agents repo.
- `vendor/rro-maryta` is a submodule with the Checkbox skills (`redmine`, `sentry`, `gitlab-mr`, and others). `bin/sync-vendors`, also a copy from the agents repo, updates it and links each of its skills into `skills/`.
- `bin/sync-agents` renders `agents/*.md` per harness and links `global/AGENTS.md`. It derives a role's name from its file name, so upstream's role files stay unedited. A role runs on the parent's model.
- `skills/deslop/` is Cursor's `deslop` skill from `cursor-team-kit`, copied verbatim (see `UPSTREAM`). pstack invokes it by name.

## Patches to upstream files

- `skills/make-bot-ui/` is deleted. It drives Cursor-hosted bot routines.
- `skills/poteto-mode/SKILL.md`: `name: poteto-mode`, the directory name, in place of `Poteto Mode`.
- `.gitignore`: `agents/.build/`, the output of `bin/sync-agents`.

Keep this list short. Fix a Cursor-ism in `global/AGENTS.md` unless only code can fix it.

## Update from upstream

```sh
bin/sync-upstream            # new commits, changed files, Cursor terms in added lines
bin/sync-upstream --diff     # plus the full diff
bin/sync-upstream --apply    # vendor onto `upstream`, merge into main, refresh deslop, bump UPSTREAM
```

It keeps a sparse clone of cursor/plugins in `~/.cache/pstack-upstream`. After `--apply`, check the reported Cursor terms against the map in `global/AGENTS.md`, then deploy.

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
