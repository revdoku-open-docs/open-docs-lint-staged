---
title: Troubleshooting
description: Causes and fixes for lint-staged error messages, hook output without colors, IDE commits, files staged with --intent-to-add, a missing backup and lost changes.
---

Find the message that you see, then read its fix. To see more detail for any problem, run `npx lint-staged --debug`.

## lint-staged could not find any valid configuration

```text
✖ lint-staged could not find any valid configuration.
```

- Make sure that the configuration file is in the project folder or in a parent folder, and that Git does not ignore it.
- If `package.json` holds the configuration, make sure that the key is `lint-staged`.
- If the file is YAML, install the `yaml` package. See [YAML files need the yaml package](/configuration#yaml-files-need-the-yaml-package).
- If a JavaScript file has a syntax error, lint-staged first shows `✖ Failed to read config from file`. Fix the error.

## lint-staged could not find any staged files

```text
→ lint-staged could not find any staged files.
```

This is not an error. You ran lint-staged with nothing staged. Stage files with `git add`, or use `--all` or `--diff`.

A similar message, `could not find any staged files matching configured tasks`, means that no staged file matches a glob pattern. Make sure that the patterns are correct. See [Filtering files](/globs).

## Failed to get staged files

```text
✖ Failed to get staged files!
```

With `--diff`, Git cannot use the revisions that you gave. Often, a CI job has a shallow clone without the target branch. See [Run in CI](/guides/ci#github-actions).

## lint-staged refused to run with files staged with `--intent-to-add`

```text
✖ lint-staged refused to run with files staged with "--intent-to-add":

- /home/me/project/new-file.js
```

`git add -N` (intent to add) records a new file without its content. A Git stash cannot store these files, so lint-staged stops (since 17.5.0). Stage the file fully with `git add new-file.js`, or remove it from the index with `git reset new-file.js`.

## lint-staged prevented an empty git commit

The tasks undid all staged changes, for example a formatter removed your only change. To make the empty commit anyway, use `--allow-empty`. See [How it works](/how-it-works#when-the-tasks-undo-all-staged-changes).

## Skipping backup because there is no initial commit

```text
⚠ Skipping backup because there’s no initial commit yet. This might result in data loss.
```

A backup stash needs one commit. Make the first commit without lint-staged, for example with `git commit --no-verify`. After that, lint-staged makes backups.

## A command works in the terminal but fails in lint-staged

lint-staged does not use a shell. Pipes, `&&`, environment variables and shell globs in a command do not work. See [lint-staged does not use a shell](/tasks#lint-staged-does-not-use-a-shell).

On Windows, start a JavaScript file with `node`: `node scripts/check.js`, not `scripts/check.js`.

## Some of your tasks use git add

```text
⚠ Some of your tasks use `git add` command. Please remove it from the config since all modifications made by tasks will be automatically added to the git commit index.
```

Since lint-staged 10, lint-staged stages the task changes itself. Remove `git add` from the configuration. A `git add` in a task can conflict with the Git commands of lint-staged.

## lint-staged staged files that I did not stage

Since 17.6.0, lint-staged stages all changes that the tasks make to tracked files. If a task edits a file that you did not stage, for example `prettier --write .`, that file goes into the commit. Make the task work only on the staged files, or use `--hide-unstaged` to hide your other changes while the tasks run.

## The hook output has no colors or shows lines twice

Git 2.36.0 changed how hooks connect to the terminal, and Git 2.37.0 fixed it. lint-staged 17 needs Git 2.32.0 or later, so a version in this range is possible. Update Git first.

If that does not help, send the hook output to the terminal. In `.husky/pre-commit`:

```shell
if sh -c ": >/dev/tty" >/dev/null 2>/dev/null; then exec >/dev/tty 2>&1; fi

npx lint-staged
```

To force colors in other cases, set the `FORCE_COLOR=1` environment variable (since 17.5.0).

## Commits from an IDE do not include the task changes

Current JetBrains IDEs run Git hooks like the command line. With old versions, the commit can contain the files without the changes that the hook staged. Update the IDE. If you cannot update, add a `post-commit` hook that runs `git update-index --again`.

## Angular: ng lint checks the full project

`ng lint` checks a full Angular project, so it does not fit lint-staged. Run `ng lint` directly in the pre-commit hook, or run ESLint on the staged files through lint-staged. See issue [#951](https://github.com/lint-staged/lint-staged/issues/951).

## My changes are missing after a failed run

Look for the backup stash:

```bash
git stash list --format="%h %s"
```

Then apply it:

```bash
git apply --index <hash>
```

If you used `--fail-on-changes`, the task changes are in your working tree, not lost. See [Get lost changes back](/how-it-works#get-lost-changes-back).

## Two runs at the same time damaged the repository

Do not run lint-staged twice at the same time in one working tree, for example from a hook and a file watcher. Some Git commands lock the repository. Separate Git worktrees are safe since 17.6.0.

## Still a problem?

Search the [lint-staged issues](https://github.com/lint-staged/lint-staged/issues). If you found a bug in lint-staged, report it there with the output of `npx lint-staged --debug`. If a page on this site is wrong, [report it here](https://github.com/revdoku-open-docs/open-docs-lint-staged/issues).
