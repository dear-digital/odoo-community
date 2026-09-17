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

### How it works

The default path is **server-side**. A fork and its parent share an object store
on GitHub, so the upstream commits already exist on GitHub's side; the job simply
asks GitHub to move our `19.0` ref (`POST /repos/.../merge-upstream` - the API
behind the "Sync fork" button). It takes seconds and transfers nothing.

Before doing that it compares the two branches and **only proceeds when the
status is `behind`**, meaning our tip is an ancestor of upstream's. Anything else
(`ahead`, `diverged`) fails the job rather than creating a merge commit on
`19.0`, which would break the mirror.

### The git fallback

Dispatch the workflow with **method = `git`** to use the original route: clone
the branch, fetch upstream, push a fast-forward. That downloads the full branch
history - about 1.3 GB and a few minutes - so it is not the daily path, but it is
the known-good one if the API route ever fails.

Deliberate choices in that path, do not "simplify" them away:

- **no `--mirror`, no `--prune`, `--no-tags`** - each would delete the
  `deployed/*` tags that keep pinned commits alive.
- **no `--force`** - the push is fast-forward only, so a rewritten upstream fails
  loudly instead of destroying pinned commits.
- the job **verifies** the new tip is a descendant of the old one before pushing.

## Related

- `dear-digital/comoo` — the superproject that pins this repository as a submodule.
- Odoo Knowledge: *Comoo — DeltaBlue hosting* for the full procedures.
