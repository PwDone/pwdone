# Releasing

Maintainer notes for publishing a PwDone build as a GitHub Release.

This repository is the **product hub**. Installers are published as release
assets and are never committed to git. The application source lives in a
separate private repository.

## 1. Build

Build the edition(s) you ship from the application source:

```powershell
pnpm run all          # or: pnpm run pdf / archives / office ...
```

Default Tauri artifact layout:

```text
src-tauri\target\release\bundle\nsis\PwDone_<version>_x64-setup.exe
src-tauri\target\release\bundle\msi\PwDone_<version>_x64_en-US.msi
src-tauri\target\release\bundle\dmg\PwDone_<version>_aarch64.dmg
```

Single-format builds use that edition's name in the paths.

## 2. Checksums

```powershell
Get-ChildItem -Recurse -Include *.exe,*.msi,*.dmg | ForEach-Object {
  "{0}  {1}" -f (Get-FileHash $_.FullName -Algorithm SHA256).Hash.ToLower(), $_.Name
} | Set-Content SHA256SUMS.txt -Encoding ascii
```

On macOS:

```bash
shasum -a 256 *.dmg > SHA256SUMS-macos.txt
```

## 3. Tag and release

The tag must match the application version: `v0.1.0`.

```powershell
gh auth login

gh release create v0.1.0 `
  --title "PwDone 0.1.0" `
  --notes-file RELEASE_NOTES.md `
  --draft `
  "D:\path\to\PwDone_0.1.0_x64-setup.exe" `
  "D:\path\to\SHA256SUMS.txt"
```

Run this inside the product-hub repository (its remote must point at the repo),
or pass `--repo <owner>/<repo>`.

Review the draft, download the assets and verify the checksums, then publish:

```powershell
gh release edit v0.1.0 --draft=false --latest
```

Add an asset later:

```powershell
gh release upload v0.1.0 "D:\path\to\PwDone_0.1.0_aarch64.dmg" --clobber
```

## 4. Release notes

Write `RELEASE_NOTES.md` (or use `--generate-notes` when the commit history is
useful). Include:

- Version and supported platforms.
- Download file names.
- Changes in this release.
- The checksum file name.
- Signing status, for example: "The Windows build is not code-signed yet;
  SmartScreen may warn."
- For macOS: publish only after notarization, or mark the build as unnotarized.

## 5. Rules

- Windows first; publish macOS after notarization.
- Never commit installers, bundled engine payloads or keys.
- Do not upload user data, logs with personal paths, or license keys.
- Keep the tag and the application version in sync.
- Prefer a draft release; verify the downloads before publishing.
- GitHub Releases is a mirror. `downloads.pwdone.com` remains the primary
  download host used by the website.
