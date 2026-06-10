# Caller examples

These are **documentation snippets**, not live workflows. They live under `examples/` (not `.github/workflows/`) on purpose, so they do not auto-run in this library repository. Copy the relevant snippet into a workflow file under `.github/workflows/` in **your own** repository to call the reusable workflows.

| Example | Shows |
| --- | --- |
| [`call-node-ci.md`](call-node-ci.md) | Calling `node-ci.yml` with `with:` inputs and `secrets:`, including `secrets: inherit`. |
| [`call-release-notes.md`](call-release-notes.md) | Consuming a workflow `outputs` value from a called workflow via `needs`. |

## Reminders for callers

- Reference a reusable workflow at **job** level with `uses:`, pointing at `owner/repo/.github/workflows/file.yml@ref`.
- Pin `@ref` to a tag or full commit SHA in real projects. Examples here use `@v1` for clarity.
- Pass inputs with `with:` and secrets with `secrets:`.
- Read a called workflow's outputs through `needs.<job-id>.outputs.<name>`.

See [`../docs/README.md`](../docs/README.md) for best practices and org governance, and the [course content](https://github.com/CertyPro/certy-gh200-course-content) for the full GH-200 study path.
