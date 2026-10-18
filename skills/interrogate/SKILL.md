---
name: interrogate
description: "Use for \"interrogate\", \"adversarial review\", \"multi-model review\", \"challenge this\", \"stress test this code\", \"find blind spots\", or \"tear this apart\". Reviewers on models other than the author's challenge changes, one for a small diff and a max-effort panel with a refute pass for a large or risky one. Force a tier with \"light\" or \"deep\"."
disable-model-invocation: true
---

# Interrogate

A skill named in bold or in backticks, or run as `/<name>`, is the file `../<name>/SKILL.md` beside this skill's directory, and a principle named without its prefix is `../principle-<name>/SKILL.md`. Read the file and follow it. Most pstack skills are user-only, so your skill tool cannot load them. Every `delegate_task` call follows the defaults in the Subagents section of `../poteto-mode/SKILL.md`.

Spawn adversarial reviewers on models other than the author's, one for a small change and a max-effort panel for a large or risky one. The adversarial signal comes from model diversity, not assigned personas. The deliverable is a synthesized verdict. Do NOT auto-apply changes.

## Step 1, Scope

Identify what to review from context:

- If the user points at specific files or a diff, use that.
- If on a feature branch, run `git diff main...HEAD` (or the appropriate base branch) for the full changeset.
- If the user's message references recent work, gather the relevant files.

Package the diff or file contents with any surrounding context files the reviewers need to understand the code.

## Step 2, Intent

Before spawning reviewers, state the intent in one clear paragraph from the user's message, commit messages, the PR description if one exists, and the code. If you're unsure about the intent, ask the user before proceeding.

## Step 3, Tier

Pick the tier before you spawn. `/interrogate light` or `/interrogate deep` overrides the rule.

- **Light.** At most 200 changed lines (added plus deleted, from `git diff --shortstat`), at most 5 files, and no risky path.
- **Deep.** Everything else.

A risky path touches auth or permissions, secrets or crypto, billing, a database migration or schema, concurrency or locking, a public API or exported contract, deploy or infra config, or data deletion. An **org-context** brief that names an incident in the area also makes it Deep. State the tier and the reason in one line.

## Step 4, Spawn Reviewers

**Never review with the author's model.** The author model wrote the diff: the code role's entry when a playbook delegate wrote it, else the parent thread's model. Every reviewer runs on a different model, from a different provider when `orchestrator_capabilities` lists one with `canRunChildTask`. When only the author's provider is left, use its most capable other model and say so.

- **Light.** One reviewer, labeled Reviewer A, from the `interrogate light reviewer` line in `~/.config/pstack/pstack-models.md`, at `high` effort. If the line is missing or names the author model, use the most capable non-author model per the rule above.
- **Deep.** One reviewer per entry on the `interrogate reviewers` line in `~/.config/pstack/pstack-models.md`, labeled Reviewer A, B, and so on, at `max` effort. Replace an entry that names the author model with the most capable model of a provider not yet on the panel, or drop it when none is left. The panel keeps at least two reviewers. If the file or that line is missing, take the most capable non-author model of each provider that `orchestrator_capabilities` reports with `canRunChildTask`, up to three, and add the most capable other model of the author's provider when that leaves fewer than two. Say so and suggest `/setup-pstack`. In the same message, spawn one more child titled `interrogate blast-radius` on a non-author model. Its brief gives the absolute path of `../blast-radius/SKILL.md` to follow in full, says to write scratch scripts only under a fresh temp directory, and forbids repo edits and git commands that change state.

Launch every child in a single message. Each reviewer gets the title `interrogate reviewer <label>`, a brief that says read-only: inspect only, no writes, no git commands, and `target` set from its entry per the Subagents section. Set the effort the tier names, or the nearest option the model lists, and keep the provider and model. Omit `target` for an `auto` or `inherit-parent` entry, so that reviewer runs on the parent model, unless the parent model is the author model. Then pick a non-author model instead.

If `delegate_task` rejects a configured entry, run that reviewer on the closest non-author model of the same provider that `orchestrator_capabilities` lists and say so. Do not block the review on it.

Read `references/reviewer-prompt.md` and fill in the same template for every reviewer with the stated intent, the diff or file contents, the absolute path of the **org-context** brief (or `none`), the repo as `owner/repo`, the rubric from `references/rubric.md`, and the code-quality lens from `references/code-quality-review.md`.

## Step 5, Synthesize

As results come back, build a unified picture. Parse every reviewer's findings. Merge findings that describe the same issue differently, and note which models raised each one. Findings two or more models raised independently are the highest signal. Read a lone model's finding, but weight it accordingly. Note disagreements. If one model flags something and another explicitly says the opposite, that is context for the verdict. Fold the blast-radius report in as findings of its own.

## Step 6, Refute (Deep only)

Every finding headed for **Act on** gets a refuter. Group the candidates by the model that raised them. For each group, spawn one fresh child titled `interrogate refuter <model>` on a panel model that raised none of them, at `max` effort, read-only. Its brief holds the findings, the diff, and the org-context brief path, and asks it to disprove each finding: trace the real call path, and run code when that is cheap. It returns `confirmed`, `refuted`, or `unclear` per finding, each with its evidence. A refuted finding goes to **Dismissed** with the refuter's evidence. An unclear one goes to **Consider**.

## Step 7, Lead Judgment

You are the lead reviewer, a pragmatic senior engineer, not a neutral aggregator. Read `references/lead-judgment.md` for the full framework. Put every finding in one of four buckets, with the model(s) that raised it and a one-line rationale. A finding without a concrete failure scenario cannot be **Act on**.

- **Act on**. Real issues affecting correctness, security, or maintainability given the actual goals. They would block a real PR.
- **Consider**. Legitimate, but you're not sure they outweigh the cost of addressing them now. Worth the user's attention.
- **Noted**. Technically valid but not actionable. Context-dependent, premature optimization, or low-impact at this stage.
- **Dismissed**. Wrong, nitpicky, or missing context, with a brief reason.

## Output Format

### Intent
> [The stated intent paragraph from Step 2]

### Tier
[Light or Deep, the reason, and the author model]

### Reviewers
- Reviewer [label]: [model name], [N findings] (one bullet per reviewer, plus blast-radius and refuters on Deep)

### Act On
[Each: description, which models raised it, why it matters.]

### Consider
[Each: description, which models raised it, the tradeoff.]

### Noted
[Brief list.]

### Dismissed
[Each with a brief rationale.]

### Agreement Map
[Where models agreed and diverged, and what that pattern tells us.]
