# tunnel

Small macOS `zsh` CLI for managing SSH local port forwards created by this tool.

It uses SSH control sockets so tunnels can be checked, closed, reopened, and pruned reliably.

## Requirements

- `zsh`
- `ssh`
- `lsof` optional, used for local port diagnostics when available
- `ss`, `lsof`, or `netstat` on the SSH host for automatic port discovery

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
tunnel open HOST
tunnel open PORT
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

Discover listening ports on `bee` and choose one:

```bash
tunnel open bee
```

Choose a known SSH host for port `5743`:

```bash
tunnel open 5743
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

With no argument, the command shows known SSH hosts from `~/.ssh/config`.
Use Up/Down and Enter to choose a host, or select `Enter another host...` to type
a full host name. The command then discovers the host's listening TCP ports and
shows the same arrow-key picker:

```text
Listening TCP ports on bee:
Open which port?
    22
  > 5743
    8080

Use Up/Down and Enter (Esc to cancel)
```

`tunnel open HOST` skips the host prompt and shows the same port picker.
`tunnel open PORT` skips port discovery and asks which SSH host to use.
`tunnel close` and `tunnel reopen` use the same arrow-key picker.

When input or output is not attached to a terminal, interactive commands retain
the numbered prompts so selections can be piped into the command.

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

Automatic discovery lists TCP listeners bound to a loopback or wildcard address,
because those listeners are reachable through the same remote `localhost:PORT`
target. Address-specific and UDP listeners are not shown.

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
