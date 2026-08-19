# djonehub-release

Self-hosted release host for **DJOneHub** core binaries — the DJI 4G / eSIM
management backend that runs on OpenWRT routers and is controlled from the
iPhone app and the LuCI web page.

The companion package
[`luci-app-djonehub`](https://github.com/563617356/luci-app-djonehub) downloads
its core binary from **this repo** at install / update time. This is the single
place you control which three-architecture Linux binaries ship — you no longer
depend on anyone else's release.

## Asset naming (must match exactly)

For a release tagged `vX.Y.Z` the package expects three assets:

| Asset | Architecture | Router example |
|-------|--------------|----------------|
| `djonehub_vX.Y.Z_linux_arm64` | aarch64 / arm64 | NanoPi, RPi (64-bit), most ARM routers |
| `djonehub_vX.Y.Z_linux_amd64` | x86_64 / amd64 | x86 mini-PC, VM, Protectli |
| `djonehub_vX.Y.Z_linux_armv7` | armv7l / armv7 | older 32-bit ARM routers |

Each asset is a plain Linux ELF executable (no installer). The package drops it
at `/etc/djonehub/bin/djonehub` and the init script starts it with
`exec /etc/djonehub/bin/djonehub -c /etc/djonehub/config/config.yaml`.

## How to publish / update binaries

### Option A — automated (recommended)

Push a tag `vX.Y.Z` to this repo, **or** open the **Build Binaries** workflow
from the Actions tab and pass a version. The workflow clones
[`563617356/DJOneHub-mac-enhanced`](https://github.com/563617356/DJOneHub-mac-enhanced),
cross-compiles the three architectures with `CGO_ENABLED=0` (no libusb/cgo — the
DJI USB-AT path uses `go.bug.st/serial`), and uploads the assets to a GitHub
release of the same tag.

> If `DJOneHub-mac-enhanced` is ever made private, add a repo secret
> `SOURCE_TOKEN` (a PAT with `repo` on that repo); the workflow falls back to it.

### Option B — manual (build locally, upload here)

Build from the source repo with its `build.sh`:

```bash
cd DJOneHub-mac-enhanced
./build.sh vX.Y.Z          # -> dist/djonehub_vX.Y.Z_linux_{arm64,armv7,amd64}
```

Then upload the three files to a GitHub release tagged `vX.Y.Z`:

```bash
gh release create vX.Y.Z \
  --repo 563617356/djonehub-release \
  --title "vX.Y.Z" \
  dist/djonehub_vX.Y.Z_linux_arm64 \
  dist/djonehub_vX.Y.Z_linux_armv7 \
  dist/djonehub_vX.Y.Z_linux_amd64
```

Or just drag-and-drop the three files into a new release in the GitHub web UI.

## Consuming from luci-app-djonehub

The package reads `option release_repo` from `/etc/config/djonehub`
(default `https://github.com/563617356/djonehub-release`). To pin a version:

- set `djonehub_version` when you run the package's **Release Packages** workflow, **or**
- in the LuCI page, pick the version (it queries this repo's latest release), **or**
- `uci set djonehub.main.version='vX.Y.Z'; uci commit djonehub` then install core.
