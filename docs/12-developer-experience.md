# 12 · Development compatibility: VS Code, Node, NVM, MCP, native tools

## Requirement

“Install Node.js, launch VS Code from the desktop, run an MCP server, and have Node/npm usable without shell gymnastics.” Keep NixOS's reproducibility without forcing every experimental CLI to be packaged as a Nix derivation.

## Runtime layers

| Layer | Use case | Example |
|---|---|---|
| Stable system runtime | Default Node/Python/compiler, GUI-launched tools | `environment.systemPackages = [ pkgs.nodejs ... ];` |
| Project runtime | Reproducible SDK versions | `nix develop`, `direnv` |
| Traditional tool runtime | NVM/npm global, downloaded binaries | `nix-ld`, supported runtimes, possibly `steam-run`/FHS env |
| Escape hatch | FHS-assuming arbitrary tools | Distrobox Ubuntu/Arch with VS Code/MCP explicitly launched within it |

## Global Node visibility

Install a stable Nix-managed Node. A path like `/run/current-system/sw/bin/node` is a reliable *system-profile executable path* when installed system-wide. `PATH` in GUI-launched VS Code may differ from terminal interactive shell; test via a GUI-launched process, not only `which node` in terminal.

An MCP stdio config should accept an **explicit runtime path** or a generated wrapper ensuring stable PATH and required env; this prevents hidden dependencies on `.bashrc` or `nvm use` in another shell.

Example conceptual MCP configuration for a VS Code workspace:

```json
{
  "servers": {
    "local": {
      "type": "stdio",
      "command": "/run/current-system/sw/bin/node",
      "args": ["${workspaceFolder}/mcp/server.js"]
    }
  }
}
```

**Verify exact VS Code MCP schema** against the current installed VS Code version before implementing presets.

## NVM support

- `nvm` initialization is per shell and not magically propagated into already-running IDE/MCP processes.
- Node binaries fetched outside Nix may require `nix-ld` and additional libraries. This is expected, not an invitation to rewrite global store paths.
- Provide `kirictl doctor node`: system Node version, shell Node version, GUI process PATH, NVM found, project runtime, MCP child spawn test and dynamic linker diagnostics.
- Avoid automatic sourcing of interactive shell RC files from privileged daemons.

## nix-ld

Use native NixOS `programs.nix-ld.enable = true` in Developer profile. Keep `programs.nix-ld.libraries` intentionally minimal, log errors clearly and add missing dependencies with tests. `nix-ld` increases compatibility but does not make all Linux binaries portable, especially complicated proprietary launchers.

## VS Code

- Package and extensions via documented supported route; unfree licensing if applicable.
- Test desktop launcher, integrated terminal, extension-provided binaries, Node debugger, git, language servers, terminal PATH, MCP stdio launch.
- Avoid multiple competing versions of VS Code from Nix and Flatpak unless explicitly configured; Flatpak sandbox introduces extra discovery complications.

## Distrobox fallback

- NixOS enables Podman and Distrobox under developer profile.
- In a container, Pacman/AUR workflows are container-scoped, not native host installations. `distrobox-export` bridges launchers/commands but not every service/runtime dependency.
- For problematic MCP tools, run VS Code **inside the same container** or use an explicit `distrobox enter ... -- command` MCP wrapper and test IPC/filesystem paths.

## Test suite: developer confidence

1. `node --version` + `npm --version` in terminal.
2. GUI-launched VS Code's integrated terminal sees intended Node.
3. A tiny Node stdio MCP smoke server can be spawned by GUI VS Code and emits expected JSON-RPC initialization flow (agent must implement test fixture).
4. NVM-installed version executes, switches within shell and does not break global default outside shell.
5. Rust Cargo, Python venv with native package, C++ compiler + debugger, Podman/Distrobox smoke.
6. External binary linked against glibc exhibits clear diagnostics and either runs under `nix-ld` or is explicitly documented unsupported.

References: https://wiki.nixos.org/wiki/Nix-ld ; https://wiki.nixos.org/wiki/Hyprland ; https://code.visualstudio.com/docs/copilot/customization/mcp-servers
