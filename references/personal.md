# Personal operating context

## Agreed preferences

- Primary purpose: best practices for maintaining NixOS, presented as a concise decision guide with task-specific runbooks.
- Reusable core; personal observations stay in this reference.
- Maintain the skill source in `/home/fonnesbeck/Repos/nixos-admin` under Git. Install it globally through `~/.omp/agent/skills/nixos-admin`, linked to this checkout. Do not maintain a separate global source copy. This skill needs no automatic scheduler, executable helper, or fleet framework.
- Preserve existing workflows; prefer pinned flakes for genuinely new configurations, without automatic migration of channel-based systems.
- Controlled updates; no automatic activation. In-scope edits/evaluation/builds may proceed; approval is required for live activation and the other actions specified in the core contract.
- Evidence of recovery readiness is required before risky maintenance. Missing evidence means prepare or investigate, not assume recoverability.
- Include baseline security, secrets hygiene, backup/restore, disk health, and service monitoring. Do not introduce a new security or monitoring platform by default.
- Cover Home Manager and development shells; keep packaging work proportional to administration needs.
- Preserve the existing secret toolchain rather than selecting agenix or sops-nix merely for consistency with an example.
- Discover locally first. Ask before connecting to remote hosts. No remote fleet was established during skill creation.
- Follow the user's global/project instructions for authentication, browser use, project environments, document storage, and protection of existing work. This reference does not replace those instructions.

## Observed on 2026-10-28; revalidate before use

These are historical observations, not a live inventory or a promise of health.

| Item | Observation |
| --- | --- |
| Local host | `avalon`, `x86_64-linux`; NixOS `26.05.20261026.718f670`; Nix `2.34.9`. |
| Configuration source | `/home/fonnesbeck/home-manager-config`. `/etc/nixos` resolved to that checkout. Do not edit the older `dotfiles-chezmoi` checkout by inference. |
| System output | `nixosConfigurations.avalon`; host module `hosts/avalon/default.nix`. |
| Inputs | `nixpkgs` on `nixos-26.05`; Home Manager on `release-26.05`, following `nixpkgs`; a separately fixed `toolchain` revision exists for selected applications. Preserve its purpose. |
| User integration | NixOS integrates Home Manager for `fonnesbeck`, with `useGlobalPkgs` and `useUserPackages`. The flake exports no standalone Home Manager configuration. Use system activation for this user. |
| Application inventory | Root `applications.nix` controls application membership. Both NixOS and Home Manager consume it. `pkgs/applications.nix` defines packages, not a second installation inventory. |
| Existing system application paths | Brave, Vesktop, Git, GitHub CLI, Wget, and uv use `environment.systemPackages`. Preserve their system executable paths and applicable desktop launchers. |
| Native application integrations | Firefox and 1Password GUI/CLI use NixOS modules. Obsidian uses Home Manager. Their native modules follow application inventory membership. |
| Application services | Collie and Dropbox services follow the inventory. Voxtype's daemon stays separately opt-in. |
| Compatibility defaults | `system.stateVersion` and `home.stateVersion` were `26.05`; not upgrade targets. |
| Boot/desktop/network declarations | systemd-boot, `linuxPackages_latest`, GNOME/GDM, NetworkManager, Tailscale. Verify the actual running kernel/session/network before drawing conclusions. |
| Secrets boundary (agreed repair) | SSH private keys stay user-owned at their working paths. Home Manager must not replace them with runtime links. Shell and marimo secrets retain runtime generation under `~/.local/state/dotfiles/secrets/current` and explicit refresh. Activation does not refresh credentials. Do not inspect secret contents or refresh them for routine verification. |
| Project tools | The flake declares a development shell including Nix formatting and Python test tooling, plus a checks output. Inspect the current project workflow before running it; this skill itself is Markdown only. |
| OMP | `18.13.3`; native global skill layout is `~/.omp/agent/skills/<name>/SKILL.md`. |

Unknown at creation: backup coverage/restore evidence, independent console recovery, boot-entry usability, retention policy, remote hosts, approved downtime, and health of services/disks. Do not mark those checks as satisfied from this file.

## Application management on avalon

Edit root `applications.nix` to add or remove an application. Apply the change through the existing system rebuild workflow. Do not run a separate Home Manager switch or add another package list.

Read `~/home-manager-config/README.md` for the current command. The package-management guide is `~/Documents/Agents/Projects/home-manager-config/Docs/2026-10-28-nixos-package-management.md`.

One inventory does not mean one installation profile or one configuration file. Preserve the existing host modules, Home Manager modules, package definitions, selected versions, and lockfile.

## Applying the profile

For an avalon maintenance request, revalidate the source and output, then load only the runbooks needed for that request. Do not run a full audit when the user asks a conceptual question or a small configuration change.

For another host, discover its actual setup. Do not substitute avalon's user, architecture, bootloader, paths, kernel choice, or Home Manager activation model.
