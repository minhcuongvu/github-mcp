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

## References

- [GitHub MCP Server](https://github.com/github/github-mcp-server)
- [OpenCode MCP Docs](https://opencode.ai/docs/mcp-servers/)
- [Model Context Protocol](https://modelcontextprotocol.io/)
- [GitHub: Signing Commits](https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-commits)
