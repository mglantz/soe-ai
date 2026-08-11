# soe-ai

Experimentation with building a Linux Standard Operating Environment (SOE)
out of specialized AI agents, defined as Claude Code skills. Each skill owns
one configuration domain, and its deliverable is an **Ansible role** that
audits and remediates that domain — instead of one monolithic hardening
script.

## Layout

```
ansible/
  site.yml                    # top-level playbook: runs every domain role
  inventory/hosts.ini          # sample inventory — add managed hosts here
  roles/
    timesync/                  # reference implementation
    accounts_policy/           # local account/password policy
    audit_setup/                # auditd configuration and rules
    boot_parameters/             # GRUB/kernel command line baseline (audit-only)
    cron_setup/                   # cron service and access control
    vm_guest_agent/                # guest agent matching the detected hypervisor
    motd_issue/                     # /etc/motd and /etc/issue* banners
    system_keyboard/                # console/X11 keyboard layout
    system_locale/                   # system locale (LANG, etc.)
    timezone/                         # system timezone
    troubleshooting_tools/             # baseline troubleshooting package set
    usbguard_setup/                     # USBGuard policy (service enablement is opt-in)
.claude/skills/
  soe/SKILL.md                # orchestrator: how/when to run ansible/site.yml
  <domain>/SKILL.md            # per-domain: baseline + how to run/extend the role
.github/workflows/
  ansible-ci.yml              # syntax-check + ansible-lint on every PR touching ansible/**
docs/
  ARCHITECTURE.md            # design rationale, audit/remediate conventions, safety rules
```

All 12 domain roles are fully implemented — real, idempotent Ansible tasks,
not stubs.

## Audit vs. remediate

Every domain maps onto Ansible's own check mode, so there's no bespoke
report format to maintain:

```sh
# Audit (read-only, safe)
ansible-playbook ansible/site.yml --tags timesync --check --diff

# Remediate (modifies the system)
ansible-playbook ansible/site.yml --tags timesync

# Everything at once
ansible-playbook ansible/site.yml --check --diff
```

See `docs/ARCHITECTURE.md` for why `command`/`shell`-based checks needed
`check_mode: false` to behave correctly under `--check`, and for the
per-role safety conventions (e.g. `usbguard_setup` never enables
enforcement by default — see its `SKILL.md`).

First-class target platform is RHEL-family Linux (RHEL, CentOS Stream,
Fedora).

## How each skill maintains its role

No skill/agent pushes changes to its role directly. Each proposes changes
on a branch (`soe/<domain>/<short-desc>`) and opens a PR
(`[<domain>] <what changed>`) for a human operator to review and merge.
`.github/workflows/ansible-ci.yml` runs `--syntax-check` and `ansible-lint`
on every such PR automatically. See `docs/ARCHITECTURE.md`'s "Contribution
workflow" for the full process.

See `docs/ARCHITECTURE.md` for the full design.
