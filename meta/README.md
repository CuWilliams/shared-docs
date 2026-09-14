# meta — CLAUDE.md tooling

Repo-agnostic. Nothing here is about any particular product.

- **[CLAUDE_MD_TEMPLATE.md](CLAUDE_MD_TEMPLATE.md)** — standard structure for a `CLAUDE.md`
- **[CLAUDE_MD_CHECKLIST.md](CLAUDE_MD_CHECKLIST.md)** — what to review, and how often
- **[check-claude-md.sh](check-claude-md.sh)** — scores a `CLAUDE.md` out of 6

## Running the health check

```bash
# From a repo root — finds CLAUDE.md / claude.md / Claude.md automatically
/path/to/shared-docs/meta/check-claude-md.sh

# Or point it at a file
/path/to/shared-docs/meta/check-claude-md.sh path/to/CLAUDE.md
```

Score: 80%+ healthy, 50–79% needs attention, below 50% refactor now.

## Treat the score as a prompt, not a target

The checks are proxies — line count, absence of session logs, presence of a status section. A file
can score 6/6 and still be wrong, and the failure that actually costs you is a confidently stated
claim that stopped being true, which no script can catch. Use the checklist for that.
