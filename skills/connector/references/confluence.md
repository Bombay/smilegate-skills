# Confluence 설정 가이드

Confluence MCP 연결을 위한 토큰 발급, 설정, 테스트 방법을 안내한다.

> ⚠️ Confluence MCP는 **사내망 또는 VPN 연결이 필요**합니다. 연결 상태를 먼저 확인해주세요.

## Confluence PAT 토큰 발급

안내 사항을 코드 블록 없이 마크다운으로 출력한다. URL은 반드시 마크다운 링크 형식으로 표시한다:

① 아래 링크를 브라우저에서 열어주세요:
   👉 [Confluence 토큰 발급 페이지](https://wiki.smilegate.net/plugins/personalaccesstokens/usertokens.action)

② "토큰 만들기(Create token)" 버튼 클릭 → 이름 입력 (예: "Claude Code 연동용")

③ ⚠️ "만료 날짜" 옵션에서 **자동 만료를 해제**하는 것을 권장합니다 (체크 해제)

④ "만들기(Create)" 버튼 클릭 → ⚠️ 생성된 토큰을 반드시 복사하세요! (다시 볼 수 없습니다)

## 토큰 입력

AskUserQuestion으로 토큰을 입력받는다:
- question: "발급받은 Confluence PAT 토큰을 붙여넣어 주세요"
- options: [
    {label: "직접 입력하기", description: "복사한 Confluence 토큰을 붙여넣으세요"},
    {label: "아직 발급 안 했어요", description: "토큰 발급 절차를 함께 진행합니다"}
  ]
- "아직 발급 안 했어요" 선택 시: 위의 Confluence 토큰 발급 절차를 스텝별로 다시 안내하고, "어디까지 진행하셨나요?"라고 물어보며 함께 진행한다. 완료 후 다시 토큰 입력을 요청한다.

## MCP 설정

`~/.claude.json`의 `mcpServers`에 추가할 JSON:

```json
"confluence": {
  "type": "http",
  "url": "http://mcp.sginfra.net/confluence-wiki-mcp",
  "headers": {
    "x-confluence-wiki-username": "{사용자ID}",
    "x-confluence-wiki-token": "{PAT토큰}"
  }
}
```

## 연결 테스트

- `mcp__confluence__test_confluence_connection()` 호출
- 성공하면: 연결 정보 표시
- 실패하면: 트러블슈팅 안내

## 관련 URL

| 항목 | URL |
|------|-----|
| Confluence MCP 서버 | `http://mcp.sginfra.net/confluence-wiki-mcp` |
| Confluence 토큰 발급 | https://wiki.smilegate.net/plugins/personalaccesstokens/usertokens.action |
| Confluence 프로필 페이지 | https://wiki.smilegate.net/users/viewmyprofile.action |
| 공식 설치 가이드 | https://wiki.smilegate.net/pages/viewpage.action?pageId=589459355 |
