# Backups, rollback, and recovery

## Establish recovery readiness before risk

Recovery evidence must match the affected state. A known-good package closure is not evidence that a database downgrade is safe. For the proposed maintenance, establish:

- A retained known-good closure and a usable boot entry where boot recovery is relevant; identify it explicitly rather than assuming “previous” means good.
- A working access path independent of the planned change: local console, authorised out-of-band console, or an established tested recovery mechanism. A second SSH session over the same network is useful but not independent recovery access.
- Recent backups of affected persistent data and configuration, with a credible restore method and sufficiently recent successful restore evidence. Record the location, age, coverage, consistency method, encryption/key availability, and acceptable data loss/downtime. Do not print keys or secret payloads.
- For stateful upgrades, documented schema/data compatibility and an application-consistent backup or snapshot. Coordinate database dumps, quiescing, or filesystem snapshots with the service's own requirements; copying live database files may be inconsistent.
- Enough capacity to build, activate, retain recovery material, and write boot files. A full EFI filesystem can prevent the expected recovery entry from being installed.

Unknown readiness blocks risky activation. Do not infer backups from Dropbox/synchronisation, filesystem snapshots on the same disk, a timer definition, or an old success log. Synchronisation can propagate deletion; local snapshots do not survive disk loss. No backup product is mandated: preserve and verify the existing workflow.

## Backup and restore maintenance

Include the configuration repository and lockfile, necessary persistent application state, irreplaceable user data, and the means to recover encrypted secrets and host identity. State which material must remain private. Keep a copy independent of the primary machine and its failure domain where the data's value warrants it.

Check backup job failures and age, repository integrity using its supported tools, and retention against recovery objectives. Do not start an expensive integrity check or prune a repository as an incidental read-only probe. Coordinate access and credentials under the user's authentication policy.

A useful restore exercise recovers representative data into an isolated location and checks content or application usability without overwriting production state. Full-system recovery additionally needs a way to install/boot NixOS, retrieve trusted sources and pinned inputs or retained closures, unlock storage, recover keys, and restore data. Backup creation, restore, and repository pruning require explicit scope/approval; backup configuration alone is not proof.

## Configuration rollback

First inspect what actually changed: running closure, booted closure, system profile/default, failed activation output, service state, and data migrations. Do not assume a failed rebuild left everything unchanged.

List system generations using the installed rebuild implementation or the profile tool:

```bash
nix-env --list-generations -p /nix/var/nix/profiles/system
```

Resolve the chosen generation and check that its closure still exists. A generation number, a Git commit, a store closure, and a boot entry are different things. Ordinary `nixos-rebuild switch --rollback` chooses the preceding system-profile generation, not necessarily the configuration that was running before a `test` activation.

For a precisely identified retained closure, modern rebuild implementations supporting `--store-path` can use an approved action against that exact closure. Otherwise use the installed version's documented rollback/generation-selection workflow. If directly invoking a generation's `switch-to-configuration`, understand that activation, profile selection, and future boot state are distinct: do not claim the profile moved unless verified. Do not mix methods blindly.

Before rollback, assess data-format compatibility. A previous service binary can fail or corrupt newer state. When a data migration is involved, follow the application's documented restore/downgrade procedure or choose a compatible forward fix. Never present Nix rollback as database rollback.

Use the existing Home Manager activation owner during rollback. Verify the affected user profile and repeat the [application access checks](configuration.md#preserve-application-access).

After an approved rollback verify the running closure, system profile/boot default, affected functionality, and logs. Reconcile configuration sources deliberately so the next rebuild does not silently redeploy the broken configuration. Preserve unrelated work; do not reset the repository or silently revert another contributor's edits.

## Boot failure

1. Use the existing boot menu to select the explicitly retained known-good entry, with the user's console access and approval. Boot-menu selection does not prove the configuration repository or persistent data was restored.
2. Once booted, inspect the actual booted/running/profile state and relevant previous-boot logs if available. Identify the failed kernel, initrd, root mount, encryption, hardware, or bootloader change before rebuilding.
3. If no entry works, use a trusted compatible rescue medium and the actual disk/encryption/RAID layout. Identify devices and mounts read-only first; do not assume `/dev/sda`, the EFI mount point, or filesystem type.
4. Prepare a case-specific rescue procedure for unlocking and mounting the intended root plus required boot/persistent filesystems; verify identity and mount state before writes. Use the installed manual's `nixos-enter`/installation recovery procedure as appropriate. Do not run a copied partition/format recipe on the installed system.
5. Obtain explicit approval for mounts that can replay journals, filesystem repairs, bootloader installation, restored files, or activation. A supposedly read-only assessment is not authority to modify disks.
6. After recovery, test the intended boot path through an approved reboot; report remaining uncertainty if no reboot was performed.

Do not overwrite `hardware-configuration.nix` during rescue without understanding the existing imports and hardware facts. Do not change `stateVersion`, reformat disks, or rotate storage keys to solve an unexplained boot failure.

## Disk, encryption, and impermanence incidents

For I/O errors, preserve evidence and minimise writes; do not begin scrubs, repairs, balance, resilver, or long self-tests without understanding the device and obtaining approval. Follow filesystem/device-specific documentation. Repair can be destructive; retain an appropriate backup/image where feasible and escalate confirmed hardware faults rather than masking them with retries.

For LUKS, verify the exact device, unlock method, key availability, and header backup procedure. Header backups and recovery material are sensitive. Never put passphrases or reusable unlock keys in a derivation, initrd copied to an unencrypted boot partition, command line, or transcript. Remote unlock depends on initrd networking and its independent security model; a running-system SSH test does not prove it works at boot.

With impermanence, distinguish intended ephemeral paths from persistent mounts and application state. Confirm that SSH host keys, secret identities, network bootstrap, machine identity as required by the setup, and backup state survive reboot. Wipe-on-boot logic is destructive and never a routine cleanup mechanism. Do not introduce impermanence or migrate root storage during ordinary maintenance.
