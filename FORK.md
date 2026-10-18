# Fork notes

This fork of [mariusflorescu/pstack-t3](https://github.com/mariusflorescu/pstack-t3) adds two things:

- **`org-context`.** A new skill. It queries Unblocked (CLI first, MCP second) for decisions, rejected approaches, team rules, in-flight work, and incidents, and writes a cited brief. The Feature, Bug fix, Refactoring, and Perf issue playbooks run it before step 1.
- **Tiered `interrogate`.** Light (at most 200 changed lines, at most 5 files, no risky path) runs one reviewer at `high` effort. Deep runs the `interrogate reviewers` panel at `max`, a blast-radius child, and a refute pass. No reviewer runs on the author's model. New role: `interrogate light reviewer`. The code playbooks run `interrogate` on every change, not only contested designs.

Install: `bunx skills add jeffrey-ng/pstack-t3 -g -a claude-code -a codex -s '*' -y`.

Sync upstream: `git fetch upstream && git merge upstream/main`, then `node scripts/check-t3-native.mjs`. Conflicts land in the files above.
