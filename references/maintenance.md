# Routine maintenance and controlled change

## Read-only maintenance assessment

Choose probes based on the request; do not run every expensive check by default. Establish a baseline before repairing anything. Commands here run on the target and may need read privileges; do not escalate or authenticate without following the user's procedure.

| Concern | Evidence | Interpretation and next decision |
| --- | --- | --- |
| Service health | `systemctl --failed --no-pager`; affected unit's status and bounded journal | Distinguish historical failure from a current fault; inspect cause before restart or reset-failed. |
| User services | `systemctl --user --failed --no-pager` in the relevant user's session | A missing user bus is not proof that all user services failed. |
| Logs | `journalctl -b -p err --no-pager -n 100`; `journalctl -b -u <unit> --no-pager -n 100` | Correlate timestamps and changes; redact sensitive contents. Do not dump the full journal or environment. |
| Capacity | `df -h / /nix/store /boot`; `df -i / /nix/store /boot`; relevant `findmnt` | Confirm actual EFI mount and shared filesystems. Check inode exhaustion as well as bytes. |
| Boot state | Resolve current, booted, and system-profile links; list generations | Different paths require investigation, not an automatic reboot. |
| Network | Relevant interface/routes, resolver status, and a scoped DNS/application probe | Existing SSH can survive a change that blocks new connections. Do not weaken access controls or website access. |
| Storage health | Kernel I/O errors; existing SMART/NVMe/RAID/Btrfs/ZFS tooling appropriate to detected devices | Do not run filesystem-specific commands on an assumed layout. Scrubs/self-tests may be intensive; do not start them during a read-only assessment. |
| Backups | Existing backup job status, repository accessibility, recent successful backup, restore evidence | A configured timer, synced folder, or successful process exit alone does not prove restorability. |
| Update exposure | Actual locked/channel revision, support status, relevant advisories and pending service/kernel changes | Stable branch names do not mean a host is patched. Check current support from release sources, not this file's age. |

Report faults with their evidence, separate recommendations, and explicitly name unknown backup/recovery or patch status. Monitoring should cover actionable failures, capacity/inodes/EFI, backup age and failures, disk errors, and affected services using existing tools. Do not install a stack or notification webhook by default, or transmit logs to a new destination without approval.

## Updating within a release

1. Inspect source ownership and unrelated edits. Establish recovery evidence and service-specific risks. Do not combine cleanup with the upgrade.
2. Read relevant changelogs/advisories; decide the scope and maintenance window. Establish a cadence appropriate to each host and urgent fixes, without automatic activation.
3. For flakes, intentionally update only the needed inputs. First inspect `nix flake update --help`; modern Nix supports the following. Preserve intentionally fixed secondary snapshots and shared-input relationships.

   ```bash
   nix flake update --flake "$repo" nixpkgs
   ```

   Review the lockfile's old/new revisions and dependency changes. Updating an input can affect its transitive inputs. Do not hand-edit lock hashes, discard the lockfile, or use an all-input update merely to hide a mismatch.
4. For channels, establish the system/root channel, retain its previous revision for reproducibility, then obtain approval for changing the live channel state. Update only the intended channel, then build separately. Root channel updates are not flake updates; `--upgrade` is not a flake refresh.
5. Evaluate and build the exact target, inspect its closure and proposed activation, then follow the approval/check procedure below.

A pinned version is reproducible, not automatically secure. Check whether a service needs restart or the kernel needs reboot to run patched code. Do not report a vulnerability fixed solely because the new package exists in the store.

## Release upgrades

Check the release's current support status and version-specific manual/release notes. Identify removed/renamed options, changed defaults, service/database migrations, boot changes, hardware compatibility, and a compatible Home Manager branch. Prefer a supported upgrade path; do not assume skipping releases is harmless.

Change the intended Nixpkgs/Home Manager branches and update their pins deliberately. Preserve `system.stateVersion` and `home.stateVersion`: they select compatibility defaults, not the desired package release. Change them only as a separate, understood migration with state-specific backups and approval. Do not migrate storage layouts or switch kernels as incidental cleanup.

## Evaluate, build, preview

For a trusted local locked flake, with discovered `repo` and output `host`:

```bash
nix eval --no-update-lock-file --raw \
  "$repo#nixosConfigurations.$host.config.system.build.toplevel.drvPath"
nix build --no-update-lock-file --no-link --print-out-paths \
  "$repo#nixosConfigurations.$host.config.system.build.toplevel"
```

Retain the exact output path returned by the build as `candidate`; validate that it is the intended NixOS closure. A no-link build does not create a persistent GC root: keep it rooted with the established workflow if it must survive garbage collection before activation. Compare against the target's running closure, not the controller's when deploying remotely.

```bash
nix store diff-closures /run/current-system "$candidate"
```

Use project checks where relevant, but `nix flake check` alone does not necessarily build each NixOS configuration or exercise activation. VM tests can test selected behaviour, not real hardware, EFI capacity, or data migrations. A passing build does not prove bootability.

Consult installed rebuild help. On implementations supporting `--store-path`, this previews the already-built closure without rebuilding another revision:

```bash
nixos-rebuild dry-activate --store-path "$candidate"
```

Treat dry activation as a partial preview; inspect custom activation scripts first and do not execute untrusted scripts just because the action is called dry. Privileges may be required for a meaningful preview. Without this flag, use the installed tool's documented flake/configuration mode with fixed inputs and revalidate the resulting closure. Do not copy unsupported flags from another release.

## Activation: approval and functional proof

| Action | Meaning | Required caution |
| --- | --- | --- |
| `build` | Build only | Resource use and untrusted build code; no service-health proof. |
| `dry-activate` | Build/preview unless using an existing closure | Incomplete prediction, not a real activation test. |
| `test` | Activate now without making it boot default | Can stop services, migrate data, and break SSH; no automatic rollback timer. |
| `switch` | Activate now and make boot default | Live and boot-state change; partial failure is possible. |
| `boot` | Make boot default without normal live activation | Not proof of successful boot; reboot needs separate approval. |

Before any approved activation, record the target's running and boot-default closures and recovery route. Select `test`, `switch`, or `boot` for the actual risk; do not mechanically run all three. A separate test activation can help, but should not repeat an irreversible migration or be mistaken for a network safety mechanism.

For package ownership changes, follow the [application access checks](configuration.md#preserve-application-access). A closure comparison alone can miss lost executable paths and desktop launchers.

For installed implementations with `--store-path`, an approved local action can be `sudo nixos-rebuild test --store-path "$candidate"` or `sudo nixos-rebuild switch --store-path "$candidate"`. Do not run these examples without approval and recovery evidence. Confirm candidate identity immediately before use; if source or candidate changes, revalidate the approval's scope.

After activation:

- Check actual running/profile paths and the command's exit status. A failed exit can leave partially applied state; inspect before retrying.
- Compare failed units and new relevant logs to the baseline. Test the actual service endpoint or user workflow, not just `systemctl is-active`.
- For remote changes, establish a fresh SSH transport as described in the [remote runbook](remote-provisioning.md#remote-activation-safety). Do not count that transport as independent recovery access. Verify DNS, routing, and application reachability. Keep the original session open.
- For desktop changes verify the affected program/session; systemd success alone does not prove GUI integration. Do not log the user out or restart their session without approval.
- For kernel/initrd/bootloader or mount changes, report boot verification pending until an approved reboot and post-boot checks. Never chain reboot after switch.
- If unhealthy, follow the recovery runbook and obtain approval for the precise rollback/restore action. Do not retry activation until its failure is understood.

## Storage pressure and garbage collection

First distinguish store growth, `/boot`/EFI growth, logs, user data, snapshots, and inode exhaustion. Preserve evidence before purging logs. Check generation profiles, retained known-good closures and their boot entries, backups, and GC roots:

```bash
nix-env --list-generations -p /nix/var/nix/profiles/system
nix-store --gc --print-roots
nix-store --gc --print-dead
```

These inspect; `--print-dead` can be slow on large stores and is not an exact promise of reclaimable bytes. Root-owned metadata may require authorised privileges. Avoid disclosing private root paths unnecessarily.

Agree on retention by host, age, known-good state, and recovery needs, not an arbitrary fixed count. Identify exact profiles/generations before seeking deletion approval. System, root, standalone Home Manager, and other users' profiles are separate; do not sweep them all. Keep the active, booted, selected-next-boot, and validated recovery paths safe. A result symlink can be a deliberate GC root; do not delete it merely because it exists.

Only after approved targeted pruning, collect unreachable store paths through Nix. Do not use `nix-collect-garbage -d` as a default: it deletes non-current generations in its scope and can erase needed rollback history. Age-based pruning alone does not guarantee a known-good generation survives. Do not manually remove store paths, `/run/booted-system`, or bootloader files to force space reclamation.

Bootloader entry limits do not themselves reclaim the entire store or guarantee recovery retention. Update boot entries using the supported NixOS mechanism after approved pruning; check EFI capacity before doing so. Store optimisation deduplicates content; it is not generation pruning or data backup and can be I/O-intensive. Propose scheduling GC, optimisation, log retention, or filesystem maintenance only as a separate justified, approved policy change.
