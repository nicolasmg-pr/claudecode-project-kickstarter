---
name: feature-start
description: Start a new feature with clean context and a clean git branch. Use when the user says "new feature", "let's build X next", "start working on X", or reaches a feature boundary.
argument-hint: "[short feature description]"
---

# Feature Start

Rule: **new feature = new context = new branch.**

## Steps

1. Context check: if this session already carries unrelated work, tell the user to run `/clear` and invoke this skill again — a fresh conversation for a fresh feature. (You cannot run `/clear` for them.)
2. Run `git status`. If the tree is dirty, ask: commit, stash, or abort. Never branch over uncommitted work silently.
3. Create the branch: short, descriptive, kebab-case, named after the feature — `add-payments`, `user-profile-page`, `dark-mode`. No ticket numbers or dates. `git checkout -b <name>` from the main branch unless told otherwise.
4. Restate the feature in 2-3 sentences and confirm scope before writing any code.
5. During the feature: delegate test runs to `test-runner` and commit messages to `commit-writer`.

## If the feature goes wrong midway

Throw the branch away and start clean — that is what it is for. The rest of the project stays untouched.
