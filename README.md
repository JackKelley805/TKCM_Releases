# TKCM application releases

## Compliance Board v1.2.0

The compliance application now connects to the management server for licensing, heartbeats and manual update checks.

- [Release notes and downloads](https://github.com/JackKelley805/TKCM_Releases/releases/tag/compliance-v1.2.0)
- [Windows x64 portable ZIP](https://github.com/JackKelley805/TKCM_Releases/releases/download/compliance-v1.2.0/ComplianceBoard-1.2.0-windows-x64.zip)
- [Linux x64 portable archive](https://github.com/JackKelley805/TKCM_Releases/releases/download/compliance-v1.2.0/ComplianceBoard-1.2.0-linux-x64.tar.gz)
- [Client checksums](https://github.com/JackKelley805/TKCM_Releases/releases/download/compliance-v1.2.0/SHA256SUMS)

These are portable applications, not the Windows Setup installer. Read START_HERE.txt inside the archive. Preserve existing configuration and data when updating. After signing in, activate under **License & updates** using a `compliance-board` product license from your management server. All four management timing settings must be configured. No private credentials or compliance data are bundled.

Download the management application's Linux deployment kits from [GitHub Releases](https://github.com/JackKelley805/TKCM_Releases/releases).

## Management Application v0.1.3

- [Linux `.tar.gz` deployment kit](https://github.com/JackKelley805/TKCM_Releases/releases/download/v0.1.3/management_application-0.1.3.tar.gz)
- [ZIP deployment kit](https://github.com/JackKelley805/TKCM_Releases/releases/download/v0.1.3/management_application-0.1.3.zip)
- [SHA256 checksums](https://github.com/JackKelley805/TKCM_Releases/releases/download/v0.1.3/SHA256SUMS)
- [Release manifest](https://github.com/JackKelley805/TKCM_Releases/releases/download/v0.1.3/RELEASE-MANIFEST.json)

The kit contains the combined management application's source, Linux installer, Docker/Compose files, RDS configuration example, and setup documentation. Docker builds the application on the server; no host .NET SDK is required. Version 0.1.3 adds `POST /api/v1/installations/version`. Send the existing installation bearer token with JSON `{"version":"1.2.3"}` to update that installation's running version and last contact. The response contains `installation_id`, `version`, and `reported_at`; valid SemVer is required. Reporting never renews license validation or extends offline access. Existing heartbeat version reports remain supported, and this endpoint requires no schema change.

## Update an existing Linux server

```bash
(
  set -euo pipefail
  management_install_dir=/opt/management_application
  update_dir=$(mktemp -d "$HOME/tkcm-management-update.XXXXXX")
  cd "$update_dir"
  curl --fail --location --output management_application-0.1.3.tar.gz \
    'https://github.com/JackKelley805/TKCM_Releases/releases/download/v0.1.3/management_application-0.1.3.tar.gz'
  curl --fail --location --output SHA256SUMS \
    'https://github.com/JackKelley805/TKCM_Releases/releases/download/v0.1.3/SHA256SUMS'
  sha256sum --check --ignore-missing SHA256SUMS
  tar -xzf management_application-0.1.3.tar.gz
  cd management_application-0.1.3
  sudo test -f "$management_install_dir/.management-install"
  sudo bash install.sh --install-dir "$management_install_dir"
  sudo docker compose --env-file "$management_install_dir/.env" --project-directory "$management_install_dir" -f "$management_install_dir/compose.yaml" ps
)
```

These commands use the default managed installation directory, `/opt/management_application`; substitute your existing directory if different. The installer retains saved settings, database mode, accounts/passwords, licenses, policy durations, and signing/authentication keys. Updating does not require entering credentials again. Preserve the existing `/data/keys/` authentication-key ring: it also decrypts saved license keys. Older hash-only keys become viewable after their next successful validation; an explicit replacement invalidates the previous key while retaining installation entries.

For a fresh server, read the extracted `README.md` and `RELEASE-INFO.md`. Prepare a dedicated MySQL/RDS database, private password file, and CA certificate, then run:

```bash
sudo bash install.sh --database external \
  --database-config /etc/management_application/database.json \
  --admin-email you@example.com
```

The external mode starts one management container and uses your existing database. Local MySQL mode is also available. A fresh server needs administrator setup; the archives contain no development-computer accounts, credentials, database records, or signing keys. Public HTTPS, AWS network access, backups, and any existing-data migration require server configuration.

The downloadable kits are the named assets above. GitHub's automatically generated source snapshots contain this distribution repository's documentation.

The application currently targets .NET 8; review its runtime lifecycle before public production deployment. See the release notes for validation details and deployment limitations.
