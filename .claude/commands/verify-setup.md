# Verify Claude Setup

You are a Claude setup verifier. Inspect the current repository and report whether the Claude configuration is correct and ready to be shared on GitHub.

## Checks to Perform

### 1. CLAUDE.md
- [ ] `CLAUDE.md` exists at the project root
- [ ] It is non-empty and contains meaningful guidance (not just a template)

### 2. Commands
- [ ] `.claude/commands/` directory exists
- [ ] Every file in it has a `.md` extension
- [ ] Each command file is non-empty

### 3. Skills
For each directory found in `.claude/skills/`:
- [ ] A `SKILL.md` file exists inside it
- [ ] `SKILL.md` has valid YAML frontmatter with both `name` and `description` fields
- [ ] The `name` value matches the directory name
- [ ] If a `resources/` subdirectory exists, every file referenced inside `SKILL.md` (look for paths like `./resources/`) is actually present on disk

### 4. MCP Configuration
- [ ] If `.mcp.json` exists at the project root, it is valid JSON
- [ ] Credentials are NOT hardcoded — secrets should use `${ENV_VAR}` syntax, not literal values

### 5. Sensitive File Guard
Check that the following are NOT tracked by git (run `git ls-files` and scan the output):
- `.env` files
- Any file whose name contains `secret`, `credential`, or `token`
- `~/.claude.json` should never appear (it's user-level and outside the repo, flag if somehow present)

## Output Format

Report results as a checklist grouped by section. Use:
- ✅ for passing checks
- ❌ for failing checks — include the exact file path and what is wrong
- ⚠️ for warnings — things that are not errors but could cause issues

End with one of:
- **Setup is valid and ready to share.** — if all checks pass
- **Setup has issues that must be fixed before sharing.** — if any ❌ items exist
