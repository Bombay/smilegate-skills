# smilegate-skills

스마일게이트 업무 도구를 AI 코딩 에이전트에 연결하는 스킬 모음.

[Agent Skills](https://agentskills.io) 표준을 따르며, Claude Code / Codex CLI / Gemini CLI / Cursor / Windsurf 등에서 사용할 수 있습니다.

## 설치

`-g` 플래그로 전역 설치하면 모든 프로젝트에서 스킬을 사용할 수 있습니다.

```bash
# Claude Code (전역)
npx skills add Bombay/smilegate-skills --agent claude-code -g --yes
```

```bash
# Cursor (전역)
npx skills add Bombay/smilegate-skills --agent cursor -g --yes
```

```bash
# Windsurf (전역)
npx skills add Bombay/smilegate-skills --agent windsurf -g --yes
```

> 프로젝트 단위로 설치하려면 `-g` 플래그를 제거하세요.

## 포함된 스킬

| 스킬 | 설명 | 트리거 |
|------|------|--------|
| connector | Slack, Jira, Confluence, BISKIT 연결 설정 | "커넥터 설정해줘", "jira 연결해줘" |

## 참고

- 사내망 또는 VPN 환경에서만 Jira/Confluence/BISKIT MCP가 동작합니다.
- 공식 설치 가이드: [Confluence 가이드 페이지](https://wiki.smilegate.net/pages/viewpage.action?pageId=589459355)
