# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

### Changed

### Deprecated

### Removed

### Fixed

- Corrected the downstream proxy contract: dropped the deprecated
  `LETSENCRYPT_HOST` spelling and made the service HTTP-only in every variant
  (no TLS opt-in), so it is served over plain HTTP whether the proxy cluster is
  LAN-only or internet-facing.

### Security
