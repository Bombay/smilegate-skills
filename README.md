# smilegate-skills

스마일게이트 업무 도구를 Claude Code에 연결하는 플러그인.

## 설치

### 1. 마켓플레이스 등록 (최초 1회)

```bash
claude plugin marketplace add Bombay/smilegate-skills
```

### 2. 플러그인 설치

```bash
claude plugin install smilegate-skills@smilegate-marketplace
```

> Claude Code를 재시작하면 플러그인이 활성화됩니다.

### 레거시 설치 (npx)

```bash
npx skills add Bombay/smilegate-skills --agent claude-code -g --yes
```

## 포함된 스킬

| 스킬 | 설명 | 트리거 |
|------|------|--------|
| connector | Slack, Jira, Confluence, BISKIT, API Docs, GitLab 연결 설정 | "커넥터 설정해줘", "jira 연결해줘" |
| automation | 반복 업무를 대화로 자동화 스킬 생성 | "자동화 만들기", "업무 자동화" |

## 참고

- 사내망 또는 VPN 환경에서만 Jira/Confluence/BISKIT MCP가 동작합니다.
- 공식 설치 가이드: [Confluence 가이드 페이지](https://wiki.smilegate.net/pages/viewpage.action?pageId=589459355)
