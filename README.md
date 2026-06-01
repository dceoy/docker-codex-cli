# docker-codex-cli

Dockerfile and compose.yml for [Codex CLI](https://developers.openai.com/codex/cli/)

[![CI/CD](https://github.com/dceoy/docker-codex-cli/actions/workflows/ci.yml/badge.svg)](https://github.com/dceoy/docker-codex-cli/actions/workflows/ci.yml)

This repository provides a Docker Compose workflow that:

- installs [Codex CLI](https://developers.openai.com/codex/cli/) (`codex`) during the image build
- runs the container as a non-root `agent` user by default
- mounts the current repository at `/workspace`
- persists Codex CLI and application state in named Docker volumes

## Included tools

- [Codex CLI](https://developers.openai.com/codex/cli/) (`codex`)
- [GitHub CLI](https://cli.github.com/) (`gh`)
- git, jq, npm, pipx, python3-pip, ripgrep, rsync, tree, unzip, uv, vim, wget, zsh
- [Oh My Zsh](https://ohmyz.sh/)
- [print-github-tags](https://github.com/dceoy/print-github-tags)

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) with the Compose plugin
- Either a ChatGPT account (sign-in happens interactively on first launch — see
  [Authentication](#authentication) below) or an OpenAI API key

## Quick start

1. Build the image:

   ```bash
   docker compose build
   ```

2. Start an interactive Codex CLI session:

   ```bash
   docker compose run --rm codex-cli
   ```

   The service starts in `/workspace` and runs:

   ```bash
   zsh -lc 'codex --dangerously-bypass-approvals-and-sandbox'
   ```

   > [!WARNING]
   > `--dangerously-bypass-approvals-and-sandbox` disables Codex's approval
   > prompts and sandboxing. It is convenient inside a disposable container, but
   > only run it on code you trust. Drop the flag to keep the default,
   > approval-gated behaviour:
   >
   > ```bash
   > docker compose run --rm codex-cli -lc 'codex'
   > ```

## Authentication

Codex CLI supports two authentication methods:

- **Sign in with ChatGPT** — on first launch, `codex` prints an authorization
  URL; open it on your local machine and complete the sign-in. Credentials are
  persisted in `/home/agent/.codex` via the `codex-data` Docker volume, so you
  only need to sign in once per volume lifetime. Run `codex logout` to clear
  saved credentials.
- **OpenAI API key** — export `OPENAI_API_KEY` on the host before launching;
  Compose forwards it into the container automatically.

## Common commands

Override the default command to run a specific tool inside the container:

```bash
docker compose run --rm codex-cli -lc 'codex --version'
docker compose run --rm codex-cli -lc 'gh auth status'
docker compose run --rm codex-cli -lc 'git status'
```

## Runtime layout

- The repository root is mounted to `/workspace`.
- Codex CLI state is persisted in the `codex-data` volume at `/home/agent/.codex`.
- Application config is persisted in the `config-data` volume at `/home/agent/.config`.
- The Compose service repairs volume ownership on startup when previously created as `root:root`.
- Build args `USER_NAME`, `USER_UID`, and `USER_GID` default to `agent`, `1001`, and `1001`.

## Environment variables

| Variable         | Required | Description                                           |
| ---------------- | -------- | ----------------------------------------------------- |
| `OPENAI_API_KEY` | No       | Authenticate Codex CLI with an OpenAI API key         |
| `GITHUB_TOKEN`   | No       | Authenticate `gh` CLI operations inside the container |

If `OPENAI_API_KEY` is not set, `codex` falls back to interactive ChatGPT
sign-in. See [Authentication](#authentication).
