# modal-deploy-ssh

A portable [Agent Skill](https://agentskills.io/specification) that creates
named Modal cloud sessions you can reach with plain SSH and VS Code
Remote-SSH — no browser terminal and no `code tunnel` login required.

It packages the full lifecycle into three commands:

```sh
python3 scripts/modal_ssh_session.py launch --name g-data-linkup
python3 scripts/modal_ssh_session.py status --name g-data-linkup
python3 scripts/modal_ssh_session.py stop   --name g-data-linkup
```

`launch` starts a detached Modal sandbox, waits for the raw-TCP SSH relay,
writes or replaces a managed `Host` block in `~/.ssh/config`, and verifies a
real login. Because the config block is rewritten on every launch, a respawn
with the same name keeps the same SSH alias — the endpoint changes, your
commands do not.

## What you get

- CPU-only or GPU sessions (L4, A10, L40S, A100, H100, H200, B200, multi-GPU)
- Direct SSH with your existing key; password auth disabled
- A persistent Modal volume mounted at `/workspace` (default `<name>-ws`)
- Automatic `~/.ssh/config` management with clear begin/end markers
- A verified connection before the command returns
- Optional one-shot provisioning script (`--provision`)

## Requirements

- Python 3.9+ (the manager is stdlib-only)
- An authenticated Modal CLI: `modal profile list` lists a profile
- An SSH keypair, by default `~/.ssh/id_ed25519`
- macOS or Linux with network access to `modal.com`

The remote container runs Debian 12 (bookworm). Provisioning scripts should
support Debian/Ubuntu.

## Install

The skill follows the [Agent Skills specification](https://agentskills.io/specification),
so any agent that discovers `SKILL.md` files can use it.

### Pi

From a checkout of this monorepo:

```sh
pi install ./packages/modal-deploy-ssh
```

Or install the whole monorepo:

```sh
pi install git:github.com/rajivmehtaflex/pi-power-utility@main
```

### Claude Code

```sh
cp -R skills/modal-deploy-ssh ~/.claude/skills/modal-deploy-ssh
```

### Codex

```sh
cp -R skills/modal-deploy-ssh ~/.codex/skills/modal-deploy-ssh
```

### ZCode

```sh
cp -R skills/modal-deploy-ssh ~/.zcode/skills/modal-deploy-ssh
```

### Generic agents

Copy the skill directory where the agent looks for skills, commonly
`~/.agents/skills/` or `<project>/.agents/skills/`:

```sh
cp -R skills/modal-deploy-ssh ~/.agents/skills/modal-deploy-ssh
```

The important part is that `SKILL.md` sits at the root of the copied folder
and `scripts/` stays beside it.

## Quick start

```sh
# 1. Create a CPU-only session (8 CPU, 32 GB, 24 h, volume <name>-ws)
python3 scripts/modal_ssh_session.py launch --name g-data-linkup

# 2. Connect from a terminal
ssh g-data-linkup

# 3. Or connect VS Code:
#    Remote-SSH: Connect to Host -> g-data-linkup
#    The workspace root is /workspace (persistent volume)

# 4. Stop when done
python3 scripts/modal_ssh_session.py stop --name g-data-linkup
```

## Command reference

### `launch`

| Flag | Default | Description |
|---|---|---|
| `--name` | required | Session/app name and SSH alias |
| `--cpu` | `8` | CPU cores (float allowed) |
| `--memory` | `32768` | Memory in MiB |
| `--gpu` | none | GPU spec: `L4`, `A10`, `H100`, `H100:2`, ... |
| `--timeout` | `86400` | Sandbox lifetime in seconds |
| `--volume` | `<name>-ws` | Persistent volume name |
| `--pubkey` | `~/.ssh/id_ed25519.pub` | Public key authorized in the container |
| `--pubkey2` | none | Optional second public key |
| `--provision` | off | Run `cd /workspace && curl -fsSL URL \| bash`; no URL uses the maintainer script |
| `--modal` | auto | Explicit modal CLI path |

### `status`

Shows matching Modal apps (ID, state, task count) and the managed
`~/.ssh/config` block for the session.

### `stop`

Stops every non-stopped app with that description, then removes the managed
SSH block. `--keep-config` leaves the block in place. The persistent volume is
**never** deleted.

## Examples

CPU-only development box:

```sh
python3 scripts/modal_ssh_session.py launch --name g-data-linkup
```

L4 for a 24 GB workload:

```sh
python3 scripts/modal_ssh_session.py launch --name g-train --gpu L4
```

Large session with two GPUs and a 48 h lifetime:

```sh
python3 scripts/modal_ssh_session.py launch --name g-big \
  --gpu H100:2 --cpu 32 --memory 65536 --timeout 172800 --volume g-big-ws
```

Launch and provision in one step:

```sh
python3 scripts/modal_ssh_session.py launch --name g-data-linkup --provision
```

Launch with a custom provisioning script:

```sh
python3 scripts/modal_ssh_session.py launch --name g-data-linkup \
  --provision https://example.com/setup.sh
```

## Lifecycle

- The sandbox runs until `--timeout` expires or the app is stopped.
- Every launch gets a new relay host/port. The managed `~/.ssh/config` block is
  replaced in place, so the alias and VS Code host stay stable.
- Only `/workspace` (the volume) survives a respawn. System packages, Node,
  Python environments outside `/workspace`, and running processes do not —
  re-run `--provision` after relaunching.
- Stopping an app does not delete its volume. Delete volumes only when you
  intend to erase the data: `modal volume delete <name>`.

## Cost

Approximate Modal list prices at the time of writing (2026-10):

| Session | Approx. cost/hour |
|---|---|
| 8 CPU + 32 GB (CPU-only) | ~$0.6 |
| 8 CPU + 32 GB + L4 | ~$1.6 |
| 32 CPU + 64 GB + H100 | ~$7 |

Idle sessions bill until stopped or until the timeout expires. Stop sessions
when you are done.

## How SSH works on Modal

Modal containers have no public IP. The app starts `sshd` and requests
`unencrypted_ports=[22]`, which makes Modal relay raw TCP from its edge to port
22 inside the container. "Unencrypted" refers to Modal not terminating TLS on
that relay; SSH still encrypts the session end to end, and only the public
keys injected at startup can log in. Modal's own `modal launch machine`
command uses the same mechanism.

## Troubleshooting

See [the skill's troubleshooting guide](skills/modal-deploy-ssh/references/TROUBLESHOOTING.md)
for authentication failures, stale endpoints, GPU queues, volume questions,
and VS Code reconnect steps.

## License

Apache-2.0. See the repository [LICENSE](../../LICENSE).
