---
title: CLI flags
description: Reference of all lint-staged 17.6 command line flags, with their defaults, what each flag changes and which flags turn on other flags.
---

```bash
npx lint-staged [options]
```

lint-staged exits with code 0 when all tasks pass. It exits with code 1 when a task fails, when it stops an empty commit, or when `--fail-on-changes` finds changes.

## All flags

| Flag | Short | Default | What it does |
| --- | --- | --- | --- |
| [`--all`](#--all) | | off | Runs tasks on all tracked files, not only staged files |
| [`--allow-empty`](#--allow-empty) | | off | Allows a commit when the tasks undo all staged changes |
| [`--concurrent <number\|boolean>`](#--concurrent) | `-p` | `true` | Sets how many tasks run at the same time |
| [`--config [path]`](#--config) | `-c` | search | Uses one configuration file, or `-` for standard input |
| [`--continue-on-error`](#--continue-on-error) | | off | Runs all tasks to the end, also after a task fails |
| [`--cwd [path]`](#--cwd) | | current folder | Runs all tasks in this folder |
| [`--debug`](#--debug) | `-d` | off | Shows debug information |
| [`--diff [string]`](#--diff) | | `--staged` | Gets the file list from a `git diff` that you choose |
| [`--diff-filter [string]`](#--diff-filter) | | `ACMR` | Sets which kinds of changed files are included |
| [`--fail-on-changes`](#--fail-on-changes) | | off | Fails when a task changes a tracked file |
| [`--no-hide-partially-staged`](#--no-hide-partially-staged) | | hide is on | Does not hide unstaged changes of partially staged files |
| [`--hide-unstaged`](#--hide-unstaged) | | off | Hides the unstaged changes of all tracked files |
| [`--hide-all`](#--hide-all) | | off | Hides all unstaged changes and untracked files |
| [`--max-arg-length [number]`](#--max-arg-length) | | platform limit | Sets the maximum length of one command line |
| [`--quiet`](#--quiet) | `-q` | off | Shows only the output of the tasks |
| [`--relative`](#--relative) | `-r` | off | Gives relative paths to the tasks |
| [`--no-revert`](#--no-revert) | | revert is on | Does not restore the original state after an error |
| [`--no-stash`](#--no-stash) | | stash is on | Does not make a backup stash |
| [`--verbose`](#--verbose) | `-v` | off | Shows the output of tasks that pass |
| `--help` | `-h` | | Shows the help text |
| `--version` | `-V` | | Shows the version number |

The help text says `default: 0` for `--max-arg-length`. Since lint-staged 17.0.5, the real default is the limit of your operating system. See [`--max-arg-length`](#--max-arg-length).

## Flags that turn on other flags

| If you use | lint-staged also uses |
| --- | --- |
| `--all` | `--no-stash` and `--allow-empty` |
| `--diff` | `--no-stash` |
| `--no-stash` | `--no-revert` |
| `--fail-on-changes` | `--no-revert` |
| `--hide-all` | Replaces `--hide-unstaged` and the default hiding of partially staged changes |
| `--hide-unstaged` | Replaces the default hiding of partially staged changes |

A flag that you give yourself wins over these implied values. For example, `--all --stash` makes a backup stash.

## --all

Includes all files that Git tracks, not only the staged files. Git exclusions, such as `.gitignore`, still apply. Use it to run the tasks on the full project, for example in CI or after you change the configuration:

```bash
npx lint-staged --all
```

## --allow-empty

If the tasks undo all staged changes, lint-staged stops the commit by default. With this flag, lint-staged lets Git make the empty commit.

## --concurrent

Sets how many tasks run at the same time:

| Value | Effect |
| --- | --- |
| `true` | No limit. This is the default. |
| `false` | One task at a time |
| A number, such as `2` | At most this number of tasks at the same time. `1` is the same as `false`. |

This flag controls the tasks of different glob patterns. The commands in one array still run one after the other.

## --config

Uses this configuration file and does not look for others. The value can be a file path or the name of an npm package. If the file does not exist, lint-staged shows an error.

With `-`, lint-staged reads JSON from standard input:

```bash
cat ci-lint-staged.json | npx lint-staged --config -
```

## --continue-on-error

By default, lint-staged stops the other tasks when one task fails. With this flag, all tasks run to the end, and lint-staged fails after that. Use it to see all errors at once.

## --cwd

Runs all tasks in this folder, not in the folder of each configuration file. The path can be absolute or relative to the current folder. In a monorepo, `--cwd packages/web` limits the run to one package. See [Monorepos](/guides/monorepo).

## --debug

Shows the configuration files that lint-staged found, the parsed options, the files it matched, the tasks it runs and each Git command. Use it when a task does not run.

## --diff

Gets the list of files from `git diff` with the revisions that you give, in place of `git diff --staged`. lint-staged does not make a backup stash with this flag.

```bash
npx lint-staged --diff="main...my-branch"
```

With three dots, the list contains the changes on `my-branch` since it split from `main`. With two dots, it also contains the changes on `main` that `my-branch` does not have. See [Run in CI](/guides/ci).

## --diff-filter

Sets which kinds of changes are included. The default is `ACMR`: added, copied, modified and renamed files. The other letters of `git diff --diff-filter` are `D` (deleted), `T` (type changed), `U` (unmerged), `X` (unknown) and `B` (pairing broken).

## --fail-on-changes

If a task changes a tracked file, lint-staged does not stage the change. It fails with exit code 1, and the commit stops. The changes stay in your working tree, so you can look at them and stage them yourself:

```text
✖ lint-staged failed because `--fail-on-changes` was used.
```

The backup stash stays. Use this flag when tasks must only check files, for example in CI.

## --no-hide-partially-staged

By default, lint-staged hides the unstaged changes of partially staged files while the tasks run. With this flag, the tasks see these changes, and lint-staged stages them with the task changes.

## --hide-unstaged

Hides the unstaged changes of all tracked files while the tasks run, not only of partially staged files. lint-staged restores them after the tasks. Use it when a task changes files that you did not stage.

## --hide-all

Hides all unstaged changes and all untracked files while the tasks run. The backup stash includes the untracked files, and lint-staged restores them after the tasks. Use it for tools that read the full project, such as Knip, which finds unused files and exports.

## --max-arg-length

Sets the maximum length of one command line. If a command with all file paths is longer, lint-staged runs it more than once, with part of the files each time. The default limits are:

| Platform | Default |
| --- | --- |
| Windows | 8,191 |
| macOS | 262,144 |
| Linux and other systems | 131,072 |

To never divide the file list, use `--max-arg-length=Infinity` (since 17.1.1).

## --quiet

Turns off the progress output of lint-staged. The output of failed tasks and the error messages still show.

## --relative

Gives the tasks file paths relative to the folder where lint-staged runs, not absolute paths:

```text
/home/me/project/src/app.js    (default)
src/app.js                     (with --relative)
```

## --no-revert

By default, lint-staged restores the original state after a task fails. With this flag, it does not restore it. It stages the changes that the tasks made, and then it stops the commit.

## --no-stash

Does not make the backup stash, and thus also does not restore the original state after an error. After a task fails, the changes that the tasks made stay in the index. lint-staged still hides the unstaged changes of partially staged files. Use this flag only if you have another backup.

## --verbose

Shows the output of all tasks, also of the tasks that pass. By default, lint-staged shows only the output of failed tasks.
