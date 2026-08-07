# Wender CLI

Send files and folders between devices on your local network, from the command line.

Every download bundles its own Java runtime — there is nothing to install first.

> This repository distributes the built CLI. The application source lives elsewhere and is not public.

## Install

### macOS (Apple Silicon)

```bash
brew install bartwell/tap/wender
```

Or download `wender-<version>.dmg` from [Releases](https://github.com/bartwell/wender-cli/releases).

### Windows

```powershell
scoop bucket add bartwell https://github.com/bartwell/wender-cli
scoop install wender
```

Scoop is the smoother route: it strips the "downloaded from the internet" mark, so Windows does not
warn about the program.

The `.msi` and the `.zip` from [Releases](https://github.com/bartwell/wender-cli/releases) work too,
but they are not code-signed, so SmartScreen shows "Windows protected your PC". Choose **More info →
Run anyway**, or unblock the file first:

```powershell
Unblock-File .\wender-<version>.msi
```

Extracting the zip in Explorer does not avoid this: the mark is copied onto the extracted files.

### Debian, Ubuntu, Raspberry Pi OS (64-bit)

```bash
curl -fsSL https://bartwell.github.io/wender-cli/apt/wender.gpg \
  | sudo tee /etc/apt/keyrings/wender.gpg > /dev/null

sudo tee /etc/apt/sources.list.d/wender.sources > /dev/null <<'EOF'
Types: deb
URIs: https://bartwell.github.io/wender-cli/apt
Suites: ./
Signed-By: /etc/apt/keyrings/wender.gpg
EOF

sudo apt update && sudo apt install wender
```

Covers `amd64` and `arm64`, so 64-bit Raspberry Pi OS is included, and updates arrive with
`apt upgrade` like any other package. The repository carries the current release only; older
versions stay in [Releases](https://github.com/bartwell/wender-cli/releases) and install with
`sudo dpkg -i wender_<version>_<arch>.deb`.

32-bit Raspberry Pi OS (armhf) is not supported. 64-bit has been the default since Bookworm.

### Portable archive

Every platform also ships a plain archive, for machines where you would rather not install anything:

```bash
tar xzf wender-<version>-linux-x64.tar.gz
./wender-<version>/wender/bin/wender --help
```

On macOS the archive contains an app bundle, so the launcher is at
`wender-<version>/wender.app/Contents/MacOS/wender`.

### Verifying a download

Each release ships `SHA256SUMS`:

```bash
sha256sum -c SHA256SUMS --ignore-missing
```

## Usage

```
wender send --to <ip> [--name <name>] <path>...   Send files or directories to a peer
wender receive [--out <dir>] [--name <name>]      Wait for an incoming transfer
wender discover [--timeout <seconds>]             List Wender devices on this network
```

Global options: `--verbose` prints library logs to stderr, plus `--help` and `--version`.

### Example

On the receiving machine:

```bash
wender receive --out ~/Downloads
```

It prints the port it is listening on and waits. On the sending machine:

```bash
wender discover              # find the other device's address
wender send --to 192.168.0.42 report.pdf photos/
```

Directories are sent whole, and progress is shown while the transfer runs. Both commands exit
non-zero if the transfer fails, so they compose in scripts.

### Networking

Transfers use TCP port 2904, and discovery uses multicast on the same port. Two things commonly get
in the way:

- **A VPN.** Multicast leaves through the tunnel interface instead of the LAN one, so `discover`
  silently finds nothing. Sending still works if you pass the address yourself with `--to`.
- **A firewall.** On Windows, allow `wender.exe` on private networks.

Guest and "client isolation" Wi-Fi networks block device-to-device traffic entirely; neither
discovery nor transfer will work there.

### Where data is kept

Transfer history goes to the per-user data directory: `~/.local/share/wender` on Linux (honouring
`XDG_DATA_HOME`), `~/Library/Application Support/Wender` on macOS, `%APPDATA%\Wender` on Windows.
Set `WENDER_DATA_DIR` to put it somewhere else.
