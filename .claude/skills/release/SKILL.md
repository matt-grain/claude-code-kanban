---
name: release
description: This skill should be used when the user wants to create a new release — bump version, tag, push, create GitHub release, and optionally publish to npm. Use when user says "release", "bump version", "publish", or "cut a release".
disable-model-invocation: true
user-invocable: true
argument-hint: "[version]"
allowed-tools: Read, Bash(git *), Bash(gh *), Bash(bun *)
---

# Release

Bump version, tag, push, create a GitHub release with auto-generated notes, and present npm publish command.

## Inputs

- `$ARGUMENTS` — target version (e.g. `1.15.0`). If not provided, suggest bump type based on recent changes.

## Workflow

### Step 1: Verify Clean Working Tree

Run `git status --short`. If there are uncommitted changes, warn the user and **stop**.

### Step 2: Determine Version

Read current version from `package.json`. If `$ARGUMENTS` is provided, use it directly.

Otherwise, analyze commits since the last tag to suggest a bump type:
- **major** — breaking changes
- **minor** — new features (✨ feat commits)
- **patch** — bug fixes, chores, docs only

Present the suggested bump type and resulting version to the user using `AskUserQuestion` with options: patch, minor, major (put the recommended one first with "(Recommended)" suffix).

### Step 3: Bump Version

Update `version` field in `package.json` to the target version.

### Step 4: Commit & Push

```bash
git add package.json
git commit -m "🔖 chore: Bump version to <version>"
git push origin main
```

### Step 5: Tag & Push Tag

```bash
git tag v<version>
git push origin v<version>
```

### Step 6: Generate Release Notes

Collect commits since previous tag:

```bash
git log --oneline <prev-tag>..HEAD
```

Group by type:
- Features (✨ feat)
- Fixes (🐛 fix)
- Other notable changes

Write concise user-facing notes (not raw commit messages). Include a **Full Changelog** compare link using the repository URL from `package.json`.

### Step 7: Create GitHub Release

```bash
gh release create v<version> --title "v<version>" --notes "<notes>"
```

### Step 8: Present npm Publish

Show the release URL. Then present the user with the manual publish command:

```
npm publish
```

Do **not** run `npm publish` automatically — let the user decide.
