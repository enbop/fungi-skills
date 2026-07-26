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

Offer to inspect `https://fungi.rs/install.sh` before executing it. The default destination is `~/.local/bin`; ensure it is on `PATH`. For unsupported platforms, use the assets at `https://github.com/enbop/fungi/releases/latest`. Do not build from source unless requested or no release fits.

On Linux the installer may create `~/.config/systemd/user/fungi.service`:

```bash
systemctl --user start fungi.service
systemctl --user status fungi.service
journalctl --user -u fungi.service -n 200
```

Otherwise run `fungi daemon` in a persistent foreground terminal. `fungi init` creates configuration and key material. Use `fungi init --upgrade-config` only to rewrite an older config intentionally.

## Device onboarding

Run on each device:

```bash
fungi info id
fungi device mdns
```

Save and inspect a peer:

```bash
fungi device add NAME DEVICE_ID
fungi device add NAME DEVICE_ID --addr /ip4/ADDRESS/tcp/PORT
fungi device list
fungi device get NAME
```

On the device receiving incoming management, save and trust the controller:

```bash
fungi device add CONTROLLER_NAME CONTROLLER_DEVICE_ID
fungi device trust CONTROLLER_NAME
fungi device trusted
```

Repeat in the opposite direction only when mutual access is wanted. Diagnose with:

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

- Put `-f /path/to/fungi` before the command and use it consistently.
- `fungi info build --json` describes the local binary without requiring the daemon.
- `fungi info config-path` and `fungi info rpc-address` identify the active daemon configuration and endpoint.
- `fungi security show` displays runtime boundaries. `security allow-path` expands host access and needs explicit user intent.
- Re-run the official installer to update, then restart an already-running daemon.
