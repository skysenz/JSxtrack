# JSxtrack

`JSxtrack` is an open source JavaScript asset recovery tool for security research, source map reversal, and AI-agent-ready code review workflows.

It runs a local ingestion server, receives browser/proxy traffic from the included Caido plugin, mirrors JavaScript and HTML assets to disk, beautifies minified code, discovers lazy-loaded chunks, fetches source maps, and reconstructs original source trees when source maps contain `sourcesContent`.

## Features

- URL-mirrored directory structure for captured HTML and JavaScript.
- Automatic JS and HTML beautification without overwriting originals.
- Webpack, Vite, and Next.js chunk discovery.
- Recursive lazy-loaded chunk fetching with configurable rate limiting.
- Automatic source map discovery from comments, headers, inline data URLs, and sibling `.map` URLs.
- Source map reversal into original source layout with safe path sanitization.
- JXScout-inspired AST findings for security-oriented JavaScript review.
- Agent-ready JSONL metadata for assets, relationships, and findings.
- Caido plugin that streams matching responses into the local server and provides a manual send action for selected HTTP history entries.
- GitHub Actions for tests, multi-platform binaries, Caido plugin builds, and releases.

## Quick Start

```bash
cargo install --path crates/jxstrack-cli
jsxtrack
```

Running `jsxtrack` opens an interactive prompt with the available tools. Type `serve --project default` to start the local receiver, or start it directly with `jsxtrack serve --project default`. Keep the receiver running in a terminal, then build and install the Caido plugin from `caido-plugin/dist/plugin_package.zip` (see [INSTALL.md](INSTALL.md)). The plugin sends captured assets to the local JSxtrack receiver:

```text
http://127.0.0.1:3333
```

This is a separate port from Caido. For example, if Caido uses port `3030` for its interface or proxy listener, leave it on `3030`; JSxtrack receives plugin data on `3333`. `127.0.0.1` means the same computer, and the plugin forwards matching Caido responses to JSxtrack.

Captured projects are written to (Windows: `%USERPROFILE%\jxstrack\<project>`):

```text
~/jxstrack/<project>
```

See [INSTALL.md](INSTALL.md) for full installation and Caido setup instructions.

## Cara kerja singkat

1. Jalankan `jsxtrack serve` agar server lokal menerima data.
2. Plugin Caido mengirim respons HTML, JavaScript, dan source map ke `http://127.0.0.1:3333`.
3. JSxtrack menyimpan file asli, versi beautified, hasil pembalikan source map, dan metadata di `~/jxstrack/<project>` (Windows: `%USERPROFILE%\jxstrack\<project>`).
4. Buka folder proyek itu dengan editor untuk meninjau aset dan temuan analisis. Anda juga bisa mengirim request tersimpan secara manual dari menu klik kanan di HTTP History.

## Project Layout

```text
original/                 Raw captured HTML and JavaScript
beautified/               Formatted versions for review
sourcemaps/raw/           Downloaded or extracted source maps
sourcemaps/reversed/      Recovered original sources
analysis/                 Per-file AST analysis JSON
metadata/assets.jsonl     Asset index for tools and AI agents
metadata/relationships.jsonl
metadata/findings.jsonl
```

## CLI

```bash
jsxtrack serve \
  --host 127.0.0.1 \
  --port 3333 \
  --project default \
  --rate-per-second 2 \
  --fetch-concurrency 5
```

Useful flags:

- `--scope <pattern>` filters captured URLs. Can be repeated.
- `--output <path>` changes the output root.
- `--rate-per-second <n>` controls chunk and source map fetches.
- `--rate-per-minute <n>` applies a minute-level cap.
- `--fetch-concurrency <n>` controls concurrent follow-up downloads.
- `--max-body-bytes <n>` rejects oversized captured responses.

## Security Notes

`JSxtrack` stores application code and metadata locally. Only run it for systems you are authorized to test. The chunk discovery engine does not execute arbitrary JavaScript; it uses static parsing and constrained string evaluation for known bundler runtime shapes.

## License

MIT. See [LICENSE](LICENSE).
