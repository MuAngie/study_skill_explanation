# Skill Agent Workbench Design

## Goal

Build a local personal workbench for studying Codex skills. The app lets the user browse, create, import, load, and run skills through the local Codex CLI, while showing the selected skill instructions, execution log, final output, and exit status.

## Assumptions

- The app is for one local user only.
- The first version uses the local `codex` command as the execution engine.
- The workspace root is the app's controlled execution area.
- New skills created by the app are stored in the project `skills/` directory first, not written directly into the global Codex skills directory.
- Installed skills can be read from `C:\Users\gao'lin'jie\.codex\skills` and plugin cache skill directories when available.
- The first version supports one run at a time.
- Codex's existing permission and sandbox behavior controls file edits and command execution.

## Non-Goals

- No multi-user accounts, cloud sync, billing, or sharing.
- No full marketplace browser.
- No replacement for the Codex desktop app.
- No custom model runner in the first version.
- No complex workflow automation or concurrent job queue.

## User Experience

The app opens directly into a three-panel workbench.

Left panel: Skill Library
- Search skills by name, description, and path.
- Show installed, project-local, and imported skills.
- Provide actions to create a skill and import a skill.

Middle panel: Skill Detail
- Show skill name, description, source, and filesystem path.
- Show the full `SKILL.md` content.
- Let the user select the skill for a run.

Right panel: Run Panel
- Show selected skills.
- Provide a task input box.
- Start a Codex CLI run.
- Stream execution output.
- Show final status, elapsed time, and result text.

The interface should be clean and utility-focused. It should feel like a debugger or local control panel, not a marketing page.

## Core Features

### Skill Discovery

The backend scans configured skill roots and returns normalized skill records:

- `id`
- `name`
- `description`
- `path`
- `source`
- `content`

The parser reads frontmatter from `SKILL.md` when present and falls back to the folder name when metadata is missing.

### Skill Creation

The user can create a new project-local skill by entering:

- name
- description
- instruction body

The app writes:

```text
skills/<slug>/SKILL.md
```

The generated file uses standard frontmatter:

```markdown
---
name: <name>
description: <description>
---

<instruction body>
```

### Skill Import

The first version supports:

- importing a local folder that contains `SKILL.md`
- downloading a raw `SKILL.md` URL into a new project-local skill folder

Imported skills are copied into `skills/` so experiments do not mutate external sources.

### Codex Run

When a run starts, the backend:

1. Builds a prompt that explicitly names the selected skills and includes the user task.
2. Starts `codex` in the project workspace.
3. Streams stdout and stderr to the UI.
4. Records exit code, timestamps, selected skills, task text, and final output.

The first version does not try to parse every internal Codex tool event. It shows the raw process stream and groups it into a readable run log.

## Architecture

Use a small local web application:

- Frontend: React or equivalent lightweight UI.
- Backend: Node.js server.
- Storage: local JSON files under `.skill-agent/`.
- Execution: child process wrapper around local `codex`.

Suggested directories:

```text
src/
  server/
    skills.ts
    runs.ts
    codex.ts
  app/
    components/
    pages/
skills/
.skill-agent/
```

## Data Flow

1. Frontend requests skill list.
2. Backend scans skill roots and returns metadata.
3. User selects skills and enters a task.
4. Frontend starts a run.
5. Backend launches Codex CLI and streams output.
6. Frontend appends log lines and displays completion state.
7. Backend stores a run record for later inspection.

## Error Handling

Keep errors plain and actionable:

- Missing `codex` command: show setup message.
- Invalid skill folder: show that `SKILL.md` is required.
- Failed download: show URL and network error.
- Codex exits non-zero: keep the full log and mark the run as failed.
- File write fails: show the target path and reason.

Do not add broad recovery flows in the first version.

## Validation

The first version is complete when:

1. The app lists skills from at least the project `skills/` directory.
2. The app can display a selected skill's metadata and full instructions.
3. The app can create a new skill with a valid `SKILL.md`.
4. The app can import a local skill folder.
5. The app can start a Codex CLI run with selected skill context.
6. The UI shows live run output and final exit status.
7. A basic manual test confirms the flow: create skill, select it, run a task, inspect logs.

## First-Version Decisions

- Global installed skills are visible by default, but new and imported skills still go into the project `skills/` directory.
- URL imports support raw `SKILL.md` files only.
- Run history is retained locally, with no delete flow in the first version.
