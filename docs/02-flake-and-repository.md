# 02 · Flake design and repository conventions

## Proposed implementation tree

```text
repo/
  flake.nix                 flake.lock
  hosts/{xps,thinkstation,arm-vm,vm-test}/
  modules/{core,desktop,developer,gaming,performance,hardware}/
  home/{common,programs,desktop,profiles}/
  installer/{iso,tui,executor,profiles}/
  desktop/{quickshell,hyprland,theme}/
  tools/kirictl/
  services/performance/
  contracts/                 tests/{eval,unit,integration,vm,hardware}/
  docs/adr/                  .github/workflows/
```

## Flake outputs required

- `nixosConfigurations.xps`, `.thinkstation`, `.arm-vm`, `.vm-test`.
- `packages.x86_64-linux.iso` and `packages.aarch64-linux.iso` (the actual Nix output derivations may be exposed by `nixosConfigurations.<installer>.config.system.build.isoImage`; a convenience `packages` output should point to them if practical).
- `packages.<system>.kirictl`, installer frontend, performance backend, shell distribution.
- `checks.<system>.*`: format, module evaluation, JSON schema, CLI unit tests, ISO build smoke where CI budget allows.
- `devShells.<system>.default` for agents and developers.

## Flake inputs

Pin `nixpkgs/nixos-26.05`, `home-manager/release-26.05`, `disko` to a tested revision (or a stable compatible branch), plus only strictly necessary integrations. Avoid dozens of unpinned community inputs. Include `home-manager.inputs.nixpkgs.follows = "nixpkgs";` as recommended, then use `home-manager.useGlobalPkgs = true` and `home-manager.useUserPackages = true` as applicable.

## Version policy

- `system.stateVersion` and `home.stateVersion` are **compatibility baselines**, not “current version” selectors. Do not automatically bump on NixOS upgrades.
- Keep distro version, desktop version, schema version and NixOS release separate.
- Lock upgrades must be reviewed and tested. Record flake input commit IDs with release metadata.

## Initial illustrative module

```nix
# modules/desktop/hyprland.nix (verify against pinned options)
{ ... }: {
  programs.hyprland = {
    enable = true;
    withUWSM = true;
    xwayland.enable = true;
  };
  # Home Manager Hyprland module must avoid duplicate systemd integration:
  home-manager.sharedModules = [{
    wayland.windowManager.hyprland.systemd.enable = false;
  }];
}
```

If `home-manager` is not yet imported at this point, attach the shared module only in the composition layer. This is an architectural example, not a tested file.

## Host composition rules

Each host imports:
1. `core/common.nix`;
2. exactly one hardware profile/auto-detection fallback;
3. desktop module (or VM-safe alternative);
4. developer profile if selected;
5. gaming only when x86_64 + graphics verified;
6. performance backend with host capabilities;
7. Home Manager for the initial user.

Prefer enabling module options over copying settings. Hardware configuration generated from an actual machine must be retained per host. Do not check machine-specific disk UUIDs or secret keys into generic installer templates.

## Nix build considerations

- Large-source tree copied into Nix store can invalidate builds; separate the shell source to its derivation where useful.
- Support `nix build .#iso` alias later, but distinguish `.#packages.x86_64-linux.iso` and `.aarch64-linux.iso` explicitly.
- Do not promise native x86_64 compilation on an ARM Mac without an x86_64 builder or emulation. Prefer a native x86_64 Linux runner and a native ARM64 Linux VM builder.
- Use binary caches first; cache custom artifacts only after signing and trust configuration.

## Definition of done

Evaluation for both architectures passes and x86_64 VM builds without accidentally pulling NVIDIA or Steam into ARM64. A failure on one host cannot prevent constructing unrelated host outputs.
