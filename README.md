# vinet-cicd-actions

A collection of reusable composite GitHub Actions for CI/CD workflows. These actions are designed to be referenced from other repositories to standardise authentication and action discovery patterns.

## Actions

### `checkout-custom-actions`

Checks out this (or another) CI/CD actions repository and moves its actions into the `./actions/` directory so they can be referenced by relative path in subsequent steps.

**Path:** `.github/actions/checkout-custom-actions`

#### Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `actions_repository` | Yes | — | Repository containing the CI/CD actions to check out (e.g. `Vinetau/vinet-cicd-actions`). Set `GH_TOKEN` as a job-level env var for private repositories. |
| `actions_ref` | No | `main` | Branch or tag to check out |

#### Usage

```yaml
- uses: Vinetau/vinet-cicd-actions/.github/actions/checkout-custom-actions@main
  with:
    actions_repository: Vinetau/vinet-cicd-actions
    actions_ref: main
  env:
    GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

After this step, actions from the checked-out repository are available under `./actions/<action-name>` and can be called with:

```yaml
- uses: ./actions/my-action
```

---

### `github-app-token`

Generates a short-lived GitHub App installation token and exports it as the `GH_TOKEN` environment variable for subsequent steps in the job.

**Path:** `.github/actions/github-app-token`

#### Inputs

| Input | Required | Description |
|-------|----------|-------------|
| `client_id` | Yes | GitHub App Client ID |
| `private_key` | Yes | GitHub App private key (PEM format) |

#### Usage

```yaml
- uses: Vinetau/vinet-cicd-actions/.github/actions/github-app-token@main
  with:
    client_id: ${{ secrets.APP_CLIENT_ID }}
    private_key: ${{ secrets.APP_PRIVATE_KEY }}

# GH_TOKEN is now set for all subsequent steps in this job
- run: gh api /repos/${{ github.repository }}
```

The token is scoped to the repository owner (`github.repository_owner`) and has a short lifespan, making it preferable to long-lived personal access tokens.

---

## Typical Workflow Pattern

A common pattern is to generate an app token first, then use it to check out private action repositories:

```yaml
jobs:
  example:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: Vinetau/vinet-cicd-actions/.github/actions/github-app-token@main
        with:
          client_id: ${{ secrets.APP_CLIENT_ID }}
          private_key: ${{ secrets.APP_PRIVATE_KEY }}

      - uses: Vinetau/vinet-cicd-actions/.github/actions/checkout-custom-actions@main
        with:
          actions_repository: Vinetau/vinet-cicd-actions

      - uses: ./actions/some-other-action
```

## Notes

- All shell steps use PowerShell (`pwsh`) for cross-platform compatibility.
- Private repository access requires `GH_TOKEN` to be set at the job level before calling `checkout-custom-actions`.
- Actions in this repository are versioned by branch/tag. Pin to a specific tag (e.g. `@v1.2.0`) in production workflows to avoid unexpected changes.