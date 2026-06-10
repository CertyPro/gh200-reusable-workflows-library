# Best practices and governance for reusable workflows

This guide accompanies the [gh200-reusable-workflows-library](../README.md) and covers the parts of reusable workflows that GH-200 examines most closely: safe referencing, secret handling, versioning, and organisation-level governance.

---

## 1. Pin the called workflow to a tag or SHA

When a caller references a reusable workflow, the `@ref` after the path decides which version runs:

```yaml
# Branch - moves every time the branch updates. Convenient, least stable.
uses: CertyPro/gh200-reusable-workflows-library/.github/workflows/node-ci.yml@main

# Tag - stable and readable. Good default for most teams.
uses: CertyPro/gh200-reusable-workflows-library/.github/workflows/node-ci.yml@v1

# Full commit SHA - immutable, the strongest supply chain guarantee.
uses: CertyPro/gh200-reusable-workflows-library/.github/workflows/node-ci.yml@8e8db7ba2c1f6a0d9e3b4c5d6e7f8a9b0c1d2e3f
```

Guidance:

- For production and anything security sensitive, pin to a **full commit SHA**. A tag can be moved by a maintainer; a SHA cannot.
- A **tag** (ideally an immutable release tag) is a sensible default that balances stability and readability.
- Avoid pinning to a **branch** such as `@main` outside of experiments, because the behaviour can change underneath you without warning.

> The same pinning advice applies to **actions** referenced at step level with `uses:`. Pinning by SHA is a recognised supply chain control.

---

## 2. Passing secrets: explicit vs `secrets: inherit`

Reusable workflows never see the caller's secrets unless the caller forwards them. There are two ways to do that.

**Explicit (recommended, least privilege):**

```yaml
jobs:
  ci:
    uses: CertyPro/gh200-reusable-workflows-library/.github/workflows/node-ci.yml@v1
    secrets:
      npm-token: ${{ secrets.NPM_TOKEN }}
```

The called workflow receives only the named secrets, and only ones it has declared under `workflow_call.secrets`.

**Inherit (convenient, broad):**

```yaml
jobs:
  ci:
    uses: CertyPro/gh200-reusable-workflows-library/.github/workflows/node-ci.yml@v1
    secrets: inherit
```

`secrets: inherit` forwards **all** of the caller's secrets to the called workflow. Use it only when caller and called workflow are within the same organisation and you trust the called workflow. Prefer explicit secrets to limit blast radius.

Other points:

- A secret declared `required: true` in the reusable workflow must be supplied by the caller.
- Secrets are masked in logs, but never echo them deliberately.
- The `GITHUB_TOKEN` is not a secret you pass with `secrets:`; its scopes come from the caller's `permissions:` block.

---

## 3. Permissions

A reusable workflow executes with the `GITHUB_TOKEN` permissions granted by the **caller**. Declaring `permissions:` inside the reusable workflow (as `label-pr.yml` does) documents what it needs, but the caller must grant at least those scopes. Apply least privilege on both sides: start from `permissions: read-all` or an empty set and add only what is required.

---

## 4. Versioning the library

Treat this library like any other shared dependency:

- Tag releases with semantic versions, for example `v1.0.0`, and maintain a **major** tag such as `v1` that you move forward to the latest compatible release. Callers that pin `@v1` then receive non-breaking updates.
- Make breaking changes only on a new major tag (`v2`), so existing callers on `@v1` are unaffected.
- Record changes in a changelog and in release notes so callers know when to bump.
- Document each workflow's inputs, secrets, and outputs (as in the top-level README) and treat them as a public contract.

---

## 5. Organisation-level governance (domain 4.0)

At scale, organisations standardise CI/CD by centralising reusable workflows and controlling how repositories may use them.

- **Central library repository.** Keep shared reusable workflows in one repository (such as this one) owned by a platform team, rather than copying YAML across many repos.
- **Required workflows / rulesets.** GitHub lets organisation owners enforce that selected workflows run on pull requests across chosen repositories. This is how an organisation mandates, for example, a security scan or a standard CI on every repo.
- **Actions access policy.** Restrict which actions and reusable workflows are allowed to run: limit to actions created by GitHub, verified creators, or an explicit allow list. This complements pinning by controlling provenance org-wide.
- **Sharing scope.** A reusable workflow can be called from other repositories when it is in a public repository, or in a private repository whose Actions settings permit access from within the organisation or enterprise.
- **Environments and approvals.** Combine reusable workflows with protected environments and required reviewers so that sensitive jobs (deployments, releases) need explicit approval, regardless of which repository calls the workflow.

These controls let an enterprise enforce consistent, auditable, least-privilege automation while still letting individual teams call shared building blocks.

---

## Further study

- Course content and the full GH-200 path: <https://github.com/CertyPro/certy-gh200-course-content>
- Hands-on labs: <https://github.com/CertyPro/gh200-student-actions-lab> and <https://github.com/CertyPro/gh200-custom-actions-lab>
- Debugging practice: <https://github.com/CertyPro/gh200-broken-workflows>
- Enterprise administration: <https://github.com/CertyPro/gh200-enterprise-admin-sim>
- Security challenges: <https://github.com/CertyPro/gh200-security-challenges>
- Certy platform: <https://certy.pro>
