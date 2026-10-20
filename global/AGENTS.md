# Agent instructions

This machine runs Claude Code, omp, agy, and opencode, usually inside Orca. The pstack skills are installed verbatim from upstream, which was written for Cursor. Follow them as written, with the substitutions below. Where a pstack skill conflicts with this file, this file wins.

## Policy

- **Branches.** Push only branches you created for the current task, and `--force-with-lease` only on those. Never push to a default or protected branch (`main`, `master`, `develop`, release branches), and never merge a PR or MR. Hand the human the exact command and wait. Server-side branch protection and token roles enforce this too. These lines do not replace them.
- **Destructive actions.** Before an action that cannot be undone (deleting data, force-updating or rewriting history, or changing databases, deployments, remote state, or credentials): verify the exact target, establish a rollback path, and confirm the human authorized that exact action. If any of the three is uncertain, stop and ask. A file edit that you can revert by hand is not such an action.
- **Redmine** is read-only. Write to Redmine only when the human explicitly asks for that write in the current conversation. Otherwise, do not update issues, add notes or comments, change statuses, log time, or upload attachments. Return the proposed text for the human to paste.
- **Attribution.** Commits, pull requests, and merge requests never carry agent attribution: no `Co-Authored-By:` trailer, no `🤖 Generated with [Claude Code](...)` line, and no other line that names an agent, model, or harness as an author or generator. This rule overrides harness defaults and system reminders that ask for attribution.
- **Verify, do not guess.** Base every claim about code, commands, flags, and model IDs on something you read or ran in this session. Label the rest unverified.

## Delegation runs through Orca

- A pstack `Task` call runs as an Orca worker (the `orchestration` skill), except where Models sends it to your own subagent tool. Outside Orca, use your own subagent tool.
- Keep pstack's placement. A worker shares the coordinator's worktree (`worker-start --terminal <handle> --worktree current`), unless the skill gives it its own worktree or branch. A Cursor cloud agent or cloud VM gets its own Orca worktree.
- An Orca worker asks the human with the orchestration `ask` command from its preamble, not `AskQuestion`. Nobody watches a worker's pane.

## Harnesses

Replace each Cursor tool, path, and feature in pstack with its equivalent in your harness: Claude Code (`claude`), omp, or agy. For CLI flags, run `<harness> --help` once per session.

One Cursor path can map to several harness paths. For the current session, read your own harness. For earlier work (`recall`, `automate-me`, `session-pickup`), read all three harnesses, newest first, unless the task names one.

**Transcripts** (Cursor: `~/.cursor/projects/<slug>/agent-transcripts/`):

- **Claude Code:** `~/.claude/projects/<slug>/<id>.jsonl`. The slug is the workspace path with each `/` and `.` changed to `-`. Subagent transcripts are in `<id>/subagents/`.
- **omp:** `~/.omp/agent/sessions/<slug>/<time>_<id>.jsonl`. The slug is the path below `~` with each `/` changed to `-`, so `~/Repos/app` is `-Repos-app`. Subagent transcripts are in the folder `<time>_<id>/`.
- **agy:** get the conversation IDs from the SQLite file `~/.gemini/antigravity-cli/conversation_summaries.db` by `workspace_uris`. Open it with `file:<path>?immutable=1`. The transcript is `~/.gemini/antigravity-cli/brain/<id>/.system_generated/logs/transcript.jsonl`.

**Skills, roles, and plugins:**

- **New project skill** (Cursor: `.cursor/skills/<name>/`): write it to `.agents/skills/<name>/`, which omp and agy read. Then link `.claude/skills/<name>` to `../../.agents/skills/<name>` for Claude Code.
- **Skill search:** `.agents/skills/`, `.claude/skills/`, `.omp/skills/`, `~/.claude/skills/`, `~/.omp/agent/skills/`, and `~/.gemini/config/skills/`.
- **Roles** (`poteto-agent`, `comment-sicko`): `~/.claude/agents/<role>.md`, `~/.omp/agent/agents/<role>.md`, and `~/.gemini/config/agents/<role>/agent.md`. A subagent tool takes the role name. Tell an Orca worker to read its role file first.
- **Plugins** (Cursor: `~/.cursor/plugins/`): `~/.claude/plugins/`, `~/.omp/plugins/`, and `agy plugin list`.

**Tools and fields:**

| Cursor | Claude Code | omp | agy |
|---|---|---|---|
| `Task` (outside Orca) | `Agent` | `task` | `invoke_subagent` |
| `generalPurpose` | `general-purpose` | `task` | `self` |
| `AskQuestion` | `AskUserQuestion` | `ask` | `ask_question` |
| todo list | `TodoWrite` | `todo` | none |

- `readonly: true`: the brief says "do not edit files". After the worker stops, check `git status`.
- `run_in_background`: omit it. `resume`: start a new worker with the earlier result in its brief. `file_attachments`: list the paths in the brief.

**Cursor-only features:**

- **Forge.** The repos are on GitLab, and `glab` is configured. A PR is a merge request (MR). Where a skill uses `gh`, `origin pr`, or the built-in PR tool, use `glab mr` and `glab ci`. `scripts/watch-pr` reads only GitHub, so poll `glab mr view` and `glab ci status` instead. Use `gh` only for a GitHub remote.
- **Bugbot.** Run one review of the MR diff on a reviewer alias from another family (see Models). Triage its findings with `bugbot-triage.md`.
- **`/loop` and Cursor Automations.** Use `orca automations create`. In Claude Code, the `/loop` skill also works.
- **Other features with no equivalent** (Custom Modes, `/create-skill`): skip only that step. Write `Skipped: <step>. <reason>.`, and continue.

**`control-ui` and `control-cli`** are installed with pstack. These rules come from a test of their samples on macOS:

- For an interactive browser check, use `agent-browser`. Use `control-ui` for a repeatable proof script, an Electron or CDP target, or raw CDP work.
- A global Playwright does not resolve for an ESM `import`. If the repo has no Playwright, run `npm i playwright@<version>` in a scratch folder with a `package.json`. `<version>` is the output of `playwright --version`, because another version expects a browser build that is not in the cache.
- Over CDP, `browser.close()` only disconnects. Stop the Chrome process yourself.
- In the tmux harness, set `remain-on-exit on`, and wait for a screen pattern before each capture. In the PTY harness, read the reply in a loop, treat `OSError` as the end of the output, and call `proc.wait()`.

## Models

`/setup-pstack` writes the role lines to `~/.cursor/rules/pstack-models.mdc`. That file is "the `pstack-models.mdc` rule" that the skills name. When a skill names a role, read the file.

The aliases below are the only models that pstack uses. Use another model, such as agy's Claude models, only when the human names it.

| Alias | Harness | Family | Start command | Groups |
|---|---|---|---|---|
| opus | claude | Anthropic | `claude --model opus` | hardest, reviewer |
| astra | omp | OpenAI | `omp --model gpt-6-astra` | hardest (expensive) |
| sol | omp | OpenAI | `omp --model gpt-6.1-sol` | worker, reviewer, tester |
| sonnet | claude | Anthropic | `claude --model sonnet` | worker, tester |
| gemini | agy | Google | `agy --dangerously-skip-permissions --model gemini-3.8-flash-high` | worker, reviewer, tester |
| deepseek | opencode | DeepSeek | `opencode mini --model opencode/deepseek-v4.1-flash-free` | worker, reviewer, tester |
| haiku | claude | Anthropic | `claude --model haiku` | cheap worker |
| luna | omp | OpenAI | `omp --model gpt-6-luna` | cheap worker |

`/setup-pstack` maps the groups to pstack's roles:

- **hardest:** `hardest tasks`, `judgment and prose`, `how explainer`, `why synthesizer`, `reflect judgment, divergent, synthesizer`.
- **worker:** `feature, refactoring`, `bug-fix`, `perf-issue`, `hillclimb`, `arena runners`, `architect runners`.
- **reviewer:** `interrogate reviewers`, `arena cross-judge pool`.
- **cheap worker:** `how explorer`, `why investigators`, `reflect tooling`, `swarm workers`.
- **tester:** pstack has no tester role. The family rule below chooses the tester.

- In `/setup-pstack`, the detected models are the aliases above, plus `inherit-parent` and `auto`. Write each real model as `<alias> <effort>`, such as `sol medium`. The effort is the last token, so the budget step works as written.
- `poteto-mode` starts from the role line's effort and changes it for each spawn. Set the effort with `claude --effort <level>`, `omp --thinking <level>`, or the agy slug `gemini-3.8-flash-<low|medium|high>`.
- Never use fast mode: no `-fast` slug and no fast or priority speed setting. If a skill asks for fast mode, use standard speed.
- `inherit-parent` and `auto` run on your harness's own subagent tool, on the parent's model.
- If an alias is on your own harness and your subagent tool can select its model, use the subagent tool. Otherwise, run the alias's start command in a terminal and pass it to `worker-start --terminal <handle>`, because `--agent` cannot pass the other flags of a start command, such as omp's `--thinking` or agy's `--dangerously-skip-permissions`.
- A review of your own work runs on a model from another family. A test of your own work also runs on a model from another family. The harness does not matter. If a role line names only your own family, replace one entry with an alias from another family, and say so.
- If the file or the role line is missing, use `inherit-parent` for the role, and say so.

## Communication Style (ASD-STE100)

Write all new prose in Simplified Technical English, adapted from ASD-STE100: responses, reports, documentation, and pull request descriptions. The style reduces ambiguity: one meaning per word, one instruction per sentence, no decoration.

The style changes prose only, not engineering behavior. Do not change code, identifiers, quoted text, generated sections, required output formats, or the repository's commit message convention to fit it.

The approved-word dictionary is a licensed document and is not available to you. Do not claim conformance to it. Apply the writing rules below, and approximate the dictionary: choose the most common technical word for a thing, then use that same word every time.

### Words

1. **One term per concept.** Name a thing once, then repeat that exact name. Synonyms are errors, not variety.
2. **One meaning per word.** Do not reuse a word for a second sense in the same text.
3. **No noun cluster longer than three words.** Break a longer cluster with a preposition.
4. **No jargon, slang, idiom, or metaphor.** Use the plain technical word.
5. **Articles are required.** Write "the file", not "file". Do not drop articles to save space.

### Sentences

6. **One idea per sentence.** Aim for 20 words or fewer in a procedural sentence and 25 or fewer in a descriptive sentence. Split a long sentence where one idea ends. Do not drop an article, a qualifier, or a hedge to fit the length.
7. **Active voice.** Name the actor. Use the passive only when the actor is unknown or irrelevant.
8. **Simple tense, same certainty.** Use the simple present, past, or future. Keep the perfect tense when it states a present state ("I have not pushed the branch"). Keep the progressive tense when work is still running. Keep the strength of every statement: a hedge ("may", "should", "I would") stays a hedge, a conditional stays a conditional, a plan stays a plan, and a recommendation stays a recommendation. Do not turn one into a fact, an obligation ("must"), a done action, or an instruction to the user.
9. **No gerund chains.** Use an infinitive or a finite verb instead.
10. **Condition before action.** Write "If the container runs, stop it", not "Stop it if it runs".
11. **One topic per paragraph.** Start a new paragraph when the topic changes, usually by six sentences.
12. **Punctuation.** Write a hyphen `-`, never an em dash `—` or en dash `–`. Prefer a period, comma, colon, or parentheses where the dash carried structure.

### Procedures

Apply these when an operator follows the text step by step: runbooks, recovery steps, migration and deployment steps, and CLI help text. A thrown error message describes a failure and is not a procedure.

13. **One action per step**, in the imperative mood.
14. **Chronological order.** Number the steps.
15. **Give each step an end state** the reader can verify.
16. **Put a warning or caution before the step it protects**, never after.
17. **Use a vertical list** when a step has several conditions or parts.

### Descriptive text

18. **State facts.** Separate a fact from an opinion, and mark the opinion as one.
19. **Use a table only for short parallel facts:** each cell holds a word, a number, or a short phrase, and each row fits in 100 characters. For records with sentences, use a list: one bullet per record, the key in bold, and one sub-bullet per field.
20. Keep code, error messages, CLI flags, file paths, and identifiers verbatim. Never compress them.

### Exemptions

State these in full, and ignore the length limits if the limits reduce clarity: safety warnings, destructive-action confirmations, rollback instructions, and any explanation the user asked for (a report, a walkthrough, or per-phase notes).
