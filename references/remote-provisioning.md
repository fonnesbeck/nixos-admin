# Remote operations and supporting provisioning

These support system maintenance; they are not a reason to adopt a fleet framework, repartition an existing host, or redesign working infrastructure.

## Separate controller, builder, and target

Record which machine evaluates the configuration, which builds Linux derivations, and which activates. Discover authorised SSH aliases, architectures, release/tool versions, and privilege mechanisms; inspect SSH configuration without reading private keys. Verify host keys through the established trust workflow. Never disable host-key checking, silently accept a changed host key, forward agents by default, or connect to inferred hosts.

A non-NixOS Linux or macOS controller can evaluate configurations with Nix, but it cannot necessarily build the target's Linux closure locally. Use an existing appropriate Linux builder, the target, documented cross-compilation, or emulation only when configured and justified. Verify that the particular rebuild implementation supports the controller platform and flags. Do not copy `--fast`, `--use-remote-sudo`, or `--sudo` across implementations without checking help.

`--target-host` selects where activation happens; `--build-host` selects where building happens. One does not imply the other. Omitting a target can activate the controller; resolve the exact command and target before approval. Do not install Nix or deploy software to a remote machine merely to inspect it.

For an established modern rebuild workflow, a non-activating remote build may take this shape, after checking installed help and permission for the named remote build:

```bash
nixos-rebuild build --flake "$repo#$host" \
  --no-update-lock-file --build-host "$builder"
```

Retain and identify the actual output closure; do not assume a stale `result` symlink came from this invocation. A remote build may copy data and consume resources; a build approval is not activation approval. Use the existing authenticated/signed closure-transfer method; do not grant root-equivalent Nix trust to work around copy failures.

## Remote activation safety

Before activation, follow the core approval contract plus:

1. Verify exact host identity, current and boot-default closures, architecture, target privileges, disk/EFI capacity, and candidate closure/input identity.
2. Check independent recovery access and backups appropriate to the change. Network, firewall, DNS, SSH, sudo, users, Tailscale, initrd, and disk changes can remove the only access path. Keep the original session open, but do not count it as independent recovery.
3. Use the existing deployment tool's documented rollback/confirmation mechanism if configured; inspect its actual settings and limits. `nixos-rebuild test` alone has no timed rollback and can disconnect SSH immediately.
4. For connectivity-risk changes, require a verified out-of-band path or an established, tested rollback mechanism that remains effective if connectivity fails. Do not invent an untested background sleep-and-switch shell job as the safety net. Do not proceed merely because the user can reconnect now.
5. Present the exact target-specific activation action, recovery route, expected outage, and post-change checks for approval. Never use an all-host selector for a single-host request.
6. After approved activation, open a fresh SSH connection. An existing multiplexed transport is not sufficient: use a new transport (for standard OpenSSH, `-o ControlMaster=no -o ControlPath=none`) and verify the target identity again. Check application health from the relevant client/network, failed units, and new logs.
7. Complete any deployment confirmation only after the checks required by that tool. Report a timed rollback distinctly from a successful deployment. A tool's activation acknowledgement is not proof of data integrity or application health.

For fleet changes, discover dependencies and choose a canary before a scoped staged rollout. Do not deploy all hosts in parallel by default. Stop expansion on a failed canary; preserve service quorum, capacity, and recovery access. deploy-rs and Colmena are options when already used or justified, not mandatory fleet-size thresholds. Consult their pinned documentation before writing configuration.

## Provisioning and installation

Provisioning is a separate authorised task with explicit targets and data-loss decisions. Use the version-matched NixOS manual and existing installation tooling; standard installation is a valid default. If the infrastructure already uses disko or nixos-anywhere, inspect their pinned configuration and documentation rather than adding a competing method.

Before any destructive command:

- Identify the exact physical/virtual target, device paths, stable IDs, serials, size, partition table, mounts, holders, RAID/LVM/encryption layers, and data ownership. Use read-only `lsblk`/`findmnt` or appropriate existing tools; never assume the first disk is expendable.
- Resolve BIOS/UEFI, Secure Boot, filesystem, swap/hibernation, encryption/unlock/recovery, persistent state, network bootstrap, SSH trust, and console access from hardware facts and user requirements.
- Obtain explicit approval for the exact disks and erased data, after relevant backups and recovery keys are verified. Do not provide an executable destructive one-liner with a guessed device.
- Preserve or generate hardware configuration for this host in an appropriate context, review it, and include it in the actual configuration. Decide initial `stateVersion` for the fresh install deliberately; do not copy another host's historical value as an upgrade instruction.
- Build/evaluate beforehand where possible. Installation, kexec, formatting, encryption, and bootloader writes require their own understood scope; success cannot be inferred from the flake building.
- Verify boot, mounts, unlock, fresh SSH, intended services, backups, and retrieval of trusted configuration after the install. A remote installer exiting successfully is not proof of post-reboot reachability.

For nixos-anywhere, determine current requirements and what its chosen mode will erase or replace. Do not assume a fixed minimum memory, architecture support, kexec availability, or that every target is a fresh NixOS installation. The user's existing Linux system can contain irreplaceable data.

## Images and VMs

Discover the existing image tooling and target format. Inspect the pinned configuration's image outputs and installed `nixos-rebuild build-image`/VM support, or the existing pinned nixos-generators integration. Do not install a generator merely because a wiki example uses it.

Build separately from flashing or launching. Check architecture, firmware, output format, disk size, credentials, secret exposure, host mounts, networking, and port forwards. A VM launch can write disk images, bind ports, and access shared directories: explain the scope and follow applicable approval requirements. Never embed production private keys/passwords in distributable images.

For a requested boot test, launch the actual image in an appropriate isolated VM, verify console/boot and the intended service, then stop only owned processes and remove only approved throwaway artifacts. VM success does not prove host hardware, LUKS remote unlock, Secure Boot, or real-disk bootloader behaviour. Flashing an image is a destructive disk write requiring exact-device approval.

Load the recovery runbook for LUKS, initrd access, or impermanence. These need case-specific, version-checked procedures and persistent-state review, not generic copy-and-paste configurations.
