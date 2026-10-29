# Configuration, packages, and user environments

## Find the owner of the setting

Follow imports before editing. Reuse the existing host/module structure; introduce shared modules only for demonstrated reuse. Hardware configuration belongs to the actual host. Preserve generated hardware facts and do not replace them with an example from another machine.

Use NixOS options for system services, users, networking, boot, and system packages. Use Home Manager for its declared user files, programs, packages, and user services. Project dependencies belong in the project's existing environment, not automatically in either global package list.

Do not edit `/nix/store`, generated `/etc` files, or Home Manager symlink targets. Inspect ownership before replacing any existing file. A collision can protect valuable user state; do not suppress it with forced overwrite.

## Verify options against the actual configuration

The following examples assume a trusted local flake with an existing lockfile. Set `repo` and `host` from discovery; `host` is the flake output name. Use `--no-update-lock-file` so a lookup cannot silently resolve missing or changed pins. If it fails, investigate; a lockfile update is a separate deliberate change.

```bash
repo=/absolute/path/to/configuration
host=discovered-output-name
nix eval --no-update-lock-file --json \
  "$repo#nixosConfigurations.$host.options.services.openssh.enable.declarations"
nix eval --no-update-lock-file --json \
  "$repo#nixosConfigurations.$host.config.services.openssh.enable"
nix eval --no-update-lock-file --raw \
  "$repo#nixosConfigurations.$host.config.system.stateVersion"
```

Read the indicated module declarations, types, descriptions, defaults, assertions, and release migrations. Use an appropriate path for the actual option, not this example regardless of task. An unset optional value can fail to evaluate even when its declaration exists. `nixos-option` depends on its evaluation context; do not assume it interrogates the running closure or selects the repository's locked inputs.

For channels, inspect the configured root/system channel, `NIX_PATH`, and `nixos-config`. A user's `nix-channel --list` does not identify root's system inputs. Use the established configuration evaluator and version-matched manual. Do not mix an unrelated registry `nixpkgs` with the host's package set.

## Module and dependency discipline

- Keep related settings together. Prefer explicit `pkgs.<package>` references and small `let` bindings; avoid broad `with` scopes and unnecessary recursive attribute sets.
- Respect module merge semantics. Use `lib.mkDefault`, `lib.mkIf`, or ordering modifiers only where warranted. `mkForce` can conceal competing ownership; find the conflicting definition first. The `//` operator is a shallow attribute-set update, not a module merge.
- Pass intentional module inputs; avoid unexplained `--impure`, ambient environment variables, floating fetches, registry lookups, and unpinned imports in a pinned configuration. Retain necessary existing exceptions until their purpose is understood.
- Do not stage all files. Inspect new imports and use narrowly scoped staging only for intended, non-secret files when required: Git-backed flakes omit untracked files but include tracked working-tree modifications. A dirty checkout does not mean Nix built the committed revision.
- Follow the project's formatter command. If none is declared, use its existing formatter tooling when available rather than installing a new tool just to reformat unrelated files.
- Verify package names/versions in the target's package set. Prefer existing packages; a one-package issue does not justify upgrading the whole system or adding a second Nixpkgs input.
- Review overlays for scope and rebuild effects. Use the established `final`/`prev` convention, avoid self-referential overrides, and prefer a narrow package override over replacing the package set.
- `nix.settings.trusted-users` grants effectively root-equivalent Nix daemon power. Do not add users, trust caches, disable signature checks, or weaken sandboxing to work around a build/copy failure.

## Home Manager: one activation owner

Identify standalone versus NixOS-module use first. An exported `homeConfigurations` output does not prove the live host uses standalone activation.

For NixOS-module use, manage the user through the system configuration and approved system activation. Do not also run `home-manager switch` for the same managed user. Preserve the existing `useGlobalPkgs`, `useUserPackages`, and input-following conventions unless the task requires changing them. Keep the Home Manager release compatible with Nixpkgs.

For an established standalone configuration, build its `activationPackage` without activating:

```bash
user=discovered-output-name
nix build --no-update-lock-file --no-link \
  "$repo#homeConfigurations.$user.activationPackage"
```

Standalone switching or invoking an activation package is a live change requiring approval. Preserve `home.stateVersion`. A Home Manager rollback does not restore arbitrary user data or mutable application settings.

For executable/version drift, inspect `type -a <command>`, resolved executable paths, current profiles, shell environment, and project activations before adding packages. Use the appropriate read-only profile listing for the installed Nix version (`nix profile list` or legacy `nix-env -q`). Never mass-remove profiles to fix one shadowed command.

Verify shell behaviour in a fresh appropriate shell and user services in the actual user/session context. A system service becoming healthy does not prove the corresponding GUI or user service works; session variables and D-Bus matter.

## Preserve application access

A package in the Nix store does not prove that the user can start it. Changes between system and user profiles can change executable paths, desktop launchers, and session access without deleting the package.

Keep the existing installation owner unless the task needs a change. A single application inventory can supply both NixOS and Home Manager without moving applications between profiles.

Before changes to application ownership, package membership, or package sources:

1. Record the baseline application inventory, selected versions, executable paths, desktop entry IDs, and profile owners.
2. Compare the candidate with that baseline across system and user profiles. Include native program modules and dependent services.
3. Stop for unexplained removals, version regressions, or lost entry points. Preserve existing entry points unless the user approves their replacement.
4. For desktop applications, check launch commands, icons, default handlers, and favorites where relevant. Check the desktop session environment, not only a fresh terminal.

After approved activation, repeat the comparison against the active profiles. Exercise the affected command and desktop launcher in the existing user session. Preserve browser profiles, application settings, and open sessions. Do not delete user data, force an application restart, or create duplicate installations to hide a failure.

Report package membership, version, entry-point availability, session discovery, and actual launch as separate checks. A missing system path does not prove an application disappeared from every profile. A successful application query does not prove that the desktop shows the launcher. State any visual or runtime verification limit.

## Secrets and mutable configuration

Store references or encrypted material in configuration, not plaintext secret contents. `builtins.readFile` does not make a secret safe: evaluation can copy the value into a derivation or generated world-readable store file. Treat flake source snapshots as public to local users. Inspect files before staging or copying them into the store.

Preserve existing agenix, sops-nix, external password-manager, or runtime-file workflows. Follow the actual module's runtime path, owner, mode, timing, and restart requirements; `/run/secrets` is not universal. Check availability and metadata without reading secret contents into the transcript. Do not unlock a vault, rotate a key, refresh credentials, or access a password manager as an incidental build step.

Applications that rewrite settings need a deliberate mutable-state boundary: generated read-only configuration, seeded writable state, or a supported merge procedure. Do not point a self-updating application at an immutable file and then disable its errors.

## Development shells

Prefer a project's existing Pixi, virtual environment, Conda, Poetry, uv, or Nix development workflow rather than replacing it. A requested new Nix environment can use pinned `devShells` and `nix develop`; inspect `.envrc` before approving `direnv allow`, because it executes code. These are development-environment activations, not system rebuilds, but still must respect trust and project instructions.

Use a project environment for Python tooling; do not use system pip. Do not introduce `nix-ld`, FHS wrappers, unfree packages, or permissive loader paths as a blanket response to an executable failure. Establish interpreter/loader requirements and choose the smallest justified solution. Nix on another OS and nix-darwin are distinct from NixOS; use their own version-matched documentation when they are actually relevant.
