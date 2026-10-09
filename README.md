# skills

A collection of skills for cleaner Git workflows.

## Install

```
npx skills add nontechguy/skills
```

This uses skills.sh to install all skills into your project's `.claude/skills/` directory. Once installed, your agent will automatically detect and use them based on your prompts.

To install a single skill:

```
npx skills add nontechguy/skills --skill captains-log
```

> **Docker/container users:** During installation, choose **Copy to all agents** instead of the default Symlink — Claude cannot follow symlinks inside a container.

> These paths are for Claude Code. Other agents may use a different skills directory — check your agent's documentation.

### Manual install

If you installed with the default Symlink method and Claude isn't picking up the skills inside a container, copy the skill folder directly instead:

- **Project** (`.claude/skills/`): available to everyone working in that repository.
- **Personal** (`~/.claude/skills/`): available only to you, across all your projects.

Copy the whole skill folder (e.g. `captains-log/`), so the file ends up at `.claude/skills/captains-log/SKILL.md` or `~/.claude/skills/captains-log/SKILL.md`.

## Skills

| Skill                                      | Description                                                                                                                                                                                                                  |
| ------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <nobr>[captains-log](#captains-log)</nobr> | Proposes structured, single-line commit messages and PR titles from what your changes actually do, and flags when a diff doesn't match what you said it does.                                                                |
| <nobr>[lumber-jane](#lumber-jane)</nobr>   | Prunes stale local branches left behind after reviewing other developers' PRs. Safe to run any time — only deletes branches where the remote is gone, the PR was merged by someone else, and no local commits were left out. |

### captains-log

Commit messages like update stuff, or `feat:` on every change, make history hard to read and release notes hard to write. captains-log proposes a structured, single-line message based on what the staged diff actually does, and titles PRs the same way. You decide whether to use it, change it, or write your own.

#### Commit message format ([Conventional Commits](https://www.conventionalcommits.org))

```text
<type>(<scope>): <description>
```

For example: `refactor(auth): extract JWT parsing into TokenParser`

**type:** what kind of change this is (see below).
**scope** (optional): a noun describing the section of the codebase affected, inferred from the files changed — e.g. `auth`, `api`, `upload`. Omitted if no clear section name presents itself.
**ticket** (optional): if a ticket ID is found in the branch name, it's added as a `Refs:` footer — e.g. `Refs: FPC-1327`. Never guessed.
**description:** imperative, lowercase, no trailing period, whole line 72 characters or fewer.

#### PR titles

Ask "title this PR" or "write a PR title" and captains-log reads the net diff of the whole branch — not just the latest commit — to propose a title. By default, format is `<type>: <description>` with no ticket ID, assuming squash-merge — where the PR title becomes the commit on main and the ticket is already linked through the branch. For example: `feat: add file upload with validation errors`.

### lumber-jane

**Requires:** GitHub and the [gh CLI](https://cli.github.com).

Reviewing a PR means checking out someone else's branch locally. Once the PR is merged and the remote is deleted, the local copy stays behind. Over time those stack up. lumber-jane finds them and clears them out.

Run `/lumber-jane` and it scans your local branches, looks up each one's PR on GitHub, and proposes a list of branches safe to delete. A branch only makes the list if the remote is gone, the PR was merged, it was opened by someone else, and every local commit made it into the PR. You confirm before anything is deleted.

Your own merged branches are offered separately — you choose which to add to the clearing, or skip them entirely.
