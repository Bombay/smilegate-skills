# API Docs 설정 가이드

API Docs는 인증이 필요 없어 가장 간단하게 설치할 수 있다.

## 설치 명령

아래 Bash 명령을 실행하여 API Docs MCP를 설치한다:

```bash
claude mcp add --transport http -s user apidocs http://mcp.sginfra.net/api-docs-mcp
```

## 설치 완료 안내

API Docs MCP가 설치되었습니다!

### 다른 MCP 서비스도 함께 설정하는 경우 (혼합 설치)

⚠️ 다른 MCP 서비스(Jira/Confluence/BISKIT)도 함께 설정하는 경우:
- API Docs 설치만으로는 재시작하지 않고, 나머지 서비스 설정이 모두 끝난 후 **한 번만 재시작**합니다.
- 이 안내 후 Phase 3으로 이동하여 나머지 MCP 서비스를 설정한다.

### API Docs만 설치하는 경우 (단독 설치)

다른 MCP 서비스 설정이 없는 경우 (API Docs만 설치):
- **Claude Code를 재시작**해야 API Docs 도구를 사용할 수 있습니다.
- 재시작 방법: `Ctrl+D` → 터미널에서 `claude` 실행
- 재시작 후 `/resume` 입력으로 대화를 이어갈 수 있습니다.

## 연결 테스트

API Docs만 설치한 경우 재시작 후 연결 테스트:
- ToolSearch로 `+apidocs search` 검색
- `mcp__apidocs__search_api_spec(query="결제 API")` 호출로 실제 테스트
- 성공하면: 검색 결과 표시 + Phase 5로 이동
- 실패하면: 아래 트러블슈팅 항목 확인 → 진전 없으면 [트러블슈팅 및 이슈 등록](troubleshoot.md) 절차 진행

## 트러블슈팅

연결 실패 시 확인할 것:
1. Claude Code를 재시작했는지 확인
2. `claude mcp add` 명령이 정상 실행되었는지 확인 (에러 메시지가 없었는지)
3. `~/.claude.json`(Mac/Linux) 또는 `%USERPROFILE%\.claude.json`(Windows)에 `apidocs` 설정이 반영되었는지 확인
4. 네트워크에서 mcp.sginfra.net 접근이 가능한지 확인 (사내망 또는 VPN 필요)

## 관련 URL

| 항목 | URL |
|------|-----|
| API Docs MCP 서버 | `http://mcp.sginfra.net/api-docs-mcp` |
| API Docs MCP 가이드 | https://wiki.smilegate.net/pages/viewpage.action?pageId=606711534 |
| 공식 설치 가이드 | https://wiki.smilegate.net/pages/viewpage.action?pageId=589459355 |
