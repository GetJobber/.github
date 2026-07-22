# GetJobber organization workflows

## Codex vulnerability triage

[`codex-vulnerability-triage.yml`](.github/workflows/codex-vulnerability-triage.yml)
is the reusable implementation behind Barnacle's dependency-vulnerability
triage. A target repository receives a `barnacle-vulnerability-triage`
`repository_dispatch` event and calls this workflow with the finding context.

The workflow keeps privileges separated:

1. The analysis job has read-only GitHub permissions. Codex can edit only the
   runner workspace and can reach an allowlist of dependency and advisory
   hosts. It cannot access a GitHub write token.
2. The publishing job does not receive the OpenAI key. It applies the generated
   patch, opens a draft pull request when required, and uploads the final
   Barnacle result artifact.

Callers must make `OPENAI_API_KEY` and either the read-only
`JOBBER_SKILLS_TOKEN` or existing `CODE_OWNERSHIP_GH_TOKEN` available and pass
them with `secrets: inherit`. The repository setting that allows GitHub Actions
to create pull requests must also be enabled. Barnacle's GitHub App needs
repository `Contents: write` to dispatch the event and `Actions: read` to poll
the result artifact.
