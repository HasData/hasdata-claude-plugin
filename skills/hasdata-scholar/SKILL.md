---
name: hasdata-scholar
description: |
  Google Scholar paper search and citation formats. Use this skill when the user wants academic papers, a Scholar search, or a citation for a paper. Triggers on "Google Scholar", "papers about", "scholarly articles on", "cite this paper". Returns structured JSON. Use the Scholar search tool and the citation-format tool. Do not use the case-law tool.
allowed-tools:
  - Bash(hasdata *)
---

# Google Scholar

## How to fetch

1. If a HasData tool is connected, call the one whose name matches the task below. Read that tool's schema and fill the fields it asks for. Do not run a shell command when the tool exists.
2. If no HasData tool is connected, ask the user to press Connect on the HasData connector for this chat. Do not install software and do not ask them to paste an API key.
3. Use the `hasdata` command only when the connector cannot be connected and the binary is already on PATH. Run `hasdata --help` and use a Scholar command only if it is listed.

## Which tool

| Ask | Tool name contains | Pass |
| --- | --- | --- |
| Find papers | `google_scholar_scholar` | `q` |
| Citation formats for one result | `google_scholar_cite` | `q` (the cite id from the search result, not a new keyword) |

`asYlo` and `asYhi` are the start and end years. `num` is how many results. `cites` limits results to papers that cite a known result.

Do not call a tool whose name contains `case_law`. That tool is not part of the connector this plugin ships.

Return title, authors, year, and link from the tool output. Do not invent a citation the cite tool did not return.
