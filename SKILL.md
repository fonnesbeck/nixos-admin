---
name: nixos-admin
description: Guide best practices for maintaining NixOS systems. Use for system administration, declarative configuration, controlled upgrades, nixos-rebuild, failed services, disk or Nix store pressure, generations, rollback, backup and boot recovery, remote deployment, provisioning, Home Manager, and Nix development environments. Prioritise reproducibility, recovery readiness, minimal changes, and verified service health.
license: MIT
metadata:
  version: "1.0.3"
---

# NixOS administration

## Purpose

Inform and apply best practices for maintaining a NixOS system. Explain the decision, its tradeoffs, and the evidence needed; do not turn each request into a platform redesign. Keep systems reproducible, changes reviewable, upgrades controlled, recovery viable, and health observable.

This is a Markdown decision guide, not an automation service or security boundary. It grants no privileges, schedules nothing, and does not override user or project instructions.

## Load only the relevant runbook

Resolve these paths relative to this skill directory, or use `skill://nixos-admin/references/<file>` in OMP.

| Task | Read before acting |
| --- | --- |
| Configure modules, verify options, manage packages, Home Manager, or development shells | [Configuration](references/configuration.md) |
| Assess health, update dependencies or releases, build, activate, or clean storage | [Maintenance](references/maintenance.md) |
| Establish backups, restore data, roll back, or recover boot/storage | [Recovery](references/recovery.md) |
| Remote/fleet maintenance, installation, disk layouts, images, LUKS, or impermanence | [Remote and provisioning](references/remote-provisioning.md) |
| Work on Chris's machines, including avalon | [Personal context](references/personal.md); revalidate observations |
| Resolve documentation conflicts or update this skill | [Sources and upkeep](references/sources.md) |

## Operating contract

- Read-only discovery and requested, in-scope configuration edits, evaluation, and builds may proceed without a separate approval. Builds consume resources, write the store, and can execute code from dependencies: inspect untrusted inputs first; check capacity before substantial builds.
- Ask for explicit approval of the exact host, change, and action before live activation or deployment. This includes `nixos-rebuild test`, `switch`, `boot`, rollback, Home Manager activation, service restarts, and changes to credentials or live network state. An explicit request approving that exact action is sufficient; do not ask twice. General permission to maintain a system is not approval of an unspecified activation.
- Ask separately before reboot/shutdown, generation or data deletion, garbage collection, formatting, partitioning, filesystem repair, or encryption/key changes. Show the target and consequences; never bundle destructive work into a routine update chain.
- Before risky maintenance, require evidence of an applicable recovery path: retained known-good closure/boot entry, access independent of the thing being changed, and restorable backups for affected persistent state. Unknown recovery readiness blocks risky activation, even if a build succeeds. Continue safe investigation and preparation instead.
- Local inspection does not authorise connecting to other hosts. Establish authorised targets; never scan or deploy to an inferred fleet.
- Use existing deployment, secret, backup, monitoring, and configuration tools. Prefer pinned flakes for new configurations; support existing channels without silently migrating them. No new daemon, MCP server, timer, binary cache, or dependency merely to use this skill.
- Do not expose secrets in chat, logs, derivations, source files copied into the store, or command arguments. Follow the user's authentication procedure before authentication. Secret refresh is not a side effect of a rebuild.
- Preserve application access and selected versions when changing package ownership. Follow the [application access checks](references/configuration.md#preserve-application-access) before and after activation.
- Preserve working credential paths and existing ownership. Follow the [credential path checks](references/configuration.md#preserve-credential-paths) before an approved ownership change.

## Establish facts, then ask what remains unknown

Use available files and read-only commands before asking. Batch only questions that change the action or its safety. Do not demand a host inventory for a conceptual question.

For operational work establish:

1. **Intent and scope:** advice, read-only assessment, edit/build, or approved activation; exact target and success criteria.
2. **Execution topology:** editor/controller, evaluator, builder, and activation target; OS, architecture, Nix and rebuild versions, SSH identity, privilege mechanism, and specialisation where relevant.
3. **Source of truth:** resolve `/etc/nixos` and symlinks; inspect the actual flake output or channel/configuration path, lockfile, imports, and Home Manager integration. Preserve unrelated edits and staging. Never assume hostname equals flake output.
4. **Three distinct states:** source checkout, running closure `/run/current-system`, and booted closure `/run/booted-system`; also inspect the system profile/boot default. They can differ. Reverting a file alone does not revert the live system.
5. **Recovery and exposure:** stateful services, data migrations, free space/inodes and EFI capacity, backups and restore evidence, recovery credentials, console access, acceptable downtime, and generation retention.

Examples for a NixOS target; select commands relevant to the task rather than running a blanket audit:

```bash
hostname
uname -m
nixos-version
nix --version
readlink -f /etc/nixos
readlink -f /run/current-system
readlink -f /run/booted-system
readlink -f /nix/var/nix/profiles/system
systemctl --failed --no-pager
```

Do not run target-only commands on a non-NixOS controller. Verify installed command flags in local help/man pages; classic and newer `nixos-rebuild` differ.

## Maintenance principles

1. **Declare durable state.** Change the owning NixOS/Home Manager module, not a generated file or store object. Imperative repairs can be necessary during an incident; record and reconcile them, rather than concealing drift.
2. **Pin and review inputs.** Preserve the lockfile. Prefer targeted updates and a supported release; read matching release notes. Do not update all inputs, switch channels, or add unstable merely to fix one package without justification.
3. **Validate against the target revision.** Use the pinned configuration's `options`, `config`, and package set. Manuals and search pages must match that revision as closely as possible. A remembered option or current wiki example is not evidence.
4. **Make the smallest coherent change.** Separate package updates, service migrations, kernel changes, network changes, and cleanup when practical. Keep `system.stateVersion` and `home.stateVersion` unchanged during ordinary upgrades.
5. **Separate build from activation.** Evaluate, build the exact target closure, inspect the change, obtain approval, then activate. `test` is live activation, `boot` changes the boot default, and `dry-activate` is only a partial preview. None proves health or bootability.
6. **Recover configuration and data separately.** Nix generations can recover retained system closures, not databases, home files, firmware, disk layouts, LUKS headers, or secret availability. Activation is not a transaction over all machine state.
7. **Verify the user's outcome.** A successful build is not proof that SSH, a service, a desktop session, a backup, or a new kernel works. Observe the affected function after approved activation; compare with the pre-change baseline. Reboot-dependent changes remain unverified until an approved reboot and checks.
8. **Retain recovery before reclaiming space.** Identify roots, profiles, known-good generations, and boot entries. Never manually delete `/nix/store` content or remove `/run/booted-system` to defeat retention.

## Response and completion

Lead with the decision or observed result. State relevant evidence, remaining risk, and the next concrete action. For a maintenance assessment distinguish confirmed faults, recommendations, and unknowns; do not report missing evidence as a verified failure.

Before activation present: exact target and candidate closure, meaningful configuration/package changes, expected restarts/downtime, recovery evidence, rollback versus data-restore plan, checks, and the precise action requiring approval. Stop there if approval or recovery evidence is missing.

After approved work report: what changed, which closure is running/selected for boot, exercised functional checks, unresolved failures, and whether reboot or restoration is still unverified. Do not claim an audit or recovery succeeded because commands were merely proposed.
