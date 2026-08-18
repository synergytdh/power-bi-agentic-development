# Synergy overrides (synergytdh fork)

This fork is the marketplace source for Troy Heinemann's Claude Code install
(`extraKnownMarketplaces.power-bi-agentic-development` in the tracked
`settings.json` of the claude-config repo). Upstream is
[data-goblin/power-bi-agentic-development](https://github.com/data-goblin/power-bi-agentic-development)
(GPL-3.0; attribution and license unchanged). The fork exists so that a small
set of local overrides survives `claude plugin marketplace update`, which
pulls this repo, not upstream.

## What is overridden (2026-08-18)

| Plugin | Change | Why |
|---|---|---|
| `custom-visuals` | `.mcp.json` removed (upstream declares a `pbiviz` MCP server: `npx -y powerbi-visuals-tools mcp`) | Every Claude Code session spawned an npx fetch plus a node process for a custom-visual toolchain that is not used here. The four visual skills (deneb, svg, python, r) do not need the server. |
| `fabric-cli` | `fabric-sql` entry removed from `.mcp.json` (`microsoft-learn` kept) | Its `Authorization: Bearer ${FABRIC_PBI_TOKEN}` header with an unset variable produced a 401 at every session start and disables Claude Code's OAuth fallback for that endpoint. If the Fabric SQL endpoint MCP is ever wanted, re-add the entry WITHOUT the header so OAuth can run. |
| `pbip` | `hooks/hooks.json` removed (scripts left in place for reference) | The PostToolUse hooks ran three bash spawns on every Edit/Write in every project. Validation here is the repo's own pre-push stack (`pbir validate`, Tabular Editor 2 BPA, `validate-pbir.py`, `preview-pages.py`). |
| `pbi-desktop` | `hooks/hooks.json` removed; `connect-pbid/SKILL.md` line about active hooks rewritten | The PreToolUse gates fired four bash spawns on every Bash call and, once `jq` is present, deny TOM/PowerShell commands (`.Measures.Add`, `-File *.ps1`) that the local scripts use. |

Overridden plugins carry a patch bump over upstream (26.31.0 -> 26.31.1 for
custom-visuals, fabric-cli, pbip, pbi-desktop) because `claude plugin update`
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
`plugins/pbi-desktop/hooks/hooks.json`) and a content conflict on
`plugins/fabric-cli/.mcp.json`. That is deliberate: it forces a look at what
upstream changed before the override is re-applied. Resolve by keeping the
deletion (or the fabric-cli file without `fabric-sql`), re-check the table
above, and push. Do not use `gh repo sync --force`; it would drop the overrides.

After a sync lands on `main`, refresh the local installs:

    claude plugin marketplace update power-bi-agentic-development
    claude plugin update <plugin>@power-bi-agentic-development      # per installed plugin
