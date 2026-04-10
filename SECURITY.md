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
- Misleading command examples that could cause data loss or unintended `p6a` operations

## Out of Scope

- Vulnerabilities in the `p6a` CLI itself. Report those to the [p6a-cmd](https://github.com/partoska/p6a-cmd) repository.
- Vulnerabilities in your agentic platform. Report those to your vendor.
