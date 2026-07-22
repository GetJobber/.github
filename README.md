# GetJobber organization workflows

## Codex vulnerability triage with GitHub Agentic Workflows

[`codex-vulnerability-triage-agentic.md`](.github/workflows/codex-vulnerability-triage-agentic.md)
is the source for the compiled
[`codex-vulnerability-triage-agentic.lock.yml`](.github/workflows/codex-vulnerability-triage-agentic.lock.yml)
workflow behind Barnacle dependency-vulnerability triage. It uses the GitHub
Agentic Workflows Codex engine, firewall, threat detection, cost controls, and
guarded safe outputs.

A target repository receives a `barnacle-vulnerability-triage`
`repository_dispatch` event and calls the compiled reusable workflow with the
finding context.

The workflow keeps privileges separated:

1. Codex has read-only GitHub permissions, works only in the runner workspace,
   and can reach an allowlist of dependency and GitHub hosts.
2. Repository writes go through `safe-outputs.create-pull-request`, which
   restricts changed paths, enforces draft pull requests, and never merges.
3. A deterministic final job normalizes the outcome into the
   `barnacle-triage-<attempt_id>` artifact consumed by Barnacle.

Callers must make `OPENAI_API_KEY` and either the read-only
`JOBBER_SKILLS_TOKEN` or existing `CODE_OWNERSHIP_GH_TOKEN` available and pass
them with `secrets: inherit`. The repository setting that allows GitHub Actions
to create pull requests must also be enabled. Barnacle's GitHub App needs
repository `Contents: write` to dispatch the event and `Actions: read` to poll
the result artifact.

The Agentic Workflow source and generated lock file must be committed together.
After changing its frontmatter or prompt, run:

```sh
gh aw compile codex-vulnerability-triage-agentic --strict --validate --approve
```
