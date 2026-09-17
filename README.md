# MIRROR — DO NOT COMMIT

This repository is a **mirror** of `odoo/odoo`, maintained by dear digital so that
the Comoo deployment on DeltaBlue can pin Odoo core to an exact commit.

## Rules

1. **Never commit to `19.0`.** That branch must stay an exact, fast-forward
   continuation of upstream. Any extra commit breaks the next sync.
2. **Never force-push or delete `19.0`.** Commits pinned by `dear-digital/comoo`
   live there. Making them unreachable breaks every future deploy, silently,
   until the next build fails to check out a submodule.
3. **Never delete a `deployed/*` tag.** Those tags anchor the commits that
   production is running.

Branch protection and a tag ruleset enforce 1-3. This README exists because a
README stops people before they act; a ruleset only stops them afterwards.

## Branches

| Branch | Purpose |
|---|---|
| `19.0` | Exact copy of upstream `odoo/odoo` `19.0`. Machine-maintained. |
| `mirror-admin` | This branch. Holds the README and the sync workflow. **Default branch.** |

`mirror-admin` is an **orphan** branch: it shares no history with `19.0`. It has
to be, because the workflow file cannot live on `19.0` without breaking the
fast-forward push. It is the repository's default branch because GitHub only
runs scheduled workflows from the default branch.

## Syncing

`.github/workflows/sync.yml` runs daily and can be started by hand from the
Actions tab (**Run workflow**).

Syncing **never changes what any environment runs.** It is safe at any time.
Only moving a submodule pin in `dear-digital/comoo` changes a deployment.

Deliberate choices in the workflow, do not "simplify" them away:

- **no `--mirror`, no `--prune`, `--no-tags`** — each of those would delete the
  `deployed/*` tags that keep pinned commits alive.
- **no `--force`** — the push is fast-forward only. If upstream ever rewrites
  history the job fails loudly instead of destroying pinned commits.
- the job **verifies** the new tip is a descendant of the old one before pushing.

## Related

- `dear-digital/comoo` — the superproject that pins this repository as a submodule.
- Odoo Knowledge: *Comoo — DeltaBlue hosting* for the full procedures.
