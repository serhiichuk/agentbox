# Agent instructions

This machine runs Claude Code, omp, and agy, usually inside Orca. The pstack skills are installed verbatim from upstream, which was written for Cursor. Follow them as written, with the two substitutions below. Where a pstack skill conflicts with this file, this file wins.

## Policy

- **Branches.** Push only branches you created for the current task, and `--force-with-lease` only on those. Never push to a default or protected branch (`main`, `master`, `develop`, release branches), and never merge a PR or MR. Hand the human the exact command and wait. Server-side branch protection and token roles enforce this too. These lines do not replace them.
- **Destructive actions.** Before deleting, overwriting, force-updating, or rewriting history, databases, deployments, remote state, credentials, or user data: verify the exact target, establish a rollback path, and confirm the human authorized that exact action. If any of the three is uncertain, stop and ask.
- **Redmine** is read-only. Write to Redmine only when the human explicitly asks for that write in the current conversation. Otherwise, do not update issues, add notes or comments, change statuses, log time, or upload attachments. Return the proposed text for the human to paste. This covers the `redmine` skill's `log`, `status`, and `raw` write calls.
- **Commits** never carry `Co-Authored-By` or any other agent attribution line.
- **Verify, do not guess.** Base every claim about code, commands, flags, and model IDs on something you read or ran in this session. Label the rest unverified.

## Delegation runs through Orca

- A pstack `Task` call or cloud agent runs as an Orca worker (the `orchestration` skill). Outside Orca, use the harness's own subagent tool.
- Keep pstack's placement. A worker shares the coordinator's worktree (`worker-start --terminal <handle> --worktree current`), unless the skill gives it its own worktree or branch. A cloud agent gets its own Orca worktree.
- An Orca worker asks the human with the orchestration `ask` command from its preamble, not `AskQuestion`. Nobody watches a worker's pane.

## Models

@~/.agents/pstack-models.md

The models file is `~/.agents/pstack-models.md`. `~/.cursor/rules/pstack-models.mdc`, the path the skills name, links to it. If its role lines are not in your context, read it whenever a skill names a role.

- A role value names who runs the role. Each harness is one model family: `claude` is Anthropic, `omp` is OpenAI, `agy` is Google.
  - `self` is your own harness, `others` is the two other harnesses, and `all` is all three. `strong` picks each harness's strong alias from `## families`.
  - `cheap` is `haiku` or `gemini`, whichever is not your own harness. In omp, `cheap` is `haiku`.
  - An alias name, such as `opus`, runs that alias.
- `self strong` and `inherit-parent` run on the harness's own subagent tool, on the parent's model.
- Start every other alias with its command from the file's `## aliases` block, in the worker's Orca terminal.
- A review or a test of your own work always runs on `others`.
- If the file or the role line is missing, use `self strong` for code and `others strong` for review. Say so.
