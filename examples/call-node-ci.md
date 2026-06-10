# Example: calling `node-ci.yml`

This caller workflow lives in **your** repository at `.github/workflows/ci.yml`. It calls the reusable `node-ci.yml` from this library, passes an input, and supplies a secret.

```yaml
name: CI
on:
  pull_request:
  push:
    branches: [main]

jobs:
  build-and-test:
    # Reference the reusable workflow at JOB level with `uses:`.
    # Format: owner/repo/.github/workflows/file.yml@ref
    uses: CertyPro/gh200-reusable-workflows-library/.github/workflows/node-ci.yml@v1
    with:
      node-version: '20'        # maps to the `node-version` input
    secrets:
      npm-token: ${{ secrets.NPM_TOKEN }}   # maps to the `npm-token` secret
```

## Passing secrets explicitly vs inheriting

The example above passes each secret **explicitly**. This is the most controlled approach and is recommended: the called workflow only receives exactly what you list.

Alternatively, you can forward **all** of the caller's secrets with `secrets: inherit`:

```yaml
jobs:
  build-and-test:
    uses: CertyPro/gh200-reusable-workflows-library/.github/workflows/node-ci.yml@v1
    with:
      node-version: '20'
    secrets: inherit
```

Use `secrets: inherit` sparingly. It is convenient when the caller and called workflow are in the same organisation and you trust the called workflow, but it grants the called workflow every secret available to the caller. Prefer explicit secrets for least privilege.

## Reading the output

`node-ci.yml` exposes a `result` output. A later job can read it:

```yaml
jobs:
  build-and-test:
    uses: CertyPro/gh200-reusable-workflows-library/.github/workflows/node-ci.yml@v1
    with:
      node-version: '20'

  gate:
    needs: build-and-test
    runs-on: ubuntu-latest
    steps:
      - run: echo "CI result was ${{ needs.build-and-test.outputs.result }}"
```

## Things to notice for GH-200

- The caller uses `uses:` at job level, with no `runs-on` or `steps` for that job - the reusable workflow supplies those.
- Inputs go under `with:`, secrets under `secrets:`.
- Pin `@v1` (a tag) or a full commit SHA in production rather than a moving branch like `@main`.
