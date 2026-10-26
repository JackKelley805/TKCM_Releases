# TKCM management application releases

Download the management application's Linux deployment kits from [GitHub Releases](https://github.com/JackKelley805/TKCM_Releases/releases).

## Version 0.1.0

- [Linux `.tar.gz` deployment kit](https://github.com/JackKelley805/TKCM_Releases/releases/download/v0.1.0/management_application-0.1.0.tar.gz)
- [ZIP deployment kit](https://github.com/JackKelley805/TKCM_Releases/releases/download/v0.1.0/management_application-0.1.0.zip)
- [SHA256 checksums](https://github.com/JackKelley805/TKCM_Releases/releases/download/v0.1.0/SHA256SUMS)
- [Release manifest](https://github.com/JackKelley805/TKCM_Releases/releases/download/v0.1.0/RELEASE-MANIFEST.json)

The kit contains the combined management application's source, Linux installer, Docker/Compose files, RDS configuration example, and setup documentation. Docker builds the application on the server; no host .NET SDK is required.

## Download on Linux

```bash
mkdir -p ~/tkcm-downloads
cd ~/tkcm-downloads
curl --fail --location --output management_application-0.1.0.tar.gz \
  https://github.com/JackKelley805/TKCM_Releases/releases/download/v0.1.0/management_application-0.1.0.tar.gz
curl --fail --location --output SHA256SUMS \
  https://github.com/JackKelley805/TKCM_Releases/releases/download/v0.1.0/SHA256SUMS
sha256sum --check --ignore-missing SHA256SUMS
tar -xzf management_application-0.1.0.tar.gz
cd management_application-0.1.0
```

Read the extracted `README.md` and `RELEASE-INFO.md`. Prepare a dedicated MySQL/RDS database, private password file, and CA certificate, then run:

```bash
sudo bash install.sh --database external \
  --database-config /etc/management_application/database.json \
  --admin-email you@example.com
```

The external mode starts one management container and uses your existing database. Local MySQL mode is also available. A fresh server needs administrator setup; the archives contain no development-computer accounts, credentials, database records, or signing keys. Public HTTPS, AWS network access, backups, and any existing-data migration require server configuration.

The downloadable kits are the named assets above. GitHub's automatically generated source snapshots contain this distribution repository's documentation.

The application currently targets .NET 8; review its runtime lifecycle before public production deployment. See the release notes for validation details and deployment limitations.
