---
description: Public company profile from Google and the company's own site. A person lookup needs their company. No home addresses, no guessed emails.
argument-hint: company name, or a person plus their company
allowed-tools:
  - Bash(hasdata *)
---

# /hasdata:enrich

Subject: **$ARGUMENTS**

This is a company lookup on public pages. If the user gave only a person's name, ask for the company and wait. Decline home addresses, personal phone numbers, family details, and location tracking.

Do not guess an email from a pattern. Do not run a name-only search across the web. Return a field only when a tool result contains it, and say when a field was not found.

## Connector

If HasData tools are connected, do only this and stop. Use the full results tool, whose name contains `google_serp_serp` and does not contain `light`. The light tool drops the knowledge panel this lookup needs.

1. Search the company name. From that response, take the official site and the knowledge panel if they are present.
2. Run three more queries on the same tool: `site:linkedin.com/company COMPANY`, `site:github.com COMPANY`, `site:crunchbase.com COMPANY`.
3. If step 1 returned an official site, call the tool whose name contains `web_scraping` on that URL. Ask for markdown.
4. Reply with company name, official site, and whichever of LinkedIn, GitHub, and Crunchbase the searches returned, each with its link. Quote only text that appeared in a tool result.

A handful of calls is enough. Do not fan out into dozens of queries.

If no HasData tool is connected, ask the user to press Connect. In Claude Code, authenticate with `/mcp`. Do not install a binary and do not ask for an API key.

## CLI fallback

Only when the connector cannot be connected and `hasdata` is already on PATH. Use `hasdata google-serp` for the company and the three site queries, then `hasdata web-scraping` on the official site. Do not install the CLI. Do not add email-harvest queries.
