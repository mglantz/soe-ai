---
name: boot_parameters
description: Maintains the ansible/roles/boot_parameters Ansible role that audits GRUB/kernel boot command-line parameters on RHEL-family systems as part of the Linux SOE (audit-only — no automated remediation). Use when checking or hardening kernel boot parameters.
---

# boot_parameters

Maintains `ansible/roles/boot_parameters/`. See `docs/ARCHITECTURE.md` for
the shared conventions. **This role is audit-only by design** — see
"Notes" below.

## Baseline

Encoded in `ansible/roles/boot_parameters/defaults/main.yml`:

- Required on the kernel cmdline: `audit=1` (feeds auditd from early boot;
  pairs with `audit_setup`).
- Forbidden unless explicitly approved: `mitigations=off`, `nopti`,
  `nospectre_v2`, `init=/bin/bash`.

The role reads `GRUB_CMDLINE_LINUX` from `/etc/default/grub` and asserts
required params are present / forbidden params are absent.

## What to do

**Audit** (the only mode this role supports):

```
ansible-playbook ansible/site.yml --tags boot_parameters --check --diff
```

A failed `assert` names the specific missing/forbidden parameter and tells
the user exactly what to edit.

**Remediation is manual**, not automated by this role:

1. Edit `GRUB_CMDLINE_LINUX` in `/etc/default/grub`.
2. Regenerate: `grub2-mkconfig -o /boot/grub2/grub.cfg` (BIOS) or the
   UEFI-specific output path for that host — get this path right, a wrong
   path silently does nothing.
3. Tell the user a **reboot is required** for the change to take effect,
   and do not reboot the host without their explicit confirmation.

**Propose a change to the role itself** (e.g. adding/removing a
required/forbidden parameter to the baseline): never commit directly. On a
branch named `soe/boot_parameters/<short-desc>`, edit the role, validate
locally (`--syntax-check`, `ansible-lint roles/boot_parameters/`,
`--check --diff`), push, and open a PR titled
`[boot_parameters] <what changed>` with the `--check --diff` output in the
body — then stop for human review. See `docs/ARCHITECTURE.md`'s
"Contribution workflow".

## Notes

- Boot parameter changes are high blast-radius: a bad `GRUB_CMDLINE_LINUX`
  edit can leave a host unbootable or without console access, and the
  effect is invisible until the next reboot. That's why this role only
  asserts state instead of attempting an automated fix — see
  `docs/ARCHITECTURE.md`'s "Safety conventions".
