---
name: github-mcp-tools
description: Guide for AI assistants on using GitHub MCP server tools to interact with GitHub repositories, issues, and pull requests
license: MIT
compatibility: opencode, claude-code
metadata:
  category: git
  audience: ai-assistants
---

## What I do

Enable AI assistants to interact with GitHub via the GitHub MCP server, providing access to repositories, issues, pull requests, commits, branches, and code search.

## Environment Setup

The GitHub MCP server requires a GitHub Personal Access Token with appropriate scopes:

| Scope | Purpose |
|-------|---------|
| `repo` | Full access to private repositories |
| `read:org` | Read org membership |
| `read:user` | Read user profile |
| `read:discussion` | Read team discussions |

### Token Setup

1. Generate token at: https://github.com/settings/tokens
2. Configure in OpenCode: `~/.config/opencode/opencode.json`
3. Or set env var: `GITHUB_TOKEN=your_token`

## Available Tools by Category

### Repository Tools

| Tool | Purpose |
|------|---------|
| `github_search_repositories` | Find repos by name, description, topics |
| `github_get_file_contents` | Read files/directories from repos |
| `github_list_branches` | List branches |
| `github_get_commit` | Get commit details with diff |
| `github_list_commits` | List commit history |
| `github_list_tags` | List git tags |
| `github_list_releases` | List releases |
| `github_get_latest_release` | Get latest release |
| `github_get_release_by_tag` | Get release by tag name |

### Issue Tools

| Tool | Purpose |
|------|---------|
| `github_list_issues` | List issues (open/closed) |
| `github_search_issues` | Search issues across repos |
| `github_issue_read` | Get issue details, comments, labels |
| `github_list_issue_types` | List supported issue types |

### Pull Request Tools

| Tool | Purpose |
|------|---------|
| `github_list_pull_requests` | List PRs |
| `github_search_pull_requests` | Search PRs across repos |
| `github_pull_request_read` | Get PR details, diff, comments, reviews |

### Code Search

| Tool | Purpose |
|------|---------|
| `github_search_code` | Search code across all GitHub |

### Label Tools

| Tool | Purpose |
|------|---------|
| `github_get_label` | Get label details |

## Common Workflows

### Search for Repositories

```
Tool: github_search_repositories
  query: "machine learning in:name stars:>1000 language:python"
```

### Read a File

```
Tool: github_get_file_contents
  owner: "facebook"
  repo: "react"
  path: "README.md"
```

### List Open Issues

```
Tool: github_list_issues
  owner: "facebook"
  repo: "react"
  state: "OPEN"
```

### Get Issue Details

```
Tool: github_issue_read
  method: "get"
  owner: "facebook"
  repo: "react"
  issue_number: 12345
```

### List Pull Requests

```
Tool: github_list_pull_requests
  owner: "facebook"
  repo: "react"
  state: "open"
```

### Review a PR

```
Tool: github_pull_request_read
  method: "get"
  owner: "facebook"
  repo: "react"
  pullNumber: 123
```

### Get PR Diff

```
Tool: github_pull_request_read
  method: "get_diff"
  owner: "facebook"
  repo: "react"
  pullNumber: 123
```

### Search Code

```
Tool: github_search_code
  query: "function useState language:typescript repo:facebook/react"
```

### List Commits

```
Tool: github_list_commits
  owner: "facebook"
  repo: "react"
  sha: "main"
```

### Get Commit Details

```
Tool: github_get_commit
  owner: "facebook"
  repo: "react"
  sha: "abc123"
  include_diff: true
```

## Tool Transparency

Before executing any GitHub MCP tool, announce it:

```
**I am Kimi (kimi-k2.5)**

**Using MCP**: github_search_repositories to find relevant projects...
**Using MCP**: github_list_issues to check open bugs...
**Using MCP**: github_pull_request_read to review the changes...
```

## Working with Local Repos vs GitHub API

| Task | Use |
|------|-----|
| Read local files | `Read` tool |
| Modify local files | `Edit`/`Write` tools |
| Check GitHub issues/PRs | `github_*` MCP tools |
| Search across GitHub | `github_search_*` tools |
| Commit local changes | `Bash`: git commit |
| Read remote file content | `github_get_file_contents` |

## AI Attribution Rules

When committing changes to a local repo that you worked on with GitHub data:

```
git commit -S -m "Analyze React PR trends

Co-authored-by: Kimi (kimi-k2.5) <noreply@moonshot.cn>"
```

| Model | Attribution Line |
|-------|------------------|
| Claude Opus 4.5 | `Co-authored-by: Claude (claude-opus-4-5) <noreply@anthropic.com>` |
| Claude Opus 4.6 | `Co-authored-by: Claude Opus 4.6 <noreply@anthropic.com>` |
| Claude Sonnet 4 | `Co-authored-by: Claude (claude-sonnet-4) <noreply@anthropic.com>` |
| Grok | `Co-authored-by: opencode (grok-4-1-fast) <grok@x.ai>` |
| GPT-4 | `Co-authored-by: GPT (gpt-4) <noreply@openai.com>` |
| Kimi K2.5 | `Co-authored-by: Kimi (kimi-k2.5) <noreply@moonshot.cn>` |

## Pagination

Many list tools support pagination:

```
Tool: github_list_issues
  owner: "facebook"
  repo: "react"
  perPage: 50
  page: 2
```

Or cursor-based:

```
Tool: github_list_issues
  owner: "facebook"
  repo: "react"
  perPage: 50
  after: "Y3Vyc29yOnYyOpHOABcd"
```

## Error Handling

| Problem | Solution |
|---------|----------|
| 401/403 auth error | Check GITHUB_TOKEN is set and has required scopes |
| 404 not found | Verify owner/repo names are correct |
| Rate limit | Wait or use authenticated requests (higher limits) |
| Empty results | Check query syntax or try broader search |

## MCP Server Setup

### Option 1: OpenCode (stdio)

Config in `~/.config/opencode/opencode.json`:

```json
{
  "mcpServers": {
    "github": {
      "command": "docker",
      "args": [
        "run", "-i", "--rm", "-e", "GITHUB_TOKEN",
        "ghcr.io/github/github-mcp-server:latest"
      ],
      "env": {
        "GITHUB_TOKEN": "your_token_here"
      }
    }
  }
}
```

### Option 2: HTTP Server

```bash
docker-compose up -d  # Server at http://localhost:8766/mcp
```

Requires Bearer token header on each request.

## References

- [GitHub MCP Server](https://github.com/github/github-mcp-server)
- [GitHub API Docs](https://docs.github.com/en/rest)
- [Model Context Protocol](https://modelcontextprotocol.io/)
