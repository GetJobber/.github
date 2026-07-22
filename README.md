# GetJobber organization workflows

## Codex vulnerability triage

Barnacle's dependency-vulnerability triage has two interchangeable reusable
backends:

- [`codex-vulnerability-triage.yml`](.github/workflows/codex-vulnerability-triage.yml)
  invokes `openai/codex-action` directly.
- [`codex-vulnerability-triage-agentic.md`](.github/workflows/codex-vulnerability-triage-agentic.md)
  is the source for the compiled
  [`codex-vulnerability-triage-agentic.lock.yml`](.github/workflows/codex-vulnerability-triage-agentic.lock.yml)
  GitHub Agentic Workflow, using its Codex engine, firewall, threat detection,
  and guarded safe outputs.

A target repository receives a `barnacle-vulnerability-triage`
`repository_dispatch` event and calls one backend with the finding context. Set
the target repository Actions variable `BARNACLE_CODEX_BACKEND` to
`agentic-workflow` to select `gh-aw`; leave it unset to use the direct action.

The workflow keeps privileges separated:

1. Codex has read-only GitHub permissions, works only in the runner workspace,
   and can reach an allowlist of dependency and GitHub hosts.
2. The direct backend publishes through a separate credential-free job. The
   Agentic Workflow publishes through `safe-outputs.create-pull-request`; both
   paths enforce draft pull requests and never merge.
3. Both backends normalize the outcome into the same
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
