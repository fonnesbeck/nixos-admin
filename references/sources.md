# Sources, qualification, and upkeep

## Evidence order

1. The trusted target's pinned module/tool source, evaluated configuration, installed help/manual, and observed runtime behaviour.
2. Official NixOS, Nix, Nixpkgs, and relevant Home Manager/tool documentation matching that release or revision. A moving `stable` URL is not a permanent version pin.
3. The official NixOS Wiki for practical context and recipes, checked against the above.
4. Third-party skills and guides as leads, never as operational authority.

If sources conflict, identify the version/context difference and verify the specific option or command. Do not silence errors, invent an option, or weaken security to make a recipe work. Unknown behaviour calls for investigation, not an unverified fallback.

## References

Reviewed during initial authoring on 2026-10-28:

- [michalzubkowicz/nixos-management-skill](https://github.com/michalzubkowicz/nixos-management-skill), especially its [entry point](https://github.com/michalzubkowicz/nixos-management-skill/blob/main/nixos-managing/SKILL.md) and [deployment reference](https://github.com/michalzubkowicz/nixos-management-skill/blob/main/nixos-managing/vm-management.md). Inspiration for task routing and breadth. This skill is independently written guidance, not a vendored copy or installed plugin; upstream carries its own MIT licence.
- [Official NixOS Wiki](https://wiki.nixos.org/wiki/NixOS_Wiki), [nixos-rebuild](https://wiki.nixos.org/wiki/Nixos-rebuild), [Updating NixOS](https://wiki.nixos.org/wiki/Updating_NixOS), and [Storage optimization](https://wiki.nixos.org/wiki/Storage_optimization).
- [NixOS manual](https://nixos.org/manual/nixos/stable/), including [upgrades](https://nixos.org/manual/nixos/stable/#sec-upgrading). The fetched stable manual identified itself as 26.05 at authoring.
- [nix.dev best practices](https://nix.dev/guides/best-practices.html) for explicit dependencies, pinning, attribute-set semantics, and scope discipline.
- [Home Manager manual](https://nix-community.github.io/home-manager/) for integration and activation modes; select the matching release documentation/source during operational work.
- Installed NixOS 26.05 `nixos-rebuild --help`, identifying the local action semantics and current `--sudo`, `--ask-sudo-password`, and `--store-path` support. Do not infer these flags exist on every target.
- [OMP skill documentation at v18.13.3](https://github.com/can1357/oh-my-pi/blob/v18.13.3/docs/skills.md) and [Agent Skills specification](https://agentskills.io/specification) for discovery, frontmatter, relative references, and progressive disclosure.

Additional primary entrypoints for task-specific verification, not a claim that all their procedures have been exercised:

- [Nix manual](https://nix.dev/manual/nix/stable/) and [Nixpkgs manual](https://nixos.org/manual/nixpkgs/stable/).
- [deploy-rs](https://github.com/serokell/deploy-rs), [Colmena](https://github.com/zhaofengli/colmena), [disko](https://github.com/nix-community/disko), [nixos-anywhere](https://github.com/nix-community/nixos-anywhere), and [nixos-generators](https://github.com/nix-community/nixos-generators).
- [sops-nix](https://github.com/Mic92/sops-nix), [agenix](https://github.com/ryantm/agenix), and [impermanence](https://github.com/nix-community/impermanence). Consult the revision actually used by the target.

## Deliberate corrections to reference material

- `nixos-rebuild test` is live activation, not a safe dry run or timed rollback. Remote access can fail immediately.
- NixOS generations do not provide universal rollback of mutable state, service migrations, encryption, firmware, or disk changes.
- A registry-based option lookup does not establish the target's pinned options; `nixos-option` is not runtime introspection of the running system.
- `--fast` is not a portable “skip if unchanged” flag. Check the installed implementation rather than copying old workarounds.
- Root/user channel state and flake pins are distinct. Finding a user channel is not proof the system uses channels.
- Do not use broad generation deletion as routine cleanup or remove `/run/booted-system` to defeat a GC root. Some wiki recipes sacrifice recovery for space.
- Manual activation of a numbered generation is not proof that the system profile now selects it. Inspect running, profile, and boot state independently.

## Maintaining this skill

Maintain source files in the `nixos-admin` Git repository. Install the skill through a link from the native OMP skills directory to the checkout. The repository owns changes. Do not replace it with an unversioned global source copy.

Revisit command examples after a supported release/tool transition, a changed personal workflow, or a demonstrated error. Do not make bulk wording changes or automatically pull upstream instructions. Preserve user-approved authority and recovery requirements unless the user explicitly changes them.

Keep the core concise and the personal observations separate. Update source links and dates only for material actually reviewed. New host facts require fresh evidence. Keep this skill free of credentials, live tokens, private keys, and automatic installation hooks.

Verify native discovery with `omp read skill://nixos-admin`. Then read the core with `omp read skill://nixos-admin/SKILL.md`. Load relevant references through `skill://nixos-admin/references/<file>`. Use a fresh OMP session if the current session cached its skill list.

Exercise representative maintenance decisions with tools disabled or an explicitly read-only environment. No scenario test authorizes live administration.

Useful behavioural scenarios: a routine pinned update; an SSH/firewall change with no independent recovery; full EFI/store and a request for `-d`; a database upgrade/downgrade; a Home Manager ownership collision; and a channel-based host reached from a different architecture. Judge the decision, missing evidence, approval boundary, and verification plan, not exact wording. Syntax/discovery checks do not prove operational advice is safe.

Also check package-list consolidation without profile migration, and a requested system-to-user package migration. A passing build alone must not count as application access proof. Include a case where a launcher path disappears but another profile retains the application.

## Change history

- 1.0.3 — Preserve working credential paths during Home Manager changes. Explain collision backups and runtime target checks. Keep SSH private keys user-owned.
- 1.0.2 — Keep source in the project repository and link the global OMP installation to it. Add project installation instructions and an MIT license.
- 1.0.1 — Add application-access comparisons before and after ownership changes. Correct avalon's Home Manager output and inventory facts. Clarify SSH transport checks and rollback verification. Activation approval rules stay unchanged.
- 1.0.0 — Initial maintenance-first decision guide and runbooks: controlled updates, revision-specific option checks, approval before activation, recovery readiness, safe storage retention, supporting remote/provisioning workflows, and separately dated avalon observations. Markdown only; no executable automation.

Initial verification on 2026-10-28: OMP 18.13.3 resolved the core and all six reference URIs; relative links resolved and nine Bash blocks passed syntax checks. A fresh OMP print-mode run with tools disabled handled all six scenarios above without permitting unsafe activation. Pinned option lookups and target derivation evaluation succeeded on avalon, and the documented build expression passed a build dry run. No system build, activation, reboot, remote connection, disk cleanup, or data restore was performed. These checks validate discovery and sampled decisions, not every operational procedure.

Verification for 1.0.1 on 2026-10-28: native OMP reads resolved the core and all six references. All ten relative links resolved, including heading links. All ten shell blocks passed Bash syntax checks.

A fresh OMP run with tools disabled answered nine maintenance scenarios. Its answers preserved activation approval, recovery limits, application ownership, and separate application-access checks. Current flake outputs and evaluated Home Manager options matched the corrected personal reference.

This skill review made no NixOS configuration changes and ran no system activation. These checks do not prove future agent behavior, desktop access, boot recovery, or data restoration.

Verification for 1.0.2: Git reports all nine project files. The global skill link resolves to this checkout. Fresh OMP reads load the core and all six references.

The README installation command created the expected link under a temporary home directory. A repeated installation refused the existing target without replacing it. README and core links resolve, and the skill metadata names the MIT license.

Review for 1.0.3 on 2026-10-28: this review found no instruction that mandates credential replacement. The configuration reference lacked explicit credential path, collision backup, and runtime link safeguards. Activation approval rules stay unchanged.

Verification for 1.0.3: native OMP reads resolved the installed skill and configuration reference. All 21 relative file links resolved. All 12 Bash blocks passed syntax checks.

A fresh OMP session with tools disabled answered three credential ownership scenarios. It rejected migration to an absent target and kept exact restoration and activation approval. It accepted an explicitly selected, verified out-of-store workflow.

These checks cover guidance loading and sampled decisions, not live system recovery or future agent behavior.
