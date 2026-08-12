# Connecting Goose to gnt-brain

gnt-brain is gnt's rules-governance MCP server. Once Goose is connected, it can use
`check_action`, `search_rules`, `get_rule`, `list_skill_packs`, and `get_skill_pack`.
Connection only makes those tools available. Load [`TOOLS.md`](TOOLS.md) as Goose instructions as
well so the agent checks before taking a side effectful action.

## Configure the MCP extension

Goose reads its main configuration from `~/.config/goose/config.yaml` on macOS and Linux, or
`%APPDATA%\Block\goose\config\config.yaml` on Windows, as described in its [configuration
guide](https://github.com/aaif-goose/goose/blob/main/documentation/docs/guides/config-files.md).
Add the following entry under the existing top-level `extensions` key:

```yaml
extensions:
  gnt-brain:
    type: streamable_http
    name: gnt-brain
    enabled: true
    uri: "https://api.gntai.dev/mcp/"
    headers:
      Authorization: "Bearer ${GNT_MCP_KEY}"
    env_keys:
      - GNT_MCP_KEY
    envs: {}
    timeout: 300
    available_tools:
      - check_action
      - search_rules
      - get_rule
      - list_skill_packs
      - get_skill_pack
```

Keep the trailing slash in the MCP URL. Goose resolves each name in `env_keys` from the matching
environment variable first and from its secret storage as a fallback. It then substitutes the
resolved value in the URI and headers, so the credential does not need to be stored in
`config.yaml`.
This behavior is implemented in Goose's [extension manager](https://github.com/aaif-goose/goose/blob/main/crates/goose/src/agents/extension_manager.rs#L538-L611).

Create a key with `gnt keys create`, or use an existing key from `gnt keys list`. To use the shell
environment as the secret source, make the key available to the process that starts Goose:

```bash
export GNT_MCP_KEY="gnt_live_..."
```

PowerShell:

```powershell
$env:GNT_MCP_KEY = "gnt_live_..."
```

Command Prompt (`cmd.exe`):

```bat
set "GNT_MCP_KEY=gnt_live_..."
```

Do not commit the key or paste it into `config.yaml`, a project hint file, an issue, or a log. If
Goose is launched by its desktop app, make sure `GNT_MCP_KEY` is available to the app process, or
use Goose's configured secret storage for `env_keys` instead of putting the value in the
configuration file.

## Load the action policy

Goose loads project context from `.goosehints`. Add the contents of [`TOOLS.md`](TOOLS.md) to the
project's `.goosehints` file, or reference the file with Goose's `@` syntax:

```text
@path/to/integrations/goose/TOOLS.md
```

For a guardrail that is injected on every turn, point Goose's [persistent-instructions
setting](https://github.com/aaif-goose/goose/blob/main/documentation/docs/guides/context-engineering/using-persistent-instructions.md)
at this file:

```bash
# Run this from the gnt checkout, or replace $PWD with the file's absolute path.
export GOOSE_MOIM_MESSAGE_FILE="$PWD/integrations/goose/TOOLS.md"
```

PowerShell:

```powershell
$env:GOOSE_MOIM_MESSAGE_FILE = Join-Path $PWD "integrations/goose/TOOLS.md"
```

Command Prompt (`cmd.exe`):

```bat
set "GOOSE_MOIM_MESSAGE_FILE=%CD%\integrations\goose\TOOLS.md"
```

The persistent-instructions option is useful when the `check_action` requirement should be
injected on every turn. If Goose is launched from its desktop app, make sure these environment
variables are available to the app process, or use `.goosehints` instead.

## Verify the connection

Start a new Goose session after changing `config.yaml`. Confirm that the `gnt-brain` extension is
enabled and that its tools are listed. Before asking Goose to perform an action with an external
side effect, verify that it calls `check_action` first and follows the returned verdict. A missing
or unclear verdict is not permission to continue.

If the extension does not load, check the YAML indentation and confirm that `GNT_MCP_KEY` is available
either in the environment inherited by Goose or in Goose's configured secret storage. Goose checks the
inherited environment first, then falls back to secret storage for names listed under `env_keys`. Do not
put the raw key in a bug report while troubleshooting.
