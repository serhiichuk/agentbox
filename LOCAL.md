# Local copy

This repo is upstream pstack from [`cursor/plugins`](https://github.com/cursor/plugins) (see [`UPSTREAM`](./UPSTREAM)), unedited except for the patches below. It runs on Claude Code, omp, and agy. An instructions file, `global/AGENTS.md`, adapts the Cursor-specific steps instead of edits to the skills, so upstream updates apply without conflicts.

## Branches

- `upstream` holds upstream's `pstack/` verbatim, one commit per update, without `.cursor-plugin/`, `assets/`, and `docs/`.
- `main` is `upstream` plus the local files and patches.

## Local files

- `global/AGENTS.md` maps pstack's Cursor tools, paths, and model slugs to this setup, and sets the push and merge policy. It is untracked. `bin/sync-agents` links it as each harness's global instructions when it exists.
- `bin/sync-skills` links `skills/*` into each harness.
- `bin/sync-agents` renders `agents/*.md` per harness with models from `agents/models.json`, and links `global/AGENTS.md`. It derives a role's name from its file name and its model class from the `roles` map in `models.json`, so upstream's role files stay unedited.
- `agents/models.json` mirrors the classes in the agents repo.

## Patches to upstream files

- `skills/make-bot-ui/` is deleted. It drives Cursor-hosted bot routines.
- `skills/poteto-mode/SKILL.md`: `name: poteto-mode`, the directory name, in place of `Poteto Mode`.
- `.gitignore`: `agents/.build/`, the output of `bin/sync-agents`.

Keep this list short. Fix a Cursor-ism in `global/AGENTS.md` unless only code can fix it.

## Update from upstream

```sh
git clone --filter=blob:none --no-checkout https://github.com/cursor/plugins /tmp/plugins
git -C /tmp/plugins sparse-checkout set pstack && git -C /tmp/plugins checkout <sha>
git switch upstream
rsync -a --delete --exclude .git --exclude .cursor-plugin --exclude assets --exclude docs /tmp/plugins/pstack/ ./
git add -A && git commit -m "chore: vendor upstream pstack from cursor/plugins@<sha>"
git switch main && git merge upstream
```

Set `commit:` in `UPSTREAM` to the new sha. Then grep the new upstream for Cursor terms the map in `global/AGENTS.md` lacks (`Task`, `subagent_type`, `cloud`, `origin pr`, `cursor-team-kit`, `AskQuestion`, `/loop`, `create-skill`, `.cursor/`), and add rows for them.

## Deploy

On the target machine, from a clone of this repo:

```sh
bin/sync-skills          # interactive
bin/sync-agents          # dry run
bin/sync-agents --apply
```

Copy `global/AGENTS.md` there separately, since it is untracked. Put the models file at `~/.cursor/rules/pstack-models.mdc`, where the skills look for it, or run `/setup-pstack`.
