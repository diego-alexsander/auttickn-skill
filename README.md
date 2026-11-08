# auttickn skill

Agent skill that turns a feature of a web app into a narrated tutorial video,
from the app's own code. The agent reads the repository, declares the path the
viewer should see, and auttickn validates, records and narrates it.

The skill is in [`skills/auttickn/SKILL.md`](skills/auttickn/SKILL.md) and
follows the open SKILL.md format, so any agent that reads skills can use it.

## Requirements

The auttickn MCP must be connected, with OAuth or an organization API key.
With OAuth, request the scopes `auttickn:mcp` and `offline_access`. In Kiro
CLI, set them in `mcp.json`:

```json
"oauthScopes": ["auttickn:mcp", "offline_access"]
```

Without that, Kiro CLI adds the `stub` scope and the server answers
`invalid_scope`.

## Install

Claude Code:

```
/plugin marketplace add diego-alexsander/auttickn-skill
/plugin install auttickn@auttickn
```

Other agents: copy `skills/auttickn/` into the agent's skills folder.

## License

[Apache-2.0](LICENSE). The license does not grant rights to the auttickn name
or trademark.
