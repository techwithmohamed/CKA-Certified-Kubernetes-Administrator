# Contributing

Thanks for considering a contribution. This repo is my personal CKA study guide, but fixes and additions are welcome.

## How to Contribute

1. Fork the repo
2. Create a branch (`git checkout -b fix/your-fix`)
3. Make your changes
4. Run `bash scripts/validate-local.sh` to lint any YAML you added
5. Test any commands on Kubernetes v1.35
6. Commit (`git commit -m "fix: description"`)
7. Push and open a PR

## Commit Convention

Use a prefix so the changelog stays readable:

- `fix:` — broken command, bad YAML, typo, dead link
- `feat:` — new exercise, skeleton, troubleshooting scenario
- `docs:` — README edits, comments, wording changes
- `chore:` — CI, tooling, repo housekeeping

Examples: `fix: correct etcd restore flags in exercise 07`, `feat: add exercise 18 — CSI snapshots`

## What's Helpful

- Fixing outdated commands or YAML
- Correcting typos or broken links
- Adding practice exercises that match CKA exam style
- Improving explanations where something is unclear
- Sharing exam feedback via the issue template

## What to Avoid

- Do not submit real exam questions. Sharing actual exam content violates the CNCF exam policy.
- Do not add AI-generated filler text
- Do not add content for certifications other than CKA
- Do not refactor working YAML for style preference — if it works, leave it

## Style

- First person, casual tone — matches the rest of the repo
- No emojis
- Valid YAML and working kubectl commands
- Use the aliases defined in `scripts/exam-setup.sh` (k, do, now)

## Local Validation

Before pushing, run:

```bash
bash scripts/validate-local.sh
```

This checks YAML syntax across `skeletons/` and `exercises/`. CI runs the same checks, but catching errors locally is faster.

## Questions?

Open an issue. I'll get back to you.
