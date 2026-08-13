# repo-template

A minimal starting point for a new `imrohitagrawal` repo: a PR quality gate
(lint/typecheck/test/security scanning, via
[`imrohitagrawal/.github`](https://github.com/imrohitagrawal/.github)'s
reusable workflow), a Codex review config, a CODEOWNERS file, and Dependabot.
That's it — this repo does not contain `.github`'s own documentation,
internal `templates/`, or a copy of the reusable workflow's implementation.
It only has the small set of files a fresh repo actually needs to start
consuming that workflow.

## What's here

```text
.github/workflows/pr-quality.yml   caller workflow, pinned to imrohitagrawal/.github's v7 tag by commit SHA
.github/CODEOWNERS                 default owner (* @imrohitagrawal) -- edit as the repo grows
.github/dependabot.yml             github-actions + npm + pip update tracking
AGENTS.md                          Codex's review config (severity rules, checklist, output format)
Makefile.python                    discrete lint/typecheck/test/security targets for a Python repo
Makefile.node                      discrete lint/typecheck/test/build/audit targets for a Node repo
```

## How to use this template

**Option A -- GitHub's "Use this template" button.** On
`imrohitagrawal/repo-template`'s GitHub page, click **Use this template ->
Create a new repository**. This copies every file above into your new repo
with a fresh git history (no shared commits with this template, unlike a
fork). You still need to do the setup steps below afterward -- the button
does not configure branch protection or enable Codex for you.

**Option B -- copy the files by hand** into an existing repo, or use
`imrohitagrawal/.github`'s retrofit script (`scripts/retrofit-quality-gate.sh`
in that repo, part of the same work that produced this template -- check
that repo directly, since it may still be on a feature branch rather than
`main` depending on when you're reading this) if the target repo already
exists and has some of these files already; it automates the copy step and
won't silently overwrite anything that differs from these templates.

## Setup steps after creating a repo from this template

This template gets you through the file-copying part of onboarding. It does
**not** get you a working gate by itself -- you still need to:

1. **Pick your Makefile.** If your repo is Python, rename `Makefile.python`
   to `Makefile` and delete `Makefile.node`. If it's Node, do the reverse.
   If it's a monorepo with both a Python and a Node component, you'll need
   to merge the two by hand into one `Makefile` with `lint`/`typecheck`/
   `test` targets that dispatch into the right subdirectory (e.g.
   `make -C backend lint`) -- this template deliberately does not guess that
   merge for you, since silently picking one stack's Makefile would suppress
   the other stack's checks with the gate still reporting green. You'll also
   need to add `python-directory: backend` / `node-directory: frontend`
   (or whatever your subdirectories are named) to the `with:` block of
   `.github/workflows/pr-quality.yml`'s `quality-gate` job -- without those
   inputs, the reusable workflow only looks for manifests at the repo root
   and won't find either stack in a subdirectory.
2. **Open a PR** and let the `quality-gate` check run once. A green check on
   a brand-new repo does not by itself prove every listed check ran and
   would have caught a real problem -- several checks in the reusable
   workflow are conditional (e.g. native npm scripts run only if the script
   exists in `package.json`; security scanners are advisory, not blocking,
   until you opt into promotion; see `imrohitagrawal/.github`'s own gate
   severity table). If you want real confidence the gate blocks something,
   introduce a deliberate failure (a bad lint rule, a failing test) and
   confirm the check actually goes red before relying on green meaning
   anything.
3. **Configure branch protection** to require the `quality-gate` check --
   see `imrohitagrawal/.github`'s
   [`docs/branch-protection.md`](https://github.com/imrohitagrawal/.github/blob/main/docs/branch-protection.md).
   Without this step, a red check is visible but doesn't block merge. If
   you're the sole maintainer, do **not** also turn on "require review from
   code owners" -- GitHub won't let you approve your own PR, so that setting
   deadlocks every PR unless you configure an explicit bypass, at which
   point it isn't really enforcing anything. `.github/CODEOWNERS` is still
   useful for routing review *requests* even without that setting.
4. **Enable Codex review** for the repo (requires a paid ChatGPT Plus
   subscription or higher) -- see
   [`docs/codex-pr-review.md`](https://github.com/imrohitagrawal/.github/blob/main/docs/codex-pr-review.md).
5. **Open a test PR** to confirm the check goes green and Codex actually
   posts a review comment, before considering the repo onboarded.

None of this is instant. Budget real time for steps 3-5 in particular --
they involve two separate products (GitHub Settings and the ChatGPT UI) and
can't be automated by copying files. If you want the full picture of what
each step does and why, read
[`imrohitagrawal/.github`'s onboarding doc](https://github.com/imrohitagrawal/.github/blob/main/docs/repo-onboarding.md),
which this template exists to shorten, not replace.

## Staying current

`.github/workflows/pr-quality.yml` pins the reusable workflow by commit SHA
(with a `# v7` comment for readability), not the mutable tag, per
`imrohitagrawal/.github`'s versioning policy. When a new tag ships, bump the
SHA and the comment together -- see the comment above the `uses:` line in
that file for the exact command.

`.github/dependabot.yml` in this template tracks `github-actions`, `npm`,
and `pip` on a weekly cadence with minor/patch grouping. It's a starting
point, not a complete config for every repo shape -- delete whichever of the
`npm`/`pip` blocks doesn't apply to your repo (an ecosystem with no matching
manifest just finds nothing, but a config entry for a directory that will
never have one is dead weight worth removing), and add ecosystems (e.g.
Docker) or per-directory targeting as your repo actually needs them.

## Before you start writing your own code

This file (`README.md`) describes the template, not your project. Once
you've done the setup steps above, replace this README's content with your
own project's description -- otherwise your new repo will keep presenting
itself as `repo-template` to anyone who visits it.
