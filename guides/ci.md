---
title: Run in CI
description: Run lint-staged in a CI pipeline on the files that a pull request changes, with --diff, a GitHub Actions workflow, the full history that the diff needs, and --all for the full project.
---

A pre-commit hook runs only on the computer of each developer, and a developer can skip it with `git commit --no-verify`. To make sure that the checks also pass for each pull request, run lint-staged in CI.

In CI, nothing is staged. Give lint-staged a list of files in a different way:

| Goal | Command |
| --- | --- |
| Check the files that a branch changed | `npx lint-staged --diff="origin/main...HEAD"` |
| Check all tracked files | `npx lint-staged --all` |

Both flags turn off the backup stash, so lint-staged does not change the Git state of the CI job.

## Check the changes of a pull request

`--diff` uses `git diff` with the revisions that you give. With three dots, `origin/main...HEAD` lists the files that changed on the current branch since it split from `main`:

```bash
npx lint-staged --diff="origin/main...HEAD"
```

To see the list before you run the tasks, run the same `git diff`:

```bash
git diff --diff-filter=ACMR --name-only origin/main...HEAD
```

If you use two dots, the list also contains the files that changed on `main` after the branch split. Usually you do not want these files.

You can also compare with the commit where the branch split:

```bash
npx lint-staged --diff="$(git merge-base origin/main HEAD)"
```

## Use commands that only check

In CI, a task that changes files does not help, because nobody commits the result. Use check commands, or let the job fail when a task changes a file:

| Local command | CI command |
| --- | --- |
| `prettier --write` | `prettier --check` |
| `eslint --fix` | `eslint` |
| `stylelint --fix` | `stylelint` |

If you keep one configuration for both, run CI with `--fail-on-changes`. Then the job fails when a task changes a file. A second configuration file for CI is also possible, with `--config`.

## GitHub Actions

```yaml
# .github/workflows/lint-staged.yml
name: lint-staged
on: pull_request

jobs:
  lint-staged:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
        with:
          fetch-depth: 0
      - uses: actions/setup-node@v7
        with:
          node-version: lts/*
          cache: npm
      - run: npm ci
      - run: npx lint-staged --diff="origin/${{ github.base_ref }}...HEAD" --fail-on-changes
```

- `fetch-depth: 0` gets the full Git history and all branches. By default, the checkout action gets only one commit and no other branches. Then `origin/main` is missing, and lint-staged stops with `✖ Failed to get staged files!`.
- `github.base_ref` is the target branch of the pull request, such as `main`.
- `lts/*` selects the newest LTS version of Node.js. lint-staged 17.6 needs Node.js 22.22.1 or later.

## Check the full project

`--all` runs the tasks on all tracked files. Use it after you change the lint-staged configuration or a linter rule, or in a scheduled job:

```bash
npx lint-staged --all --fail-on-changes
```

`--all` also implies `--allow-empty`. On a large project, a full run can be slow. lint-staged then divides long file lists into parts. See [Long lists of files](/tasks#long-lists-of-files).
