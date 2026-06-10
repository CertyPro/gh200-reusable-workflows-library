# gh200-reusable-workflows-library

A small library of **reusable workflows** for the [Certy](https://certy.pro) GitHub Actions certification track (**GH-200**). Learners call these workflows from their own repositories to practise the `workflow_call` trigger, typed inputs, secrets, and outputs, and to recognise how callers reference a centrally maintained workflow with `uses:`.

Part of the CertyPro learning platform: <https://github.com/CertyPro>

- Course content: <https://github.com/CertyPro/certy-gh200-course-content>
- Student actions lab: <https://github.com/CertyPro/gh200-student-actions-lab>
- Custom actions lab: <https://github.com/CertyPro/gh200-custom-actions-lab>
- Broken workflows (debugging practice): <https://github.com/CertyPro/gh200-broken-workflows>
- Enterprise admin simulation: <https://github.com/CertyPro/gh200-enterprise-admin-sim>
- Security challenges: <https://github.com/CertyPro/gh200-security-challenges>

> Maps to GH-200 domain **1.0 Author and manage workflows** and domain **4.0 Manage GitHub Actions in the enterprise** (org-level reusable workflow governance).

---

## What is a reusable workflow?

A reusable workflow is a complete workflow file that another workflow can call, rather than copying and pasting the same jobs into many repositories. You define it once with the `workflow_call` trigger, then call it from a caller workflow using the `uses:` keyword at **job** level:

```yaml
jobs:
  ci:
    uses: CertyPro/gh200-reusable-workflows-library/.github/workflows/node-ci.yml@v1
```

Key points for the exam:

- Reusable workflows are triggered with `on: workflow_call`.
- A caller references them at **job level** with `uses:`, not at step level (step-level `uses:` runs an **action**, which is a different building block).
- Data flows in through `inputs:` and `secrets:`, and flows out through `outputs:`.
- The called workflow runs in the context of the **caller**, so the caller controls which secrets are shared.
- You can pin the called workflow to a branch, a tag, or a full commit SHA after the `@`.

These workflows live under `.github/workflows/`, so they exist on GitHub but **do not run on their own** - they only execute when another workflow calls them. That is the expected behaviour.

---

## Workflows in this library

| Workflow file | Purpose | Inputs | Secrets | Outputs |
| --- | --- | --- | --- | --- |
| `node-ci.yml` | Reusable Node.js CI: checkout, setup-node, `npm ci`, `npm test`. | `node-version` (string, default `'20'`) | `npm-token` (optional) | `result` |
| `lint.yml` | Reusable lint stage with a configurable working directory. | `working-directory` (string, default `'.'`) | none | none |
| `release-notes.yml` | Generates a release notes string for a given tag and exposes it as a workflow output. | `tag` (string, required) | none | `notes` |
| `label-pr.yml` | Demonstrates job `permissions` and a simple labelling step. | `label` (string, default `'needs-review'`) | none | none |

All `uses:` examples below are pinned to the `v1` tag. In your own work you should pin to a tag or a full commit SHA for stability and supply chain safety. See [`docs/README.md`](docs/README.md).

---

## How to call each workflow

### `node-ci.yml`

Reusable Node.js continuous integration.

| Name | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| Input `node-version` | string | no | `'20'` | Node.js version passed to `actions/setup-node`. |
| Secret `npm-token` | secret | no | - | Token for installing private npm packages. |
| Output `result` | string | - | - | `'passed'` when the test job completes. |

```yaml
name: CI
on:
  pull_request:
  push:
    branches: [main]

jobs:
  build-and-test:
    uses: CertyPro/gh200-reusable-workflows-library/.github/workflows/node-ci.yml@v1
    with:
      node-version: '20'
    secrets:
      npm-token: ${{ secrets.NPM_TOKEN }}
```

### `lint.yml`

Reusable lint stage.

| Name | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| Input `working-directory` | string | no | `'.'` | Directory the lint step runs in. |

```yaml
jobs:
  lint:
    uses: CertyPro/gh200-reusable-workflows-library/.github/workflows/lint.yml@v1
    with:
      working-directory: ./app
```

### `release-notes.yml`

Generates a notes string and exposes it as a workflow output so the caller can consume it.

| Name | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| Input `tag` | string | yes | - | The release tag to generate notes for. |
| Output `notes` | string | - | - | A short generated release notes string. |

```yaml
jobs:
  notes:
    uses: CertyPro/gh200-reusable-workflows-library/.github/workflows/release-notes.yml@v1
    with:
      tag: v1.4.0

  publish:
    needs: notes
    runs-on: ubuntu-latest
    steps:
      - run: echo "Notes were ${{ needs.notes.outputs.notes }}"
```

### `label-pr.yml`

Demonstrates explicit job `permissions` and a simple step.

| Name | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| Input `label` | string | no | `'needs-review'` | Label name used by the demonstration step. |

```yaml
jobs:
  label:
    uses: CertyPro/gh200-reusable-workflows-library/.github/workflows/label-pr.yml@v1
    with:
      label: needs-review
```

---

## Caller examples and best practices

- Worked caller examples: [`examples/`](examples/)
- Best practices, pinning, secrets, and org governance: [`docs/README.md`](docs/README.md)

---

## Licence

Released under the MIT Licence. See [`LICENSE`](LICENSE).
