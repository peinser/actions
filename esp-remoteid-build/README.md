
# Build ESP RemoteID

Reusable workflow for building ESP-IDF firmware projects inside the official `espressif/idf` Docker container.

## Usage

```yaml
jobs:
  firmware:
    uses: peinser/actions/.github/workflows/esp-remoteid-build.yml@v1
    with:
      idf-version: release-v5.5
      target: esp32s3
    permissions:
      contents: read
```

With a custom sdkconfig overlay (e.g. from the Remote ID web configurator):

```yaml
jobs:
  firmware:
    uses: peinser/actions/.github/workflows/esp-remoteid-build.yml@v1
    with:
      idf-version: release-v5.5
      target: esp32s3
      sdkconfig-overlay-b64: ${{ inputs.sdkconfig-overlay-b64 }}
      artifact-name: firmware-${{ inputs.build-id }}
      artifact-retention-days: 1
    permissions:
      contents: read
```

## Inputs

| Input | Type | Default | Description |
|---|---|---|---|
| `idf-version` | string | `release-v5.5` | `espressif/idf` Docker image tag |
| `target` | string | `esp32s3` | Chip target (`esp32s3`, `esp32`, `esp32c3`, ...) |
| `project-path` | string | `.` | Path to the ESP-IDF project root (contains `CMakeLists.txt`) |
| `sdkconfig-overlay-b64` | string | _(empty)_ | Base64-encoded `sdkconfig.defaults` overlay, merged on top of the project's own defaults |
| `artifact-name` | string | `firmware` | Name for the uploaded artifact |
| `artifact-retention-days` | number | `7` | Artifact retention period in days (1-90) |

## Outputs

| Output | Description |
|---|---|
| `artifact-id` | GitHub Actions artifact ID |
| `artifact-url` | Artifact URL (requires a GitHub token to download) |

## How it works

1. Checks out the calling repository with submodules.
2. Restores a ccache compiler cache keyed on IDF version, chip target, and source file hashes.
3. If `sdkconfig-overlay-b64` is provided, decodes it to `sdkconfig.overlay` and sets `SDKCONFIG_DEFAULTS=sdkconfig.defaults;sdkconfig.overlay` so ESP-IDF merges it on top of the project defaults.
4. Runs `idf.py set-target <target> build` inside the `espressif/idf` container.
5. Uploads the app binary, bootloader, partition table, and flash argument files as a single artifact.

ccache (not the CMake build directory) is cached across runs, avoiding stale CMake state while still accelerating recompilation of unchanged translation units.
