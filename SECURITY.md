# Security Policy

## Supported Versions

Only the latest commit on `main` is actively maintained.

## Reporting a Vulnerability

Please **do not** open a public GitHub issue for security vulnerabilities.

Use [GitHub's private vulnerability reporting](https://github.com/partoska/p6a-skills/security/advisories/new) to submit a report. You can expect:

- Acknowledgement within **168 hours**
- A fix or mitigation plan within **90 days**, depending on severity
- Credit in the release notes if you would like it

## Scope

This repository contains agent skill definitions with no compiled code, no server, and no dependency tree. The attack surface is limited to:

- Prompt injection via malicious skill content loaded into an agent
- Misleading instructions that could lead an agent into unintended Partoska actions, such as making a private event public, approving photos the user would reject, or revoking approvals the user wants kept

## Out of Scope

- Vulnerabilities in the Partoska service or its MCP server. Report those to Partoska directly.
- Vulnerabilities in your agentic platform. Report those to your vendor.
