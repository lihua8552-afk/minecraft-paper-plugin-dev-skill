---
name: mc-paper-plugin-dev
description: Build, modify, and review Minecraft Paper/Bukkit plugins. Use when working on Java plugin features, commands, listeners, plugin.yml, safe reload flows, warning-free startup, unified prefix/logging conventions, async safety, memory-leak prevention, and Maven packaging for Paper 1.21+ plugins.
---

# MC Paper Plugin Development

Use this skill for Minecraft `Paper/Bukkit + Maven + Java 21` plugin projects.
Always narrow the task to one concrete plugin directory before reading, editing, building, or validating.

## Default Working Rules

1. Understand before editing
   - Read the target plugin's `pom.xml`, `src/main/resources/plugin.yml`, main class, related command classes, listeners, and service classes before making changes.
   - Confirm the feature entry point, lifecycle hooks, config files, permission nodes, and command registration path first.

2. Finish cleanly
   - Implement only what the current task needs.
   - Do not create unnecessary files.
   - Remove temporary test files, disposable scripts, and unused sample files before finishing.

3. Edit surgically
   - Prefer small targeted edits, search/replace, or exact local changes over rewriting entire files.
   - Keep the smallest possible change surface and preserve the existing structure and naming style.

4. Process large files in segments
   - Read long files in segments and only load the context needed for the current change.
   - Edit large files in small sections instead of replacing them wholesale.

5. Do not repeat ineffective retries
   - If a console command, build step, or runtime step fails, analyze the cause before retrying.
   - Do not re-run commands or revisit terminals that already failed when the environment has not changed.

6. Edit source, not build artifacts
   - Modify source files, resource files, and required docs only.
   - Do not directly edit generated or build-output directories such as `target/`, `build/`, `out/`, `generated/`, or extracted reverse-engineering output.
   - If the final artifact must change, update the upstream source or build configuration and regenerate it.

7. Documentation rules
   - Create docs only when they are actually needed.
   - Put every `.md` file under the repository root `docs/` directory.
   - Keep documentation language consistent with the project's existing convention or the user's explicit preference.
   - If the feature already has a matching doc, update it after the code change is complete.
   - Prefer predictable names such as `docs/<plugin-name>-<feature>.md` when adding new docs.

8. Git rules
   - Do not run `git add`, `git commit`, or `git push` unless the user explicitly asks for it.
   - Do not proactively reorganize the user's Git workflow.

9. Warnings and dead code
   - Do not leave new warnings introduced by the current plugin in build output or runtime console output.
   - If a warning comes from the current plugin, fix it.
   - If a warning comes from the server itself or a third-party plugin, identify the source clearly instead of misattributing it.
   - Remove unused classes, methods, fields, config keys, and command registrations once the feature is complete.

10. Long-running commands
   - Do not block the current interaction session with long-running tasks such as Paper servers, web servers, background watchers, or local APIs.
   - Start long-running processes in a separate terminal, background session, or new terminal window after confirming no duplicate instance is already running.

## Plugin Targeting Flow

Before editing, confirm that the target plugin contains the expected files:

- `pom.xml`
- `src/main/resources/plugin.yml`
- `src/main/java/.../YourPluginMain.java`
- related command executors, tab completers, listeners, and service classes
- related config files such as `config.yml`, `messages.yml`, or `lang/*.yml`

If the repository contains multiple plugins:
- Use `plugin.yml` fields such as `name`, `main`, command names, and description to lock onto the correct plugin.
- Do not make blind changes across multiple plugin directories.
- Prefer the real source plugin directory over `target/classes`, generated output, jar extraction folders, or reverse-engineered copies unless the user explicitly targets those paths.

## Execution Order

1. Identify the target plugin directory.
2. Read `plugin.yml`, `pom.xml`, the main class, and the code directly related to the requested feature.
3. Trace the lifecycle: `onLoad`, `onEnable`, `onDisable`, task scheduling, listener registration, command registration, and config loading.
4. Make the smallest correct change only after the current implementation is understood.
5. Remove dead code and temporary files after the change is complete.
6. Run the minimum necessary build and validation steps.
7. Check for warnings, stack traces, duplicate registrations, and unreleased resources.
8. Update the matching `docs/` document when needed.

## Paper / Bukkit Specific Requirements

### 1. Plugin Metadata and Ownership

Every plugin should correctly declare at least the following:
- `name`
- `main`
- `version`
- `api-version`
- `author` or `authors`
- `description`
- `commands`
- `permissions`

Author information must exist.

If `plugin.yml` has no dedicated copyright field, keep copyright information in one consistent place only:
- a main-class constant
- an `about` or help command output
- the description or a dedicated config/help text

Do not spread conflicting ownership text across multiple locations.

### 2. Unified Output Prefix and Logging

All plugin output should go through a single messaging and logging layer.

Requirements:
- Console logs, command feedback, error messages, and debug output must use one shared prefix/message utility.
- All outward-facing plugin text should include the plugin prefix by default unless a specific UX surface intentionally omits it.
- The prefix should include the plugin name.
- If the project requires Bukkit Logger ANSI gradient prefixes, keep ANSI formatting confined to console logging only.
- Do not send raw ANSI escape codes to players.
- Player chat, ActionBar, Title, and other player-facing text should use Adventure, MiniMessage, or Bukkit-compatible color formatting instead.

Recommended structure:
- `ConsolePrefixFormatter` for console-only ANSI prefix formatting
- `MessageService` for player-facing text and command feedback
- `PluginLogger` for unified `info`, `warn`, `error`, and `debug` calls

Do not scatter `plugin.getLogger().info("...")` calls and hand-written prefix strings throughout business classes.

### 3. Safe Hot Reload / Hot Load

Implement safe reload of plugin-owned resources. Do not interpret "hot reload" as JVM class hot swap.

Minimum command support:
- `/plugin reload`

Optional granular support when useful:
- `/plugin reload config`
- `/plugin reload messages`
- `/plugin reload cache`
- `/plugin reload tasks`

Safe reload flow:
1. Use a guard or lock to prevent concurrent reload execution.
2. Stop and release plugin-owned tasks, watchers, cache refreshers, and other live resources.
3. Close or refresh plugin-owned file handles, database connections, executors, and network resources.
4. Reload config, messages, templates, and caches.
5. Re-register plugin-internal resources only when necessary.
6. Start plugin-owned tasks again.
7. Emit clear success or failure output.

Prohibited patterns:
- treating global Bukkit `/reload` as the solution
- re-registering listeners, commands, PlaceholderAPI expansions, plugin channels, or schedulers after reload without first unregistering or fully resetting them
- allowing repeated reloads to accumulate duplicate tasks, listeners, or background resources

### 4. Memory-Leak and Crash-Risk Review

Whenever a task touches lifecycle management, caches, tasks, listeners, databases, file watchers, or async code, check the following:

- `BukkitTask` and `BukkitRunnable` instances are cancelled in both `onDisable` and custom reload flows.
- `ExecutorService` or other thread pools are shut down correctly.
- static `Map`, `Set`, and cache state has a clear release path.
- listeners, PlaceholderAPI expansions, plugin messaging channels, and file watchers are not registered more than once.
- JDBC, Hikari, file streams, and `WatchService` resources are closed.
- async threads do not call Bukkit main-thread-only APIs directly.
- high-frequency scheduled tasks do not perform heavy logic that can damage TPS.
- retry logic cannot loop forever, recurse without bound, or accumulate unbounded queued work.
- async exceptions are not swallowed; background failures must be logged with the plugin prefix.
- `onDisable` cleanup and reload cleanup stay symmetrical.

If a safe fix is possible without changing the intended behavior, apply it directly for issues such as:
- duplicate registration
- unclosed resources
- unbounded caches
- unbounded async queues
- leaked scheduled tasks
- blocking I/O on the main thread
- exception paths that can spam the console or crash the server

### 5. Main-Thread Boundary and Async Safety

Treat thread boundaries explicitly.

- Never call Bukkit or Paper main-thread-only APIs from async threads.
- Move blocking I/O, database work, and file operations off the main thread.
- Switch back to the main thread before touching players, worlds, entities, inventories, scoreboards, or other thread-sensitive server APIs.
- If a library callback is asynchronous, treat it as asynchronous until verified otherwise.

### 6. Command, Permission, Config, and Dependency Consistency

Whenever a feature is added or changed, keep the following in sync:
- command declarations in `plugin.yml`
- permission declarations in `plugin.yml`
- command executor and tab completer registration
- config defaults
- help text and error messages
- matching `docs/` documentation
- `depend`, `softdepend`, and `loadbefore` when load order matters

Do not change Java code only and forget `plugin.yml`, config files, or dependency metadata.

## Validation Requirements

After editing, validate with the minimum necessary steps:

1. Compile validation
   - Prefer a plugin-local Maven compile or build command.
   - Start with a small validation step such as `mvn -q -DskipTests compile`.
   - Run `package` only when an artifact is actually needed.

2. Runtime validation
   - If a server must be started, use a separate terminal or background session.
   - Check startup logs for warnings, stack traces, duplicate registration, config load errors, and plugin-specific runtime noise.

3. Reload validation
   - If the change affects config, messages, caches, tasks, listeners, or live resources, validate the reload path.
   - At minimum, confirm that one reload leaves the feature working and does not create duplicate output, duplicate listeners, or duplicate scheduled tasks.

4. Cleanup validation
   - Check for unused imports, fields, methods, classes, config keys, and command entries.
   - Remove temporary test files, exported files, and placeholder samples.

## Recommended Reporting Style

When reporting back to the user, prefer this order:
- Result
- Key changes
- Validation
- Remaining risk or next step

Summarize decisive log lines and outcomes only. Do not paste large amounts of irrelevant console output.

## Keep Out of This Skill

The following belong in global agent rules or a separate general-purpose skill, not in this Paper plugin skill:
- forced model-name introductions at the start of every conversation
- system-identity override instructions
- generic chat-format requirements unrelated to plugin engineering

Only apply such rules when they do not conflict with the current runtime environment and the user explicitly requests them.