
운영체제에 맞는 claude 데스크탑 버전 설치
https://claude.ai/download

MCP 서버 목록
- [awesome-mcp-servers](https://github.com/punkpeye/awesome-mcp-servers)

❌ firecrawl mcp server 설치하여 크롤링을 해본다 
- https://github.com/mendableai/firecrawl-mcp-server
- 크레딧 500제한으로 하지 않음

`claude_desktop_config.json`
```text
{
  "mcpServers": {
    "mcp-server-firecrawl": {
      "command": "npx",
      "args": ["-y", "firecrawl-mcp"],
      "env": {
        "FIRECRAWL_API_KEY": "YOUR_API_KEY_HERE",

        "FIRECRAWL_RETRY_MAX_ATTEMPTS": "5",
        "FIRECRAWL_RETRY_INITIAL_DELAY": "2000",
        "FIRECRAWL_RETRY_MAX_DELAY": "30000",
        "FIRECRAWL_RETRY_BACKOFF_FACTOR": "3",

        "FIRECRAWL_CREDIT_WARNING_THRESHOLD": "2000",
        "FIRECRAWL_CREDIT_CRITICAL_THRESHOLD": "500"
      }
    }
  }
}
```

크롤러 대체 서버
https://github.com/devflowinc/trieve/tree/main/clients/mcp-server

슬랙
https://github.com/modelcontextprotocol/servers/tree/main/src/slack

```text
{
  "mcpServers": {
    "slack": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-slack"
      ],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-your-bot-token",
        "SLACK_TEAM_ID": "T01234567"
      }
    }
  }
}
```