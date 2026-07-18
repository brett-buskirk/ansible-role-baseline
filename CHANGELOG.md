# Changelog

All notable changes to ansible-role-baseline are documented here. The format is based on
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial `baseline` Ansible role — a host-hardening baseline for fresh Debian servers, built on
  `brett-buskirk.secure_user` (a role dependency that creates the sudo user, installs its key, and
  disables SSH password auth).
- **SSH:** disables direct root login (`PermitRootLogin no`), validated with `sshd -t` before restart.
- **UFW:** installs and enables the firewall with a default-deny-inbound / allow-outbound policy,
  allowing configured app profiles (SSH first, so it never locks you out) and numeric ports.
- **fail2ban:** installs the service and writes a `jail.local` (enabling the `sshd` jail by default).
- **Docker:** installs Docker Engine + the Compose plugin from Docker's official apt repo and adds
  configured users to the `docker` group.
- **Tailscale:** installs Tailscale from its official apt repo; brings the node onto the tailnet when a
  (vault-supplied) auth key is set.
- Config-driven via `defaults/main.yml`; every block is opt-out. Galaxy metadata targeting Debian
  (bookworm), `min_ansible_version: 2.9`.
