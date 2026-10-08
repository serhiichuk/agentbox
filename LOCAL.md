# Local copy

This repo is upstream pstack from [`cursor/plugins`](https://github.com/cursor/plugins) (see [`UPSTREAM`](./UPSTREAM)), unedited except for the patches below. It runs on Claude Code, omp, and agy. An instructions file, `global/AGENTS.md`, adapts the Cursor-specific steps instead of edits to the skills, so upstream updates apply without conflicts.

## Branches

- `upstream` holds upstream's `pstack/` verbatim, one commit per update, without `.cursor-plugin/`, `assets/`, and `docs/`.
- `main` is `upstream` plus the local files and patches.

## Local files

- `global/AGENTS.md` sets the push and merge policy, runs pstack's delegation through Orca, and imports the models file `~/.agents/pstack-models.md`. `bin/sync-agents --apply` links it as each harness's global instructions.
- `bin/sync-skills` links `skills/*` into each harness.
- `bin/sync-agents` renders `agents/*.md` per harness with models from `agents/models.json`, and links `global/AGENTS.md`. It derives a role's name from its file name and its model class from the `roles` map in `models.json`, so upstream's role files stay unedited.
- `agents/models.json` mirrors the classes in the agents repo.
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
bin/sync-skills          # interactive
bin/sync-agents          # dry run
bin/sync-agents --apply
```

Put the models file at `~/.agents/pstack-models.md`: upstream's role lines with alias values, plus an `## aliases` block with one start command per alias. Link it at the path the skills name: `mkdir -p ~/.cursor/rules && ln -s ~/.agents/pstack-models.md ~/.cursor/rules/pstack-models.mdc`. Claude Code and omp read it through the import in `global/AGENTS.md`. agy does not expand the import and reads the file when a skill names it. Do not run `/setup-pstack`: it overwrites the file with Cursor model names.
