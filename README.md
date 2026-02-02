# Resonite Download Action

A GitHub Action to download and cache [Resonite](https://resonite.com/) game files via Steam's depot system using [hazre.ResoniteDownloader](https://github.com/hazre/ResoniteDownloader).

## Features

- Downloads Resonite game files from Steam
- Automatic caching based on version and branch
- Support for specific versions or manifest IDs
- Support for public and protected branches (e.g., headless)
- Outputs version metadata for use in subsequent workflow steps

## Usage

### Basic Example

```yaml
- name: Download Resonite
  uses: hazre/resonite-download-action@v1
  with:
    steam-username: ${{ secrets.STEAM_USERNAME }}
    steam-password: ${{ secrets.STEAM_PASSWORD }}
```

### Download Specific Version

```yaml
- name: Download Resonite 2024.1.28.1342
  uses: hazre/resonite-download-action@v1
  with:
    steam-username: ${{ secrets.STEAM_USERNAME }}
    steam-password: ${{ secrets.STEAM_PASSWORD }}
    version: 2024.1.28.1342
```

### Download Headless Branch

```yaml
- name: Download Resonite Headless
  uses: hazre/resonite-download-action@v1
  with:
    steam-username: ${{ secrets.STEAM_USERNAME }}
    steam-password: ${{ secrets.STEAM_PASSWORD }}
    steam-beta-password: ${{ secrets.STEAM_BETA_PASSWORD }}
    branch: headless
```

### Download by Manifest ID

```yaml
- name: Download Specific Build
  uses: hazre/resonite-download-action@v1
  with:
    steam-username: ${{ secrets.STEAM_USERNAME }}
    steam-password: ${{ secrets.STEAM_PASSWORD }}
    manifest-id: 1234567890
```

### Use Outputs in Subsequent Steps

```yaml
- name: Download Resonite
  id: resonite
  uses: hazre/resonite-download-action@v1
  with:
    steam-username: ${{ secrets.STEAM_USERNAME }}
    steam-password: ${{ secrets.STEAM_PASSWORD }}

- name: Display Version Info
  env:
    VERSION: ${{ steps.resonite.outputs.version }}
    BUILD_ID: ${{ steps.resonite.outputs.build-id }}
    MANIFEST_ID: ${{ steps.resonite.outputs.manifest-id }}
    CACHE_HIT: ${{ steps.resonite.outputs.cache-hit }}
  run: |
    echo "Downloaded version: $VERSION"
    echo "Build ID: $BUILD_ID"
    echo "Manifest ID: $MANIFEST_ID"
    echo "Cache hit: $CACHE_HIT"
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `steam-username` | Steam username for authentication | Yes | - |
| `steam-password` | Steam password for authentication | Yes | - |
| `steam-beta-password` | Beta password for protected branches (e.g., headless) | No | `""` |
| `version` | Specific Resonite version to download | No | Latest |
| `manifest-id` | Specific manifest ID to download | No | `""` |
| `resonite-path` | Path where Resonite will be installed | No | `${{ github.workspace }}/Resonite` |
| `branch` | Branch of Resonite to use (`public`, `headless`, etc.) | No | `public` |

## Outputs

| Output | Description |
|--------|-------------|
| `version` | Version of Resonite that was downloaded |
| `build-id` | Build ID (version without dots) used for cache keys |
| `manifest-id` | Manifest ID that was downloaded |
| `cache-hit` | Whether the cache was hit (`true` or `false`) |

## Caching

This action automatically caches downloaded Resonite files based on:
- **Version/Build ID**: Ensures each version is cached separately
- **Branch**: Public and headless branches are cached independently

When a cache hit occurs, the download step is skipped entirely, significantly reducing workflow execution time and Steam API usage.

Cache keys follow the format: `Resonite-{build-id}-{branch}`

Example: `Resonite-202401281342-public`

## Requirements

- GitHub Actions runner with Ubuntu (recommended) or Windows
- Valid Steam account with access to Resonite
- [DepotDownloader](https://github.com/SteamRE/DepotDownloader) is automatically installed by the ResoniteDownloader tool

## Security Considerations

Store Steam credentials as [GitHub encrypted secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets):

1. Go to your repository **Settings** → **Secrets and variables** → **Actions**
2. Add secrets:
   - `STEAM_USERNAME`: Your Steam username
   - `STEAM_PASSWORD`: Your Steam password
   - `STEAM_BETA_PASSWORD`: Beta password (if using headless or other protected branches)

**Never** commit credentials directly in workflow files.

## Examples

### Matrix Testing Across Branches

```yaml
jobs:
  test:
    strategy:
      matrix:
        branch: [public, headless]
    runs-on: ubuntu-latest
    steps:
      - uses: hazre/resonite-download-action@v1
        with:
          steam-username: ${{ secrets.STEAM_USERNAME }}
          steam-password: ${{ secrets.STEAM_PASSWORD }}
          steam-beta-password: ${{ secrets.STEAM_BETA_PASSWORD }}
          branch: ${{ matrix.branch }}
          resonite-path: ./resonite-${{ matrix.branch }}
```

### Custom Installation Path

```yaml
- name: Download to Custom Path
  uses: hazre/resonite-download-action@v1
  with:
    steam-username: ${{ secrets.STEAM_USERNAME }}
    steam-password: ${{ secrets.STEAM_PASSWORD }}
    resonite-path: /opt/resonite
```

## Troubleshooting

### Authentication Failures

If you encounter Steam authentication errors:
- Verify your credentials are correct
- Check if Steam Guard is enabled (It's not supported, please a dedicated steam account with Steam Guard disabled)
- Ensure your account has access to Resonite (Meaning it's added to your steam library)

### Cache Issues

To force a fresh download and bypass cache:
1. Go to **Actions** → **Caches** in your repository
2. Delete the relevant `Resonite-*` cache entry
3. Re-run the workflow

### Protected Branch Access

For headless or other protected branches:
- Ensure `steam-beta-password` is provided
- Verify you have access to the beta branch in Steam

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Related Projects

- [hazre.ResoniteDownloader](https://github.com/hazre/ResoniteDownloader) - The underlying .NET CLI tool
- [DepotDownloader](https://github.com/SteamRE/DepotDownloader) - Steam depot downloading tool