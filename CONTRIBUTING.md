# Contributing

Thanks for improving `JSxtrack`.

## Development

```bash
cargo fmt --all
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```

Build the Caido plugin:

```bash
cd caido-plugin
corepack pnpm@10 install --frozen-lockfile
corepack pnpm@10 typecheck
corepack pnpm@10 build
```

## Pull Requests

- Keep changes focused.
- Add tests for new parsing, chunk discovery, source map, and AST behavior.
- Update `README.md` or `INSTALL.md` when user-facing behavior changes.
- Do not commit captured third-party application code.
