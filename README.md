# skills

A collection of skills for product development workflows

Copy skills to your project `.claude/skills/` directory or to `~/.claude/skills/`. Once copied, your agent will automatically detect and use them based on your prompts. You can also invoke a skill directly by name, e.g. `/captains-log`.

**Project** (`.claude/skills/`): available to everyone working in that repository. The skill becomes a tracked file in git.
**Personal** (`~/.claude/skills/`): available only to you, across all your projects. Nothing is added to the repository.

Copy the whole skill folder (e.g. `captains-log/`), so the file ends up at `.claude/skills/captains-log/SKILL.md` or `~/.claude/skills/captains-log/SKILL.md`.

> These paths are for Claude Code. Other agents may use a different skills directory — check your agent's documentation.

## Skills

| Skill                         | Description                                                                                                                                                   |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [captains-log](#captains-log) | Proposes structured, single-line commit messages and PR titles from what your changes actually do, and flags when a diff doesn't match what you said it does. |

### captains-log

Commit messages like update stuff, or `feat:` on every change, make history hard to read and release notes hard to write. captains-log proposes a structured, single-line message based on what the staged diff actually does, and titles PRs the same way. You decide whether to use it, change it, or write your own.

#### Commit message format

```text
<type>(<scope>): <description>
```

For example: `refactor(FPC-1327): extract JWT parsing into TokenParser`

**type:** what kind of change this is (see below).
**scope** (optional): the ticket ID from any project management tool — Jira, Shortcut, Linear, and so on. Taken from your branch name, e.g. `feature/FPC-1327-auth-cleanup` or `feature/SC-42-auth-cleanup`. Left out if no ticket ID is found; it's never guessed.
**description:** imperative, lowercase, no trailing period, whole line 72 characters or fewer.

#### PR titles

Ask "title this PR" or "write a PR title" and captains-log reads the net diff of the whole branch — not just the latest commit — to propose a title. By default, format is `<type>: <description>` with no ticket ID, assuming squash-merge — where the PR title becomes the commit on main and the ticket is already linked through the branch. For example: `feat: add file upload with validation errors`.
