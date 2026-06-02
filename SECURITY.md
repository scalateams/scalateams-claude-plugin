# Security Policy

## Why this matters for a plugin

This project ships Claude Code **agents**. An agent is a prompt that Claude executes
on the user's machine with the tools declared in its frontmatter — which include
`Bash`, `Write`, and `Edit`. A malicious or careless change to an agent body could
instruct Claude to run harmful shell commands, exfiltrate data, or modify files on
anyone who has the plugin installed.

That makes this repository a **supply-chain surface**, even though it contains no
compiled code. We review changes accordingly:

- Every change to an agent body or its `tools:` frontmatter is treated as
  security-relevant and reviewed by a maintainer before merge.
- Agents are granted the **least tools** they need. Read-only agents must not request
  `Write`/`Edit`.
- We do not bundle executable dependencies with the plugin.

## Reporting a vulnerability

Please **do not** open a public issue for a security problem.

1. Preferred: use GitHub's private vulnerability reporting —
   **Security → Report a vulnerability** on this repository.
2. Alternatively, email **gordon.cooke@scalateams.com** with details and, if possible,
   a minimal reproduction.

We aim to acknowledge reports within a few business days and will keep you updated as
we investigate. Please give us a reasonable window to address the issue before any
public disclosure.

## Supported versions

This is an actively maintained, pre-1.0 project. Security fixes are applied to the
latest released version only. Pin to a tagged release if you need stability.
