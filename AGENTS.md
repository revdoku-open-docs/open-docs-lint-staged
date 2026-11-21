# AGENTS.md

This repository is the source of the unofficial lint-staged community docs, published by Revdoku as a docs site (Mintlify format: `docs.json` plus `.md` and `.mdx` pages). Revdoku keeps this file in the bucket but never publishes it.

## Rules

1. The site is unofficial. Keep the banner in `docs.json`, the note on `index.mdx` and the page `about.md`. Use "community docs". Do not use the word "official" for this site, and do not use the lint-staged logo.
2. Write original text. Do not copy or lightly reword text from the lint-staged README, `MIGRATION.md`, the changelog or the release notes. Write from the source code and from your own test runs.
3. Test each command and configuration example with the version in `about.md` before you publish it. Do not publish an example that you did not run.
4. Before each push, run the overlap check from the open-docs workspace:
   `open-docs/bin/check-overlap open-docs/lint-staged open-docs/_upstream/lint-staged/README.md open-docs/_upstream/lint-staged/MIGRATION.md open-docs/_upstream/lint-staged/CHANGELOG.md`
   It must pass (5% or less of shared 8-word sequences on each page).
5. Plain English (ASD-STE100 based): short sentences, active voice, no "should/may/might/could", no "below/above" for places in the text, bold only for interface labels.
6. Keep `docs.json` in step with the pages. Each page has `title` and `description` front matter. Check each link and anchor.
7. When lint-staged releases a version that changes behavior, update the pages, the version in `about.md`, `index.mdx` and the banner, and the check date.
8. Never put secrets or private data in this repository. Each file except agent files, dot files and scripts is published.

## Format

- Components (`Steps`, `Card`, `CardGroup`, `Note`, `Warning`, `Tabs`, `CodeGroup`, `Accordion`) only in `.mdx` files. MDX runs no code, and `{expressions}` outside code are dropped.
- `revdoku.json` holds the Ask welcome line (plain text, no links, 200 characters) and up to 8 starter questions (90 characters each).
- Format reference: https://docs.revdoku.com/docs-sites.md
