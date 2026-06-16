# tunnel

Small macOS `zsh` CLI for managing SSH local port forwards created by this tool.

It uses SSH control sockets so tunnels can be checked, closed, reopened, and pruned reliably.

## Requirements

- `zsh`
- `ssh`
- `lsof` optional, used for local port diagnostics when available

## Install

Run directly from this repo:

```bash
./tunnel list
```

Or place it somewhere on your `PATH`:

```bash
chmod +x tunnel
cp tunnel /usr/local/bin/tunnel
```

## Usage

```bash
tunnel list
tunnel status
tunnel open HOST:PORT
tunnel open
tunnel reopen PORT
tunnel reopen all
tunnel reopen
tunnel close PORT
tunnel close all
tunnel close
tunnel prune
```

## Examples

Open a tunnel from local `localhost:5743` to `bee`'s `localhost:5743`:

```bash
tunnel open bee:5743
```

This runs roughly:

```bash
ssh -fN -M -S "$socket" -L 5743:localhost:5743 bee
```

List managed tunnels:

```bash
tunnel list
```

Check control socket and local port state:

```bash
tunnel status
```

Close a managed tunnel by local port:

```bash
tunnel close 5743
```

Close all live managed tunnels:

```bash
tunnel close all
```

Reopen a stale managed tunnel:

```bash
tunnel reopen 5743
```

Reopen all stale managed tunnels:

```bash
tunnel reopen all
```

Remove stale metadata and socket files without touching live tunnels:

```bash
tunnel prune
```

## Interactive Mode

Open interactively:

```bash
tunnel open
```

The command lists known SSH hosts from `~/.ssh/config`.
Enter either a printed host index or a full host name, then enter the port:

```text
SSH host:
Port:
```

Close interactively:

```bash
tunnel close
```

Reopen interactively:

```bash
tunnel reopen
```

## Runtime Files

Runtime files are stored in:

```bash
${XDG_RUNTIME_DIR:-/tmp}/tunnel-$USER
```

Each managed tunnel has:

```text
<host>-<port>.sock
<host>-<port>.meta
```

Metadata is shell-style key/value text:

```bash
ssh_host=bee
local_port=5743
remote_host=localhost
remote_port=5743
socket=/tmp/tunnel-flo/bee-5743.sock
created_at=2026-06-16T12:34:00Z
```

## Behavior

For MVP syntax, `HOST:PORT` means:

```text
localhost:PORT -> HOST:localhost:PORT
```

Before opening, `tunnel` refuses to continue if:

- Another managed live tunnel already uses the local port.
- Stale metadata exists for that local port.
- The local port is already listening and `lsof` is available.

`tunnel list` only shows tunnels created by this tool. It does not discover arbitrary external `ssh -L` processes.

`tunnel close all` closes live managed tunnels and removes their metadata, matching `tunnel close PORT`.

`tunnel reopen all` only reopens stale managed tunnels that still have metadata.

`tunnel prune` only removes stale metadata and socket files. It does not close live tunnels.

## Exit Codes

- `0`: success
- `1`: user or runtime error
- `2`: usage error
