# TKCM application releases

## Compliance Board v1.2.0

The compliance application now connects to the management server for licensing, heartbeats and manual update checks.

- [Release notes and downloads](https://github.com/JackKelley805/TKCM_Releases/releases/tag/compliance-v1.2.0)
- [Windows x64 portable ZIP](https://github.com/JackKelley805/TKCM_Releases/releases/download/compliance-v1.2.0/ComplianceBoard-1.2.0-windows-x64.zip)
- [Linux x64 portable archive](https://github.com/JackKelley805/TKCM_Releases/releases/download/compliance-v1.2.0/ComplianceBoard-1.2.0-linux-x64.tar.gz)
- [Client checksums](https://github.com/JackKelley805/TKCM_Releases/releases/download/compliance-v1.2.0/SHA256SUMS)

These are portable applications, not the Windows Setup installer. Read START_HERE.txt inside the archive. Preserve existing configuration and data when updating. After signing in, activate under **License & updates** using a `compliance-board` product license from your management server. All four management timing settings must be configured. No private credentials or compliance data are bundled.

Download the management application's Linux deployment kits from [GitHub Releases](https://github.com/JackKelley805/TKCM_Releases/releases).

## Management Application v0.1.5

- [Release notes and downloads](https://github.com/JackKelley805/TKCM_Releases/releases/tag/v0.1.5)
- [Linux `.tar.gz` deployment kit](https://github.com/JackKelley805/TKCM_Releases/releases/download/v0.1.5/management_application-0.1.5.tar.gz)
- [ZIP deployment kit](https://github.com/JackKelley805/TKCM_Releases/releases/download/v0.1.5/management_application-0.1.5.zip)
- [SHA256 checksums](https://github.com/JackKelley805/TKCM_Releases/releases/download/v0.1.5/SHA256SUMS)
- [Release manifest](https://github.com/JackKelley805/TKCM_Releases/releases/download/v0.1.5/RELEASE-MANIFEST.json)

The kit contains the management application's source, Linux installer, Docker/Compose files, MySQL/RDS examples, and setup/API documentation. Docker builds the .NET 8 application on the server; no host .NET SDK is required.

Version 0.1.5 makes enrollment automatic by default:

- **Manual enrollment approval** starts **Off**, including upgrades from versions without this setting. Valid license credentials and device-key proof approve enrollment without a separate administrator step.
- Existing pending requests can advance on their next authenticated retry, status poll, or validation claim after the normal proof/license/ownership/installation checks. Saving the setting does not bulk approve entries.
- Turn manual approval on in Settings when you want new/pending requests reviewed by an administrator. This is independent of Migration/Enforced mode; existing approved/bound keys remain valid.
- Rejected/revoked requests stay denied. Automatic approval never replaces another already approved key for the same installation ID. Bound tokens still require DPoP; seat limits, timing policy, token rotation, nonce/replay protection, and lease rules remain unchanged.
- The API uses the same request fields/routes/statuses: continue on `approved`, or poll on `pending`. Read `API.md`, `LICENSING-INTEGRATION.md`, and `INSTALLATION-AUTHENTICATION.md` **inside the downloaded kit** for implementation details.

Migration remains the default for new deployments; an upgrade retains the existing authentication mode. Keep Migration while legacy clients require Bearer compatibility, and enable Enforced after their enrollment/signed-request rollout. The existing version/MAC/IP telemetry and Settings product catalog are retained. Automatic GitHub release discovery is not implemented by this release. Key possession does not attest an unmodified official executable on a customer-controlled computer.

This is a management release only. It does not update the Compliance Board packages above or deploy software to your running server. Validation passed **56/56 automated scenarios** in the isolated MySQL integration run and a Windows Release build with **zero warnings/errors**. Optional external verified-MySQL-TLS testing was not configured.

## Update an existing Linux server

Plan a maintenance window and verify backups of the management MySQL database, persistent signing/authentication key volume (including `/data/signing-key.pem` and `/data/keys/`), and private deployment configuration before deploying. Keep the same installation directory and Compose project identity.

This release retains security **schema 3**, upgrades older schemas, and adds the approval-policy setting to existing schema-3 databases. Stop **all older management server instances** before starting it; do not run a mixed-version pool. Versions before v0.1.4 refuse schema 3, while v0.1.4 does not honor the new approval setting. Migration preserves old client compatibility, not old server compatibility. A deliberate rollback requires matching pre-upgrade database/key backups and a plan for later state/policy; changing the binary alone is insufficient.

Run the following only when you choose to deploy after completing that preparation. It downloads/checks the kit, stops only the installer-managed `management` service, and runs the installer; the database service remains running. Stop additional application instances separately first. Change `management_install_dir` if your existing installation uses another path.

```bash
(
  set -euo pipefail
  management_install_dir=/opt/management_application
  update_dir=$(mktemp -d "$HOME/tkcm-management-update.XXXXXX")
  cd "$update_dir"
  curl --fail --location --output management_application-0.1.5.tar.gz \
    'https://github.com/JackKelley805/TKCM_Releases/releases/download/v0.1.5/management_application-0.1.5.tar.gz'
  curl --fail --location --output SHA256SUMS \
    'https://github.com/JackKelley805/TKCM_Releases/releases/download/v0.1.5/SHA256SUMS'
  sha256sum --check --ignore-missing SHA256SUMS
  tar -xzf management_application-0.1.5.tar.gz
  cd management_application-0.1.5
  sudo test -f "$management_install_dir/.management-install"
  sudo docker compose --env-file "$management_install_dir/.env" --project-directory "$management_install_dir" -f "$management_install_dir/compose.yaml" stop management
  sudo bash install.sh --install-dir "$management_install_dir"
  sudo docker compose --env-file "$management_install_dir/.env" --project-directory "$management_install_dir" -f "$management_install_dir/compose.yaml" ps
)
```

Continue only if the archive checksum reports `OK`. The installer retains saved settings, database mode/connection configuration, accounts/passwords, licenses, policy durations, signing/authentication keys, and Compose identity, and checks readiness after rebuilding. Credentials do not need to be entered again. The existing authentication mode is preserved; keep Migration while legacy clients need it. Manual approval starts Off when upgrading from versions without this setting. If installation fails, diagnose it before restarting an older binary against schema 3. Do not delete volumes as part of an update.

For a fresh server, read `README.md` and `RELEASE-INFO.md` inside the extracted kit. Prepare a dedicated MySQL/RDS database, private password file, and CA certificate, then run:

```bash
sudo bash install.sh --database external \
  --database-config /etc/management_application/database.json \
  --admin-email you@example.com
```

External mode starts one management container using your existing database; local MySQL mode is also available. A fresh server needs administrator setup. The archives contain no development-computer accounts, credentials, database records, or signing keys. Public HTTPS, AWS network access, backups, and any existing-data migration require server configuration.

Use the named downloadable kit assets above. GitHub's automatically generated source snapshots contain this distribution repository's documentation. The compliance application is released separately.
