# Getting started with DeepSeek Harness

English | [中文](getting-started.zh.md)

This tutorial takes a first-time user from an empty machine to a running Web UI with a working agent session. It assumes you can install software on your own computer; a DeepSeek API key is needed only when the agent calls a model.

## What DeepSeek Harness is

DeepSeek Harness (`dsh`) is an open-source agent harness from DeepSeek AI. Every capability is a plugin running on Cordis: the agent loop, tools, shell and filesystem capabilities, web search, LLM providers, and the Web UI itself can all be replaced or extended. See the [architecture overview](../architecture.md) for how the pieces compose.

## Prerequisites

- Node.js 22.19+ or 24+.
- pnpm (this repo pins pnpm 11.7.0; corepack activates it from the lockfile).
- Git 2.26+.
- Optional but recommended: a DeepSeek API key for real agent tasks.

Check your tools:

```sh
node --version
git --version
```

If `pnpm` is missing, activate the pinned version with corepack:

```sh
corepack enable
corepack prepare pnpm@11.7.0 --activate
pnpm --version
```

If corepack shims are not on your PATH, run the same commands through `corepack pnpm` instead.

## Get the source

```sh
git clone https://github.com/deepseek-ai/deepseek-harness.git
cd deepseek-harness
```

## Install and build

```sh
pnpm install
pnpm run build
```

`pnpm install` installs every workspace package and configures the Lefthook Git hooks. `pnpm run build` compiles the TypeScript packages and builds the Web frontend; the source checkout must build successfully before it can run.

## Start the Web UI

```sh
pnpm dsh web
```

The command prints its URL; the UI is served at `http://127.0.0.1:3080` by default. Keep the process running and open the URL in a browser.

## Configure a model

Open **Settings → Models**, enter a DeepSeek API key, and save it. The model route becomes usable immediately without a restart. Other providers and custom OpenAI-compatible endpoints are covered in the [provider guide](./providers.md).

For the command-line modes, put the key in a root `.env` file instead:

```sh
DEEPSEEK_API_KEY=sk-...
```

## Run your first task

Click **Choose workspace**, add a project directory, and select it. Start a session and send a prompt, for example:

> Summarize this repository and identify its main packages.

The agent can read and edit workspace files, run commands, delegate work, and maintain a plan. Operations that need approval under the active permission policy ask you first.

## Other entry points

- The [Web UI guide](./index.md) covers model and workspace setup in more depth.
- Headless mode answers one task and exits: `pnpm dsh --profile headless "run the tests"`.
- The ACP automation server and the self-referential Cordis demo need a key: `pnpm run demo:acp` and `pnpm run demo:cordis`.
- The [Python SDK](./python-sdk.md) drives agents from Python.
- Plugin development starts with the [first plugin tutorial](../develop/basic/).
- The [CLI reference](../../../apps/cli/README.md) documents profiles and launcher flags such as `--profile`, `--patch`, and `--dump-config`.

## Development commands

After editing code, run the focused checks:

```sh
pnpm run typecheck
pnpm run test
pnpm run test:snapshot
pnpm run lint
pnpm run build
```

Non-trivial changes carry Agent Notes, tests, and snapshots; read [CONTRIBUTING.md](../../../CONTRIBUTING.md) and [docs/development.md](../../development.md) before contributing.

## Troubleshooting

- `pnpm: command not found` — activate pnpm through corepack as above, or run the same commands via `corepack pnpm`.
- `Port 3080 already in use` — stop the process holding the port (`lsof -iTCP:3080 -sTCP:LISTEN`), or check the port flags the web profile exposes (`pnpm dsh web --help`).
- Boot fails with an `EPERM` creating `~/.dsh` — the process cannot write its profile home; check directory permissions, or run without a sandboxed permission manager.
- The UI opens but tasks fail without a model — configure the API key in **Settings → Models** or in the root `.env`; the server does not need a restart.

## Next steps

- [Web UI guide](./index.md) — model and workspace configuration.
- [Architecture overview](../architecture.md) — how the plugin system composes.
- [Development guide](../../development.md) — the contributor setup and daily workflow.
- [Cookbook](../../cookbook/adding-a-package.md) — step-by-step package how-tos.
