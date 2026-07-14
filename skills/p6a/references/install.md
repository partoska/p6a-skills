# p6a Install & Login Reference

Use this reference to install `p6a`, complete the one-time OAuth login, and troubleshoot setup. For the command reference and workflows, see `../SKILL.md`.

The CLI requires installing the binary and completing a one-time OAuth login. If the user's environment cannot install software or open a browser, the CLI path is not viable — tell the user the task isn't possible in this environment.

## Step 1: Install p6a

### Option A: npm (simplest — no platform-specific installer needed)

```bash
npm install -g @partoska/p6a
```

Or run without installing (useful for one-off use or environments where global installs are restricted):

```bash
npx @partoska/p6a login
npx @partoska/p6a <command>
```

Prefer this option when the user has Node.js available — it works on macOS, Linux, and Windows without choosing a platform-specific asset.

### Option B: Platform-specific binary from GitHub Releases

Download the latest release for the user's platform from GitHub:

**[https://github.com/partoska/p6a-cmd/releases](https://github.com/partoska/p6a-cmd/releases)**

If network access is available and the user wants exact commands, resolve the latest release via the GitHub Releases API instead of inventing versioned URLs:

```bash
curl -L https://api.github.com/repos/partoska/p6a-cmd/releases/latest
```

Select the installer from `assets[].name`, use its `browser_download_url`, and verify against its `digest` when possible. Do **not** hard-code a version or construct a release URL by guessing.

### Choose the right asset

| Platform | Preferred interactive install | Portable/headless fallback |
| --- | --- | --- |
| macOS | `p6a_<version>_darwin_universal.pkg` | `p6a_<version>_darwin_universal.tar.gz` or bare `p6a_<version>_darwin_universal` |
| Debian/Ubuntu amd64 | `p6a_<version>_linux_amd64.deb` | `p6a_<version>_linux_amd64.tar.gz` or bare `p6a_<version>_linux_amd64` |
| Debian/Ubuntu arm64 | `p6a_<version>_linux_arm64.deb` | `p6a_<version>_linux_arm64.tar.gz` or bare `p6a_<version>_linux_arm64` |
| Fedora/RHEL amd64 | `p6a_<version>_linux_amd64.rpm` | `p6a_<version>_linux_amd64.tar.gz` or bare `p6a_<version>_linux_amd64` |
| Fedora/RHEL arm64 | `p6a_<version>_linux_arm64.rpm` | `p6a_<version>_linux_arm64.tar.gz` or bare `p6a_<version>_linux_arm64` |
| Windows amd64 | `p6a_<version>_windows_amd64_setup.exe` | `.msi`, `.zip`, or bare `p6a_<version>_windows_amd64.exe` |
| Windows arm64 | `p6a_<version>_windows_arm64_setup.exe` | `.msi`, `.zip`, or bare `p6a_<version>_windows_arm64.exe` |

Use `uname -s` for OS and `uname -m` for CPU architecture. Map `x86_64` to `amd64`; map `aarch64` or `arm64` to `arm64`. On Linux, inspect `/etc/os-release` to choose `.deb` for Debian/Ubuntu-family systems and `.rpm` for Fedora/RHEL-family systems.

### Install examples

Window package:

```bat
p6a_<version>_windows_amd64_setup.exe
```

macOS package:

```bash
sudo installer -pkg ./p6a_<version>_darwin_universal.pkg -target /
```

Debian/Ubuntu:

```bash
sudo apt install ./p6a_<version>_linux_amd64.deb
```

Fedora/RHEL:

```bash
sudo dnf install ./p6a_<version>_linux_amd64.rpm
```

Portable Linux/macOS tarball:

```bash
tar -xzf ./p6a_<version>_<platform>.tar.gz
chmod +x ./p6a
mkdir -p ~/.local/bin
mv ./p6a ~/.local/bin/p6a
```

Portable bare binary:

```bash
chmod +x ./p6a_<version>_<platform>
mkdir -p ~/.local/bin
mv ./p6a_<version>_<platform> ~/.local/bin/p6a
```

Make sure `~/.local/bin` is on `PATH` when using portable installs.

### Verify downloaded assets

When the release API provides a `digest` such as `sha256:<hex>`, verify the downloaded file before installing:

```bash
shasum -a 256 ./p6a_<version>_<platform>.<ext>
```

Compare the output hash to the asset digest from the release API.

> **Windows note:** The shell examples in this reference use Bash syntax. On Windows, use PowerShell or `.bat` equivalents unless you are running inside MSYS2, Git Bash, Cygwin, or WSL — in those environments the Bash examples work as-is.

Verify the install:

```bash
p6a version
```

## Step 2: Log in

```bash
p6a login
```

This starts an OAuth device flow — follow the prompt to authorize in a browser. Credentials are saved to `~/.p6a/` and reused automatically. To use a custom config directory instead of `~/.p6a/`, pass `-D <path>` to any command (e.g. `p6a login -D ~/work/.p6a`).

Verify the connection:

```bash
p6a list
```

> If events are listed, the setup is complete. If `p6a` is not found, check that the binary is on `PATH`.

## Install Troubleshooting

- `p6a: command not found`: open a new terminal, verify the installer updated `PATH`, or use the full path to the binary.
- Portable install still not found: add the install directory (for example `~/.local/bin`) to `PATH`.
- macOS blocks the binary or package: prefer the `.pkg` installer; if macOS still blocks execution, use the Security & Privacy prompt in System Settings to allow it.
- Linux package install fails for permissions: run the package manager command with `sudo`, or use the portable tarball in a user-owned directory.
- Linux package format is wrong: use `.deb` for Debian/Ubuntu-family systems and `.rpm` for Fedora/RHEL-family systems.
- Windows terminal cannot find `p6a`: close and reopen the terminal after install; if needed, use the full install path or rerun the setup installer.
