---
title: "Getting tests into git core"
description: "git merge-base --is-ancestor is used in scripts everywhere and had no tests. Here's what it took to add some — and what the git contribution process is actually like."
date: 2026-08-03
tags: ["open-source", "git", "testing"]
draft: false
---

`git merge-base --is-ancestor A B` answers a yes/no question — is A an ancestor of B? — and returns it as an exit code: `0` for yes, `1` for no. Shell scripts lean on it constantly ("has this branch been merged yet?"). It's load-bearing plumbing.

It also had no tests.

I noticed while reading `t/t6010-merge-base.sh` for something unrelated. There were tests for `--all`, for the octopus cases, for the plain merge-base — but nothing exercising `--is-ancestor` at all. No test pinned the exit-code contract that every script depends on. So I wrote some.

## What the tests actually cover

Three things, all about the contract rather than the happy path:

- **exit 0** when A is an ancestor of B
- **exit 1** when it isn't (this is the one scripts branch on — it must not be conflated with an error)
- **exit 128**, not 1, when you hand it a bad argument — because a script that treats "error" as "not an ancestor" will silently do the wrong thing

Plus two guard cases: `--is-ancestor` can't be combined with `--all`, and the error message names *both* options so the user knows what conflicted; and it requires exactly two commits (too few is a usage error, too many is rejected explicitly).

None of this changes behavior. It documents and locks the behavior that was already there — the kind of test that only matters the day someone refactors the option parser and quietly breaks the exit codes.

## Contributing to git is not a pull request

Here's the part that surprised people when I described it: git doesn't take GitHub pull requests. The `github.com/git/git` repo is a read-only mirror. Development happens on a **mailing list** — you send patches as emails, and review happens as email replies.

I used [GitGitGadget](https://gitgitgadget.github.io/), a bot that bridges the two: you open a PR against its fork, it turns your commits into a properly-formatted patch series and posts them to the list for you, then relays the review back onto the PR. It handles the mechanics (`git format-patch`, threading, the `Signed-off-by` trailer) so you can focus on the change.

The review itself is old-school and exacting. Two rounds:

- **v1** got feedback from Phillip Wood ("thanks for adding some tests") and from Junio Hamano, the maintainer — reuse the existing `E---D---C---B---A` history in the test file instead of setting up a fresh repo; add a case for the exactly-two-commits requirement.
- **v2** addressed both. Junio's reply: *"OK. Will queue."*

## The "cooking" pipeline

git integrates changes through a sequence of branches, and watching your topic move through them is oddly satisfying:

1. **`seen`** — proposed, integrated for testing. (Your patch shows up here first.)
2. **`next`** — accepted, cooking toward the next release.
3. **`master`** — shipped.

My topic, `ns/merge-base-is-ancestor-tests`, went the whole distance: after cooking in `next`, Junio [merged it to `master`](https://github.com/git/git/commit/89454a60ed3c). The tests are now part of git itself.

## Why bother

A test-only change to a 20-year-old tool isn't glamorous. But it's exactly the kind of contribution that's *welcome* — it's low-risk, it's obviously correct, it fills a real gap, and it respects the maintainers' time. That's the whole trick to contributing to a project you didn't start: find the thing that's clearly missing, make it small, make it verifiable, and follow the local customs to the letter — even when the local customs are a mailing list and a bot from 2019.

More on how I pick those changes — and how I've landed 30-odd of them across the CNCF, Python, and Rust ecosystems — in a later post.
