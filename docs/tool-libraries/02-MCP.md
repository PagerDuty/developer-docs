# MCP (Model Context Protocol) at PagerDuty

If you're looking to use MCP with PagerDuty, you've come to the right place!

MCP (or [Model Context Protocol]) is designed to give agentic clients the tools they need to perform complex
interactions with web businesses like PagerDuty, based on [JSON-RPC].

## Local MCP server

If you'd like to run our MCP server yourself, you can check out the instructions [in Github][open-source MCP server].
Pull requests welcome!

## Remote MCP server

You can also call our MCP API remotely.

Key URLs:

- MCP Base URL: `https://mcp.pagerduty.com/mcp`
- OAuth Metadata: `https://identity.pagerduty.com/global/oauth/anonymous/.well-known/openid-configuration`

You can see the full list of tools by calling the `list/tools` method, or by browsing our [open-source MCP server].

### Getting Started

The MCP API supports the same authentication methods as the REST API, including API tokens and OAuth. See our
[authentication documentation] for more information.

Here is a sample MCP configuration for Microsoft VSCode:

```json
{
    "servers": {
    "pagerduty-mcp": {
        "url": "https://mcp.pagerduty.com/mcp",
        "type": "http",
        "headers": {
        "Authorization": "Token token=${input:pagerduty-api-key}"
        }
    }
    },
    "inputs": [
    {
        "type": "promptString",
        "id": "pagerduty-api-key",
        "description": "PagerDuty API Key",
        "password": true
    }
    ]
}
```

  [Model Context Protocol]: https://modelcontextprotocol.io/specification/2025-06-18/basic
  [JSON-RPC]: https://www.jsonrpc.org/
  [open-source MCP server]: https://github.com/PagerDuty/pagerduty-mcp-server
  [authentication documentation]: https://developer.pagerduty.com/docs/rest-api-v2/authentication/