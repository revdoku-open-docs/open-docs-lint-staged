---
title: TypeScript type checks
description: Run the TypeScript compiler from lint-staged without file paths, so that tsc uses tsconfig.json - and what error TS5112 means.
---

To check types before a commit, run `tsc --noEmit` from a function that does not pass file paths:

```js
// lint-staged.config.js
export default {
  '*.{ts,tsx}': () => 'tsc --noEmit',
}
```

The command runs once when one or more TypeScript files are staged. It checks the full project with the settings in `tsconfig.json`.

## Why the string form fails

With the string form, lint-staged adds the staged files to the command:

```json
{
  "*.ts": "tsc --noEmit"
}
```

lint-staged then runs `tsc --noEmit /path/to/a.ts`. When you give `tsc` file names, it does not read `tsconfig.json`.

Current TypeScript versions stop with an error in this case. We tested TypeScript 7.0.2:

```text
✖ tsc --noEmit
error TS5112: tsconfig.json is present but will not be loaded if files are specified on commandline. Use '--ignoreConfig' to skip this error.
```

Older TypeScript versions show no error. They check the files with default settings, so you get wrong errors, such as:

```text
TS17004: Cannot use JSX unless the '--jsx' flag is provided.
```

`--ignoreConfig` removes the TS5112 error, but `tsc` still ignores your settings. Use the function form.

## Check types and lint at the same time

```js
// lint-staged.config.js
export default {
  '*.{ts,tsx}': [() => 'tsc --noEmit', 'eslint --max-warnings=0 --no-warn-ignored'],
}
```

The function returns a command without file paths, and the ESLint command gets the staged files. The two commands run one after the other. To run them at the same time, put them in an inner array:

```js
// lint-staged.config.js
export default {
  '*.{ts,tsx}': [[() => 'tsc --noEmit', 'eslint --max-warnings=0 --no-warn-ignored']],
}
```

If one of the two fails, lint-staged stops the other one.

## Speed

`tsc --noEmit` checks the full project, so it can take some seconds in a large project. Options:

- Use `"incremental": true` in `tsconfig.json`. Then `tsc` reuses the result of the last check.
- Run the type check in CI only, and keep the pre-commit hook for fast tools.
