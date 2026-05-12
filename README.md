# Lima OpenClaw Runner

This repo contains a local launcher for a Lima VM that installs and runs the
OpenClaw gateway inside Ubuntu 24.04, configured for DeepSeek and reachable from
the macOS host through Lima port forwarding.

## Quick Start

On a new Mac, install Lima first:

```bash
brew install lima
```

Then create the local config and start a VM:

```bash
scripts/lima-claw init
# edit lima-claw.env and set DEEPSEEK_API_KEY
scripts/lima-claw start openclaw
scripts/lima-claw chat openclaw "Say hello in one short sentence."
```

The default host URL is:

```text
http://127.0.0.1:18789/
```

The gateway also enables the OpenAI-compatible HTTP chat endpoint:

```bash
curl -sS http://127.0.0.1:18789/v1/chat/completions \
  -H 'Content-Type: application/json' \
  --data '{"model":"openclaw","messages":[{"role":"user","content":"Say hello"}],"stream":false}'
```

## Configuration

`scripts/lima-claw` reads `lima-claw.env` by default. If only the old
`openclaw-lima.env` exists, it will use that for compatibility. Config files
are ignored by git because they contain the DeepSeek API key.

The default VM shape is intentionally modest for running several instances on a
Mac:

```dotenv
INSTANCE_NAME=openclaw
CPUS=2
MEMORY=4GiB
DISK=30GiB
GUEST_PORT=18789
HOST_PORT=18789
DEEPSEEK_MODEL=deepseek/deepseek-v4-flash
OPENCLAW_VERSION=latest
OPENCLAW_POST_INSTALL_COMMANDS=
```

`OPENCLAW_POST_INSTALL_COMMANDS` runs inside the VM after Node/OpenClaw are
installed and before the gateway starts. It is intended for non-interactive
OpenClaw plugins or integration installers, for example:

```dotenv
OPENCLAW_POST_INSTALL_COMMANDS="openclaw plugins install '@example/openclaw-plugin'"
```

The script records a marker for the exact command string, so it runs again only
when you change that config value.

For installers that print a QR code or ask for input, SSH into the VM instead:

```bash
scripts/lima-claw ssh openclaw
npx -y @tencent-weixin/openclaw-weixin-cli@latest install
```

The VM setup exposes OpenClaw's bundled Node toolchain in SSH shells, so `node`,
`npm`, and `npx` are available after `scripts/lima-claw start`.

For three concurrent VMs, create one config per instance and vary at least
`INSTANCE_NAME` and `HOST_PORT`, for example `18789`, `18790`, and `18791`.

## Commands

```bash
scripts/lima-claw list                  # show Lima VMs
scripts/lima-claw start [name]          # create/start VM, configure OpenClaw, wait for /readyz
scripts/lima-claw resume [name]         # start an existing stopped VM and gateway
scripts/lima-claw status [name]         # show Lima status plus /healthz and /readyz
scripts/lima-claw chat [name] "hello"   # send one OpenAI-compatible HTTP chat request
scripts/lima-claw ssh [name]            # SSH into the VM
scripts/lima-claw shell [name]          # open a limactl shell in the VM
scripts/lima-claw stop [name]           # stop the gateway process and Lima VM
scripts/lima-claw destroy [name]        # delete the Lima VM and generated state
```
