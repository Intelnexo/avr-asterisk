# Changelog

All notable changes to the **avr-asterisk** project will be documented in this file.

## [1.2.0] - 2026-01-27

### Added
- **Configuration:**
    - `asterisk/conf/campaign-extensions.conf.example`: Example configuration for automated voice campaigns.
- **Documentation:**
    - Added `FEATURES.md` with a detailed list of supported PBX capabilities.
    - Added `TROUBLESHOOTING.md` for common setup and audio issues.
    - Added `estructura-funcional.md` describing the internal dialplan logic.

### Changed
- **Deployment:**
    - Updated `Dockerfile` for better compatibility with ARM and AMD64 architectures.
    - Refined `docker-compose-asterisk.yml` with optimized volume mounts and environment variables.
- **Maintenance:**
    - Repository cleanup: Switched to `develop` as the primary branch and removed legacy branches (`main`, `enable-amd`).

### Fixed
- Cleaned up `.DS_Store` files and other OS-specific metadata from the repository.
- Improved persistence of Asterisk logs and CDRs within Docker volumes.
