# Update Release Tags

A GitHub Action that automatically creates or updates sliding version tags (e.g. `v1`, `v1.3`) when a new semver tag is released. Supports multi-level sliding, prerelease skipping, and dry-run mode.

## Usage

```yaml
- uses: carry0987/update-release-tags@v1
  with:
    tag: ${{ github.event.release.tag_name }}
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `tag` | Yes | — | The source semver tag to slide from (e.g. `v1.3.0`) |
| `levels` | No | `major` | Comma-separated tag levels to update: `major`, `minor` |
| `prefix` | No | `v` | Tag prefix |
| `skip-prerelease` | No | `true` | Skip tag update if the source tag is a prerelease |
| `dry-run` | No | `false` | Log what would be done without modifying tags |

## Outputs

| Output | Description | Example |
|--------|-------------|---------|
| `tags-updated` | Comma-separated list of tags created/updated | `v1,v1.3` |
| `commit-sha` | The commit SHA the source tag points to | `abc1234...` |
| `skipped` | Whether the update was skipped | `true` / `false` |

## Examples

### Basic — Update major tag only

```yaml
on:
  release:
    types: [published]

jobs:
  update-tags:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v6
        with:
          fetch-depth: 0
          fetch-tags: true

      - uses: carry0987/update-release-tags@v1
        with:
          tag: ${{ github.event.release.tag_name }}
```

When `v1.3.0` is released, this updates `v1` to point to the same commit.

### Multi-level — Update both major and minor tags

```yaml
- uses: carry0987/update-release-tags@v1
  with:
    tag: ${{ github.event.release.tag_name }}
    levels: major,minor
```

When `v1.3.0` is released, this updates both `v1` and `v1.3`.

### With Release Please

```yaml
jobs:
  release-please:
    runs-on: ubuntu-latest
    outputs:
      release_created: ${{ steps.release.outputs.release_created }}
      tag_name: ${{ steps.release.outputs.tag_name }}
    steps:
      - uses: googleapis/release-please-action@v4
        id: release

  update-tags:
    needs: release-please
    if: needs.release-please.outputs.release_created == 'true'
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v6
        with:
          fetch-depth: 0
          fetch-tags: true

      - uses: carry0987/update-release-tags@v1
        with:
          tag: ${{ needs.release-please.outputs.tag_name }}
          levels: major,minor
```

### Prerelease skipping (default)

By default, prerelease tags are skipped:

```yaml
- uses: carry0987/update-release-tags@v1
  with:
    tag: v2.0.0-beta.1
# Skipped — v2 is NOT updated
```

To disable this behavior:

```yaml
- uses: carry0987/update-release-tags@v1
  with:
    tag: v2.0.0-beta.1
    skip-prerelease: false
# v2 IS updated
```

### Dry-run

```yaml
- uses: carry0987/update-release-tags@v1
  id: dry
  with:
    tag: v1.3.0
    levels: major,minor
    dry-run: true

- run: echo "Would update: ${{ steps.dry.outputs.tags-updated }}"
```

## License

[MIT](LICENSE)
