# Contributing

This is a personal-use fork. See `LICENSE` for what's allowed.

## SKILL.md changes

SKILL.md is loaded into the agent's context on every invocation. Keep these principles in mind:

- **Every added line costs tokens** on every future use. Prefer moving verbose content to `references/` and adding a one-line pointer + routing-table entry.
- **Every rule should trace to a real failure.** If you add a constraint, note why (even one sentence is enough).
- Cross-references use the form `references/xxx.md` (relative to repo root). Absolute paths break on different installs.

## Adding a reference file

1. Create `references/your-topic.md`
2. Add a row to the routing table in `SKILL.md` (currently around line 724–748)
3. Link from the relevant section of SKILL.md using `references/your-topic.md`

## Adding a showcase sample

The showcase matrix is 8 scenes × 3 styles = 24 entries. See `assets/showcases/INDEX.md` for the naming convention and directory structure. Each entry needs both an `.html` source and a `.png` screenshot at the documented resolution.

## Script changes

Shell scripts (`scripts/*.sh`) use `set -e`. Keep that, and add binary existence checks (`command -v <tool>`) for any new external tool dependencies — see `add-music.sh` for the pattern.
