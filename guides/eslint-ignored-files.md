---
title: ESLint and ignored files
description: Stop the ESLint warning "File ignored because of a matching ignore pattern" from failing a lint-staged commit, with --no-warn-ignored or a filter for old ESLint versions.
---

lint-staged gives ESLint every staged file that matches the pattern, also files that your ESLint configuration ignores. For each ignored file, ESLint shows a warning. With `--max-warnings=0`, that warning fails the commit:

```text
src/generated/out.js
  0:0  warning  File ignored because of a matching ignore pattern. Use "--no-ignore" to disable file ignore settings or use "--no-warn-ignored" to suppress this warning

✖ 1 problem (0 errors, 1 warning)

ESLint found too many warnings (maximum: 0).
```

## ESLint 9 and later

Add `--no-warn-ignored`. ESLint then skips ignored files without a warning:

```json
{
  "*.{js,ts}": "eslint --max-warnings=0 --no-warn-ignored"
}
```

This works with the flat configuration (`eslint.config.js`), which is the only format in ESLint 10. In ESLint 8, the flag exists since 8.51.0, and only with a flat configuration.

We tested this with ESLint 10.15: the ignored file gives no warning, and real errors in other files still fail the commit.

## ESLint 8 with an .eslintrc file

The flag does not work with an `.eslintrc` configuration. Remove the ignored files in a function before ESLint runs:

```js
// lint-staged.config.js
import { ESLint } from 'eslint'

const eslint = new ESLint()

const withoutIgnored = async (files) => {
  const ignored = await Promise.all(files.map((file) => eslint.isPathIgnored(file)))
  return files.filter((_, i) => !ignored[i])
}

export default {
  '*.{js,ts}': async (files) => {
    const kept = await withoutIgnored(files)
    return kept.length ? `eslint --max-warnings=0 ${kept.join(' ')}` : []
  },
}
```

`isPathIgnored` returns a promise in ESLint 7 and later. A simpler fix is to update to a current ESLint version and use `--no-warn-ignored`.
