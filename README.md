# GitHub MCP Server

Run GitHub's MCP (Model Context Protocol) server for use with AI coding assistants like OpenCode, Claude Code, etc.

## Setup Options

### Option 1: Local MCP (Recommended for OpenCode)

Add to your `~/.config/opencode/opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "github": {
      "type": "local",
      "command": [
        "docker", "run", "-i", "--rm",
        "-e", "GITHUB_PERSONAL_ACCESS_TOKEN=your_token_here",
        "ghcr.io/github/github-mcp-server",
        "stdio",
        "--toolsets", "repos,issues,pull_requests",
        "--read-only"
      ]
    }
  }
}
```

Verify with:
```bash
opencode mcp list
```

### Option 2: HTTP Server (Docker Compose)

For sharing across multiple clients or web apps.

1. Copy `.env.example` to `.env` and add your GitHub token:
   ```bash
   cp .env.example .env
   ```

2. Start the server:
   ```bash
   docker-compose up -d
   ```

3. The MCP server will be available at `http://localhost:8766/mcp`

4. Configure your MCP client to use:
   - URL: `http://localhost:8766/mcp`
   - Auth: Bearer token (your GitHub PAT)

## Configuration

### Toolsets

Available toolsets:
- `repos` - Repository operations
- `issues` - Issue management
- `pull_requests` - PR operations
- `actions` - GitHub Actions
- `code_security` - Security features
- `discussions` - Discussions
- `gists` - Gists
- `notifications` - Notifications
- `orgs` - Organization management
- `projects` - Projects
- `users` - User operations

### Flags

- `--read-only` - Restrict to read-only operations
- `--toolsets` - Comma-separated list of toolsets to enable

## GitHub Token

Create a Personal Access Token at https://github.com/settings/tokens with appropriate scopes for the toolsets you want to use.

## Verified Commits (SSH Signing)

To make your commits show as "Verified" on GitHub:

### 1. Generate an SSH signing key

```bash
ssh-keygen -t ed25519 -C "your_email@example.com" -f ~/.ssh/id_ed25519_signing -N ""
```

### 2. Configure git to use SSH signing

```bash
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519_signing
git config --global commit.gpgsign true
```

### 3. Add the key to GitHub

```bash
# Grant the required scope
gh auth refresh -h github.com -s admin:ssh_signing_key

# Add the signing key
gh ssh-key add ~/.ssh/id_ed25519_signing.pub --type signing --title "Signing key"
```

Or manually: Go to https://github.com/settings/keys > "New SSH Key" > Key type: "Signing Key"

### 4. Verify it works

```bash
echo "test" | git commit-tree HEAD^{tree} -S
```

### Co-authoring with AI (Claude, Grok/opencode, GPT, etc.)

To have verified commits while crediting an AI assistant as co-author:

**Problem**: If you set the AI as committer (e.g., `Claude <noreply@anthropic.com>` or `opencode (grok-4-1-fast) <grok@x.ai>`), the commit will be **unverified** because the signature is from your key, not the AI's.

**Solution**: Keep yourself as author/committer and add a `Co-authored-by` trailer:

```bash
git commit -S -m "Your commit message

Co-authored-by: opencode (grok-4-1-fast) <grok@x.ai>"
```

This gives you:
- **Verified** badge (signed with your key)
- **Your name** as author/committer
- **AI attribution** shown on GitHub with bot icon

### Rules for AI Assistants

**DO NOT** change git user identity when making commits:

```bash
# WRONG - breaks verification
git config --local user.name "Grok"
git config --local user.email "grok@x.ai"
git commit -m "..."  # Unverified! Signed with human's key but AI identity

# CORRECT - use global config (human) + co-author trailer
git commit -S -m "Message

Co-authored-by: Grok <grok@x.ai>"
```

**Why this matters**:
1. SSH signing key belongs to the human, not the AI
2. GitHub verifies signature matches committer email
3. If committer = AI email, signature won't match = unverified
4. `Co-authored-by` gives AI credit without breaking verification

**For AI agents**: Never run `git config user.name/email` with AI identity. Always use the human's global git config and add yourself as co-author in the commit message.

**History cleanup** (solo repos): To remove bad commits:
```bash
git reset --hard <good-commit-before-bad>  # e.g., git reset --hard 662dfbd
git push --force
```
Re-apply good changes manually. For shared repos: `git revert <bad-commit>`.

## Usage Examples

**Prompts** (OpenCode/Claude):
- "List open issues using github"
- "Review PR #123 with github mcp"
- "Search code for auth. use github"

**AGENTS.md**:
```
For PR reviews, use github tools.
```

**Combos**: github + sentry (issues), context7 (docs).

## xAI/Grok Research (2026-03-18)

xAI repos:
- grok-1: 51k stars, last commit 2024-03-19 (cuda fix)
- grok-prompts: CI agent updates prompts (2025-11-17)
- xai-cookbook: API examples (2026-03-15)

No AI co-authors in recent commits.

## References

- [GitHub MCP Server](https://github.com/github/github-mcp-server)
- [OpenCode MCP Docs](https://opencode.ai/docs/mcp-servers/)
- [Model Context Protocol](https://modelcontextprotocol.io/)
- [GitHub: Signing Commits](https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-commits)
