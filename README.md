# Ship Safer Paper Plugins with Cursor

A lightweight Cursor skill for Paper/Bukkit plugin development with safer reloads, stronger async boundaries, and production-minded validation.

Fast to install. Easy to reuse. Built for real plugin work.

## What it improves

- safer reload and hot-load flows
- async and main-thread guardrails
- memory-leak and crash-risk checks
- tighter edits with less file churn
- warning-aware validation
- command, config, permission, and dependency consistency

## Who it's for

Paper, Bukkit, and Spigot plugin authors using Cursor with Java + Maven projects.

## Install

Copy `mc-paper-plugin-dev` into either:

- `~/.cursor/skills/` for personal use
- `.cursor/skills/` in a project repository

## Use

Ask Cursor things like:

- "Add a reload-safe config cache to my Paper plugin."
- "Review this plugin for duplicate listener registration and task leaks."
- "Implement a command and keep `plugin.yml`, permissions, and reload flow in sync."
- "Refactor this async code so it respects Bukkit main-thread boundaries."

## Included skill

- `mc-paper-plugin-dev/SKILL.md`

## Why this exists

Generic AI code often gets reload behavior, cleanup, and async boundaries wrong in Paper plugins. This skill pushes Cursor toward safer, production-minded changes instead of quick hacks.

## License

MIT