**中文** · [README.md](./README.md)

<!-- badges -->
[![Node](https://img.shields.io/badge/Node.js-≥18-339933?logo=node.js&logoColor=white)](https://nodejs.org)
[![Zero-Deps](https://img.shields.io/badge/dependencies-zero-0f9d58?logo=npm&logoColor=white)](https://www.npmjs.com/)
[![License](https://img.shields.io/github/license/huanweide/docdoctor)](LICENSE)
<!-- /badges -->

# docdoctor

A Markdown documentation health check — a zero-dependency, single-file CLI that scans your repo's `.md` files for common smells (dead links, skipped heading levels, missing image alt text, bare URLs, and more), reports a 0–100 health score, and drops straight into CI as a gate.

No dependencies to install, no network, no config file. Just run `node index.js`.

## Who should use it / Who shouldn't

- **Use it if** you maintain a `docs/` folder or lots of `.md` files; want to block dead links / missing alt / skipped headings in CI; or don't want to wire up a whole ESLint / remark config.
- **Skip it if** you already use markdownlint + markdown-link-check and are happy with their coverage (it still works as a complement); or your docs are almost entirely external links with nothing to validate locally.

## Why it exists

Dead links, skipped heading levels, images without alt — everyone trips over these doc smells, yet existing tools each cover only part of it: linkbust only checks local dead links, markdownlint needs a full ESLint/remark setup, axe needs a browser DOM. docdoctor picks a narrow but practical angle: **a zero-dependency single file that integrates multiple doc-health dimensions into one check-up**, with a health score and CI gate included.

It shares the same philosophy as devdoctor (project check-up), a11ydoctor (Web a11y), and repodoctor (git repo hygiene) in the family: no zero-dependency single-file tool integrated multi-dimensional doc health before.

## Install

No install. Just run it with Node:

```bash
node index.js [path...] [options]
```

Or link it globally:

```bash
npm link   # optional, then use the docdoctor command
```

Requires Node >= 18 (uses lookbehind assertions, etc.).

## Usage

```bash
# Scan all .md in the current directory (recursive)
node index.js

# Scan a specific directory or file
node index.js ./docs README.md

# Machine-readable output
node index.js --json

# CI gate: fail on any high-severity issue
node index.js --fail-on-high

# Tune oversized-doc threshold and issue caps
node index.js --max-lines 2000 --max-issues 10 --max-high 0
```

## Detection dimensions

| Rule | Severity | Description |
| --- | --- | --- |
| D1 dead link / anchor | high / medium | Relative path points to a missing local file (high); `#anchor` has no matching heading in the target (medium) |
| D2 oversized doc | medium / low | Doc exceeds a line threshold (suggest splitting); files over 256KB skip content scan |
| D3 heading skip | low | Broken hierarchy, e.g. jumping H1 → H3 |
| D4 anchor conflict | low | Multiple identical slug headings in one doc; GitHub anchors would point to the wrong place |
| D5 image missing alt | medium | Accessibility: image without alt text |
| D6 bare URL | low | Inline http(s) address not wrapped in a Markdown link (exempt inside code blocks) |
| D7 missing ref definition | medium | Uses ``[text][ref]`` but no matching ``[ref]: url`` definition |

All checks run outside fenced code blocks (``` or ~~~) and inline code (`` `...` ``), so examples in your docs never cause false positives.

## Health score

Per-file weighted deduction: high×3 + medium×2 + low×1. Each file is allowed 2 weighted issues for free; beyond that, each point costs 8, floored at 0. The repo health score is the average across all files.

```
health = max(0, 100 - max(0, weighted_issues - 2) × 8)
```

## CI gate

Exit code semantics:

- `0`: healthy (no high-severity doc issues)
- `1`: high-severity doc issue present (e.g. dead link)
- `2`: CI gate tripped (`--fail-on-high` fired, or issue / high counts exceeded `--max-*`), or usage error

GitHub Actions example:

```yaml
name: docs-health
on: [push, pull_request]
jobs:
  doctor:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npx -y https://github.com/huanweide/docdoctor.git --fail-on-high ./docs
```

## How it differs from alternatives

- **linkbust**: zero-dependency but only checks local dead links. docdoctor treats dead links as one of many dimensions, also covering heading structure, doc size, image alt, bare URLs, and reference links, with a health score and CI gate — its scope is "overall doc health", not "dead-link specialist".
- **markdownlint / remark**: require installation and rule-set config, geared toward "style rules". docdoctor is zero-dependency, scans out of the box, and leans toward "health check-up + gate".
- **axe / markdown-link-check**: the former needs a browser DOM, the latter needs the network for external URLs. docdoctor is fully local and deterministic — CI won't go red because some external site went down.

## Example output

```text
✗ docs/guide.md (health 82)
  L12 [HIGH] D1 link points to missing local file: ./old-api.md
  L5  [MED ] D5 image missing alt text: ![](/img/logo.png)

summary: 3 files · 1 high · 1 medium · 0 low · health 88
```

## Roadmap

Rule set is stable. Possible future additions: a TOC/anchor lint preset and JSON-schema validation for front-matter. No breaking changes planned.

## Contributing

Issues and pull requests are welcome. Keep it single-file and zero-dependency.

## ⭐ Star

If docdoctor ever saved you from shipping a doc with a dead link, a Star ⭐ helps others find it: https://github.com/huanweide/docdoctor

## Author

[huanweide](https://github.com/huanweide)

## License

MIT
