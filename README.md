# XIDL Standard Library

Reusable, transport-neutral types for
[xidlc](https://github.com/xidl/xidl). Use this repository as an XIDL library
dependency instead of copying common definitions into every contract.

This is a standard
[XIDL library](https://xidl.netlify.app/docs/xidl-toml/): `xidl.toml` declares
the package, and the root-level `*.idl` files are its inputs.

## Available Types

| File | Type | Fields |
| --- | --- | --- |
| `timestamp.idl` | `Timestamp` | `int64 seconds`, `int32 nanos` |
| `duration.idl` | `Duration` | `int64 seconds`, `int32 nanos` |

The types use the familiar `(seconds, nanos)` representation and do not impose
JSON encoding or wire-format annotations. Consuming contracts can apply their
own serialization conventions.

## Requirements

- `xidlc` 0.107.0 or newer, as declared by `compiler` in `xidl.toml`
- `git` for remote library dependencies
- `curl` and `tar` when `xidlc` must download a matching toolchain

The repository pins its compiler version with
[mise](https://mise.jdx.dev/). Install it with:

```bash
mise install
```

With [direnv](https://direnv.net/) installed, run `direnv allow` once in this
directory to load the pinned toolchain automatically.

## Use The Library

Add `std` to the `[dependencies]` section of your project's `xidl.toml`. A Git
dependency must have exactly one of `tag` or `rev`:

```toml
name = "my-service"
version = "0.1.0"

[dependencies]
"std" = { src = "github.com/xidl/std", rev = "c88182d" }
```

Pin a release tag or commit and update it deliberately. To use a local checkout
during development, point at the directory instead:

```toml
[dependencies]
"std" = { path = "../xidl-std" }
```

A relative `path` is resolved from the directory containing the manifest.

The dependency key becomes the mount name used by `#include`. Include only the
files needed by the contract:

```idl
#include "std/timestamp.idl"
#include "std/duration.idl"

struct Event {
    Timestamp at;
    Duration ttl;
};
```

Then generate as usual. For example, with the manifest above in the current
directory:

```bash
xidlc gen --out-dir generated rust .
```

`#include` inlines the definitions into the current compilation unit, which is
the simplest choice for these value types. If a project needs a separate
generated unit, use `import "std/timestamp.idl";` instead.

## Development

Run the checks against the library manifest:

```bash
mise exec -- xidlc lint *.idl
mise exec -- xidlc gen --dry-run rust .
```

Format IDL changes before committing:

```bash
mise exec -- xidlc fmt -i *.idl
```

The repository also provides pre-commit hooks for formatting and basic file
hygiene:

```bash
pre-commit install
pre-commit run --all-files
```

When adding a type:

1. Add a root-level `<name>.idl` file so the manifest includes it by default.
2. Reuse existing standard types instead of redefining equivalent structures.
3. Keep the definition transport-neutral unless a protocol mapping is part of
   the type's contract.
4. Run lint, generation, and formatting before committing.
