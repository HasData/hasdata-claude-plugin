---
description: Job listings for a role and a city
argument-hint: role and city
allowed-tools:
  - Bash(hasdata *)
---

# /hasdata:jobs

Role and place: **$ARGUMENTS**

If a city or a role is missing, ask before calling a tool.

If HasData tools are connected, call the tool whose name contains `indeed_listing`. Also call the tool whose name contains `glassdoor_listing` unless the user named only one site. Read each schema. Do not run a shell command when the tools exist.

If no HasData tool is connected, ask the user to press Connect. Do not install software and do not ask for an API key.

Use `hasdata indeed-listing --help` only when the connector cannot be connected and the binary is already on PATH.

Return title, company, location, and link. Include pay only when the tool returned it.
