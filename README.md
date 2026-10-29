# NixOS administration skill

An OMP skill for NixOS maintenance best practices. It provides a decision guide and task-specific runbooks, not automatic system administration.

## Source and installation

This repository owns the skill source. The global OMP installation links to this checkout. Git tracks changes here, not in a separate global copy.

## Coverage

- Declarative configuration, pinned options, packages, Home Manager, and development shells.
- Controlled updates, service health, activation checks, and storage retention.
- Backups, data restoration, generation rollback, and boot recovery.
- Remote maintenance, fleet changes, provisioning, encryption, and images.
- Separate personal context for the owner's systems.

## Safety

In-scope configuration edits, evaluation, and builds do not grant permission for live activation. The skill asks for approval before activation, deployment, reboot, destructive cleanup, and disk changes.

Risky maintenance needs evidence of recovery readiness. A successful build does not prove service health, bootability, or data recovery. NixOS generation rollback does not restore databases or other mutable data.

These instructions guide the agent. They are not an access-control mechanism.

## Install into OMP

From the repository root, create the native global skill link:

```bash
mkdir -p "$HOME/.omp/agent/skills"
ln -sT -- "$PWD" "$HOME/.omp/agent/skills/nixos-admin"
```

The current local installation already uses this link. Do not repeat the link command if the target exists. Inspect an existing target before replacing it. Preserve any independent changes.

The link keeps source edits and the installed skill identical. Keep this checkout at a stable path. If you move it, update the link.

## Verify and use

Verify that OMP resolves the installed source and a reference:

```bash
omp read skill://nixos-admin/SKILL.md
omp read skill://nixos-admin/references/maintenance.md
```

Start a fresh OMP session after installation so OMP discovers the skill. Invoke it explicitly:

```text
/skill:nixos-admin Review the maintenance practices for this system.
```

## Repository layout

- [SKILL.md](SKILL.md): entry point, task selection, operating contract, and maintenance principles.
- [Configuration](references/configuration.md): configuration ownership, option checks, packages, and user environments.
- [Maintenance](references/maintenance.md): health assessment, controlled updates, activation, and storage cleanup.
- [Recovery](references/recovery.md): backup evidence, data restoration, rollback, and boot recovery.
- [Remote and provisioning](references/remote-provisioning.md): remote access, deployment, installation, and images.
- [Personal context](references/personal.md): personal preferences and dated host observations.
- [Sources and upkeep](references/sources.md): source links, qualifications, verification notes, and change history.
- [LICENSE](LICENSE): MIT license.

## Maintenance and version control

Edit the files in this repository. Review personal host details before sharing the project or using them on another system. The installed link uses repository changes immediately. It does not pin a Git commit.

Before activation, revalidate the target, candidate configuration, recovery evidence, and approval scope. After skill changes, verify OMP loading and exercise relevant maintenance scenarios without live changes.

## Sources and license

The skill draws on [michalzubkowicz's NixOS management skill](https://github.com/michalzubkowicz/nixos-management-skill), the [official NixOS Wiki](https://wiki.nixos.org/wiki/NixOS_Wiki), and official manuals. References and corrections appear in [Sources and upkeep](references/sources.md). The skill uses the MIT license in [LICENSE](LICENSE).
