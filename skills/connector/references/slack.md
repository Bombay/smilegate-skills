# Slack 설정 가이드

Slack은 가장 쉽다. 브라우저에서 클릭 몇 번이면 끝.

## Slack 연결 방법

안내 사항을 코드 블록 없이 마크다운으로 출력한다:

**Step 1.** 아래 링크를 브라우저에서 열어주세요:
👉 [Slack Connectors 설정 페이지](https://claude.ai/settings/connectors)

**Step 2.** 왼쪽 메뉴에서 **"커넥터 둘러보기"**를 클릭하세요

**Step 3.** 목록에서 **"Slack"**을 찾아 클릭하세요

**Step 4.** **"허용"** 버튼을 클릭하세요

**Step 5.** Slack 페이지로 이동됩니다
- 워크스페이스 선택 화면이 나오면 **"Smilegate Internal"**을 선택하세요
- 다른 워크스페이스가 보이더라도 반드시 **Smilegate Internal**을 선택해야 사내 채널에 접근할 수 있습니다

**Step 6.** Slack 로그인 화면이 나오면 로그인하세요 (이미 로그인되어 있으면 자동으로 넘어갑니다)

**Step 7.** Slack에서 **"허용"**을 클릭하여 Claude의 접근을 승인하세요

**Step 8.** 완료! "연결됨"으로 표시되면 성공입니다

## 주의 사항

- Claude Code에서 로그인한 계정과 claude.ai 계정이 동일해야 한다
- 회사 Slack 워크스페이스 관리자가 앱 설치를 승인해야 할 수 있다

## 연결 확인

- ToolSearch로 slack 도구를 검색하여 사용 가능한지 확인
- `mcp__claude_ai_Slack__slack_search_channels(query="general")` 호출로 실제 테스트

## 트러블슈팅

Slack 연결에 문제가 있을 때 확인할 것:
1. Claude Code에서 로그인한 계정과 claude.ai 계정이 동일한지 확인
2. 회사 Slack 워크스페이스 관리자가 앱 설치를 승인했는지 확인
3. 브라우저에서 [Slack Connectors 설정 페이지](https://claude.ai/settings/connectors)에 Slack이 "연결됨"으로 표시되는지 확인
4. Claude Code를 재시작했는지 확인

트러블슈팅이 진전 없이 막히면 → [트러블슈팅 및 이슈 등록](troubleshoot.md) 절차에 따라 GitHub 이슈 등록을 제안한다.

## 완료 확인

AskUserQuestion으로 완료 여부를 확인한다:
- question: "Slack 연결이 완료되셨나요?"
- options: [
    {label: "네, 완료했어요", description: "다음 단계로 진행합니다"},
    {label: "아니요, 도움이 필요해요", description: "문제 해결을 함께 진행합니다"}
  ]
- "아니요, 도움이 필요해요" 선택 시: 어디서 막혔는지 물어보고 위의 트러블슈팅 항목을 함께 확인한다. 해결되면 다시 완료 여부를 확인한다.

## 관련 URL

| 항목 | URL |
|------|-----|
| Slack Connectors 설정 | https://claude.ai/settings/connectors |
| 공식 설치 가이드 | https://wiki.smilegate.net/pages/viewpage.action?pageId=589459355 |
