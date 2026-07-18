# ansible-role-baseline

This Ansible role lays down a **baseline for a fresh Debian host**: it hardens SSH (disables direct root
login), stands up a **UFW** firewall and **fail2ban**, and installs **Docker** and **Tailscale**. It
builds on [`ansible-role-secure_user`](https://github.com/brett-buskirk/ansible-role-secure_user) — a
role dependency that runs first to create the sudo user, install its SSH key, and disable password
authentication.

It's meant for the DigitalOcean (or any) Debian droplets you provision as code: `terraform apply` the
box, then apply this role to make it a hardened, Docker- and Tailscale-ready host.

## Requirements

* A Debian target host — **bookworm or trixie** (the tasks key off `ansible_distribution`).
* Ansible, run with root privileges (`become: true`).
* The [`community.general`](https://galaxy.ansible.com/community/general) collection (for the `ufw`
  module). It ships with the full `ansible` package; install it explicitly with
  `ansible-galaxy collection install community.general` if you're on `ansible-core`.
* The [`brett-buskirk.secure_user`](https://github.com/brett-buskirk/ansible-role-secure_user) role
  (a dependency — see below). As it requires, the new user's public key must already be in
  `/root/.ssh/authorized_keys` on the target host **before** the first run, so you don't lose access
  when root login is disabled.

## Role Variables

| Variable | Description | Default |
|---|---|---|
| `user_name` | Passed through to `secure_user` — the sudo user to create. | `None` (required) |
| `baseline_disable_root_login` | Set `PermitRootLogin no` in `sshd_config`. | `true` |
| `baseline_ufw_enabled` | Install and enable UFW. | `true` |
| `baseline_ufw_allow` | UFW application profiles to allow inbound (SSH is allowed before enable). | `["OpenSSH"]` |
| `baseline_ufw_allow_ports` | Extra numeric rules as `"port/proto"` (proto defaults to `tcp`), e.g. `["80/tcp", "443/tcp"]`. | `[]` |
| `baseline_fail2ban_enabled` | Install fail2ban and enable a jail. | `true` |
| `baseline_fail2ban_jail_local` | Contents of `/etc/fail2ban/jail.local`. Default enables the `sshd` jail. | `[sshd]\nenabled = true` |
| `baseline_install_docker` | Install Docker Engine + Compose plugin from Docker's apt repo. | `true` |
| `baseline_docker_users` | Users to add to the `docker` group (root-equivalent — keep tight). | `[]` |
| `baseline_install_tailscale` | Install Tailscale from its apt repo. | `true` |
| `baseline_tailscale_authkey` | Auth key to `tailscale up` non-interactively. Empty = install only. **Pass from a vault; never commit one.** | `""` |
| `baseline_tailscale_up_args` | Extra flags for `tailscale up` (e.g. `--ssh`). | `""` |

Every block is opt-out — set its toggle (`baseline_disable_root_login`, `baseline_ufw_enabled`,
`baseline_fail2ban_enabled`, `baseline_install_docker`, `baseline_install_tailscale`) to `false` to skip
it.

## Dependencies

* [`brett-buskirk.secure_user`](https://github.com/brett-buskirk/ansible-role-secure_user) — runs first
  (via `meta/main.yml`) to create the sudo user and set up key-based SSH. This role then hardens root
  login on top of it, so ordering is guaranteed.

## Example Playbook

```yaml
- hosts: your_hosts
  become: true
  vars:
    user_name: brett                     # for secure_user
    baseline_docker_users:
      - brett
    baseline_ufw_allow_ports:
      - "80/tcp"
      - "443/tcp"
    # From a vault at runtime — do not commit a real key:
    baseline_tailscale_authkey: "{{ vault_tailscale_authkey }}"
    baseline_tailscale_up_args: "--ssh"
  roles:
    - baseline                           # pulls in secure_user automatically
```

## Installation

Include both roles in your `requirements.yml`:

```yaml
---
roles:
  - name: brett-buskirk.secure_user
  - name: brett-buskirk.baseline
```

Then install them:

```bash
ansible-galaxy install -r requirements.yml
ansible-galaxy collection install community.general
```

## Role Tasks

With `secure_user` having already created the user and set up key auth, this role:

* **Hardens SSH:** disables direct root login (`PermitRootLogin no`), validating the config with
  `sshd -t` before saving and restarting SSH via handler.
* **Firewall (UFW):** installs UFW, allows the configured app profiles (SSH first) and ports, then sets
  a default-deny-inbound / allow-outbound policy and enables it — allow rules go in before enable, so it
  can't lock you out.
* **fail2ban:** installs fail2ban and writes a `jail.local` (enabling the `sshd` jail by default), then
  enables and starts the service.
* **Docker:** installs Docker Engine + the Compose plugin from Docker's official apt repo, enables the
  service, and adds `baseline_docker_users` to the `docker` group.
* **Tailscale:** installs Tailscale from its official apt repo and enables `tailscaled`; if a
  `baseline_tailscale_authkey` is supplied, brings the node onto the tailnet (`no_log`).

## Usage

1. **Provision the box** (e.g. `terraform apply`) and ensure the user's public key is in
   `/root/.ssh/authorized_keys` on it.
2. **Set `user_name`** and any `baseline_*` overrides in your playbook/inventory.
3. **Include the `baseline` role** — `secure_user` runs automatically first.
4. Supply `baseline_tailscale_authkey` from a vault if you want the host joined to your tailnet on the
   first run.

## License

MIT

## Author Information

Brett Buskirk

## Galaxy Tags

```
baseline, hardening, security, firewall, ufw, fail2ban, docker, tailscale, server, linux
```
