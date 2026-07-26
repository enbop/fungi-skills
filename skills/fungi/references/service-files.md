# Authoring `.fungi.md` service files

A service file is Markdown with strict YAML frontmatter. Unknown fields are rejected. Use `fungi service apply NAME FILE --dry-run` locally or `fungi service apply NAME@DEVICE FILE --dry-run` remotely as the authoritative validator.

## Common shape

```yaml
---
fungi: service/v1
id: example-service

run:
  provider: docker
  source:
    image: example/image:1.2.3
  args:
    - --listen
    - 0.0.0.0:8080
  env:
    EXAMPLE_MODE: production
  mounts:
    - from: $fungi.service.data
      to: /data

publish:
  web:
    tcp:
      port: 8080
    client:
      kind: web
      path: /
---

# Example service

Explain what runs, what data is mounted, what is reachable, and relevant safety assumptions.
```

`id` identifies the reusable definition. `service apply NAME ...` sets the deployed instance name, so one definition can back differently named instances.

## Choose one runtime pattern

Docker requires exactly `run.source.image`; omit `run.mode` and `publish.*.tcp.host`. Its host-facing port is allocated automatically:

```yaml
run:
  provider: docker
  source:
    image: example/image:1.2.3
```

Wasmtime requires exactly one of `run.source.url` or `run.source.file`. A relative file is resolved from the service file's directory. Add `mode: http` only for a WASI HTTP component:

```yaml
run:
  provider: wasmtime
  mode: http
  source:
    url: https://example.invalid/releases/v1/app.wasm
```

To publish an already-running local TCP service, omit `run`. Exactly one publish entry is allowed, and the host must be `127.0.0.1` or `localhost`:

```yaml
publish:
  ssh:
    tcp:
      host: 127.0.0.1
      port: 22
    client:
      kind: ssh
```

## Field rules

- `fungi` must equal `service/v1`; `id` must be non-empty; `publish` needs at least one entry.
- `run` accepts `provider`, optional Wasmtime `mode`, `source`, `args`, `env`, and `mounts`.
- Each mount contains `from` and runtime-side `to`.
- Prefer `$fungi.service.data` for private persistent app data and `$fungi.service.artifacts` for service artifacts. `$fungi.workspace` exposes the user's Fungi workspace; `$fungi.root` is broader and requires special care.
- Each publish entry contains `tcp.port` greater than zero and optional `client` metadata.
- `client.kind: web` may use `path` and `iconUrl`; `ssh` expresses SSH intent; other kinds are treated as raw clients.
- All publish entries currently need matching client metadata.
- Wasmtime and existing-TCP ports are fixed local ports and must not collide. Docker ports are mapped to automatically allocated host ports.
- Keep credentials out of frontmatter. Confirm how the workload receives secrets before deploying it.

## Validation loop

```bash
fungi info runtime
fungi service apply NAME@DEVICE ./NAME.fungi.md --dry-run
fungi service apply NAME@DEVICE ./NAME.fungi.md --start
fungi service inspect NAME@DEVICE --verbose
fungi service logs NAME@DEVICE --tail 200
```

Omit `@DEVICE` for a local service.

If validation fails, change only the reported field or runtime assumption, then rerun `--dry-run`. If startup fails, preserve the file, inspect state and bounded logs, revise, and apply again.
