# Privacy Policy

_Last updated: 2026-06-02_

This Privacy Policy describes how the **Scala Teams Claude Code plugin** (the
"plugin") handles data. In short: the plugin collects nothing.

## What the plugin is

The plugin is a collection of agent definitions — Markdown files containing prompts
and tool permissions — that are loaded into [Claude Code](https://claude.com/claude-code)
on your own machine. It is not a service. It has no backend, no servers, and no
accounts.

## Data we collect

**None.** The plugin contains no telemetry, analytics, tracking, or network calls of
its own. It does not collect, store, transmit, or have access to your code, your
prompts, your files, your personal information, or any usage data. Scala Teams, the
plugin's author, receives nothing from your use of the plugin.

## How the plugin operates

When you invoke one of its agents, the agent runs entirely inside your local Claude
Code session. Depending on the tools an agent is granted (which may include reading,
writing, and editing files and running shell commands), it operates on files in your
own environment, under Claude Code's permission system and with your approval. All of
this happens locally between you, Claude Code, and your machine. No data is sent to
the plugin author as a result.

## Third parties

The plugin runs on top of Claude Code. Your interactions with Claude — including any
content the agents help you produce — are processed by Claude Code and Anthropic's
services, and are governed by **Anthropic's** terms and privacy policy, not by this
plugin:

- Anthropic Privacy Policy: https://www.anthropic.com/legal/privacy
- Anthropic Consumer Terms / Commercial Terms: https://www.anthropic.com/legal

The plugin itself does not introduce any additional third-party data processing,
integrations, or external services.

## Changes to this policy

If this policy changes, the updated version will be published in this repository with a
new "Last updated" date.

## Contact

Questions about this policy? Open an issue at
https://github.com/scalateams/scalateams-claude-plugin or email
**gordon.cooke@scalateams.com**.
