# gh-demo-action

A demo GitHub Action for testing the Origin supply-chain-security platform.

## What it does

Logs `Hello, <name>!` (default name: `world`).

## Usage

```yaml
jobs:
  demo:
    runs-on: self-hosted-with-origin
    steps:
      - uses: resilientscale/gh-demo-action@<sha>
        with:
          name: 'GRunS'
```

## Inputs

| Name | Required | Default | Description |
|---|---|---|---|
| `name` | no | `world` | Name to greet in the hello-world step |

## Safety

This action is intended for demo and testing of Origin only. Do not depend on it for production workflows.
