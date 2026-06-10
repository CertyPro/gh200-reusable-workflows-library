# Example: consuming outputs from `release-notes.yml`

A reusable workflow can return data to the caller through workflow-level `outputs`. The caller reads those values with `needs.<job-id>.outputs.<name>`, exactly as it would for any job output.

This caller workflow lives in **your** repository, for example at `.github/workflows/release.yml`.

```yaml
name: Release
on:
  push:
    tags:
      - 'v*'

jobs:
  notes:
    # Call the reusable workflow that generates release notes.
    uses: CertyPro/gh200-reusable-workflows-library/.github/workflows/release-notes.yml@v1
    with:
      tag: ${{ github.ref_name }}   # for a tag push this is, for example, v1.4.0

  publish:
    needs: notes                     # wait for the called workflow to finish
    runs-on: ubuntu-latest
    steps:
      - name: Use the generated notes
        run: |
          echo "Publishing release with the following notes:"
          echo "${{ needs.notes.outputs.notes }}"
```

## How the output travels

1. Inside `release-notes.yml`, a step writes a value to `$GITHUB_OUTPUT`:
   `echo "notes=$NOTES" >> "$GITHUB_OUTPUT"`.
2. The job re-exposes that step output as a **job** output (`outputs.notes`).
3. The `workflow_call` block re-exposes the job output as a **workflow** output (`outputs.notes`).
4. The caller reads it through `needs.notes.outputs.notes`.

All three levels are required: step output, then job output, then workflow output. Missing any link is a common reason a caller sees an empty value, and a frequent GH-200 exam trap.

## Things to notice for GH-200

- The consuming job must declare `needs:` on the job that calls the reusable workflow.
- The `tag` input is `required: true`, so the caller must always provide it.
- Outputs are strings.
