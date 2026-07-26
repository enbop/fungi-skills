# CLI workflows

Use current `--help` output when it differs from this reference.

## Contents

- Install and daemon
- Device onboarding
- Recipes and service lifecycle
- Useful context

## Install and daemon

Official quick installer for macOS arm64, Linux x86_64, and Linux arm64:

```bash
curl -fsSL https://fungi.rs/install.sh | sh
```

Offer to inspect `https://fungi.rs/install.sh` before executing it. The default destination is `~/.local/bin`; ensure the standalone `fungi` CLI is on `PATH`, even when the desktop Fungi App is installed. Do not invoke a binary inside an app bundle. For unsupported platforms, use the assets at `https://github.com/enbop/fungi/releases/latest`. Do not build from source unless requested or no release fits.

Identify the CLI, then probe for an existing daemon before running `init`, starting a user service, or launching a foreground daemon:

```bash
fungi info build --json
fungi info version
fungi info rpc-address
fungi info config-path
```

`info build` describes the local CLI. The other commands query the daemon. If they succeed, reuse that daemon; do not start a second one. A desktop Fungi App and the CLI can share a compatible daemon in the default Fungi directory. Keep App-specific handling brief and keep agent operations CLI-first.

If versions differ or incompatibility is reported, record the CLI and daemon versions, RPC address, config path, and likely process owner first. A difference alone is not proof of incompatibility. Never stop, restart, kill, or close the daemon or Fungi App without explicit user permission. Treat a process as externally managed when ownership is unclear.

On Linux the installer may create `~/.config/systemd/user/fungi.service`:

```bash
systemctl --user start fungi.service
systemctl --user status fungi.service
journalctl --user -u fungi.service -n 200
```

Only when no daemon responds, run `fungi init` if needed and start the installed user service or run `fungi daemon` in a persistent foreground terminal. `fungi init` creates configuration and key material. Use `fungi init --upgrade-config` only to rewrite an older config intentionally.

## Device onboarding

On each CLI-capable device, run:

```bash
fungi info id
fungi device mdns
```

For an App-only target device, ask the user to copy its Device ID from Fungi App instead. Verify every Device ID through a trusted channel.

Save and inspect a device:

```bash
fungi device add NAME DEVICE_ID
fungi device add NAME DEVICE_ID --addr /ip4/ADDRESS/tcp/PORT
fungi device list
fungi device get NAME
```

Saving a device does not authorize it. Treat trust as high risk and do not place it in an unattended onboarding batch. On the device that would grant access:

```bash
fungi device add CONTROLLER_NAME CONTROLLER_DEVICE_ID
fungi device get CONTROLLER_NAME
fungi security show
```

Present the full Device ID, authorization direction, service-management capability, allowed host paths, persistence, and `fungi device untrust CONTROLLER_NAME` rollback command. Pause until the user explicitly approves this exact authorization. A general request to add or connect devices is not approval.

Only after approval, run:

```bash
fungi device trust CONTROLLER_NAME
fungi device trusted
```

Do not automate Fungi's confirmation input. Repeat the approval process independently in the opposite direction only when mutual access is wanted. Diagnose with:

```bash
fungi ping NAME
fungi connection overview --verbose
fungi connection streams --verbose
fungi connection relay-status --verbose
```

## Recipes and service lifecycle

```bash
fungi service recipe list --refresh
fungi service recipe show RECIPE
fungi service apply NAME --recipe RECIPE --dry-run
fungi service apply NAME --recipe RECIPE --start
fungi service apply NAME@DEVICE --recipe RECIPE --dry-run
fungi service apply NAME@DEVICE --recipe RECIPE --start
```

For a custom file, replace `--recipe RECIPE` with its path. Use `NAME` for a local service or `NAME@DEVICE` for a remote service:

```bash
fungi service apply NAME ./service.fungi.md --dry-run
fungi service apply NAME@DEVICE ./service.fungi.md --start
```

Observe and control local or remote instances uniformly:

```bash
fungi service list --refresh
fungi service inspect NAME@DEVICE --verbose
fungi service logs NAME@DEVICE --tail 200
fungi service start NAME@DEVICE
fungi service stop NAME@DEVICE
fungi service remove NAME@DEVICE
```

Omit `@DEVICE` from these lifecycle commands for a local service.

Connect a published remote endpoint locally:

```bash
fungi service connect NAME@DEVICE
fungi service connect NAME@DEVICE ENTRY
fungi service connect NAME@DEVICE ENTRY --local-port PORT
fungi service disconnect NAME@DEVICE
```

Remote service management succeeds only when the target device trusts the controller. If a device is offline, `fungi service remove NAME@DEVICE --local-only` forgets cached state and does not remove the service from that device.

## Useful context

- Prefer the default Fungi directory so the CLI and desktop App can reach the same daemon. When isolation is intentional, put `-f /path/to/fungi` before the command and use it consistently.
- `fungi info build --json` describes the local binary without requiring the daemon.
- `fungi info config-path` and `fungi info rpc-address` identify the active daemon configuration and endpoint.
- `fungi security show` displays runtime boundaries. `security allow-path` expands host access and needs explicit user intent.
- Re-run the official installer to update. If the update requires a daemon restart, identify its owner and obtain explicit user permission before stopping or restarting it.
