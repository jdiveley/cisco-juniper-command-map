# Cisco-Juniper Switch Command Map

Single-page tool that translates commands between Cisco and Juniper switches, with
troubleshooting playbooks. Pick a model on each side.

- Cisco: Catalyst 3850, 9300, 9400, 9500 (IOS-XE); Nexus 5548, 7009, 9300 (NX-OS)
- Juniper: EX2300, EX3400, EX4300, EX4400, EX4650, EX9200, QFX5100, QFX5120, QFX10002 (Junos)

No build step and no backend: everything is in `index.html`.

## Run locally / on the LAN

    ./serve.sh 8080

## Publish

Push to `main`. The workflow in `.github/workflows/pages.yml` deploys to GitHub Pages
(enable it once under Settings > Pages > Source: GitHub Actions).

## Adding commands

Add an `add(...)` line in the command database section of `index.html`. That line holds the
IOS-XE and Junos versions. Use `V("id", { nxos: "..." })` for NX-OS or StackWise Virtual
(`svl`) variants, and `R([...ids], "poe" | "stack" | "vpc")` to mark commands that need a
capability. Platforms and port-naming rules live in the `CISCO` and `JUNIPER` lists.
