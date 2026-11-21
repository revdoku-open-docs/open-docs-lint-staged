---
title: Tasks and commands
description: Which commands a lint-staged task can run, how lint-staged starts them without a shell, how to run commands in sequence or at the same time, and how to prevent conflicts between tasks.
---

A task is the value of a glob pattern in the configuration. It can be one command, a list of commands or a function. This page is about commands. For functions, see [JavaScript configuration](/js-config).

## Which commands work

A command can be:

- A program from `node_modules/.bin`, such as `eslint` or `prettier`. lint-staged looks for it first in the folder of the configuration file, then in the parent folders.
- A program on your `PATH`, such as `git` or `gofmt`.
- An npm script: `npm run lint --`. The `--` sends the file paths to the script.
- A script file of your project, such as `./scripts/check-size.sh`.

Install the tools that your tasks run as dev dependencies of the project. If a task uses a program that only you installed globally, the hook fails for other people.

## lint-staged does not use a shell

lint-staged starts each command directly, without a shell. It splits the command into arguments at the spaces, and text in quotes stays one argument. Then it adds the file paths as more arguments. Thus these shell features do not work in a command:

| Shell feature | Example that does not work |
| --- | --- |
| Pipes and redirects | `eslint --fix \| tee log.txt` |
| Two commands in one string | `eslint --fix && prettier --write` |
| Environment variables | `NODE_ENV=test jest` |
| Glob expansion by the shell | `rm *.tmp` |

Use these replacements:

| You want | Use |
| --- | --- |
| Two commands | An array: `["eslint --fix", "prettier --write"]` |
| An environment variable | A tool such as `cross-env`: `cross-env NODE_ENV=test jest --findRelatedTests` |
| Shell syntax | A script file. The script gets the file paths as arguments, in `"$@"`. |

An example script file:

```bash
#!/bin/sh
# scripts/check-size.sh: fail if a staged file is larger than 1 MB
for file in "$@"; do
  if [ "$(wc -c < "$file")" -gt 1048576 ]; then
    echo "Too large: $file"
    exit 1
  fi
done
```

```json
{
  "*": "./scripts/check-size.sh"
}
```

Make the script executable with `chmod +x scripts/check-size.sh`.

The `--shell` option was removed in lint-staged 16. See [Upgrade notes](/migration#v16).

### Node.js scripts on Windows

On Windows, a `.js` file is not a program. Start Node.js scripts with `node`:

```json
{
  "*.js": "node scripts/check-headers.js"
}
```

## Commands in sequence

An array of commands runs one command after the other, on the same files. If a command fails, the commands after it do not run.

```json
{
  "*.ts": ["eslint --fix", "prettier --write"]
}
```

Put the commands that change files and the commands that only check files in an order that makes sense. For example, run `eslint --fix` before `prettier --write`, so that Prettier formats the output of ESLint.

## Commands at the same time

There are two levels of concurrency.

Different glob patterns run at the same time. In this configuration, ESLint and Stylelint start together:

```json
{
  "*.ts": "eslint",
  "*.css": "stylelint"
}
```

In one glob pattern, an array in the array runs its commands at the same time. Since lint-staged 17.3, this works:

```json
{
  "*.js": [["eslint --max-warnings=0", "vitest related --run"], "prettier --write"]
}
```

lint-staged runs ESLint and the related Vitest tests at the same time. When both pass, it runs `prettier --write`.

To limit the number of tasks that run at the same time, use `--concurrent <number>`. To run all tasks one after the other, use `--concurrent false`.

## Prevent conflicts between tasks

Two tasks that change the same file at the same time can damage it. This happens when glob patterns overlap and both commands write files:

```json
{
  "*": "prettier --write",
  "*.ts": "eslint --fix"
}
```

A staged `.ts` file matches both patterns, so Prettier and ESLint edit it at the same time. To fix this, make the patterns exclusive, and put the commands for one file type in one array:

```json
{
  "!(*.ts)": "prettier --ignore-unknown --write",
  "*.ts": ["eslint --fix", "prettier --write"]
}
```

Overlapping patterns are not a problem when the commands only read files, for example `eslint` without `--fix` and `prettier --check`.

## Changes that tasks make

lint-staged stages all changes that the tasks make to tracked files, and the commit includes them. You do not need `git add` in a task. Do not add it: a `git add` in a task can conflict with the Git commands of lint-staged.

To fail the commit when a task changes files, use `--fail-on-changes`. See [How it works](/how-it-works).

## Long lists of files

An operating system limits the length of a command line. If you stage many files, lint-staged divides the file list into parts, and it runs the command once for each part. The limits are:

| Platform | Default limit (characters) |
| --- | --- |
| Windows | 8,191 |
| macOS | 262,144 |
| Linux and other systems | 131,072 |

lint-staged measures the full command string, not only the file list. A command that gets no file paths, such as a function that returns `tsc --noEmit`, is not divided. To change the limit, use `--max-arg-length`. See [CLI flags](/cli#--max-arg-length).
