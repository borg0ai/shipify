# Package channels

## Source: GitHub Release only

Every channel installs from the matching OS asset attached to the `gh release` for the tag being shipped. Publish assets for each supported macOS, Linux, and Windows target before updating channels. No registry tarball, no source build, no `head` URL in a channel.

- CLI or app: attach the platform package or archive for each supported OS and architecture. Examples include `.dmg`, `.pkg`, or `.zip` for macOS; `.deb`, `.rpm`, `.AppImage`, or `.tar.gz` for Linux; and `.exe`, `.msi`, or `.zip` for Windows.
- Node CLI: attach the `npm pack` tarball (`name-X.Y.Z.tgz`) as a release asset. Channels use that URL.
- Attach a `SHA256SUMS` file with every release.
- If a channel needs a target the release lacks, stop. Add the target to the release workflow and ship a new release. Never overwrite a published asset.

Pick the form by artifact type, never by repository name. Compute every checksum from the published asset, not from a local build: `curl -fL <url> | shasum -a 256`. Open one pull request per channel repository (see SKILL.md step 10).

## macOS: `../../systembug/luban-homebrew`

Files: `Formula/<name>.rb` for a CLI, `Casks/<name>.rb` for a macOS app. Copy the closest existing entry (`Formula/gopdf.rb`, `Casks/buzz.rb`, `Casks/karen.rb`).

### CLI → formula

- **Native binary** (Go, Rust, Zig): one archive per OS/arch from the GitHub Release.
  ```ruby
  class Name < Formula
    desc "…"
    homepage "https://github.com/OWNER/REPO"
    version "X.Y.Z"
    license "MIT"
    on_macos do
      on_arm   { url ".../name-darwin-arm64.tar.gz", sha256 "…" }
      on_intel { url ".../name-darwin-amd64.tar.gz", sha256 "…" }
    end
    def install = bin.install "name"
    test { assert_match version.to_s, shell_output("#{bin}/name --version") }
  end
  ```
  Write each `url`/`sha256` on separate lines in real code. Add `on_linux` blocks only when the Linux archives exist.
- **Node CLI**: use the release tarball.
  ```ruby
  url "https://github.com/OWNER/REPO/releases/download/vX.Y.Z/name-X.Y.Z.tgz"
  sha256 "…"
  depends_on "node"
  def install
    system "npm", "install", *std_npm_args
    bin.install_symlink libexec.glob("bin/*")
  end
  ```
- The `test` block must run the real binary. Confirm the flag exists (`--version`, `--help`) before using it.

### App → cask

```ruby
cask "name" do
  version "X.Y.Z"
  on_arm do
    url "https://github.com/OWNER/REPO/releases/download/v#{version}/Name-v#{version}-macos-arm64.dmg"
    sha256 "…"
  end
  on_intel do
    url "…-macos-amd64.dmg"
    sha256 "…"
  end
  name "Name"
  desc "…"
  homepage "https://github.com/OWNER/REPO"
  depends_on macos: ">= :monterey"
  auto_updates true   # only when the app updates itself
  app "Name.app"      # path inside the archive; "macos/Name.app" for a tar.gz layout
  zap trash: ["~/Library/Application Support/Name"]
end
```

- Use the DMG/ZIP/PKG that Developer ID signing and notarization covered. Never point a cask at an unsigned build.
- One `version` line plus `#{version}` in URLs. Replace every `PLACEHOLDER_UPDATE_BEFORE_MERGE` before opening the PR.
- Checks: `brew audit --strict --formula Formula/name.rb` or `brew audit --strict --cask Casks/name.rb`, then `brew install` / `brew install --cask` from the local tap path, run the tool once, `brew uninstall`.

## Linux: `../../systembug/cangjie-nix`

Files: `nix/<name>.nix`; add `name = pkgs.callPackage ./<name>.nix { };` to `nix/packages.nix`; add a row to the README package table. Systems: `x86_64-linux`, `aarch64-linux` (`flake.nix`). Set `meta.platforms = lib.platforms.linux` and `meta.mainProgram`.

### CLI

- **Native binary**: `stdenv.mkDerivation` with `fetchurl` per system (`selectSystem`), `nativeBuildInputs = [ autoPatchelfHook ]`, `installPhase` with `install -Dm755`.
- **Node CLI**: `fetchurl` the release tarball, then `buildNpmPackage` (`npmDepsHash`) or, when it has no runtime dependencies, install `dist/` and `makeWrapper` around `${nodejs}/bin/node` (see `nix/gopdf.nix` for the wrapper). The tarball is the `src`, not a GitHub source archive.

### App

- **AppImage**: `appimageTools.wrapType2 { pname; inherit version; src = fetchurl {…}; }`; add `extraInstallCommands` for the `.desktop` file and icon.
- **`.deb`**: `stdenv.mkDerivation` with `dpkg`, `autoPatchelfHook`, `wrapGAppsHook3`, and the runtime libraries the app needs (Tauri: `webkitgtk_4_1`, `gtk3`, `libsoup_3`; Electron: use `electron` from nixpkgs where possible). Install `usr/share/applications` and icons.

### Hashes and checks

Start with `hash = lib.fakeHash;`, run `nix build .#name`, copy the real hash from the error. Then `./result/bin/name --version`, `nix flake check`, `nix fmt`. These need a Linux host. On macOS, report "Nix build not verified" and do not claim success.

## Windows: `../../systembug/shennong-bucket`

File: `bucket/<name>.json` (filename equals app name; CI parses every manifest as JSON). Always include `version`, `description`, `homepage`, `license`, `checkver`, `autoupdate`.

### CLI

- **Native exe in a zip** (the release must publish it):
  ```json
  {
    "version": "X.Y.Z",
    "description": "…",
    "homepage": "https://github.com/OWNER/REPO",
    "license": "MIT",
    "architecture": {
      "64bit": { "url": "https://…/name-X.Y.Z-windows-amd64.zip", "hash": "…" },
      "arm64": { "url": "https://…/name-X.Y.Z-windows-arm64.zip", "hash": "…" }
    },
    "bin": "name.exe",
    "checkver": "github",
    "autoupdate": {
      "architecture": {
        "64bit": { "url": "https://…/name-$version-windows-amd64.zip" },
        "arm64": { "url": "https://…/name-$version-windows-arm64.zip" }
      }
    }
  }
  ```
- **Node CLI**: `"depends": "nodejs"`, `url` is the release tarball, `"extract_dir": "package"`, `"bin": [["dist\\cli.js", "name"]]` (Scoop runs `.js` shims with `node`). If the package has runtime dependencies, add `"installer": { "script": "npm install --omit=dev --prefix \"$dir\"" }`. Skip it when there are none.

### App

- **Portable zip/7z** (preferred): `bin` for any CLI entry, `"shortcuts": [["Name.exe", "Name"]]`, `"persist"` for user data folders.
- **Installer (`.exe`/`.msi`)**: Scoop extracts MSI; for Inno Setup add `"innosetup": true`; for other installers use `"installer": { "script": … }` and `"uninstaller"`. A silent, per-user install only.
- Windows apps must be code-signed per the repository's release contract before they reach the bucket.

### Checks

`pwsh -c "Get-Content bucket/name.json -Raw | ConvertFrom-Json"` (same as CI), then on Windows: `scoop install .\bucket\name.json`, run the tool, `scoop uninstall name`. Run Scoop's `checkver.ps1` and `formatjson.ps1` from its install `bin` folder. Without a Windows host, report "install not verified".
