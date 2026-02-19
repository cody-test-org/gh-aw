# Issue Analyzer — GitHub Agentic Workflow

An AI-powered [GitHub Agentic Workflow](https://github.github.com/gh-aw/introduction/overview/) that automatically analyzes issues opened in your repository and determines whether they contain enough information for maintainers to act on.

## What It Does

When an issue is **opened** or **edited**, the workflow:

1. **Reads repository context** — README, contributing guidelines, issue templates, and codebase structure
2. **Classifies the issue** — bug report, feature request, question, or documentation issue
3. **Evaluates completeness** based on issue type:
   - **Bug reports** — needs description, repro steps, expected vs actual behavior, environment info
   - **Feature requests** — needs use case, desired behavior, context
   - **Questions** — needs enough context and what was already tried
   - **Documentation** — needs specific section and what's wrong or missing
4. **Takes action:**
   - ✅ **Sufficient info** → adds `ready-for-review` label
   - ❌ **Insufficient info** → adds `needs-more-info` label + posts a friendly comment listing what's missing
   - ✏️ **Edited issue** → re-evaluates and swaps labels if info is now sufficient

## Prerequisites

- [GitHub CLI](https://cli.github.com/) (`gh`) installed and authenticated
- [gh-aw extension](https://github.github.com/gh-aw/setup/cli/) installed:
  ```bash
  gh extension install github/gh-aw
  ```

## Quick Start — Add to Any Repository

### Step 1: Initialize agentic workflows in your target repo

```bash
cd /path/to/your-repo
gh aw init
```

### Step 2: Add the issue-analyzer workflow

```bash
gh aw add cody-test-org/gh-aw/issue-analyzer
```

This copies `issue-analyzer.md` into your repo's `.github/workflows/` directory and compiles the lock file.

### Step 3: Create the required labels

The workflow applies these labels to issues. Create them in your repository:

| Label | Color (suggested) | Description |
|-------|-------------------|-------------|
| `ready-for-review` | `#0E8A16` (green) | Issue has sufficient information for maintainer review |
| `needs-more-info` | `#E4E669` (yellow) | Issue needs additional information from the author |

You can create them via the GitHub UI (Settings → Labels) or with the CLI:

```bash
gh label create "ready-for-review" --color "0E8A16" --description "Issue has sufficient information for maintainer review"
gh label create "needs-more-info" --color "E4E669" --description "Issue needs additional information from the author"
```

### Step 4: Configure secrets

The workflow uses the **Copilot** engine. Bootstrap the required secrets:

```bash
gh aw secrets bootstrap --engine copilot
```

### Step 5: Commit and push

```bash
git add .github/
git commit -m "Add issue-analyzer agentic workflow"
git push
```

The workflow will now automatically run whenever an issue is opened or edited.

## Alternative: Manual Installation

If you prefer not to use `gh aw add`, you can copy the files manually:

1. Copy `.github/workflows/issue-analyzer.md` from this repository into your target repo's `.github/workflows/` directory
2. Navigate to your target repo and compile:
   ```bash
   cd /path/to/your-repo
   gh aw compile
   ```
3. Commit both the `.md` and generated `.lock.yml` files
4. Create the required labels and configure secrets (Steps 3–4 above)

## Customization

### Editing Agent Instructions (No Recompilation Needed)

The markdown body of `issue-analyzer.md` contains the agent's natural language instructions. You can edit these **directly on GitHub.com** without recompilation. For example:

- Add project-specific requirements (e.g., "bug reports must include a log file")
- Adjust the tone or language of comments
- Add or remove issue categories
- Change the completeness criteria for each issue type

### Editing Frontmatter (Requires Recompilation)

If you change the YAML frontmatter (triggers, permissions, safe-outputs, engine, etc.), you must recompile:

```bash
gh aw compile
```

### Adjusting Who Triggers the Workflow

By default, `roles: all` means issues from **any user** (including external contributors) are analyzed. To restrict to repository team members only, edit the frontmatter:

```yaml
roles: [admin, maintainer, write]
```

### Adding or Changing Labels

To change the labels the workflow can apply, edit the `safe-outputs` section in the frontmatter:

```yaml
safe-outputs:
  add-labels:
    allowed: [ready-for-review, needs-more-info, your-custom-label]
  remove-labels:
    allowed: [ready-for-review, needs-more-info, your-custom-label]
```

Then recompile with `gh aw compile`.

## Security

- The agent runs with **read-only permissions** — it cannot directly modify issues, labels, or comments
- All write operations go through **[safe-outputs](https://github.github.com/gh-aw/reference/safe-outputs/)**, which are validated and executed by separate permission-controlled jobs
- Bot-triggered events are automatically skipped to prevent loops
- Concurrency control ensures only one analysis runs per issue at a time

## File Structure

```
.github/workflows/
├── issue-analyzer.md         # Agentic workflow source (edit this)
└── issue-analyzer.lock.yml   # Compiled GitHub Actions workflow (auto-generated)
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Labels not being applied | Ensure the labels exist in your repository (Step 3) |
| Workflow not triggering | Check that the `.lock.yml` file is committed and pushed |
| Authentication errors | Run `gh aw secrets bootstrap --engine copilot` |
| Need to recompile | Run `gh aw compile` after any frontmatter changes |
| Check workflow status | Run `gh aw status` to see workflow state and recent runs |
| View workflow logs | Run `gh aw logs issue-analyzer` to inspect recent runs |
