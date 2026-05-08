# VPM (Vim Package Manager)

**VPM** is a lightweight, zero-dependency Vim package manager written entirely in `tcsh`. It is specifically designed for managing plugins in secure, air-gapped, or legacy Linux workstations (common in EDA and IC design sectors). Since many professional environments use older versions of gVim (7.x) and lack internet access, VPM allows engineers to maintain a modular `pack/` structure and synchronizes it seamlessly with standard Vim runtime directories.

## Key Features

- **Zero Dependencies**: Requires only `tcsh`. No Python, Ruby, Node.js, or `git` needed.
- **Toggleable Plugins**: Instantly disable a plugin by prefixing its folder name with a dot (e.g., `.nerdtree`). VPM automatically skips hidden folders.
- **Scannable Logs**: Generates a clean, categorized list of all deployed files for easy auditing.
- **Surgical Uninstall**: Precisely removes specific packages and prunes empty parent directories.
- **Self-Protection**: The "Clean" mode includes a whitelist to ensure the `vpm` script never deletes itself.

## Installation

1. Place `vpm.csh` in your `~/.vim/` directory.
2. Grant execution permissions:
   ```bash
   chmod +x ~/.vim/vpm.csh
   ```
3. Add an alias to your `~/.cshrc` or `~/.tcshrc`:
   ```bash
   alias vpm '/disk/home/thomas/.vim/vpm.csh'
   ```

## Command Reference

| Command | Action |
| :--- | :--- |
| `vpm` | **Load**: Installs all pending packages from `pack/` to the `.vim/` root. |
| `vpm -s` | **Status**: Displays total, installed, and pending package counts on the console. |
| `vpm -a <pkg>` | **Add**: Manually installs a specific package from the `pack/` source. |
| `vpm -d <pkg>` | **Delete**: Uninstalls a specific package and removes its associated files. |
| `vpm -c` | **Clean**: Wipes all runtime folders to return to a pure state (Safe Mode). |
| `vpm --log` | **Log**: Shows the summarized status and the detailed file deployment list. |

## Directory Structure Example

VPM operates with the following structure:

```text
~/.vim/
├── vpm.csh           # The tool itself
├── pack/             # Source directory for your plugins
│   ├── tabular/      # Active plugin (will be installed)
│   ├── airline/      # Active plugin (will be installed)
│   └── .nerdtree/    # Hidden plugin (will be IGNORED)
├── plugin/           # (Managed by VPM)
├── autoload/         # (Managed by VPM)
└── doc/              # (Managed by VPM)
```

## Technical Details

Manifest Tracking: VPM maintains a hidden .vpm_manifest file. This tracks every file path copied to your root, ensuring that deletes and cleans are precise and never touch your manual configuration files.
Auto-Helptags: After every load or delete operation, VPM automatically triggers `:helptags` for the `doc/` directory, keeping your plugin documentation searchable via :`help`.

## License

MIT
