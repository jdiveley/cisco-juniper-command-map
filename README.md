# Catalyst to EX Command Map

Single-page tool that translates commands between Cisco Catalyst 3850 (IOS-XE)
and Juniper EX3400 (Junos ELS), with troubleshooting playbooks.

No build step and no backend: everything is in `index.html`.

## Run locally / on the LAN

    ./serve.sh 8080

## Publish

Push to `main`. The workflow in `.github/workflows/pages.yml` deploys to GitHub Pages
(enable it once under Settings > Pages > Source: GitHub Actions).

## Adding commands

Add an `add(...)` line in the command database section of `index.html`.
