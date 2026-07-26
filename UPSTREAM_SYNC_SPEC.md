# Fork Requirements and Upstream Sync

## A. Fork feature requirements

This fork maintains `fix/pass-e2b-api-url` on top of
`e2b-dev/langchain-e2b:main`.

The E2B provider must:

- resolve `E2B_API_URL` through the same injected environment resolver used for
  `E2B_API_KEY`;
- prefer `DEEPAGENTS_CODE_E2B_API_URL` over `E2B_API_URL`, matching the
  existing prefixed-variable behavior;
- pass a configured API URL to synchronous and asynchronous sandbox create,
  connect, and delete operations;
- preserve lazy credential resolution and all upstream provider behavior; and
- document the canonical and Deep Agents Code-prefixed settings in `README.md`.

Acceptance requires focused unit tests for canonical URL propagation and
prefixed-variable precedence, plus successful unit tests, Ruff checks, and
static type checks.

## B. Upstream synchronization workflow requirements

`main` is the default automation branch and must remain a strict superset of
the maintained feature branch. `.github/workflows/upstream-sync.yml` merges in
this order:

1. `e2b-dev/langchain-e2b:main` into `fix/pass-e2b-api-url`;
2. the updated `fix/pass-e2b-api-url` into `main`.

The workflow runs weekly and by manual dispatch, serializes runs, uses merge
commits, and never rebases or force-pushes. Clean merges are validated and
pushed in order.

If either merge conflicts, a Deep Agents Code repair runs in the protected
`actions-env` GitHub Environment. The agent must read this entire file,
preserve upstream behavior except where Section A intentionally overrides it,
resolve only the failed merge and necessary related breakage, run focused
checks when useful, and leave the merge uncommitted with no unresolved paths.

The repair agent must not push, create or merge pull requests, rewrite history,
force-push, change secrets or repository settings, or bypass validation.
Checkout credentials are not persisted and no GitHub token is supplied to the
agent. After it returns, deterministic workflow steps must confirm the merge is
still pending and conflict-free, run `make lint` and `make test`, commit the
repair, verify the incoming commit is an ancestor, push a uniquely named repair
branch, and open a human-reviewed pull request against the failed target.

Third-party actions must be pinned to immutable commit SHAs. Workflow
permissions are limited to `contents: write` and `pull-requests: write`.
Provider credentials belong only in the protected `actions-env` environment;
an optional `DEEP_AGENT_MODEL` environment variable may override the agent's
default model.
