# Roadmap

`ansible-role-baseline` is a single-role host baseline for fresh Debian servers — SSH hardening, a UFW
firewall, fail2ban, Docker, and Tailscale — layered on
[`brett-buskirk.secure_user`](https://github.com/brett-buskirk/ansible-role-secure_user). It's aimed at
the DigitalOcean droplets provisioned as code: Terraform stands up the box, this role configures it.

## Maintenance

- Keep the role working against current Debian stable releases and Ansible versions.
- Track the Docker and Tailscale apt repos and the `community.general.ufw` module for changes.
- Address bugs and security issues promptly and cut a patch release.

## Nice-to-haves

None are committed or urgent.

- [ ] Molecule + CI test coverage exercising the role against a real container on every PR.
- [ ] Optional `unattended-upgrades` / automatic security updates.
- [ ] Deeper, configurable SSH hardening (ciphers, `MaxAuthTries`, `LoginGraceTime`).
- [ ] Broaden beyond Debian bookworm — Ubuntu and newer Debian are close (the tasks already key off
      `ansible_distribution`); add them to `platforms` once verified.
- [ ] If the baseline grows, split Docker / Tailscale into their own single-purpose roles and depend on
      them here, keeping the estate's granular-role convention.
