# Update Kustomize/Helm Image Tag Action

A composite GitHub Action that updates image tags in Kubernetes manifests (kustomization.yaml) or Helm releases and commits the changes. Supports both standard GitHub Actions bot authentication and GitHub App authentication for Flux GitOps workflows.

## Features

- Updates image tags in kustomization.yaml files or Helm release manifests
- Supports two authentication methods:
  - Standard `github-actions[bot]` (default)
  - GitHub App authentication for Flux GitOps workflows
- Flexible tag pattern matching (`newTag` for kustomization, `tag` for Helm)
- Optional `[skip ci]` commit message suffix
- Configurable branch targeting

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `image-tag` | Image tag to update in the manifest | Yes | - |
| `manifest-path` | Path to the manifest file | Yes | - |
| `branch-name` | Branch name to push to | No | `github.ref_name` |
| `use-flux-app-auth` | Use GitHub App authentication | No | `false` |
| `flux-app-id` | GitHub App ID (required if using flux auth) | No | - |
| `flux-app-private-key` | GitHub App private key (required if using flux auth) | No | - |
| `update-pattern` | Pattern to update: `newTag` or `tag` | No | `tag` |
| `skip-ci` | Add `[skip ci]` to commit message | No | `false` |

## Usage Examples

### Example 1: Basic Usage (Standard Bot Authentication)

This example uses the default `github-actions[bot]` for authentication and updates a kustomization.yaml file.

```yaml
update-image-tag:
  name: Update Kustomize Image Tag in Manifest
  runs-on: ubuntu-latest
  needs: [build-and-push, create-multi-arch-manifest]
  if: needs.build-and-push.result == 'success'
  steps:
    - name: Update manifest image to ${{ inputs.image-tag || env.TAG }}
      uses: bisnow/github-actions-update-image-tag-k8s@v1.0
      with:
        image-tag: ${{ inputs.image-tag || env.TAG }}
        manifest-path: .k8s/overlays/dev/kustomization.yaml
        update-pattern: newTag
```

### Example 2: GitHub App Authentication with Kustomization

This example uses GitHub App authentication (for Flux GitOps) and includes `[skip ci]` in the commit message.

```yaml
update-image-tag:
  name: Update Kustomize Image Tag in Manifest
  runs-on: ubuntu-latest
  needs: [build-and-push, create-multi-arch-manifest]
  if: needs.build-and-push.result == 'success'
  steps:
    - name: Update manifest image to ${{ inputs.image-tag || env.TAG }}
      uses: bisnow/github-actions-update-image-tag-k8s@v1.0
      with:
        image-tag: ${{ inputs.image-tag || env.TAG }}
        manifest-path: .k8s/overlays/dev/kustomization.yaml
        update-pattern: newTag
        use-flux-app-auth: true
        flux-app-id: ${{ secrets.FLUX_APP_ID }}
        flux-app-private-key: ${{ secrets.FLUX_APP_PRIVATE_KEY }}
        skip-ci: true
```

### Example 3: GitHub App Authentication with Helm Release

This example uses GitHub App authentication and updates a Helm release manifest with the `tag` pattern.

```yaml
update-image-tag:
  name: Update Kustomize Image Tag in Manifest
  runs-on: ubuntu-latest
  needs: [build-and-push, create-multi-arch-manifest]
  if: needs.build-and-push.result == 'success'
  steps:
    - name: Update manifest image to ${{ inputs.image-tag || env.TAG }}
      uses: bisnow/github-actions-update-image-tag-k8s@v1.0
      with:
        image-tag: ${{ inputs.image-tag || env.TAG }}
        manifest-path: .k8s/dev/dev-helm-release.yaml
        update-pattern: tag
        use-flux-app-auth: true
        flux-app-id: ${{ secrets.FLUX_APP_ID }}
        flux-app-private-key: ${{ secrets.FLUX_APP_PRIVATE_KEY }}
        skip-ci: true
```

## Authentication Methods

### Standard Bot Authentication (Default)

- Uses `github-actions[bot]` for git commits
- No additional configuration required
- Suitable for repositories where the default GitHub Actions bot has write access

### GitHub App Authentication

- Uses a custom GitHub App (e.g., `bisnow-flux-gitops-app[bot]`)
- Requires `flux-app-id` and `flux-app-private-key` to be configured as secrets
- Recommended for Flux GitOps workflows where you need specific app permissions
- Allows bypassing certain branch protection rules if the app has appropriate permissions

## Update Patterns

### `newTag` Pattern

Used for kustomization.yaml files. Updates lines like:

```yaml
newTag: v1.0.0
```

to:

```yaml
newTag: v1.2.3
```

### `tag` Pattern

Used for Helm release manifests. Updates lines like:

```yaml
tag: v1.0.0
```

to:

```yaml
tag: v1.2.3
```

## Commit Message Format

- Default: `chore: update image to <tag>`
- With `skip-ci: true`: `chore: update image to <tag> [skip ci]`

The `[skip ci]` suffix prevents triggering CI pipelines on the manifest update commit, which is useful in GitOps workflows to avoid infinite loops.

## Versioning

This action uses rolling major version tags. You can pin to:

- A specific version: `@v3.1.0` (exact, never changes)
- A major version: `@v3` (recommended, gets bug fixes and new features)

When a new semantic version tag (e.g., `v3.2.0`) is pushed, a GitHub Actions workflow automatically updates the corresponding major version tag (`v3`) to point to the new release.