---
name: org-context
description: "Gather organizational context from Unblocked before any code change: past decisions, rejected approaches, team rules, in-flight work, and incidents for the area you will touch. Writes a cited brief that later delegates and reviewers read. Use for /org-context, \"org context\", \"what does the team know about X\", or as the first step of a code playbook."
disable-model-invocation: true
---

# Org context

A skill named in bold or in backticks, or run as `/<name>`, is the file `../<name>/SKILL.md` beside this skill's directory, and a principle named without its prefix is `../principle-<name>/SKILL.md`. Read the file and follow it. Most pstack skills are user-only, so your skill tool cannot load them.

Learn what the organization already knows about an area before you change it. The code shows what exists. Unblocked shows why it exists, what failed before, what the team expects, and who else works there. Run this before the first edit. It is read-only.

## Step 1, Target

Name the area with concrete identifiers: the repo as `owner/repo` (`gh repo view --json nameWithOwner -q .nameWithOwner`), the files or subsystem, the key symbols, and any ticket, PR, or doc URL the task names. Queries with real identifiers work. Keyword queries return noise.

## Step 2, Access

Run `command -v unblocked` once. If it resolves, use the CLI. Otherwise, if `context_research` is in your tool list, call the MCP tools with the same arguments. Otherwise write `org context: unavailable, Unblocked not configured` in the brief and the reply, tell the user to run `unblocked auth` or authorize the Unblocked MCP server, and continue the task. Never block on it. Never substitute web search.

## Step 3, Query in parallel

One objective per call, all calls at once. Write each query as a full question with the identifiers from Step 1.

| Objective | Call |
|---|---|
| Decisions and rejected approaches | `unblocked context-research --effort high --query "What design decisions, tradeoffs, and rejected approaches shaped <area> in <owner/repo>?" --instruction "Prefer PR discussions, design docs, and decision threads."` |
| Team rules | `unblocked context-get-rules --repo-name <owner/repo> --task code-generation --paths <files>` |
| In-flight work | `unblocked context-research --effort medium --query "What open PRs, open issues, and in-progress work touch <area> in <owner/repo>?"` |
| Incidents and user reports | `unblocked context-research --effort medium --query "What incidents, bugs, and user reports involved <area>, and how were they resolved?"` |
| The task's own spec | `unblocked context-get-urls --urls <url> ...`, only when the task names a ticket, PR, or doc URL. |

Mine the first results for stronger identifiers (PR numbers, file paths, owners) and run at most one follow-up round, only for gaps that change the plan. Results reflect the default branch. Check any claim about code against the local files.

## Step 4, Write the brief

Write `~/.local/state/pstack/org-context/<repo>-<branch>.md`, with `/` in the branch replaced by `-`. Overwrite it on a re-run. Use these sections and drop any with nothing to say:

- **Preserve.** Decisions and constraints the change must keep.
- **Avoid.** Approaches that were tried and failed or were rejected, with the reason.
- **Rules.** Team rules that apply to these files.
- **In-flight.** Open PRs, issues, or owners that overlap the change. An overlap goes to the user before the first edit.
- **Open questions.** What the sources did not settle.
- **Sources.** One line per call: what it returned, or `nothing found`. A null result is a finding.

Every line carries its source URL. Never invent a URL. Label an inference as an inference.

## Step 5, Hand off

Give the brief's absolute path to every later code delegate and every `interrogate` reviewer as a file pointer. In the reply, say in two or three sentences what the brief changed about the plan, or that it changed nothing.

Skip this skill only for a purely mechanical edit (typo, format, a rename with no behavior change), and record `skip: mechanical edit` in the task list. For the full history behind one decision, use `why`.
