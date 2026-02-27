# Jira 설정 가이드

Jira MCP 연결을 위한 토큰 발급, 설정, 테스트 방법을 안내한다.

> ⚠️ Jira MCP는 **사내망 또는 VPN 연결이 필요**합니다. 연결 상태를 먼저 확인해주세요.

## Jira PAT 토큰 발급

안내 사항을 코드 블록 없이 마크다운으로 출력한다. URL은 반드시 마크다운 링크 형식으로 표시한다:

① 아래 링크를 브라우저에서 열어주세요:
   👉 [Jira 토큰 발급 페이지](https://jira.smilegate.net/secure/ViewProfile.jspa?selectedTab=com.atlassian.pats.pats-plugin:jira-user-personal-access-tokens)

② "Create token" 버튼 클릭 → 이름 입력 (예: "Claude Code 연동용")

③ ⚠️ "만료 날짜" 옵션에서 **자동 만료를 해제**하는 것을 권장합니다 (체크 해제)

④ "Create" 버튼 클릭 → ⚠️ 생성된 토큰을 반드시 복사하세요! (다시 볼 수 없습니다)

## 토큰 입력

AskUserQuestion으로 토큰을 입력받는다:
- question: "발급받은 Jira PAT 토큰을 붙여넣어 주세요"
- options: [
    {label: "발급하고 복사했어요", description: "3번 입력란에 바로 붙여넣어주세요"},
    {label: "발급 절차를 다시 안내해주세요", description: "토큰 발급 방법을 처음부터 안내합니다"}
  ]
- 사용자가 Other(3번)를 선택하여 텍스트를 입력하면, 입력값을 토큰으로 처리한다.
- "발급하고 복사했어요" 선택 시: "토큰을 붙여넣고 엔터를 눌러주세요"라고 안내한다. 다음 사용자 메시지를 토큰으로 처리한다.
- "발급 절차를 다시 안내해주세요" 선택 시: 위의 Jira 토큰 발급 절차를 스텝별로 다시 안내하고, 완료 후 다시 토큰 입력을 요청한다.

## MCP 설정

`~/.claude.json`의 `mcpServers`에 추가할 JSON:

```json
"jira": {
  "type": "http",
  "url": "http://mcp.sginfra.net/confluence-jira-mcp",
  "headers": {
    "x-confluence-jira-username": "{사용자ID}",
    "x-confluence-jira-token": "{PAT토큰}"
  }
}
```

## 연결 테스트

- `mcp__jira__test_jira_connection()` 호출
- 성공하면: 사용자 이름, Jira URL 등 연결 정보 표시
- 실패하면: 에러 메시지와 함께 트러블슈팅 안내

## 관련 URL

| 항목 | URL |
|------|-----|
| Jira MCP 서버 | `http://mcp.sginfra.net/confluence-jira-mcp` |
| Jira 토큰 발급 | https://jira.smilegate.net/secure/ViewProfile.jspa?selectedTab=com.atlassian.pats.pats-plugin:jira-user-personal-access-tokens |
| Jira 프로필 페이지 | https://jira.smilegate.net/secure/ViewProfile.jspa |
| 공식 설치 가이드 | https://wiki.smilegate.net/pages/viewpage.action?pageId=589459355 |
