# Release Runbook

Step-by-step process for releasing a new version of product-builder-starter.

## Pre-Release Checklist

1. **Validate frontmatter** — run `./scripts/validate-skills.sh`; fix any errors before proceeding
2. **Audit version drift** — compare bundled skill versions against sources:
   ```bash
   for skill_md in skills/*/SKILL.md; do
     skill=$(basename "$(dirname "$skill_md")")
     version=$(grep "^version:" "$skill_md" | head -1)
     echo "$skill: $version"
   done
   # Cross-check against ~/.claude/skills/*/SKILL.md
   ```
3. **Update stale skills** — for any skill behind source: copy updated SKILL.md,
   add missing CHANGELOG entries, bump version
4. **Update root CHANGELOG.md** with new `## [vX.Y.Z] - YYYY-MM-DD` entry and extract
   the release notes section to `CHANGELOG_RELEASE_NOTES.md` for use in step 8
5. **Update README.md** — bump Version column in the skills table

## Normal Release (no history reset)

For routine releases, use a regular commit on main:

6. **Commit and push**:
   ```bash
   git add -A
   git commit -m "chore: release vX.Y.Z"
   git push origin main
   ```
7. **Create per-skill tags** using individual skill versions from their SKILL.md frontmatter:
   ```bash
   # Extract and tag each skill at its own version
   for skill_md in skills/*/SKILL.md; do
     skill=$(basename "$(dirname "$skill_md")")
     version=$(grep "^version:" "$skill_md" | sed 's/version:[[:space:]]*//' | tr -d '"'"'" | head -1)
     git tag "${skill}/v${version}"
   done
   git push --tags
   ```
8. **Create GitHub release**:
   ```bash
   gh release create vX.Y.Z \
     --title "vX.Y.Z — Product Builder Starter Pack" \
     --notes-file CHANGELOG_RELEASE_NOTES.md
   ```

## History-Reset Release (orphan branch — major structural changes only)

Use the orphan technique only when resetting commit history is warranted (license change,
major structural rewrite). **Must not be checked out on main when running step A.**

A. **Create orphan branch** (from a different branch or a worktree):
   ```bash
   git checkout --orphan release-vX.Y.Z
   git add -A
   git commit -m "chore: release vX.Y.Z"
   git branch -D main
   git branch -m main
   ```
B. **Disable branch protection** in GitHub repo settings before force-pushing, then:
   ```bash
   git push --force-with-lease origin main
   ```
C. Re-enable branch protection after the push.
D. Continue from step 7 (per-skill tags) and step 8 (GitHub release) above.

## Post-Release

9. Verify all skill tags appear on the GitHub releases/tags page
10. Confirm README renders correctly (Mermaid diagram, skills table)
11. Test install of one skill from the pack in a fresh Claude Code session
