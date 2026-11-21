---
title: Upgrade notes
description: What to change when you update lint-staged, from version 10 to 17.6 - Node.js and Git minimums, removed options, and behavior changes in minor releases.
---

This page lists the changes that can need work from you when you update lint-staged. It includes behavior changes in minor releases, not only in major releases. For all changes, read the [release notes](https://github.com/lint-staged/lint-staged/releases) and the [official migration file](https://github.com/lint-staged/lint-staged/blob/main/MIGRATION.md).

## Minimum versions

| lint-staged | Node.js | Git |
| --- | --- | --- |
| 17 | 22.22.1 | 2.32.0 (checked at each run) |
| 16.1 | 20.17.0 | none checked |
| 16.0 | 20.19.0 | none checked |
| 15 | 18.12.0 | none checked |
| 14 | 16.14.0 | none checked |
| 13 | 14.13.1 | none checked |
| 12 | 12.20.0 | none checked |
| 10 | 10.13.0 | none checked |

To see your versions, run `node --version` and `git --version`.

## Changes in 17.x minor releases

| Version | Change | What to do |
| --- | --- | --- |
| 17.6.0 | lint-staged stages all changes that tasks make to tracked files, also in files that you did not stage. | If a task edits more than the staged files, use `--hide-unstaged`, or limit the task to the staged files. |
| 17.6.0 | Task objects get a `log()` function. | Optional. See [Run JavaScript code as a task](/js-config#run-javascript-code-as-a-task). |
| 17.5.0 | lint-staged stops when files are staged with `git add -N`. | Stage these files fully. |
| 17.5.0 | Tools in `node_modules/.bin` are found from the folder of each configuration file again. | Nothing. This fixes monorepos on 16.3 to 17.4. |
| 17.4.0 | New `defineConfig` helper and new `--all` flag. | Optional. |
| 17.3.0 | An array in a task array runs at the same time. | Optional. |
| 17.2.0 | Task functions are no longer divided into parts for long file lists. | Nothing. |
| 17.1.1 | `--max-arg-length=Infinity` turns off the division into parts. | Optional. |

## v17

- Update Node.js to 22.22.1 or later.
- Update Git to 2.32.0 or later. lint-staged stops with an error if Git is older.
- If you use a YAML configuration file, install the `yaml` package. It is optional now. A `.lintstagedrc` file without an extension is YAML: rename it to `.lintstagedrc.json` if it contains JSON.

## v16

- The `--shell` option was removed. Move shell syntax into a script file, or use a function that returns the command. See [lint-staged does not use a shell](/tasks#lint-staged-does-not-use-a-shell).
- If you used `bash -c 'tsc --noEmit'` to avoid file paths, use a function: `'*.ts': () => 'tsc --noEmit'`.
- lint-staged starts processes with a different library. On Windows, start Node.js scripts with `node`.
- The advanced options of the configuration file from version 8 and earlier are not checked any more. lint-staged can read them as glob patterns. Remove them.

## v13.3.0

Version 13.3.0 contains the code of 14.0.0 by mistake. Its Node.js minimum is 16.14.0, like version 14.

## v12

lint-staged is an ES module since version 12. To load it in CommonJS code, use `await import('lint-staged')`.

## v10

- lint-staged stages the task changes itself. Remove `git add` from your tasks.
- lint-staged makes a backup stash before each run. A stash needs one commit, so make the first commit before you use lint-staged.
- lint-staged stops the commit when the tasks undo all staged changes. Use `--allow-empty` to allow it.
