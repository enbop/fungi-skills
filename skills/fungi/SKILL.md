---
name: fungi
description: Install, configure, and operate the Fungi CLI across trusted devices. Use when an agent needs to install or initialize Fungi, add and trust devices, diagnose connectivity, manage local or remote services, apply official recipes, author .fungi.md service files, or verify service state with inspect and bounded logs.
---

# Fungi

Operate Fungi as a private, multi-device service platform. Keep every action explicit, observable, and reversible where possible.

## Start with the current CLI

1. Run `fungi --version` and the relevant `fungi <command> --help` before mutating state. Fungi is evolving; prefer installed CLI help over remembered syntax.
2. Use the same global `--fungi-dir`/`-f` value for every command in a workflow when the user selects a non-default directory.
3. Treat the local daemon as the control plane. Most `info`, `device`, `service`, `connection`, and `ping` commands require it.
4. Distinguish the local device from the target device before changing a service. Address a remote service as `NAME@DEVICE` or use `fungi service --device DEVICE ...`.

Read [references/cli-workflows.md](references/cli-workflows.md) when installing Fungi, adding devices, operating services, or diagnosing failures. Read [references/service-files.md](references/service-files.md) before creating or modifying a `.fungi.md` file.

## Establish the baseline

1. Check whether `fungi` is on `PATH`. If it is absent, follow the platform-aware installation flow in the CLI reference and verify the binary before continuing.
2. Initialize with `fungi init` unless an existing configuration is already in use.
3. Start the daemon in the foreground or through the installed user service. Do not launch a second daemon if one is already responding.
4. Verify the baseline with:

```bash
fungi info version
fungi info id
fungi info runtime
```

Report missing runtimes before applying a service that depends on them.

## Add and trust devices

1. Obtain each device ID with `fungi info id` on that device. Have the user verify device IDs through a trusted channel.
2. Discover with `fungi device mdns` when devices share a LAN, or use a verified device ID and optional direct multiaddress.
3. Save the device with `fungi device add NAME DEVICE_ID [--addr MULTIADDR]`.
4. Explain the direction before trusting: running `fungi device trust controller` on a target device allows that controller to initiate access and manage services on the target. Trust is not automatically mutual.
5. Let the user review Fungi's security confirmation. Do not bypass it by scripting input.
6. Verify with `fungi device list`, `fungi device trusted`, `fungi ping DEVICE`, and, when needed, `fungi connection overview`.

Never trust a peer solely because it appeared in mDNS output.

## Manage a service from a recipe

1. Inspect available recipes with `fungi service recipe list` and `fungi service recipe show RECIPE`.
2. Confirm the runtime, source artifact or image, mounts, published ports, and security implications.
3. Preview without changing state:

```bash
fungi service apply NAME@DEVICE --recipe RECIPE --dry-run
```

Omit `@DEVICE` for a local service.

4. Apply, optionally adding `--start` when the user wants it running immediately. Avoid `--yes` unless non-interactive execution was explicitly requested and the preview was reviewed.
5. Close the feedback loop with `service inspect`, `service logs --tail`, and `service connect` when the service publishes an endpoint.

## Create a custom service

1. Translate the request into a provider, pinned source, arguments, environment, minimal mounts, published TCP endpoints, and client intent.
2. Prefer an official recipe when it already fits. Otherwise create a focused `.fungi.md` file using the current `service/v1` schema in the service-file reference.
3. Keep secrets out of the file and terminal output. Ask the user how secrets should be supplied rather than embedding them.
4. Prefer `$fungi.service.data` for service-owned persistent data. Expose `$fungi.workspace`, `$fungi.root`, or arbitrary host paths only when the user needs that access and understands the scope.
5. Pin container tags and artifact release URLs when practical. Do not invent an image, WASM URL, port, or runtime contract; verify uncertain upstream details first.
6. Validate before applying:

```bash
fungi service apply NAME@DEVICE ./NAME.fungi.md --dry-run
```

Omit `@DEVICE` for a local service.

7. Apply, inspect, start if necessary, inspect again, and read bounded logs. Revise the file and repeat until the observed state matches the request.

## Diagnose systematically

Use evidence in this order:

1. `fungi info version` and `fungi info runtime`
2. `fungi device get DEVICE` and `fungi device trusted`
3. `fungi ping DEVICE` and `fungi connection overview --verbose`
4. `fungi service inspect NAME@DEVICE --verbose` (omit `@DEVICE` for local)
5. `fungi service logs NAME@DEVICE --tail 200` (omit `@DEVICE` for local)

For remote logs, the default is 200 lines and the maximum is 2000. Always use a bounded `--tail` during diagnosis, including for local services. After every corrective change, rerun the smallest check that can prove it worked.

## Guard destructive and security-sensitive actions

- Explain that trusted devices can manage services and access configured host paths.
- Inspect a recipe or custom file before applying it.
- Require a clear target before `stop`, `remove`, `untrust`, or address removal. State whether the target is local or remote.
- Do not use remote `remove --local-only` as if it removed the actual service; it only forgets the local cached record.
- Do not edit Fungi's internal state directly when a CLI operation exists.
- Do not claim success from a zero exit status alone. Confirm the resulting device, connection, or service state.

## Finish with an operational summary

Report the Fungi version and config directory used, devices added or trusted and trust direction, services and target devices changed, verification performed, local connection addresses created, and any remaining security or runtime caveats.
