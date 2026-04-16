# Installing amazon-ad for Codex

Use this repository in Codex through native skill discovery.

Codex only needs access to this repository's `skills/` directory. The `.claude-plugin` metadata and `commands/` directory are for Claude Code, not Codex.

## Prerequisites

- Git

## Installation

1. **Clone this repository**

   ```bash
   git clone https://github.com/zkp8023/amazon-ad.git ~/.codex/amazon-ad
   ```

   **Windows (PowerShell):**

   ```powershell
   git clone https://github.com/zkp8023/amazon-ad.git "$env:USERPROFILE\.codex\amazon-ad"
   ```

2. **Expose the skills directory to Codex**

   ```bash
   mkdir -p ~/.agents/skills
   ln -s ~/.codex/amazon-ad/skills ~/.agents/skills/amazon-ad
   ```

   **Windows (PowerShell):**

   ```powershell
   New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills"
   cmd /c mklink /J "$env:USERPROFILE\.agents\skills\amazon-ad" "$env:USERPROFILE\.codex\amazon-ad\skills"
   ```

3. **Restart Codex**

   Quit and relaunch Codex so it can discover the newly linked skills.

## Verify

macOS / Linux:

```bash
ls -la ~/.agents/skills/amazon-ad
```

Windows (PowerShell):

```powershell
Get-ChildItem "$env:USERPROFILE\.agents\skills\amazon-ad"
```

If installation succeeded, Codex should be able to discover the skills from this repository:

- `component-standard`
- `optimize`
- `vue`
- `vueuse`

## Updating

```bash
cd ~/.codex/amazon-ad && git pull
```

## Uninstalling

macOS / Linux:

```bash
rm ~/.agents/skills/amazon-ad
rm -rf ~/.codex/amazon-ad
```

Windows (PowerShell):

```powershell
Remove-Item "$env:USERPROFILE\.agents\skills\amazon-ad"
Remove-Item "$env:USERPROFILE\.codex\amazon-ad" -Recurse -Force
```
