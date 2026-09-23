# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- `DOZZLE_GEN_SELF_SIGNED_CERT` in `env.example` / `.env`, driving the
  service's `GEN_SELF_SIGNED_CERT` proxy opt-in (default `false`).
- `ACME_HOST` and `GEN_SELF_SIGNED_CERT` TLS opt-ins on the dozzle service, so
  an internet-facing cluster serves HTTPS.

### Changed

- The service now declares both proxy TLS opt-ins instead of none. The
  self-signed path is env-controlled and defaults off, so a LAN cluster still
  serves HTTP.

### Deprecated

### Removed

### Fixed

- Corrected the downstream proxy contract: dropped the deprecated
  `LETSENCRYPT_HOST` spelling in favour of the current `ACME_*` spelling.

### Security
