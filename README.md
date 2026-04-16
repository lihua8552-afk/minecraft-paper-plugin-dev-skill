# Minecraft Paper Plugin Dev Skill for Cursor

![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)
![Cursor Skill](https://img.shields.io/badge/Cursor-Skill-blue)
![Minecraft](https://img.shields.io/badge/Minecraft-Paper%20%7C%20Bukkit%20%7C%20Spigot-5b8c2a)
![Java](https://img.shields.io/badge/Java-21%2B-orange)
![Build Tool](https://img.shields.io/badge/Maven-friendly-c71a36)

Build Minecraft Paper/Bukkit plugins with safer reloads, cleaner edits, and fewer async mistakes.

A production-minded Cursor skill for Java plugin authors who want less guesswork, less metadata drift, and fewer broken server reloads.

## Why this skill is useful

Most generic AI output is fine for quick snippets but weak on plugin lifecycle safety. This skill pushes Cursor toward safer Paper/Bukkit changes by enforcing patterns around:

- reload-safe resource handling
- async and main-thread boundaries
- listener, task, and cache cleanup
- `plugin.yml`, command, permission, and config consistency
- warning-aware validation before calling work done

## What it helps prevent

- broken or duplicated reload behavior
- leaked tasks, watchers, executors, and caches
- async access to Bukkit main-thread-only APIs
- drift between Java code and `plugin.yml`
- noisy warning-heavy plugin updates
- large, unnecessary file rewrites

## Best fit

Use this if you build or maintain:

- Paper plugins
- Bukkit plugins
- Spigot plugins
- Java + Maven Minecraft plugin projects

## Install in under a minute

Copy `minecraft-paper-plugin-dev` into either:

- `~/.cursor/skills/` for personal use
- `.cursor/skills/` in a project repository

Final structure should look like this:

```text
.cursor/
  skills/
    minecraft-paper-plugin-dev/
      SKILL.md
```

## Example prompts

Ask Cursor things like:

- "Add a reload-safe config cache to my Minecraft Paper plugin."
- "Review this plugin for duplicate listener registration and task leaks."
- "Implement a command and keep `plugin.yml`, permissions, and reload flow in sync."
- "Refactor this async code so it respects Bukkit main-thread boundaries."
- "Audit this plugin for warning sources, leaked tasks, and unsafe reload behavior."

## Included skill

- `minecraft-paper-plugin-dev/SKILL.md`

## Why this exists

Generic AI code often gets plugin reload behavior, cleanup, async boundaries, and `plugin.yml` consistency wrong. This skill pushes Cursor toward safer, more production-ready Paper/Bukkit changes instead of quick hacks.

## License

MIT