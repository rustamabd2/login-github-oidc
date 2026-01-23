# Login to CUE Registry via GitHub OIDC

A GitHub Action that authenticates to a CUE registry using GitHub's OIDC tokens.

## Features

- Authenticates using GitHub's OIDC provider (no static credentials needed)
- Configurable registry hostname
- Automatically configures `cue` CLI with registry credentials

## Prerequisites

Your workflow must have the following permissions:

```yaml
permissions:
  id-token: write
  contents: read
```

## Usage

### Basic usage (with default registry)

```yaml
- name: Login to CUE registry
  uses: rustamabd2/login-github-oidc@v1
```

### Using the access token

The action outputs an `access_token` that can be used for direct API calls:

```yaml
- name: Login to CUE registry
  id: oidc
  uses: rustamabd2/login-github-oidc@v1

- name: Test registry access
  run: |
    curl -sSL https://registry.cue.works/v2/ \
      -H "Authorization: Bearer ${{ steps.oidc.outputs.access_token }}"
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `registry` | CUE registry hostname | No | `registry.cue.works` |
| `update_logins` | Whether to update the local CUE logins.json file | No | `true` |

## Outputs

| Output | Description |
|--------|-------------|
| `access_token` | The access token obtained from the registry |

## How it works

1. Obtains a GitHub OIDC token with the registry URL as the audience
2. Exchanges the OIDC token for a registry access token via `/login/oidc/github` endpoint
3. Optionally configures the `cue` CLI with the registry credentials in `~/.config/cue/logins.json`

## Requirements

- The `cue` CLI must be installed before using this action
- Your GitHub repository/environment must be configured to trust the registry's OIDC endpoint

## Example workflow

```yaml
name: Publish CUE module

on:
  push:
    tags:
      - 'v*'

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    
    steps:
      - uses: actions/checkout@v4

      - name: Login to CUE registry
        uses: rustamabd2/login-github-oidc@v1

      - name: Install Go
        uses: actions/setup-go@v6
        with:
          go-version: '1.25'
      
      - name: Install Cue
        run: go install cuelang.org/go/cmd/cue@latest
      
      - name: Publish module
        run: |
          cue mod publish ${{ github.ref_name }}
```

## License

MIT
