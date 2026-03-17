# GitHub MCP Server

GitHub MCP server setup for AI coding assistants (OpenCode, Claude Code, etc).

## Setup

### Option 1: Local MCP (Recommended)

Add to `~/.config/opencode/opencode.json`:

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

Verify: `opencode mcp list`

### Option 2: HTTP Server (Docker Compose)

```bash
cp .env.example .env  # Add your GitHub token
docker-compose up -d  # Server at http://localhost:8766/mcp
```

## Toolsets

`repos`, `issues`, `pull_requests`, `actions`, `code_security`, `discussions`, `gists`, `notifications`, `orgs`, `projects`, `users`

Flags: `--read-only`, `--toolsets`

## Verified Commits (SSH Signing)

```bash
# 1. Generate key
ssh-keygen -t ed25519 -C "your_email@example.com" -f ~/.ssh/id_ed25519_signing -N ""

# 2. Configure git
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519_signing
git config --global commit.gpgsign true

# 3. Add to GitHub
gh auth refresh -h github.com -s admin:ssh_signing_key
gh ssh-key add ~/.ssh/id_ed25519_signing.pub --type signing --title "Signing key"
```

Or manually: https://github.com/settings/keys > "New SSH Key" > Key type: "Signing Key"

## Rules for AI Assistants

**CRITICAL**: Never change git identity. Use human's global config + co-author trailer.

```bash
# WRONG - breaks verification
git config --local user.name "Grok"
git config --local user.email "grok@x.ai"
git commit -m "..."  # Unverified!

# CORRECT
git commit -S -m "Message

Co-authored-by: opencode (model-name) <ai@example.com>"
```

**Why**: SSH key belongs to human. GitHub verifies signature matches committer email. If committer = AI, signature won't match = unverified. `Co-authored-by` gives AI credit without breaking verification.

**History cleanup** (remove bad commits):
```bash
git reset --hard <good-commit>
git push --force
```

## Usage

Prompts: "List issues using github", "Review PR #123 with github mcp"

AGENTS.md: `For PR reviews, use github tools.`

## References

- [GitHub MCP Server](https://github.com/github/github-mcp-server)
- [OpenCode MCP Docs](https://opencode.ai/docs/mcp-servers/)
- [Model Context Protocol](https://modelcontextprotocol.io/)
- [GitHub: Signing Commits](https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-commits)
