# SCARLET-X-Probe private one-tap update design (2026-10-23)

## Requirements
- Keep both source and .deb private. SE uses RootHide, ElleKit, Sileo, Filza, NewTerm.
- Do not publish Pages/APT repo, Releases, or publicly accessible URLs. Retain existing SCARLET-X-Probe/develop workflow (macOS RootHide deb in Actions artifact for 14 days).
- Distinguish build success, package installation, and X process loaded runtime version.

## Recommended MVP: authenticated Actions artifact to local Sileo
1. On SE, provision fine-grained GitHub PAT limited to *one* repo `Botanium-git/SCARLET-X-Probe`, Actions: read, Metadata: read (normally implicit); set expiration. Never put token in URL, GitHub issue, Actions variables, shell history or exported logs. Store in system keychain or a protected credential store; runtime passes via Authorization header. Rotation/revocation must be supported.
2. Updater contacts GitHub API using TLS, lists workflow runs for specific `build.yml` workflow with `branch=develop`, `event=push`, `status=success`; verify most recent eligible run and `head_branch`, `head_sha`, `conclusion`, repo, and artifact name `SCARLET-X-Probe-roothide`.
3. List run artifacts, choose non-expired artifact associated with the verified run and exact name. GET its download endpoint with Actions read; API returns a short-lived 302 URL pointing to ZIP. Download ZIP via HTTPS. NEVER publish redirect URL; token not sent to unrelated redirected host.
4. Securely unpack to temporary private directory. Reject paths that escape extraction root, symlinks, nested archives, multiple .deb, oversized ZIP, missing fields. Inspect `dpkg-deb -f`: Package must equal `com.botanium.scarletxprobe`, Architecture `iphoneos-arm64e` and version strictly newer than locally installed version. Confirm package contains only expected library/plist and package control scripts are reviewed. Reject downgraded or unexpected package. Package digest can additionally match a **separately authenticated** manifest, but hash served via same compromised source is not strong provenance.
5. MVP **does not invoke root commands from an unprivileged iOS shortcut**. It opens local package in Filza/Sileo for user's confirmation. This is near-one tap to fetch and one tap to install; require X restart and runtime Probe version check.
6. Future full one-tap: create narrowly scoped root-owned local installer/launcher with strictly fixed package ID and protected staging path, version checks, safe input validation, no arbitrary shell commands and explicit user authorization. Avoid placing an unprotected root execution service in X. An iOS shortcut cannot by default perform dpkg privileged install just by accessing HTTP URL.
7. A fully Sileo-driven alternative requires a **private authenticated APT endpoint**, implementing Release/InRelease/Packages/.deb under authorization and secure credential handling. Sileo/private APT auth support must be device-tested; do not rely on PAT in repository URL. More operational complexity than GitHub API updater, so not MVP.

## Failure and rollback
- Fail closed on missing credentials, expired artifact, GitHub API failure, build failure, incomplete downloads, wrong package, invalid version, integrity check error; do not alter installed package.
- Back up previously installed known-good .deb securely if available (not a guarantee of automatic dpkg rollback); don't claim atomic update. After installing, relaunch official X and confirm version, class snapshots and Drive export before reporting device success.
- Artifact retention is 14 days currently; GitHub run must have available artifact or fallback private distribution mechanism. Do not assume indefinite availability.

## Verified repository state
- Probe repo private; develop workflow `.github/workflows/build.yml` has `push: develop`, RootHide build and named artifact, retention 14 days. Existing `control` lists Package `com.botanium.scarletxprobe`, Architecture `iphoneos-arm64e`, version `0.1.12`.
- **Design only:** no uploader, installer, token provisioning, Sileo feed, or SE device automation implemented yet. Keep main unchanged.

References: GitHub REST API Actions artifacts docs (fine-grained PAT: Actions read; artifact download via 302 expiring after 1 minute); Sileo official site describes APT package source support.
