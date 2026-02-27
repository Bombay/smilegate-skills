# BISKIT 설정 가이드

BISKIT MCP 연결을 위한 토큰 발급, 설정, 테스트 방법을 안내한다.

> ⚠️ BISKIT MCP는 **사내망 또는 VPN 연결이 필요**합니다. 연결 상태를 먼저 확인해주세요.

## BISKIT PAT 토큰 발급

안내 사항을 코드 블록 없이 마크다운으로 출력한다. URL은 반드시 마크다운 링크 형식으로 표시한다:

① 아래 링크를 브라우저에서 열어주세요:
   👉 [BISKIT 토큰 발급 페이지](https://biskit-sync.smilegate.net?openTokenDialog)

② 우측 상단 **프로필 아이콘** → **토큰 관리** → **새 토큰 생성** 클릭

③ 토큰 이름 입력 (예: "Claude Code 연동용") → **생성** 클릭

④ 토큰 만료 설정이 나오면 **만료 없음**으로 설정하는 것을 추천합니다 (만료를 설정하면 이후 토큰을 재발급해야 합니다)

⑤ ⚠️ 생성된 토큰을 반드시 복사하세요! (다시 볼 수 없습니다)

## 토큰 입력

AskUserQuestion으로 토큰을 입력받는다:
- question: "발급받은 BISKIT PAT 토큰을 붙여넣어 주세요"
- options: [
    {label: "발급하고 복사했어요", description: "3번 입력란에 바로 붙여넣어주세요"},
    {label: "발급 절차를 다시 안내해주세요", description: "토큰 발급 방법을 처음부터 안내합니다"}
  ]
- 사용자가 Other(3번)를 선택하여 텍스트를 입력하면, 입력값을 토큰으로 처리한다.
- "발급하고 복사했어요" 선택 시: "토큰을 붙여넣고 엔터를 눌러주세요"라고 안내한다. 다음 사용자 메시지를 토큰으로 처리한다.
- "발급 절차를 다시 안내해주세요" 선택 시: 위의 BISKIT 토큰 발급 절차를 스텝별로 다시 안내하고, 완료 후 다시 토큰 입력을 요청한다.

## MCP 설정

`~/.claude.json`의 `mcpServers`에 추가할 JSON:

```json
"biskit-report-mcp": {
  "type": "http",
  "url": "https://mcp.sginfra.net/biskit-report-mcp",
  "headers": {
    "Authorization": "Bearer {PAT토큰}"
  }
}
```

## 연결 테스트

- ToolSearch로 `+biskit check_auth` 검색 후 `mcp__biskit-report-mcp__check_auth_status()` 호출
- 성공하면: "인증이 정상적으로 되어있습니다" + 프로필 정보 표시
- 실패하면: 트러블슈팅 안내

## 관련 URL

| 항목 | URL |
|------|-----|
| BISKIT MCP 서버 | `https://mcp.sginfra.net/biskit-report-mcp` |
| BISKIT 토큰 발급 | https://biskit-sync.smilegate.net?openTokenDialog |
| BISKIT MCP 가이드 | https://wiki.smilegate.net/pages/viewpage.action?pageId=642342187 |
| 공식 설치 가이드 | https://wiki.smilegate.net/pages/viewpage.action?pageId=589459355 |
