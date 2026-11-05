# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file static web app (`index.html`) that translates switch commands between Cisco (Catalyst IOS-XE, Nexus NX-OS) and Juniper (EX, QFX Junos). No build step, no dependencies, no tests, no linter. HTML, CSS and JS all live in `index.html`; the only external load is Google Fonts.

## Commands

- Serve locally or on the LAN: `./serve.sh [port]` (default 8080, wraps `python3 -m http.server`)
- Syntax-check the JS: extract the contents of the `<script>` block to a file and run `node --check` on it
- Deploy: push to `main`. `.github/workflows/pages.yml` publishes only `index.html` to GitHub Pages (live at https://jdiveley.github.io/cisco-juniper-command-map/). Any new file the page needs must be added to the workflow's "Stage site" step, which currently copies just `index.html`.
- Check a run with `gh run list --limit 1`. A failure in `configure-pages` means Pages is not enabled (Settings > Pages > Source: GitHub Actions).
- Pushing `.github/workflows/*` needs a GitHub token with the `workflow` scope (`gh auth refresh -s workflow`).

## Architecture (all inside the IIFE in `index.html`)

**Command database.** `add(id, cat, kind, title, ciscoLines, junosLines, note)` registers an entry; multi-line commands are `\n`-separated strings. `kind` is `show` or `cfg`. Entry order matters: the UI emits a category heading each time `cat` changes, so keep entries of one category contiguous, and categories appear in order of first use.

**Variants and capabilities.** The base `cisco` lines are IOS-XE. Platform-specific Cisco lines are layered on afterwards:
- `V(id, { nxos, n7k, svl })` adds Cisco variants. Each model in the `CISCO` list has a `keys` array that is searched in order (the Nexus 7009 uses `["n7k","nxos"]`, the 9400/9500 use `["svl"]`).
- `R([ids], "poe" | "stack" | "vpc")` marks entries that need a platform capability. A model lacking the flag gets a greyed "not available" pane instead of commands. A matching variant wins over the capability check. The Junos flag names are `poe`, `stack`, `vpc` (the last means MC-LAG).
- The `V(...)` and `R(...)` calls sit in one block just before the troubleshooting playbooks and must come after the `add(...)` they refer to (they throw on an unknown id).

**Placeholders.** Commands use `{port}`, `{jport}`, `{portlong}`, `{vlan}`, `{vlanname}`, `{ip}`, `{prefix}`, `{mask}`, `{remote}`, `{mac}`, `{maccisco}`, `{ae}`, `{aecount}`. `derive()` computes them from the "Your values" inputs and the selected models, and invalid input falls back to defaults and sets a `bad` flag. A new placeholder needs a key in `derive()`.

**Port translation.** A port is parsed into a canonical `{m, s, n, spd}` (member, fixed or uplink slot, port, speed) by `parseAny()`, then rendered per platform by `fmtCisco()` and `fmtJunos()` using the model rules in `CISCO` and `JUNIPER` (uplink PIC numbers in `up`, `dc: true` for data-center models where all ports are on PIC 0). Adding a model is an entry in one of those two lists.

**Rendering.** `sideData()` picks the lines for one side and one model; `pane()`, `row()`, `renderResults()` and `renderTS()` build HTML strings. Search (`search()`) indexes the commands from every variant regardless of the selected model, so pasting any platform's command finds its row. The `TS` array defines the troubleshooting playbooks as ordered `[entryId, "what to look for"]` steps that reuse database entries.

**State.** One `state` object, saved to `localStorage` key `cjmap.v2` (direction, selected models, values). Bump the key if the stored shape changes incompatibly.

## Conventions worth knowing

- Cisco and Junos pairings should be the closest functional analog, not a loose match. Examples already settled: `show ip interface brief` pairs with `show interfaces terse` (NX-OS: `show interface brief`), while `show interfaces status` pairs with `show interfaces brief` plus `show ethernet-switching interface`.
- Junos configuration lines are `set` statements and need `commit`. The pane label adds "commit to apply" automatically for Junos `cfg` entries.
- Command syntax has not been verified on real hardware. Entries with uncertain syntax carry a caution in their `note`.
