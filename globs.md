---
title: Filtering files
description: How lint-staged matches staged files to glob patterns, which paths it gives to tasks, how to ignore files and how to match files outside the project folder.
---

Each key of a lint-staged configuration is a glob pattern. A glob pattern is a text pattern for file paths, such as `*.js`. lint-staged uses [picomatch](https://github.com/micromatch/picomatch) to match the staged files to these patterns.

## Patterns without a slash

If a pattern has no slash (`/`), it matches the file name in any folder:

| Pattern | Matches | Does not match |
| --- | --- | --- |
| `*.js` | `test.js`, `src/lib/test.js` | `test.ts` |
| `*.{js,jsx}` | `app.js`, `src/App.jsx` | `app.ts` |
| `!(*test).js` | `foo.js` | `foo.test.js` |
| `!(*.css\|*.js)` | `README.md`, `logo.svg` | `app.css`, `app.js` |

## Patterns with a slash

If a pattern has a slash, it matches the path from the folder of the configuration file:

| Pattern | Matches | Does not match |
| --- | --- | --- |
| `./*.js` | `test.js` | `src/test.js` |
| `src/**/*.js` | `src/test.js`, `src/lib/test.js` | `test.js` |

## Braces with one value

A brace pattern with only one value, such as `*.{js}`, matches nothing in picomatch. lint-staged corrects it and shows a warning:

```text
⚠ Detected incorrect braces with only single value: `*.{js}`. Reformatted as: `*.js`
```

Write `*.js` to remove the warning.

## How lint-staged selects the files

For each run, lint-staged:

1. Finds the root folder of the Git repository. You do not configure it.
2. Gets the staged files that are inside the project folder.
3. Matches them to the glob patterns of the closest configuration file.
4. Gives the absolute paths of the matching files to the tasks.

lint-staged gives absolute paths, because the Git root and the folder of `package.json` can be different. To give paths relative to the current folder, use `--relative`.

By default, lint-staged includes files that are added, copied, modified or renamed. It does not include deleted files. To change this, use `--diff-filter`. See [CLI flags](/cli#--diff-filter).

## Ignore files

lint-staged always gives the tasks all staged files that match. To ignore files, configure the tool that the task runs.

For example, a code generator writes files to `src/generated/`, and you commit them. Add `src/generated/` to the ignore file of your formatter, such as `.prettierignore`. The formatter then skips these files, also when lint-staged gives them to it.

If the tool cannot ignore files, use a negated glob pattern, or filter the paths in a function. See [Ignore files from a match](/js-config#ignore-files-from-a-match).

For ESLint warnings about ignored files, see [ESLint and ignored files](/guides/eslint-ignored-files).

## Files outside the project folder

By default, lint-staged runs tasks only on files inside the project folder. The project folder is the folder where lint-staged runs. This matters only if the project folder is a subfolder of the Git repository.

To match files outside the project folder, start the pattern with `../`:

```json
{
  "../backend/**/*.py": "black"
}
```

Patterns such as `*.js` and `**/*.js` still match only the files in the project folder. For an example project, see [lint-staged-django-react-demo](https://github.com/sudo-suhas/lint-staged-django-react-demo).
