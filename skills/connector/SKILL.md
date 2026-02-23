---
name: connector
description: 스마일게이트 업무 도구(Slack, Jira, Confluence, BISKIT)를 Claude Code에 연결하는 설정 가이드. 비개발자도 따라할 수 있도록 단계별로 안내한다. "커넥터", "connector", "MCP 설정", "jira 연결", "confluence 연결", "slack 연결", "biskit 연결", "비스킷 연결" 요청에 사용.
triggers:
  - "커넥터"
  - "connector"
  - "커넥터 설정"
  - "MCP 설정"
  - "jira 연결"
  - "confluence 연결"
  - "slack 연결"
  - "biskit 연결"
  - "비스킷 연결"
  - "biskit mcp"
  - "스마일게이트 커넥터"
---

# Smilegate Connector

스마일게이트 업무 도구를 Claude Code에 연결하는 설정 스킬.
비개발자도 따라할 수 있도록 단계별로 안내한다.

## 연결 대상

| 서비스 | 연결 방식 | 난이도 |
|--------|----------|--------|
| Slack | Connectors (클릭만) | 쉬움 |
| Jira | MCP (토큰 발급 필요) | 보통 |
| Confluence | MCP (토큰 발급 필요) | 보통 |
| BISKIT | MCP (토큰 발급 필요) | 보통 |

## 실행 흐름

이 스킬이 트리거되면 아래 순서로 진행한다.

**링크 출력 규칙**: 모든 URL은 코드 블록 밖에서 마크다운 링크 형식 `[텍스트](URL)` 으로 표시한다. 코드 블록 안에 URL을 절대 넣지 않는다. 코드 블록 안의 URL은 클릭이 불가능하고 줄바꿈이 발생할 수 있다. 단, JSON 설정 예시의 URL 값은 설정값이므로 코드 블록 안에 표기한다.

### Phase 0: 현재 연결 상태 진단

스킬 시작 시 먼저 현재 연결 상태를 진단한다.

확인 방법:
1. ToolSearch로 `+slack read` 검색 → Slack 도구 존재 여부 확인
2. ToolSearch로 `+jira test` 검색 → Jira MCP 존재 여부 확인. 도구가 있으면 `mcp__jira__test_jira_connection()` 호출로 실제 연결 확인
3. ToolSearch로 `+confluence test` 검색 → Confluence MCP 존재 여부 확인. 도구가 있으면 `mcp__confluence__test_confluence_connection()` 호출로 실제 연결 확인
4. ToolSearch로 `+biskit check_auth` 검색 → BISKIT MCP 존재 여부 확인. 도구가 있으면 `mcp__biskit-report-mcp__check_auth_status()` 호출로 실제 연결 확인

진단 결과를 테이블로 보여준다:

| 서비스 | 상태 |
|--------|------|
| Slack | ✅ 연결됨 / ❌ 미연결 |
| Jira | ✅ 연결됨 / ⚠️ 재연결 필요 (토큰 만료 등) / ❌ 미연결 |
| Confluence | ✅ 연결됨 / ⚠️ 재연결 필요 / ❌ 미연결 |
| BISKIT | ✅ 연결됨 / ⚠️ 재연결 필요 / ❌ 미연결 |

- ✅ 연결됨: 도구 존재 + 연결 테스트 성공
- ⚠️ 재연결 필요: 도구는 존재하지만 연결 테스트 실패 (토큰 만료, 토큰 오류 등)
- ❌ 미연결: 도구 자체가 없음 (MCP 설정 없음)

이미 연결된 서비스(✅)는 건너뛴다. ⚠️ 또는 ❌ 상태의 서비스만 설정을 진행한다.
모두 ✅이면 "모든 서비스가 연결되어 있습니다!"를 출력하고 기본 사용법을 안내한 뒤 종료한다.

설정이 필요한 서비스(⚠️/❌)가 있으면 AskUserQuestion으로 설정할 서비스를 선택받는다:
- question: "어떤 서비스를 설정할까요? (여러 개 선택 가능)"
- options: ⚠️/❌ 상태의 서비스만 동적으로 표시
  - {label: "Slack", description: "Slack 채널 읽기/검색 (클릭 몇 번이면 끝)"}
  - {label: "Jira", description: "Jira 이슈 조회/생성/관리 (토큰 발급 필요)"}
  - {label: "Jira ⚠️ 재연결", description: "토큰 만료 또는 오류 — 토큰을 다시 발급받아 연결합니다"}
  - {label: "Confluence", description: "Wiki 페이지 조회/검색 (토큰 발급 필요)"}
  - {label: "Confluence ⚠️ 재연결", description: "토큰 만료 또는 오류 — 토큰을 다시 발급받아 연결합니다"}
  - {label: "BISKIT", description: "게임 데이터 리포트 조회 (토큰 발급 필요)"}
  - {label: "BISKIT ⚠️ 재연결", description: "토큰 만료 또는 오류 — 토큰을 다시 발급받아 연결합니다"}
- multiSelect: true
- 해당 상태의 서비스만 옵션에 표시한다 (예: Jira가 ❌이면 "Jira"만, ⚠️이면 "Jira ⚠️ 재연결"만)

#### 분기 흐름

선택 결과에 따라 아래와 같이 진행한다:
- **Slack만 선택** → Phase 1 → Phase 4 (Phase 2 건너뜀)
- **MCP 서비스만 선택** (Jira/Confluence/BISKIT) → Phase 2 → Phase 4 (Phase 1 건너뜀)
- **Slack + MCP 서비스 선택** → Phase 1 → Phase 2 → Phase 4

### Phase 1: Slack 연결 (Connectors)

Slack은 가장 쉽다. 브라우저에서 클릭 몇 번이면 끝.

안내 사항을 코드 블록 없이 마크다운으로 출력한다:

Slack 연결 방법:

① 아래 링크를 브라우저에서 열어주세요:
   👉 [Slack Connectors 설정 페이지](https://claude.ai/settings/connectors)

② 왼쪽 메뉴에서 "커넥터 둘러보기"를 클릭하세요

③ 목록에서 "Slack"을 찾아 클릭하세요

④ Slack 로그인 화면이 나오면 로그인하세요 (이미 로그인되어 있으면 자동으로 넘어갑니다)

⑤ "허용" 버튼을 클릭하세요

⑥ 끝! 이제 Claude Code에서 Slack을 사용할 수 있습니다.

주의 사항:
- Claude Code에서 로그인한 계정과 claude.ai 계정이 동일해야 한다
- 회사 Slack 워크스페이스 관리자가 앱 설치를 승인해야 할 수 있다

연결 확인:
- ToolSearch로 slack 도구를 검색하여 사용 가능한지 확인
- `mcp__claude_ai_Slack__slack_search_channels(query="general")` 호출로 실제 테스트

AskUserQuestion으로 완료 여부를 확인한다:
- question: "Slack 연결이 완료되셨나요?"
- options: [
    {label: "네, 완료했어요", description: "다음 단계로 진행합니다"},
    {label: "아니요, 도움이 필요해요", description: "문제 해결을 함께 진행합니다"}
  ]
- "아니요, 도움이 필요해요" 선택 시: 어디서 막혔는지 물어보고 함께 해결한 뒤 다시 완료 여부를 확인한다.

완료 확인 후:
- MCP 서비스(Jira/Confluence/BISKIT)를 선택했으면 Phase 2로 이동한다.
- MCP 서비스를 선택하지 않았으면 Phase 2를 건너뛰고 Phase 4로 이동한다.

### Phase 2: MCP 연결 (Jira / Confluence / BISKIT)

> ⚠️ Jira, Confluence, BISKIT MCP는 **사내망 또는 VPN 연결이 필요**합니다. 연결 상태를 먼저 확인해주세요.

**핵심 원칙**: 선택한 MCP 서비스를 모두 한 번에 설정하고, **재시작은 딱 한 번**만 한다.
토큰 발급 → 전체 입력 → 설정 파일 일괄 저장 → 재시작 → 연결 테스트 순서로 진행한다.

#### Step 1: 선택한 서비스의 토큰 발급 안내 (모두 출력)

선택한 서비스에 해당하는 토큰 발급 안내를 **한 번에 모두** 출력한다.
사용자가 브라우저에서 여러 탭을 열어 한꺼번에 발급받을 수 있도록 안내한다.

아래 내용을 코드 블록 없이 마크다운으로 출력한다. URL은 반드시 마크다운 링크 형식으로 표시한다:

##### Jira 토큰 발급 (Jira 선택 시)

Jira PAT 토큰 발급 방법:

① 아래 링크를 브라우저에서 열어주세요:
   👉 [Jira 토큰 발급 페이지](https://jira.smilegate.net/secure/ViewProfile.jspa?selectedTab=com.atlassian.pats.pats-plugin:jira-user-personal-access-tokens)

② "Create token" 버튼 클릭 → 이름 입력 (예: "Claude Code 연동용")

③ ⚠️ "만료 날짜" 옵션에서 **자동 만료를 해제**하는 것을 권장합니다 (체크 해제)

④ "Create" 버튼 클릭 → ⚠️ 생성된 토큰을 반드시 복사하세요! (다시 볼 수 없습니다)

##### Confluence 토큰 발급 (Confluence 선택 시)

Confluence PAT 토큰 발급 방법:

① 아래 링크를 브라우저에서 열어주세요:
   👉 [Confluence 토큰 발급 페이지](https://wiki.smilegate.net/plugins/personalaccesstokens/usertokens.action)

② "토큰 만들기(Create token)" 버튼 클릭 → 이름 입력 (예: "Claude Code 연동용")

③ ⚠️ "만료 날짜" 옵션에서 **자동 만료를 해제**하는 것을 권장합니다 (체크 해제)

④ "만들기(Create)" 버튼 클릭 → ⚠️ 생성된 토큰을 반드시 복사하세요! (다시 볼 수 없습니다)

##### BISKIT 토큰 발급 (BISKIT 선택 시)

BISKIT PAT 토큰 발급 방법:

① 아래 링크를 브라우저에서 열어주세요:
   👉 [BISKIT 토큰 발급 페이지](https://biskit-sync.smilegate.net?openTokenDialog)

② 우측 상단 **프로필 아이콘** → **토큰 관리** → **새 토큰 생성** 클릭

③ 토큰 이름 입력 (예: "Claude Code 연동용") → **생성** 클릭

④ BISKIT 토큰은 별도의 만료 설정이 없습니다 (Jira/Confluence와 달리 자동 만료 해제 불필요)

⑤ ⚠️ 생성된 토큰을 반드시 복사하세요! (다시 볼 수 없습니다)

#### Step 2: 토큰 전체 입력 (선택한 서비스 모두)

토큰 발급 안내 직후, AskUserQuestion으로 필요한 정보를 순서대로 입력받는다.
**재시작 전에 선택한 모든 서비스의 토큰을 한 번에 입력받는다.**

- 사용자 ID 질문 (Jira/Confluence 중 하나라도 선택한 경우):
  - question: "사용자 ID를 입력해주세요 (Jira/Confluence 공통, 예: hyuntkim)"
  - options: [
      {label: "직접 입력하기", description: "사용자 ID를 입력하세요"},
      {label: "잘 모르겠어요", description: "Jira 또는 Confluence에 로그인할 때 사용하는 ID입니다"}
    ]
  - "잘 모르겠어요" 선택 시: [Jira 프로필 페이지](https://jira.smilegate.net/secure/ViewProfile.jspa) 또는 [Confluence 프로필 페이지](https://wiki.smilegate.net/users/viewmyprofile.action)에서 사용자 이름을 확인할 수 있다고 안내한 뒤, 다시 입력을 요청한다.

- Jira PAT 토큰 질문 (Jira 선택 시):
  - question: "발급받은 Jira PAT 토큰을 붙여넣어 주세요"
  - options: [
      {label: "직접 입력하기", description: "복사한 Jira 토큰을 붙여넣으세요"},
      {label: "아직 발급 안 했어요", description: "토큰 발급 절차를 함께 진행합니다"}
    ]
  - "아직 발급 안 했어요" 선택 시: 위의 Jira 토큰 발급 절차를 스텝별로 다시 안내하고, "어디까지 진행하셨나요?"라고 물어보며 함께 진행한다. 완료 후 다시 토큰 입력을 요청한다.

- Confluence PAT 토큰 질문 (Confluence 선택 시):
  - question: "발급받은 Confluence PAT 토큰을 붙여넣어 주세요"
  - options: [
      {label: "직접 입력하기", description: "복사한 Confluence 토큰을 붙여넣으세요"},
      {label: "아직 발급 안 했어요", description: "토큰 발급 절차를 함께 진행합니다"}
    ]
  - "아직 발급 안 했어요" 선택 시: 위의 Confluence 토큰 발급 절차를 스텝별로 다시 안내하고, "어디까지 진행하셨나요?"라고 물어보며 함께 진행한다. 완료 후 다시 토큰 입력을 요청한다.

- BISKIT PAT 토큰 질문 (BISKIT 선택 시):
  - question: "발급받은 BISKIT PAT 토큰을 붙여넣어 주세요"
  - options: [
      {label: "직접 입력하기", description: "복사한 BISKIT 토큰을 붙여넣으세요"},
      {label: "아직 발급 안 했어요", description: "토큰 발급 절차를 함께 진행합니다"}
    ]
  - "아직 발급 안 했어요" 선택 시: 위의 BISKIT 토큰 발급 절차를 스텝별로 다시 안내하고, "어디까지 진행하셨나요?"라고 물어보며 함께 진행한다. 완료 후 다시 토큰 입력을 요청한다.

**중요**: 토큰은 민감정보이므로 대화 내용에 그대로 노출하지 않는다.
입력받은 토큰 값의 앞뒤 공백과 줄바꿈을 제거(trim)한 뒤 설정 파일에 저장한다.
대화에서는 마스킹하여 표시한다.

#### Step 3: 설정 파일 일괄 업데이트

OS에 따라 설정 파일 위치가 다르다:

| OS | 설정 파일 경로 |
|----|--------------|
| Mac / Linux | `~/.claude.json` |
| Windows | `%USERPROFILE%\.claude.json` (예: `C:\Users\hyuntkim\.claude.json`) |

Read 도구로 설정 파일을 읽고, `mcpServers` 객체에 **선택한 서비스를 한번에** 추가한다:

Jira 선택 시:
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

Confluence 선택 시:
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

BISKIT 선택 시:
```json
"biskit-report-mcp": {
  "type": "http",
  "url": "https://mcp.sginfra.net/biskit-report-mcp",
  "headers": {
    "Authorization": "Bearer {PAT토큰}"
  }
}
```

추가 후 사용자에게 아래 내용을 코드 블록 없이 출력한다:

설정 파일의 mcpServers에 {설정한 서비스 목록} 설정이 모두 추가되었습니다!

이제 **Claude Code를 한 번 재시작**하면 모든 서비스가 동시에 연결됩니다:

Claude Code를 재시작하는 방법:

- **Mac / Linux**: `Ctrl+D` → 터미널에서 `claude` 실행
- **Windows**: `exit` 입력 → 터미널에서 `claude` 실행

재시작 후 `/resume` 을 입력하면 **직전 대화를 이어서** 진행할 수 있습니다.

#### Step 4: 재시작 후 연결 테스트 (선택한 서비스 모두)

사용자가 재시작을 완료했다고 (또는 `/resume`으로 돌아왔다고) 알려주면, 선택한 서비스를 **모두** 테스트한다.

Jira 테스트 (Jira 선택 시):
- `mcp__jira__test_jira_connection()` 호출
- 성공하면: 사용자 이름, Jira URL 등 연결 정보 표시
- 실패하면: 에러 메시지와 함께 트러블슈팅 안내

Confluence 테스트 (Confluence 선택 시):
- `mcp__confluence__test_confluence_connection()` 호출
- 성공하면: 연결 정보 표시
- 실패하면: 트러블슈팅 안내

BISKIT 테스트 (BISKIT 선택 시):
- ToolSearch로 `+biskit check_auth` 검색 후 `mcp__biskit-report-mcp__check_auth_status()` 호출
- 성공하면: "인증이 정상적으로 되어있습니다" + 프로필 정보 표시
- 실패하면: 트러블슈팅 안내

트러블슈팅:
```
연결 실패 시 확인할 것:
1. Claude Code를 재시작했는지 (설정 변경 후 재시작 필수)
2. 토큰이 올바르게 복사되었는지 (설정 파일에서 mcpServers 직접 확인)
3. 사용자 ID가 맞는지 (Jira/Confluence 프로필에서 확인)
4. BISKIT 토큰 앞에 "Bearer "가 포함되어 있는지 (공백 1개 필수)
5. 네트워크에서 mcp.sginfra.net 접근이 가능한지 (사내망 또는 VPN 필요)
6. Windows의 경우 %USERPROFILE%\.claude.json 경로가 맞는지 확인
```

### Phase 3: 미해결 이슈 등록

스킬 진행 중 트러블슈팅이 **진전 없이 막혀있다고 판단**되면, GitHub 이슈 등록을 제안한다.

#### 트리거 조건

단순 핑퐁 횟수가 아니라, **실제로 진전이 없는 상황**에서만 제안한다.

다음 조건 중 하나 이상을 만족해야 이슈 등록을 제안한다:
- **동일 에러 반복**: 같은 에러 메시지로 연결 테스트가 3회 이상 실패한 경우 (다른 조치를 시도했는데도 결과가 동일)
- **트러블슈팅 항목 소진**: 트러블슈팅 체크리스트(재시작, 토큰 확인, ID 확인, Bearer 형식, 네트워크 등)를 모두 확인했는데도 해결되지 않은 경우
- **사용자가 포기 의사 표현**: "안 되겠다", "모르겠다", "포기" 등 해결을 중단하려는 의사를 표현한 경우

다음 상황에서는 제안하지 **않는다**:
- 사용자가 새로운 시도를 하고 있는 중 (에러 메시지가 바뀌거나, 다른 단계를 진행 중)
- 토큰 재발급, 설정 수정 등 아직 시도하지 않은 조치가 남아있는 경우

#### 이슈 등록 제안

AskUserQuestion으로 제안한다:
- question: "트러블슈팅을 시도했지만 문제가 계속되고 있습니다. GitHub에 이슈를 등록해서 추적할까요?"
- options: [
    {label: "네, 이슈 등록해주세요", description: "현재 상황과 환경 정보를 포함한 이슈를 자동으로 생성합니다"},
    {label: "아니요, 나중에 할게요", description: "이슈 등록 없이 진행합니다"}
  ]

#### 이슈 생성

"네, 이슈 등록해주세요" 선택 시, `gh issue create` 명령으로 이슈를 생성한다.

이슈 저장소: `Bombay/smilegate-skills`

이슈 제목 형식:
```
[Connector] {서비스명} 연결 실패 — {에러 요약}
```

이슈 본문에 포함할 정보:

```markdown
## 증상
{어떤 서비스에서 어떤 문제가 발생했는지 1-2문장으로 요약}

## 에러 메시지
{연결 테스트 또는 설정 과정에서 발생한 실제 에러 메시지}

## 재현 과정
1. `/connector` 스킬 실행
2. {서비스명} 선택
3. 토큰 발급 및 입력 완료
4. Claude Code 재시작 후 연결 테스트
5. {어떤 에러가 발생했는지}

## 시도한 트러블슈팅
{사용자와 함께 시도한 해결 방법 목록}

## 환경 정보
- **OS**: {darwin/linux/windows + 버전}
- **Claude Code 버전**: {Bash로 `claude --version` 실행하여 확인}
- **네트워크**: {사내망/VPN/외부망 — 대화에서 파악된 정보}
- **설정 파일 경로**: {~/.claude.json 등}

## MCP 설정 (토큰 마스킹됨)
{해당 서비스의 mcpServers 설정을 출력하되, 토큰 값은 앞 6자만 남기고 `***`로 마스킹}

## 추가 컨텍스트
{대화 중 파악된 추가 정보가 있으면 기재}
```

**중요**:
- 이슈 본문에 토큰 전체를 절대 노출하지 않는다. 앞 6자 + `***` 형태로 마스킹한다.
- Bash로 `claude --version` 을 실행하여 Claude Code 버전을 수집한다.
- Bash로 `uname -a` 를 실행하여 OS 정보를 수집한다.
- `gh issue create` 명령 실행 전 수집한 정보를 사용자에게 보여주고 확인받는다.
- labels로 `bug`와 `connector`를 추가한다.

이슈 생성 명령:
```bash
gh issue create --repo Bombay/smilegate-skills \
  --title "{제목}" \
  --body "$(cat <<'EOF'
{본문}
EOF
)" \
  --label "bug,connector"
```

이슈 생성 후 URL을 사용자에게 보여준다:
"이슈가 등록되었습니다: {이슈 URL}"

이후 Phase 4로 진행한다. (이슈 등록 여부와 관계없이 완료 리포트는 출력)

### Phase 4: 완료 리포트

모든 설정이 끝나면 최종 상태를 요약한다.

**실제 설정한 서비스만** 표시하고, 연결 테스트 결과에 따라 ✅(성공)/❌(실패)를 표시한다.

출력 형식 (코드 블록 없이 마크다운으로):

**스마일게이트 커넥터 설정 완료!**

연결 상태: (설정한 서비스만 표시)
- {서비스명}: ✅ 연결됨 또는 ❌ 연결 실패

기본 사용법: (연결 성공한 서비스만 표시)

서비스별 예시 문구:
- **Slack**: "Slack에서 #general 채널 최근 메시지 보여줘", "Slack에서 '배포' 관련 메시지 검색해줘"
- **Jira**: "나한테 할당된 Jira 이슈 보여줘", "PROJ-123 이슈 상태 알려줘"
- **Confluence**: "최근 업데이트된 Wiki 페이지 보여줘", "'프로젝트 계획' 관련 문서 검색해줘"
- **BISKIT**: "카제나 어제 DAU 알려줘", "에픽세븐 이번 주 매출은?"

연결 실패한 서비스가 있으면 트러블슈팅 안내를 함께 표시한다.

> 팁: 토큰이 만료되면 "커넥터 설정해줘" 한 마디로 이 스킬을 다시 실행할 수 있습니다.

## 참고 정보

| 항목 | 값 |
|------|-----|
| Jira MCP 서버 | `http://mcp.sginfra.net/confluence-jira-mcp` |
| Confluence MCP 서버 | `http://mcp.sginfra.net/confluence-wiki-mcp` |
| BISKIT MCP 서버 | `https://mcp.sginfra.net/biskit-report-mcp` |
| Jira 토큰 발급 | https://jira.smilegate.net/secure/ViewProfile.jspa?selectedTab=com.atlassian.pats.pats-plugin:jira-user-personal-access-tokens |
| Confluence 토큰 발급 | https://wiki.smilegate.net/plugins/personalaccesstokens/usertokens.action |
| BISKIT 토큰 발급 | https://biskit-sync.smilegate.net?openTokenDialog |
| Slack Connectors | https://claude.ai/settings/connectors |
| 공식 설치 가이드 | https://wiki.smilegate.net/pages/viewpage.action?pageId=589459355 |
| BISKIT MCP 가이드 | https://wiki.smilegate.net/pages/viewpage.action?pageId=642342187 |

## MCP 설정 위치

MCP는 항상 **전역 설정 파일**에 추가한다. OS마다 경로가 다르다:

| OS | 설정 파일 경로 |
|------|------|
| Mac / Linux | `~/.claude.json` |
| Windows | `%USERPROFILE%\.claude.json` (예: `C:\Users\hyuntkim\.claude.json`) |

> 전역 설정 파일에 저장하면 모든 프로젝트에서 사용할 수 있다.
> Git 저장소 밖에 있으므로 토큰이 실수로 커밋될 위험이 없다.
