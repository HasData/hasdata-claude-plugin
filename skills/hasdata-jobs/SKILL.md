---
name: hasdata-jobs
description: |
  Job listings and details from Indeed and Glassdoor. Use this skill when the user wants to search jobs by keyword and location, get details on a single job posting, build a hiring map for a role, or research roles at a company. Triggers on "find jobs", "Indeed search", "Glassdoor jobs", "<role> jobs in <city>", "open positions for", "job listings", "hiring for", "job posting at <url>", or any salary / company hiring research. Returns structured JSON — title, company, location, salary, posted date, URL — without HTML scraping.
allowed-tools:
  - Bash(hasdata *)
---

# hasdata jobs APIs

Job search + single-listing details from Indeed and Glassdoor.

## When to use

- User wants jobs by keyword + location
- User wants details on a single job posting URL
- User wants to research what's hiring in a market for a given role
- User wants to fan out from search → fetch each listing's full description

## APIs in this group

| Command              | Purpose                                    | Cost |
| -------------------- | ------------------------------------------ | ---- |
| `indeed-listing`     | Indeed search by keyword + location        | 5    |
| `indeed-job`         | Single Indeed job by URL                   | 5    |
| `glassdoor-listing`  | Glassdoor search by keyword + location     | 5    |
| `glassdoor-job`      | Single Glassdoor job by URL                | 5    |

## Quick start

### Indeed

```bash
# Search
hasdata indeed-listing --keyword "software engineer" --location "Remote" --pretty -o .hasdata/indeed-swe.json

# Sorted by relevance, paginated
hasdata indeed-listing --keyword "data scientist" --location "New York, NY" --sort relevance --start 10 --pretty -o .hasdata/indeed-ds-p2.json

# Different country
hasdata indeed-listing --keyword "product manager" --location "London" --domain uk.indeed.com --pretty -o .hasdata/indeed-pm-uk.json

# Single job details
hasdata indeed-job --url "https://www.indeed.com/viewjob?jk=3c916f0d7f870c71" --pretty -o .hasdata/indeed-job.json
```

### Glassdoor

```bash
# Search
hasdata glassdoor-listing --keyword "frontend engineer" --location "San Francisco, CA" --pretty -o .hasdata/glassdoor-fe.json

# Sort by recent + paginate
hasdata glassdoor-listing --keyword "designer" --location "Remote" --sort recent --pretty -o .hasdata/glassdoor-design.json
hasdata glassdoor-listing --keyword "designer" --location "Remote" --next-page-token "<token>" --pretty -o .hasdata/glassdoor-design-p2.json

# Single job details
hasdata glassdoor-job --url "https://www.glassdoor.com/job-listing/..." --pretty -o .hasdata/glassdoor-job.json
```

## Common flags

### Indeed
| Flag             | Purpose                                                         |
| ---------------- | --------------------------------------------------------------- |
| `--keyword`      | Required — search query (e.g. `"data engineer"`)                |
| `--location`     | Required — `"City, ST"` or `"Remote"`                           |
| `--domain`       | `www.indeed.com`, `uk.indeed.com`, `de.indeed.com`, etc.        |
| `--sort`         | `relevance` / `date` (default `date`)                           |
| `--start <n>`    | Pagination offset                                               |

### Glassdoor
| Flag                  | Purpose                                                         |
| --------------------- | --------------------------------------------------------------- |
| `--keyword`           | Required — search query                                         |
| `--location`          | Required — location                                             |
| `--domain`            | `www.glassdoor.com`, `www.glassdoor.co.uk`, etc.                |
| `--sort`              | `recent` / `relevant` (default `recent`)                        |
| `--next-page-token`   | Pagination token from previous response                         |

## Tips

- **Search → job fan-out** when the user wants full descriptions:
  ```bash
  hasdata indeed-listing --keyword "rust" --location "Remote" --pretty -o .hasdata/indeed.json
  for url in $(jq -r '.jobs[].url' .hasdata/indeed.json | head -20); do
    hasdata indeed-job --url "$url" --pretty -o ".hasdata/job-$(basename $url | tr '?=&' '-').json" &
  done
  wait
  ```
- **Country-specific Indeed domains** (e.g. `de.indeed.com`) return localized salary ranges and currencies.
- For **company-specific research**, search the company name as a keyword: `--keyword "Anthropic"`.
- Indeed sort `date` is best for monitoring new postings; sort `relevance` is best for general job search.

## Working with results

```bash
# Indeed → title, company, location, salary, posted, URL
jq -r '.jobs[] | "\(.title)\t\(.company)\t\(.location)\t\(.salary // "n/a")\t\(.url)"' .hasdata/indeed.json

# Glassdoor → similar
jq -r '.jobs[] | "\(.title)\t\(.company)\t\(.location)\t\(.salary // "n/a")"' .hasdata/glassdoor.json

# Filter for remote-only
jq '.jobs[] | select(.location | test("Remote"; "i"))' .hasdata/indeed.json

# Indeed job → key fields including description
jq '{title, company, location, salary, description: .description[0:500]}' .hasdata/indeed-job.json
```

## See also

- [hasdata-search](../hasdata-search/SKILL.md) — `google-serp` for company hiring pages
- [hasdata-business](../hasdata-business/SKILL.md) — Yelp / YellowPages for non-job business data
