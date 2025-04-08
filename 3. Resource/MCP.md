참고. 
- [유튜브 영상](https://www.youtube.com/watch?v=fkqXQOjj8cA)

이력
- 20250407 크롤러, 슬랙 연동 완료


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
- https://github.com/modelcontextprotocol/servers/tree/main/src/slack
- create a slack apps 
- bot token scopes configuration, Navigate to "OAuth & Permissions" and add these scopes:
	- `channels:history` - View messages and other content in public channels
	- `channels:read` - View basic channel information
	- `chat:write` - Send messages as the app
	- `reactions:write` - Add emoji reactions to messages
	- `users:read` - View users and their basic information
- `Settings > Install App`메뉴 에서 `Install to Claude`
	-  slack bot token 을 복사해서 아래 설정 파일에 추가해준다
	-  slack team id는 주소창에 T로 시작하는 정보를 복사한다
- 슬랙에 `ai 뉴스` 채널 생성 후 app을 추가해준다
	- `/invite @Claude` 실행하여 추가 가능

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

