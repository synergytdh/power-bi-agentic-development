# Synergy overrides (synergytdh fork)

This fork is the marketplace source for Troy Heinemann's Claude Code install
(`extraKnownMarketplaces.power-bi-agentic-development` in the tracked
`settings.json` of the claude-config repo). Upstream is
[data-goblin/power-bi-agentic-development](https://github.com/data-goblin/power-bi-agentic-development)
(GPL-3.0; attribution and license unchanged). The fork exists so that a small
set of local overrides survives `claude plugin marketplace update`, which
pulls this repo, not upstream.

## What is overridden (2026-08-18, re-checked 2026-11-16 at upstream 26.45)

| Plugin | Change | Why |
|---|---|---|
| `custom-visuals` | `.mcp.json` removed (upstream declares a `pbiviz` MCP server: `npx -y powerbi-visuals-tools mcp`) | Every Claude Code session spawned an npx fetch plus a node process for a custom-visual toolchain that is not used here. The four visual skills (deneb, svg, python, r) do not need the server. Upstream 26.45 still declares it. |
| `fabric-cli` | `hooks/hooks.json` removed (module files under `hooks/` left in place for reference) | Added 2026-11-16. Upstream 26.45 registers a Claude Code mod (`{"modules": ["./register.tsx"]}`): a "Fabric in Claude Code" explorer pane that opens and follows every `fab` command and spawns its own processes to populate the tree. It loads in every session at user scope, and nobody here asked for an app pane. Re-add `hooks/hooks.json` with that one line if the pane is ever wanted. The 2026-08-18 override (the `fabric-sql` entry in `.mcp.json`) is gone: upstream deleted `.mcp.json` from this plugin entirely, so there is nothing left to override there. |
| `reports` | `hooks/hooks.json` removed (module files under `hooks/` left in place for reference) | Added 2026-11-16. Same mod pattern as fabric-cli: a "Report" pane that follows every `pbir` command, and Synergy sessions run `pbir` constantly during PBIP work. Re-add `{"modules": ["./register.tsx"]}` if the pane is ever wanted. |
| `pbip` | `hooks/hooks.json` removed (scripts left in place for reference) | The PostToolUse hooks ran three bash spawns on every Edit/Write in every project. Validation here is the repo's own pre-push stack (`pbir validate`, Tabular Editor 2 BPA, `validate-pbir.py`, `preview-pages.py`). Upstream 26.45 scopes these hooks with `if` filters (`*.Report/**`, `*.tmdl`, `definition.pbir`), so the every-edit cost is gone upstream; the override stays because the checks duplicate the pre-push stack. Revisit if that stack is ever retired. |
| `pbi-desktop` | `hooks/hooks.json` removed; `connect-pbid/SKILL.md` line about active hooks rewritten | The PreToolUse gates fired four bash spawns on every Bash call and, once `jq` is present, deny TOM/PowerShell commands (`.Measures.Add`, `-File *.ps1`) that the local scripts use. Upstream 26.45 scopes them with `if` filters, which ends the every-call cost but keeps the deny on exactly those commands, so the override stays. |

Overridden plugins carry a patch bump over upstream (26.45 -> 26.45.1 for
custom-visuals, fabric-cli, pbip, pbi-desktop, reports; 26.31.0 -> 26.31.1
before the 2026-11-16 sync) because `claude plugin update`
is version-keyed: with an unchanged number it reports "already at the latest
version" and never re-copies the content. Rule: every override change bumps
the patch number of the touched plugin; after an upstream sync, re-bump each
overridden plugin ABOVE upstream's new number. Un-overridden plugins keep
upstream's numbers, so an installed-vs-upstream comparison still reads
correctly for them.

## Syncing from upstream

    git remote add upstream https://github.com/data-goblin/power-bi-agentic-development.git   # once
    git fetch upstream
    git merge upstream/main

Expect a modify/delete conflict whenever upstream touches one of the removed
files (`plugins/custom-visuals/.mcp.json`, `plugins/pbip/hooks/hooks.json`,
`plugins/pbi-desktop/hooks/hooks.json`, `plugins/fabric-cli/hooks/hooks.json`,
`plugins/reports/hooks/hooks.json`), a content conflict on each overridden
plugin's `.claude-plugin/plugin.json` version line (take upstream's number,
then re-bump), and, before the 2026-11-16 sync, a content conflict on
`plugins/fabric-cli/.mcp.json`. That is deliberate: it forces a look at what
upstream changed before the override is re-applied. Resolve by keeping the
deletion, re-check the table above (including any NEW `hooks/hooks.json` or
`.mcp.json` upstream added to an installed plugin, since those load in every
session), and push. Land the sync with a merge commit, never a squash: a
squash drops upstream's commits from the fork's history and the next sync
re-conflicts on everything. Do not use `gh repo sync --force`; it would drop the overrides.

After a sync lands on `main`, refresh the local installs:

    claude plugin marketplace update power-bi-agentic-development
    claude plugin update <plugin>@power-bi-agentic-development      # per installed plugin
