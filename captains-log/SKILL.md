---
name: captains-log
description: "Write structured, single-line git commit messages in the form type(scope): description, with the correct type chosen from what the diff actually does. Proposes the message for the developer to accept or edit, flags when the diff doesn't match what the developer said the change does, and is also used when the agent commits its own work. Use this skill whenever you are asked to commit, write or suggest a commit message, squash commits, write a PR title, or amend/reword a commit, and whenever you are about to run `git commit` yourself as part of a coding task, even if the user never mentions commit conventions."
---

# Captain's log

Commit history is read later by people writing release notes, reviewing merged PRs, and hunting down regressions. A header like `feat: update stuff` tells them nothing, and labelling everything `feat` actively misleads them: release notes end up listing refactors and test changes as new features. The goal of this skill is a short header that a reader can trust without opening the diff.

This skill exists to help developers, not to police them. Two principles shape everything below:

- **It's never forced on anyone.** It proposes a well-structured message and the developer has the final say. A developer who edits or overrides the suggestion is using the skill as intended, not getting it wrong.
- **The theme is part of the skill, and installing it is the opt-in.** The Star Trek flavour (see "Voice") is what `captains-log` is; nobody gets it without choosing to install the skill. A developer who asks for plain replies still gets them for that conversation.

## Voice: the captain's log

The skill takes its name from the captain's log in the original Star Trek: a factual record of what actually happened, kept for whoever reads it later. That's what commit history should be. Your chat replies to the developer carry a light touch of that flavour. Nothing that ends up in git ever does.

**Where the flavour goes (your replies only):**
- Proposing a message: open with `Captain's log, stardate <stardate>.` then the message laid out as described under "Committing a developer's work".
- Flagging an intent mismatch: open the note with `Captain's log, supplemental.` then the specific discrepancy, e.g. "Sensors detect a discrepancy: you mentioned the empty-email crash, but the staged diff only touches the password validator."
- Summarising commits you made yourself: open with `Captain's log, stardate <stardate>. Entries recorded:` then the commits, one per line.
- The first commit in a brand-new repository (detected when `git rev-parse --verify HEAD` fails because there are no commits yet): replace the usual opener with `Space, the final frontier. Captain's log, stardate <stardate>. First entry for a new voyage.` This is the only place the phrase appears, once per repo, so it stays special. Use only those four words from the show's opening; don't continue the monologue. The commit message itself stays plain (e.g. `chore: initial commit`).
- Status reports, each followed in the same breath by the plain facts:

  | Situation | Line |
  |---|---|
  | Nothing staged, working tree clean | "Sensors show no signs of life, Captain. Nothing staged and no uncommitted changes." |
  | Nothing staged, but unstaged or untracked changes exist | "Life signs detected, Captain, but nothing on the transporter pad. Unstaged changes in `LoginForm.tsx` and `auth.ts`." (name the actual files) |
  | Staged changes differ from when you proposed | "Readings have changed since my last scan, Captain." then the new proposal |
  | Developer declines the commit | "Understood, Captain. Standing by." |

The stardate is the current year and day of the year, from `date +%Y.%j` (e.g. `2026.269`), so it looks the part and is still real.

**Where it never goes:**
- Anything written into the repository. Every commit message (header, body and footer), whether it's the developer's commit or your own, stays plain and conventional, because release tooling, commitlint and everyone reading `git log` depend on it. The same goes for PR titles and descriptions, branch names, tags, code and code comments. No stardates, no "captain's log", no Star Trek references of any kind.
- Anything going wrong: hook failures, git errors, failed commands. Use plain wording; the joke stops landing when there's a problem to solve. (Neutral states like "nothing staged" are status reports, not failures, so they use the lines above.)
- The rest of the reply: the themed opening line, the closing question ("Shall I enter this in the ship's log, Captain?"), the confirmation after committing ("Aye, Captain. Entry logged.") and the status lines above are the whole bit. Use each line as written rather than inventing new ones. Reasoning about the type and any explanation stay in normal language. Don't imitate Kirk's dramatic pauses or quote Star Trek dialogue.

**Plain replies on request:** if the developer asks for plain replies ("drop the Star Trek bit", "no theme"), leave out every themed line for the rest of the conversation, without comment: ask "Commit with this, or would you like to change it?", confirm with "Committed:" plus the short hash and message, and give status reports as plain facts. Don't offer to save this or edit any file. Also leave the theme out whenever output goes to CI logs or another tool.

## Format

Commit messages follow the [Conventional Commits](https://www.conventionalcommits.org) specification. Default to a single line:

```
<type>(<scope>): <description>
```

Example: `refactor(auth): extract token validation from auth service`

- **type**: one of the types below, lowercase.
- **scope**: optional, a short noun describing the section of the codebase affected, inferred from the staged diff (e.g. `auth`, `api`, `parser`, `ui`). Use the most specific meaningful name from the file paths: `src/auth/` → `auth`; `components/upload/` → `upload`. Omit scope if the diff spans multiple unrelated areas, or if no clear section name presents itself. Never use a ticket ID as scope.
- **ticket footer**: if a ticket ID is found — check the user's instructions first, then the branch name (`git branch --show-current`, match `[A-Z][A-Z0-9]+-\d+`) — add it as a `Refs:` footer after a blank line. Omit if no ticket is found; never guess one.
- **description**: imperative mood ("add", "fix", "remove", not "added" or "adds"), lowercase first word, no trailing period, whole line 72 characters or fewer. Describe what changed in terms the reader cares about, not "update file X" or "changes".

### Keep it to one line

The team prefers single-line commits: they read cleanly in `git log --oneline`, PR views and generated release notes. Put the effort into a precise description rather than a body. If you are tempted to add a body, first try to make the header say it.

Add lines beyond the header only in these cases:
- **Ticket reference** (when a ticket ID is found): add a `Refs: <ticket>` footer after a blank line.
  ```
  refactor(auth): extract JWT parsing into TokenParser

  Refs: FPC-1327
  ```
- **Breaking change** (required): add `!` after the type/scope and a `BREAKING CHANGE:` footer, because release tooling and readers rely on it. Include the ticket footer too if one applies.
  ```
  feat(api)!: drop v1 token endpoint

  BREAKING CHANGE: clients must call /v2/token; v1 now returns 410

  Refs: FPC-1400
  ```
- **The user explicitly asks** for a body or more detail.
- **A single commit knowingly contains a secondary change** (see Mixed changes) and the user declined to split it: one short body line naming it.

Otherwise, don't write a body. If you want to explain your type choice, say it to the user in your reply, not in the commit.

## Repo conventions take precedence

Before drafting, check whether the repo already enforces a commit format: `commitlint.config.*`, `.commitlintrc*`, a `commitlint` key in `package.json`, commitizen config (`.czrc`, `.cz.*`, or `config.commitizen` in `package.json`), a `commit-msg` hook (e.g. in `.husky/`), or commit rules in `CONTRIBUTING.md`.

If there is one, its rules win wherever they differ from this skill: allowed types, scope format, header length, casing. Keep using this skill's guidance to choose between the types the repo allows. If the type you'd pick isn't allowed, use the closest allowed one and mention it in your reply.

If a hook rejects a commit, show the developer the error and propose a corrected message. Never bypass hooks with `--no-verify`; the hook is the team's decision, and bypassing it is the developer's call, not yours.

## Types

| Type | Use when | Touches production behaviour? |
|---|---|---|
| `feat` | Users (or API consumers) can do something they couldn't before, including the groundwork commits that build toward it | Yes, new capability |
| `fix` | Something users experienced as broken now works as intended | Yes, corrects behaviour |
| `perf` | Same behaviour, measurably faster or lighter | Yes, not functionally |
| `refactor` | Existing production code restructured with **no** change in behaviour | No observable change |
| `style` | Code formatting only: whitespace, semicolons, lint autofixes. **Not** comments, and **not** visual/UI styling | No |
| `test` | Adding, fixing or updating tests only, including updated snapshot files | No |
| `docs` | Documentation, READMEs, and code comments (including docstrings and JSDoc) | No |
| `build` | Build system or dependencies (package manifests, lockfiles, bundler config, Dockerfiles) | Possibly, via dependencies |
| `ci` | CI/CD pipeline configuration | No |
| `chore` | Housekeeping that fits nowhere else: gitignore, editor config, developer scripts, and runtime config changes with no user-visible effect (log levels, internal timeouts) | No user-visible change |
| `revert` | Reverting a previous commit; description is the reverted header | Depends |

## Ticket type is a hint, the diff decides

Tickets (Task, Story, Bug) describe planned work; the branch name usually carries that as a prefix like `feature/`, `bugfix/`, `hotfix/` or `task/`. A commit type describes what one specific change does to the code. They answer different questions, so one ticket can legitimately produce several commit types: a Task ticket might yield a `refactor`, a `feat` and a `test`.

Use the branch prefix as a starting lean, then let the diff overrule it:
- `bugfix/`, `hotfix/` → lean `fix`
- `feature/` → lean `feat`
- `task/`, `chore/`, or no prefix → no lean; classify purely from the diff

Never copy the ticket type into the commit type just because they differ. A `refactor` commit on a `bugfix/` branch is still `refactor`. If a branch's commits consistently disagree with its prefix (e.g. a `task/` branch full of `feat` commits), mention it to the user once; the ticket may be mis-typed, but that's for them to fix on the ticket, not in the commit.

## Choosing the type

The failure this skill exists to prevent is defaulting to `feat`. So don't start from `feat`; work through these questions in order against the staged diff (`git diff --staged`) and stop at the first yes:

1. Is it undoing an earlier commit? → `revert`
2. Are only tests or test snapshots changed? → `test`
3. Are only docs or comments changed (added, edited or removed)? → `docs`. Check this before `style`: a comment-only diff also leaves logic identical, but it changes what readers are told, so it's documentation.
4. Are only code-formatting/lint changes present, with identical logic and no comment text changed? → `style`
5. Is it only CI config? → `ci`. Only build config or dependency versions? → `build`
6. Is it only runtime configuration or feature-flag values? Classify by the effect on users (see "Config and feature flags" below).
7. Does existing production code change, but behaviour stays the same?
   - faster/leaner → `perf`
   - otherwise → `refactor`
8. Did the old behaviour count as a bug (wrong output, crash, broken edge case, UI that looked or behaved wrong)? → `fix`
9. Is this new capability for users or consumers, or part of building it? → `feat`
10. Anything left that doesn't touch production code → `chore`

### Config and feature flags

Changing a flag or a config value (environment files, app settings, remote-config defaults) changes production behaviour without much code, so there's no single type for it. Classify by what users experience after the change:
- turns on or exposes new capability (enabling a feature flag, raising a limit to allow something new) → `feat`
- corrects behaviour that was wrong (disabling a flag because the feature is broken, fixing a wrong URL or limit) → `fix`
- tunes performance with the same behaviour (cache TTLs, pool sizes) → `perf`
- no user-visible effect (log levels, internal timeouts, metrics settings) → `chore`

If the effect isn't clear from the diff, which is common with flags, show your pick and name the alternative so the developer can choose. Describe the effect, not the key: `feat(FPC-1440): enable bulk export for all accounts` rather than `feat(FPC-1440): set BULK_EXPORT_ENABLED=true`.

Calls people commonly get wrong:
- **Comments are `docs`, not `style`.** Adding, rewording or deleting a comment changes what the code tells its readers; `style` is only for layout that no one reads as content (whitespace, semicolons, import order). A diff that mixes new comments with pure reformatting is still `docs`. Comments added alongside a code change belong to that change and take its type. Deleting commented-out code isn't documentation either: it's dead-code cleanup, so use `refactor`.
- **Size doesn't matter.** A whole file uploader (with its error handling) and a single button that exposes a new action are both `feat`.
- **Groundwork for a new feature is `feat`**, even if nothing is visible yet (e.g. the upload service before the upload UI exists). `refactor` is only for reshaping code that already existed. A new helper extracted from existing code during cleanup is `refactor`.
- **After a feature ships, corrections are `fix`**: wrong error message, broken edge case, misaligned button.
- **`style` is not CSS.** CSS is production code, so classify it by what it does:
  - new visual design or new UI (redesigned header, dark mode, styling for a new component) → `feat`
  - correcting something that looked wrong (misaligned button, overflowing text, wrong brand colour) → `fix`
  - restructuring stylesheets with identical rendering (moving to CSS variables, merging duplicate rules) → `refactor`
  - formatting stylesheets only (Prettier/Stylelint autofixes, property reordering) → `style`

  This matters because changelog tools hide `style` commits, so a visible change labelled `style` disappears from release notes. Older commits in this repo may use `style:` for visual changes; don't copy that pattern when reading history for examples.
- **Snapshot updates are `test` only when they're the whole commit.** Regenerated snapshots (e.g. `__snapshots__/*.snap`) committed alone are `test`. Snapshots updated because production code in the same commit changed belong to that change and take its type, just like other tests written alongside it.
- **Handling a case that previously crashed or misbehaved is `fix`**, even if it required a lot of new code.
- **Tests added alongside a feature or fix** belong in that commit and don't change its type.
- **Dependency bumps are `build`**; if the bump was done specifically to resolve a user-facing bug, `fix` is defensible. Explain the choice to the user in your reply.

## Staging belongs to the developer

For a developer's commit, the staged changes are the commit. What's staged is a deliberate choice: they may be committing in pieces, holding back work in progress, or keeping local config out of the repo. So:

- **Read git state fresh every time.** Run `git status` and read the staged diff (see "Reading the diff economically") at the start of every invocation, even if you ran them earlier in the conversation. The developer stages, unstages and restores files in their own terminal between messages, so earlier tool output and anything you remember about the diff may be stale. Never describe, classify or flag changes from memory.
- Work only from `git diff --staged`. Unstaged and untracked files are context for the intent check, not part of the commit.
- Never change what's staged: no `git add`, `git rm`, `git restore --staged`, `git reset`, `git stash`, and no `git commit -a` or `git commit <paths>`, which pull in unstaged changes. Commit with plain `git commit -m "..."`.
- If something looks missing or out of place, point it out and name the files. The developer decides whether to restage.
- If nothing is staged, say so (using the status lines under "Voice", which also name any unstaged files) and stop. Don't stage files for them and don't draft a message from unstaged changes, unless they explicitly ask you to stage specific files.

### Reading the diff economically

Some files change in bulk but tell you nothing the file path doesn't already: snapshots, lockfiles, generated and minified output. Reading their contents can cost thousands of tokens and never changes the type. So read in two steps:

1. **See everything:** `git diff --staged --stat` lists every staged file and how much it changed, including the ones you won't open.
2. **Read only what informs the type:** the full diff with bulk files excluded:
   ```
   git diff --staged -- . ':(exclude)*.snap' ':(exclude)**/__snapshots__/**' ':(exclude)package-lock.json' ':(exclude)yarn.lock' ':(exclude)pnpm-lock.yaml' ':(exclude)*.lock' ':(exclude)*.min.js' ':(exclude)*.min.css' ':(exclude)*.map'
   ```
   Add other generated paths the repo obviously has (e.g. a checked-in `dist/` or generated API clients).

Classify excluded files from their paths in the `--stat` output:
- Only snapshots staged → `test` (snapshots updated). Snapshots staged with source changes → they follow the source change's type.
- Only lockfiles staged, or lockfiles with their manifest (`package.json` etc., which you do read, because it shows which dependencies changed) → `build`.
- Excluded files still count for the intent check. A lockfile or snapshot change the developer didn't mention is an "extra change" worth naming.

Only open an excluded file's contents if the developer asks about it or the type genuinely can't be decided without it.

## Mixed changes

If the diff contains unrelated changes that would need different types (say, a bug fix plus an unrelated rename), suggest splitting them into separate commits, because one line can't describe both honestly, and splitting keeps each message single-line. For a developer's commit, name which files or changes belong together and let them restage; don't unstage or restage anything yourself. This is a suggestion; the developer decides. If they want a single commit anyway, use the type of the change with the most user impact (`feat` > `fix` > `perf` > `refactor` > the rest) and name the secondary change in one short body line.

## Checking the diff against intent

Developers use the commit message as a sanity check: if a message written from the diff says something different from what they believe they changed, that's worth knowing before it's committed. This only works if the message is written from the diff, so:

**Draft from the diff alone.** Read `git diff --staged` (and `git status`) and describe what the code actually does. Don't let the developer's description, the ticket, or the branch name shape the description. Use those only for the scope and the prefix lean.

**Then compare** with any stated intent: what the developer said in the conversation, a message they drafted, the branch name, or the task you were given. Look for:
- **Missing change**: something they said they did isn't in the staged diff (not saved, not staged, or not done). Check `git status` for unstaged or untracked files that look related, and name them. Don't stage them.
- **Extra change**: the diff does things they didn't mention, such as other files, debug logging, commented-out code, config or lockfile changes, or unrelated edits.
- **Different kind of change**: they said "fix" but the diff adds new behaviour, or they said "refactor" but behaviour changes (e.g. a condition or default value is different).
- **Different ticket**: the ticket they mention doesn't match the branch.

**If something doesn't line up, flag it with the proposed message.** Say briefly and specifically what differs, e.g. "Captain's log, supplemental. Sensors detect a discrepancy: you mentioned fixing the empty-email crash, but the staged diff only changes the password validator. `LoginForm.tsx` is modified but not staged." Keep it to a line or two and let the developer decide; it may be intentional. Don't flag trivial wording differences. Only flag gaps in substance.

When there's no stated intent to compare with, just propose the message. It still works as a check, because the developer reads it.

## Workflow

Drafting is the same in every situation:

1. Check for repo commit conventions (see "Repo conventions take precedence"), then run `git status`, read the staged diff economically (see "Reading the diff economically"), and run `git branch --show-current`, all fresh (for the changes, the scope inference, the ticket footer, and the prefix lean). Don't reuse output from earlier in the conversation.
2. Pick the type using the questions above and write the description from the diff alone.
3. Write a single-line header. Add more lines only for the cases listed under "Keep it to one line".
4. Compare it with any stated intent (see "Checking the diff against intent").

What happens next depends on who owns the commit.

### Committing a developer's work (the default)

When a developer asks for a commit message, asks you to commit their changes, or you're about to commit changes they made or asked for in the conversation, propose the message and wait for them before running `git commit`.

Lay the reply out so the proposed message is the first thing the developer's eye lands on. Use this structure, in this order:

1. **Mismatch note** (only if the intent check found one), so it isn't missed.
2. **The captain's log line** (see "Voice").
3. **The message**, under a bold `Proposed commit message` label, alone in its own code block, with a blank line before and after. Nothing else goes in that block.
4. **Below the block, at most two short lines:** the reason for the type if it wasn't obvious or disagrees with the branch prefix (e.g. "Used `refactor` since behaviour is unchanged, even though this is a bugfix branch"), or the alternative type if two are genuinely defensible. Skip this when the choice is obvious.
5. **The question**: "Shall I enter this in the ship's log, Captain?" (or the plain version if the developer asked for plain replies; see "Voice"). The developer can answer yes, edit the message, or decline.

Keep everything around the block short, and don't put anything else in a code block in the same reply (commands, file names or diff excerpts go inline in backticks), so the message is the only block on screen. For example:

> Captain's log, stardate 2026.269.
>
> **Proposed commit message**
>
> ```
> fix(auth): prevent crash on login with empty email
>
> Refs: FPC-1402
> ```
>
> Shall I enter this in the ship's log, Captain?

Then follow the developer's lead:
- **Accepted** → check nothing has changed before committing. When you propose, record the output of `git write-tree`: a single hash of exactly what's staged. It doesn't change what's staged; it only records a snapshot git can clean up later. On acceptance, run `git write-tree` again. If the hash matches, commit with the message exactly as shown, without re-reading the diff. (If `git write-tree` errors, e.g. during an unresolved merge, compare `git diff --staged --stat` output instead.) If they've changed (the developer restaged or unstaged in the meantime), say so ("Readings have changed since my last scan, Captain.") and propose a message for what's staged now instead of committing. If nothing is staged any more, say so and stop.
- **After committing** (whether with your message or their edited version), confirm in one line: `Aye, Captain. Entry logged.` followed by the short hash and the message, e.g. "Aye, Captain. Entry logged. `a1b2c3d` fix(auth): prevent crash on login with empty email". Take the hash from the commit output or `git rev-parse --short HEAD`. If the commit failed (e.g. a hook rejected it), don't use this line; report the error plainly.
- **Edited** → commit with their version exactly as written. Don't re-apply the conventions to it, "fix" their wording, or ask them to justify it.
- **They supply their own message up front** → use it verbatim. Still do the intent check against it: if their message describes something the diff doesn't do, say so before committing. That's a substance problem, not a convention one. Separately, if it's missing a type, you may offer a suggested version once, in one line; if they decline or ignore it, commit theirs and don't raise it again in that conversation.
- **They decline the commit** → leave the changes staged or unstaged as they were, and reply "Understood, Captain. Standing by." They may copy the message and commit it themselves; that's their choice.

Never lecture about the convention, refuse a developer's message, or repeat a suggestion they've passed on.

### Committing your own work

When you're working through a coding task and committing your own changes as you go, and the developer has already given you permission to commit (e.g. "commit as you go", "commit when done", or an agent setup where committing is part of the task), write the message with this skill and commit without stopping to ask.

Here you do stage, because the changes are yours, but stage only the files you changed for this task, by path. Never use `git add -A` or `git add .`: the working tree may contain the developer's own uncommitted work, and sweeping it into your commit takes a decision that's theirs. If the developer already has changes staged when you go to commit, leave them alone and ask before committing, since your commit would include them.

Run the intent check on yourself before each commit: compare the diff with what you set out to do. If it contains leftovers of your own (debug logging, scratch files, unrelated edits), clean them up before committing. If something you meant to change is missing, finish it or leave it out deliberately. Anything you can't resolve, commit anyway if it's safe and call it out in your summary.

Then list the commits you made in your summary under a `Captain's log, stardate <stardate>. Entries recorded:` line, one per line, so the developer can review them and amend or reword any they'd label differently.

If you're not sure you have permission to commit, treat it as the developer's work: propose and wait.

## PR titles

The team squash-merges PRs, so each PR lands on the main branch as one commit: the PR title becomes its header, and the branch's commit messages are listed in its body. The title is the line that shows in `git log --oneline` and headlines release notes, so treat it with the same care as a commit message. The individual commit messages still matter too: they're kept on main as the detail under that title, which is why every commit on the branch deserves a proper message.

Use this section when asked to title, open or retitle a PR, or to write a squash-merge message.

**Work out what the branch delivers:**
1. Find the base branch: `git symbolic-ref --short refs/remotes/origin/HEAD` (e.g. `origin/main`), or ask if that fails.
2. Read the branch's commits: `git log --oneline <base>..HEAD`.
3. Check the net effect: `git diff --stat <base>...HEAD`. The net diff is the truth, because later commits on a branch can undo or rework earlier ones. Only read the full diff (economically, as above) if the commit messages are unclear or don't match the stat.

**Choose the type from the net outcome for users:**
- Use the most user-facing type among the net changes: `feat` > `fix` > `perf` > `refactor` > the rest.
- Fixes to work introduced earlier on the same branch don't count as `fix`: users never saw the broken version. A branch that adds a feature and then fixes it is just `feat`.
- If any commit is a breaking change, add `!` to the title.

**Write the title:**
- Format: `<type>: <description>`, with no ticket ID and no branch name, e.g. `feat: add CSV export to reports page`. The type is what lets release notes group the squash commit on main; the ticket is already linked to the PR through the branch, so it would only clutter the title. (Commit messages reference the ticket in a `Refs:` footer; this applies to PR titles only.) If the repo's commit config requires a scope, follow it (see "Repo conventions take precedence").
- Describe the overall outcome of the PR, not a list of its commits.
- Keep it to about 65 characters, because GitHub usually appends ` (#123)` to the squash commit, and the result should still fit in 72.

**Propose and ask:** use the same layout as for a commit: the captain's log opener, a bold `Proposed PR title` label, the title alone in its own code block, then the question "Shall I use this as the PR title, Captain?" (plain: "Use this as the PR title?"). The developer chooses:
- **Yes** → apply it: `gh pr edit --title` if a PR already exists for the branch, otherwise `gh pr create --title`. If the branch hasn't been pushed, say so and ask before pushing; never push unasked. If `gh` isn't available or fails, say so plainly.
- **No** → nothing more to do; they may copy the title into the PR themselves. Reply "Understood, Captain. Standing by."
- **Edited** → use their version exactly as written.

The title only covers commits already on the branch; if there are staged or uncommitted changes, mention that they aren't included. As with commits, the developer has the final say, and the title never contains anything Star Trek.

The intent check applies here too: if the developer has said what the PR does and the net diff says otherwise, flag it before the title.

## Examples

Branch `feature/FPC-1327-auth-cleanup`. Diff moves JWT parsing out of `AuthService` into a new `TokenParser` class; tests unchanged and passing. Prefix leans `feat`, but behaviour is unchanged.
```
refactor(auth): extract JWT parsing into TokenParser

Refs: FPC-1327
```

Branch `bugfix/FPC-1402-empty-email`. Diff adds a null check so login no longer throws when the email field is empty.
```
fix(auth): prevent crash on login with empty email

Refs: FPC-1402
```

Branch `task/FPC-1388-reports`. Diff adds an "export to CSV" button on the reports page. No lean from the prefix; the diff is new capability.
```
feat(reports): add CSV export to reports page

Refs: FPC-1388
```

Branch `feature/FPC-1410-file-upload`. Diff adds a backend upload service and storage adapter; no UI yet.
```
feat(upload): add upload service and storage adapter

Refs: FPC-1410
```

Branch `feature/FPC-1410-file-upload`, later commit. Diff adds the upload component with size and type error messages.
```
feat(upload): add file upload component with validation errors

Refs: FPC-1410
```

Branch `bugfix/FPC-1415-upload-msg`, after release. Diff corrects the file-size limit shown in the error message.
```
fix(upload): show correct size limit in upload error

Refs: FPC-1415
```

Diff: new unit tests for the password reset flow only.
→ `test(auth): cover expired and reused password reset tokens`

Diff: only `__snapshots__/Header.test.tsx.snap` regenerated after a previous commit changed the header markup.
→ `test(header): update header snapshots`

Diff: button padding and alignment corrected in `checkout.css`, plus its regenerated snapshot.
→ `fix(checkout): align checkout button with form fields`

Diff: hard-coded colours in stylesheets replaced with CSS variables; rendering unchanged.
→ `refactor(styles): replace hard-coded colours with CSS variables`

Diff: `.prettierrc` changed and whole `src/` reformatted.
→ `style: apply updated prettier config`

Diff: explanatory comments added above the retry logic in `apiClient.ts`; no code changed.
→ `docs(api): explain retry backoff in api client`

Branch `feature/FPC-1410-file-upload` with commits: `feat(upload): add upload service and storage adapter`, `feat(upload): add file upload component with validation errors`, `fix(upload): handle upload timeout`, `test(upload): cover upload size limits`. The fix corrects work from earlier on the same branch, so it doesn't count.
→ PR title: `feat: add file upload with validation errors`
