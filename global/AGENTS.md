# Agent instructions

This machine runs Claude Code, omp, and agy, usually inside Orca. The pstack skills are installed verbatim from upstream, which was written for Cursor. Follow them as written, with the two substitutions below. Where a pstack skill conflicts with this file, this file wins.

## Policy

- **Branches.** Push only branches you created for the current task, and `--force-with-lease` only on those. Never push to a default or protected branch (`main`, `master`, `develop`, release branches), and never merge a PR or MR. Hand the human the exact command and wait. Server-side branch protection and token roles enforce this too. These lines do not replace them.
- **Destructive actions.** Before deleting, overwriting, force-updating, or rewriting history, databases, deployments, remote state, credentials, or user data: verify the exact target, establish a rollback path, and confirm the human authorized that exact action. If any of the three is uncertain, stop and ask.
- **Redmine** is read-only. Write to Redmine only when the human explicitly asks for that write in the current conversation. Otherwise, do not update issues, add notes or comments, change statuses, log time, or upload attachments. Return the proposed text for the human to paste.
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
- A review or a test of your own work always runs on `others strong`.
- If the file or the role line is missing, use `self strong` for code and `others strong` for review. Say so.

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
