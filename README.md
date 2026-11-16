# Local status observations

`amplifier-status-emitter` publishes and reads bounded, versioned local status
snapshots using only the Python standard library. The public import is
`amplifier_status`; the reference JSON consumer is `amplifier-status`.
Python 3.11 or newer is required. Importing the package starts no publisher.

These files are observations, not commands, authorization, execution receipts or
instructions to replay work. Missing information remains unknown. An ended
observation does not establish that work succeeded, and a stale observation after
a crash does not establish that work ended.

## Publish and read

Callers explicitly choose a private local directory and own the session identity:

```python
from amplifier_status import Publisher, discover

publisher = Publisher("/absolute/private/status", host="example", version="1.0")
try:
    publisher.submit("session-id", {"now": {"status": "thinking"}})
    # Publication is asynchronous and coalesced. Read observations when needed:
    observations = discover("/absolute/private/status")
finally:
    publisher.close()
```

The publisher bounds its retained observations and writes atomic private JSON
files. It accepts allow-listed status, location, plan counts and usage fields;
prompt, transcript, draft, tool arguments/results and raw exception text are not
published. Plan text is redacted by default. There is no network endpoint.

```sh
amplifier-status --directory /absolute/private/status
amplifier-status --directory /absolute/private/status --instance INSTANCE_ID
amplifier-status --directory /absolute/private/status --watch
```

Without `--directory`, the reader uses `AMPLIFIER_STATUS_DIR`, then
`~/.amplifier/emitter`. Exact instance selection avoids walking older producer
instances. Discovery reports freshness, invalid files and truncation explicitly.

## Optional hook

The `amplifier.modules` entry point `hooks-status-emitter` loads
`amplifier_status.hook:mount`. Importing or discovering it requires no Core or
Unified installation. Actually receiving native hook events requires a compatible
host that supplies Core; the standalone distribution does not install it.

For resolvers that require an explicit source, compose the installed package
directory returned by that environment's Python:

```sh
python -c 'import amplifier_status, pathlib; print(pathlib.Path(amplifier_status.__file__).parent)'
```

```yaml
hooks:
  - module: hooks-status-emitter
    source: /absolute/installed/site-packages/amplifier_status
    config:
      directory: /absolute/private/status
      include_plan_text: false
```

Mounting defers publication until a lifecycle event, deduplicates mounts and
defers to Unified's host-owned emitter. Installing the package does not enable
publication or add control over work. The full observation contract and host
integration limits are documented in the Unified repository's
[`docs/status-emitter.md`](https://github.com/microsoft/amplifier-unified/blob/main/docs/status-emitter.md).

## Build and test independently

From this directory, in an owned environment:

```sh
uv build
uv venv .venv
uv pip install --python .venv/bin/python dist/amplifier_status_emitter-0.1.0-py3-none-any.whl pytest
.venv/bin/python -m pytest -q tests
```

On Windows, use `.venv\Scripts\python.exe`. The wheel contains the six runtime
modules and distribution metadata. The source archive also includes this README,
the MIT license and the standalone tests. To verify the installed wheel without
the author tree, copy only `tests/` and `pyproject.toml` to a separate directory
and run pytest there. Do not add the author directory to `PYTHONPATH`.

These tests exercise the installed import, metadata, publisher, reader, CLI and
optional hook declaration without Unified, a native runtime, accounts or network
services. Host event integration and crash/recovery qualification remain separate.

## Ownership and provenance

This history-preserving project is prepared as the public author home at
`bkrabach/amplifier-platform-status`, with a later transfer to Microsoft planned.
[PROVENANCE.json](PROVENANCE.json) records the original public Unified source,
pre-move history, and exact imported runtime/test bytes. Original MIT attribution
is preserved. The repository name does not rename the `amplifier-status-emitter`
distribution or `amplifier_status` import.

This import changes no runtime behavior, dependency requirement, package version,
or maintained test. Unified retains its host integration and policy. Authoring
handoff and consumer selection are separate reviewed changes; retire Unified’s
authored project at handoff rather than maintain two authoring copies.

The public `amplifier_status` namespace is distinct from Unified’s private
`amplifier_web.status_protocol`. The packages can coexist without overlapping
installed files; installing this package does not replace Unified’s selected
private emitter.

## Existing delivery and future publication

Unified continues to select the immutable [status-emitter-v0.1.0 artifacts](https://github.com/microsoft/amplifier-unified/releases/tag/status-emitter-v0.1.0)
from their original public producer. The original source receipt, revision,
subdirectory and hashes remain historical identities; this repository does not
republish or relabel those artifacts. Unified’s generated `amplifier_web/status_protocol/`
is selected by `components/status-emitter.source.json` and verified offline during
normal builds. Do not edit that generated copy by hand.

The [package workflow](.github/workflows/package.yml) builds one wheel and source
archive, then tests that wheel outside the checkout. It does not publish a release
or change any consumer. A future publication needs an exact source/artifact
receipt and review of Unified’s selector for this repository-root project layout;
the existing selector identifies historical monorepo subdirectories. Source moves
do not reserve a new version or make the original receipts invalid.
