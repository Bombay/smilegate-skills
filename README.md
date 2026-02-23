# smilegate-skills

스마일게이트 업무 도구를 AI 코딩 에이전트에 연결하는 스킬 모음.

[Agent Skills](https://agentskills.io) 표준을 따르며, Claude Code / Codex CLI / Gemini CLI / Cursor / Windsurf 등에서 사용할 수 있습니다.

## 설치

```bash
# Claude Code
npx skills add hyuntkim/smilegate-skills --agent claude-code --yes

# Cursor
npx skills add hyuntkim/smilegate-skills --agent cursor --yes

# Windsurf
npx skills add hyuntkim/smilegate-skills --agent windsurf --yes
```

## 포함된 스킬

| 스킬 | 설명 | 트리거 |
|------|------|--------|
| connector | Slack, Jira, Confluence, BISKIT 연결 설정 | "커넥터 설정해줘", "jira 연결해줘" |

## 참고

- 사내망 또는 VPN 환경에서만 Jira/Confluence/BISKIT MCP가 동작합니다.
- 공식 설치 가이드: [Confluence 가이드 페이지](https://wiki.smilegate.net/pages/viewpage.action?pageId=589459355)
