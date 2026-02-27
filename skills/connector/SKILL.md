---
name: connector
description: 스마일게이트 업무 도구(Slack, Jira, Confluence, BISKIT, API Docs, GitLab, Amplitude)를 Claude Code에 연결하는 설정 가이드. 비개발자도 따라할 수 있도록 단계별로 안내한다. "커넥터", "connector", "커넥터 설정", "jira 연결", "confluence 연결", "slack 연결", "biskit 연결", "비스킷 연결", "apidocs 연결", "api docs 연결", "gitlab 연결", "깃랩 연결", "amplitude 연결", "앰플리튜드 연결" 요청에 사용.
triggers:
  - "커넥터"
  - "connector"
  - "커넥터 설정"
  - "jira 연결"
  - "confluence 연결"
  - "slack 연결"
  - "biskit 연결"
  - "비스킷 연결"
  - "apidocs 연결"
  - "api docs 연결"
  - "gitlab 연결"
  - "깃랩 연결"
  - "amplitude 연결"
  - "앰플리튜드 연결"
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
| API Docs | MCP (인증 불필요, 클릭만) | 쉬움 |
| GitLab | CLI 설치 (토큰 발급 필요) | 보통 |
| Amplitude | MCP (OAuth 인증, 클릭만) | 쉬움 |

## 실행 흐름

이 스킬이 트리거되면 **가장 먼저 state.json을 확인**하여 분기한다.

**링크 출력 규칙**: 모든 URL은 코드 블록 밖에서 마크다운 링크 형식 `[텍스트](URL)` 으로 표시한다. 코드 블록 안에 URL을 절대 넣지 않는다. 코드 블록 안의 URL은 클릭이 불가능하고 줄바꿈이 발생할 수 있다. 단, JSON 설정 예시의 URL 값은 설정값이므로 코드 블록 안에 표기한다.

### 상태 확인 및 분기

스킬이 시작되면 Read 도구로 `~/.claude/skills/smilegate-ai-tools/state.json` 파일을 읽는다.

**하위 호환**: `connector` 키에 `status` 필드가 없고 `completed: true`만 있는 경우, `status: "completed"`로 취급한다.

아래 3가지 케이스로 분기한다:

**케이스 A: 파일 없음 또는 `connector` 키 없음 → 신규 설치**

처음 방문한 사용자다. 아래 "진단" 단계부터 전체 흐름을 진행한다:

```
진단 → 서비스 선택 → 서비스별 설정(하나씩) → 재시작(필요 시) → 연결 테스트 → 완료 리포트 → 자동화 제안(첫 실행 시)
```

**케이스 B: `connector.status` = `"installing"` → 재시작 후 복귀**

이전에 설정을 완료하고 재시작을 안내받은 사용자가 돌아왔다. `connector.pending_services` 목록을 확인하여 연결 테스트를 진행한다.

1. pending_services에 있는 서비스들의 도구가 로드되었는지 ToolSearch로 확인한다.
2. **도구가 로드됨** → 바로 "연결 테스트" 단계로 이동. pending_services에 있는 서비스만 테스트한다.
3. **도구가 로드되지 않음** → 재시작이 아직 안 된 상태. 아래 메시지를 다시 출력한다:

   아직 재시작이 완료되지 않았어요!

   **1단계: Claude Code 종료**
   - Mac / Linux: `Ctrl+D` 누르기
   - Windows: `exit` 입력

   **2단계: 다시 시작**
   - 터미널에서 `claude` 입력

   **3단계: 아래 명령어를 그대로 입력**

   > /connector

**케이스 C: `connector.status` = `"completed"` → 재방문**

이미 설정을 완료한 사용자다. 아래 "진단" 단계부터 진행하되, state.json의 `connected_services`가 **1개 이상 있고** 그 서비스들이 **모두 ✅이면** 빠른 종료 분기를 탄다.

```
진단 → (connected_services 1개 이상 & 모두 ✅) → "기존 연결이 모두 정상입니다" + 추가 설정 여부 질문
     → (connected_services 없음 또는 ⚠️/❌ 있음) → 서비스 선택 → 서비스별 설정 → 재시작(필요 시) → 연결 테스트 → 완료 리포트
```

빠른 종료 조건 충족 시:
1. "기존 연결(Slack, Jira …)이 모두 정상입니다!"를 출력한다. (실제 서비스명 나열)
2. AskUserQuestion으로 "추가로 설정할 서비스가 있나요?"를 묻는다.
   - "네" → ❌ 미연결 서비스만 서비스 선택으로 이동
   - "아니요" → 종료

> `connected_services`가 없거나 빈 배열인 경우(레거시 형식 등)는 빠른 종료를 건너뛰고 일반 진단 흐름으로 진행한다.

> 재방문(케이스 C)에서는 자동화 스킬 제안을 건너뛴다.

---

### 진단: 현재 연결 상태 확인

> 케이스 B(installing)에서는 이 단계를 건너뛴다.

스킬 시작 시 먼저 현재 연결 상태를 진단한다.

확인 방법:
1. ToolSearch로 `+slack read` 검색 → Slack 도구 존재 여부 확인
2. ToolSearch로 `+jira test` 검색 → Jira MCP 존재 여부 확인. 도구가 있으면 `mcp__jira__test_jira_connection()` 호출로 실제 연결 확인
3. ToolSearch로 `+confluence test` 검색 → Confluence MCP 존재 여부 확인. 도구가 있으면 `mcp__confluence__test_confluence_connection()` 호출로 실제 연결 확인
4. ToolSearch로 `+biskit check_auth` 검색 → BISKIT MCP 존재 여부 확인. 도구가 있으면 `mcp__biskit-report-mcp__check_auth_status()` 호출로 실제 연결 확인
5. ToolSearch로 `+apidocs search` 검색 → API Docs MCP 존재 여부 확인. 도구가 있으면 연결됨 (인증 불필요이므로 도구 존재 = 연결 성공)
6. Bash로 `glab auth status --hostname git.sginfra.net 2>&1` 실행 → GitLab CLI 인증 상태 확인. 명령이 없으면 glab 미설치
7. ToolSearch로 `+amplitude` 검색 → Amplitude MCP 존재 여부 확인. 도구가 있으면 설정 완료 (OAuth는 사용 시 자동 처리이므로 도구 존재 = 추가 작업 불필요)

진단 결과를 테이블로 보여준다:

| 서비스 | 상태 |
|--------|------|
| Slack | ✅ 연결됨 / ❌ 미연결 |
| Jira | ✅ 연결됨 / ⚠️ 재연결 필요 / ❌ 미연결 |
| Confluence | ✅ 연결됨 / ⚠️ 재연결 필요 / ❌ 미연결 |
| BISKIT | ✅ 연결됨 / ⚠️ 재연결 필요 / ❌ 미연결 |
| API Docs | ✅ 연결됨 / ❌ 미연결 |
| GitLab | ✅ 연결됨 / ⚠️ 재연결 필요 / ❌ 미연결 |
| Amplitude | ✅ 연결됨 / ❌ 미연결 |

- ✅ 연결됨: 도구 존재 + 연결 테스트 성공 (GitLab은 glab 설치 + 인증 완료)
- ⚠️ 재연결 필요: 도구는 존재하지만 연결 테스트 실패 (토큰 만료 등). GitLab은 glab 설치됨 + 인증 실패
- ❌ 미연결: 도구 자체가 없음 (MCP 설정 없음). GitLab은 glab 미설치
- Slack, API Docs, Amplitude는 수동 토큰 관리가 불필요하므로 ✅(연결됨)과 ❌(미연결) 두 가지 상태만 존재한다.

이미 연결된 서비스(✅)는 건너뛴다. ⚠️ 또는 ❌ 상태의 서비스만 설정을 진행한다.
전체 7개 서비스가 모두 ✅이면 "모든 서비스가 연결되어 있습니다!"를 출력하고 기본 사용법을 안내한 뒤 종료한다.

> 케이스 C(재방문)에서는 위 조건 대신 `connected_services` 기준 빠른 종료를 먼저 확인한다. 케이스 C 설명 참조.

### 서비스 선택

설정이 필요한 서비스(⚠️/❌)가 있으면 먼저 추천 안내를 출력한다:

**💡 추천: 처음이시라면 Slack, Jira, Confluence, BISKIT 4가지를 먼저 설정하세요!** 가장 많이 사용되는 핵심 서비스입니다.

이후 AskUserQuestion으로 설정할 서비스를 선택받는다:
- question: "어떤 서비스를 설정할까요? (⬆⬇ 화살표로 이동, Space로 선택, Enter로 확인)"
- options: ⚠️/❌ 상태의 서비스만 동적으로 표시. 사용자가 아무것도 선택하지 않고 Enter만 누르거나 "Other"로 "건너뛰기"/"나중에" 등을 입력하면, "나중에 다시 `/connector`로 설정할 수 있어요!"를 안내하고 종료한다.
  - {label: "Slack", description: "Slack 채널 읽기/검색 (클릭 몇 번이면 끝)"}
  - {label: "Jira", description: "Jira 이슈 조회/생성/관리 (토큰 발급 필요)"}
  - {label: "Jira ⚠️ 재연결", description: "토큰 만료 또는 오류 — 토큰을 다시 발급받아 연결합니다"}
  - {label: "Confluence", description: "Wiki 페이지 조회/검색 (토큰 발급 필요)"}
  - {label: "Confluence ⚠️ 재연결", description: "토큰 만료 또는 오류 — 토큰을 다시 발급받아 연결합니다"}
  - {label: "BISKIT", description: "게임 데이터 리포트 조회 (토큰 발급 필요)"}
  - {label: "BISKIT ⚠️ 재연결", description: "토큰 만료 또는 오류 — 토큰을 다시 발급받아 연결합니다"}
  - {label: "API Docs", description: "SGP API 명세 검색 (인증 불필요, 바로 설치)"}
  - {label: "GitLab", description: "사내 GitLab CLI 설치 및 연결 (토큰 발급 필요)"}
  - {label: "GitLab ⚠️ 재연결", description: "토큰 만료 또는 오류 — 토큰을 다시 발급받아 연결합니다"}
  - {label: "Amplitude", description: "프로덕트 분석 (이벤트, 차트, 퍼널, 실험) — OAuth 인증, 바로 설치"}
- multiSelect: true
- 해당 상태의 서비스만 옵션에 표시한다 (예: Jira가 ❌이면 "Jira"만, ⚠️이면 "Jira ⚠️ 재연결"만)

### 서비스별 설정

선택한 서비스를 아래 순서대로 **하나씩** 설정한다. 각 서비스의 상세 절차는 해당 가이드를 참조한다.

| 순서 | 서비스 | 가이드 | 재시작 필요 |
|------|--------|--------|-----------|
| 1 | Slack | [references/slack.md](references/slack.md) | X |
| 2 | Jira | [references/jira.md](references/jira.md) | O (MCP) |
| 3 | Confluence | [references/confluence.md](references/confluence.md) | O (MCP) |
| 4 | BISKIT | [references/biskit.md](references/biskit.md) | O (MCP) |
| 5 | API Docs | [references/apidocs.md](references/apidocs.md) | O (MCP) |
| 6 | GitLab | [references/gitlab.md](references/gitlab.md) | X |
| 7 | Amplitude | [references/amplitude.md](references/amplitude.md) | O (MCP) |

각 서비스는 해당 가이드의 절차에 따라 **설정까지만** 진행한다. 연결 테스트는 모든 설정이 끝난 뒤 일괄 수행한다.

> ⚠️ Jira, Confluence, BISKIT, API Docs, GitLab은 **사내망 또는 VPN 연결이 필요**합니다.

#### Jira/Confluence 공통 사용자 ID

Jira와 Confluence를 모두 선택한 경우, 사용자 ID는 **한 번만** 입력받는다:
- question: "사용자 ID를 입력해주세요 (Jira/Confluence 공통, 예: hyuntkim)"
- options: [
    {label: "직접 입력하기", description: "사용자 ID를 입력하세요"},
    {label: "잘 모르겠어요", description: "Jira 또는 Confluence에 로그인할 때 사용하는 ID입니다"}
  ]
- "잘 모르겠어요" 선택 시: [Jira 프로필 페이지](https://jira.smilegate.net/secure/ViewProfile.jspa) 또는 [Confluence 프로필 페이지](https://wiki.smilegate.net/users/viewmyprofile.action)에서 사용자 이름을 확인할 수 있다고 안내한 뒤, 다시 입력을 요청한다.

#### MCP 설정 파일 일괄 업데이트

토큰 기반 MCP 서비스(Jira/Confluence/BISKIT)를 1개 이상 선택한 경우:
- 모든 토큰 입력이 끝난 후, Read 도구로 설정 파일을 읽고 **기존 mcpServers의 다른 서비스 설정을 보존한 채** 선택한 서비스를 한번에 추가(신규) 또는 해당 키만 덮어쓰기(재연결)한다. `claude mcp add`로 이미 추가된 서비스(API Docs, Amplitude 등)의 설정을 삭제하지 않는다.
- 각 서비스의 MCP 설정 JSON은 해당 가이드를 참조한다.

**중요**: 토큰은 민감정보이므로 대화 내용에 그대로 노출하지 않는다.
입력받은 토큰 값의 앞뒤 공백과 줄바꿈을 제거(trim)한 뒤 설정 파일에 저장한다.
대화에서는 마스킹하여 표시한다.

### 비-MCP 서비스 즉시 테스트

MCP 서비스와 비-MCP 서비스(Slack, GitLab)를 함께 선택한 경우, 비-MCP 서비스는 재시작이 필요 없으므로 **설정 직후 바로 테스트**한다. 이 테스트 결과는 state.json의 `connected_services`에 반영된다.

> Slack, GitLab만 선택한 경우 (MCP 서비스 없음), 재시작이 필요 없으므로 설정 → 테스트 → 완료 리포트로 바로 이동한다. state.json에 installing 상태를 저장하지 않는다.

### 재시작

MCP 서비스를 1개 이상 설정한 경우, **모든 서비스의 설정이 끝난 후** Claude Code를 한 번만 재시작한다.

#### state.json에 installing 상태 저장

재시작 안내를 출력하기 **직전에**, `~/.claude/skills/smilegate-ai-tools/state.json` 파일을 생성/업데이트한다:

```json
{
  "connector": {
    "status": "installing",
    "pending_services": ["{MCP 재시작이 필요한 설정 서비스 목록}"],
    "connected_services": ["{이미 연결 완료된 전체 서비스 목록}"],
    "first_completed_at": "{기존 값 보존 또는 null}",
    "last_updated_at": "{현재 ISO 8601 시각}"
  }
}
```

- `pending_services`: 이번에 설정했지만 재시작이 필요한 MCP 서비스만 포함 (Jira, Confluence, BISKIT, API Docs, Amplitude 중 선택한 것)
- `connected_services`: 기존 state.json에 connected_services가 있었다면 그 목록을 유지하고, 이번 세션에서 즉시 테스트 성공한 비-MCP 서비스(Slack, GitLab)를 추가한다. **중복을 제거**한다.
- `first_completed_at`: 기존 state.json에 이 값이 있었다면 **반드시 보존**한다. 없었다면 포함하지 않는다.
- 파일이 이미 존재하면 기존 내용을 보존하고 `connector` 키만 업데이트한다.

#### 재시작 안내 메시지

state.json 저장 후 아래 메시지를 출력한다:

설정 파일에 {설정한 서비스 목록} 설정이 추가되었습니다!

---

⚠️ **아래 두 단계를 반드시 따라해주세요!**

**1단계: Claude Code 재시작**
- Mac / Linux: `Ctrl+D` 누르기 → 터미널에서 `claude` 입력
- Windows: `exit` 입력 → 터미널에서 `claude` 입력

**2단계: 아래 명령어를 그대로 복사해서 붙여넣기**

> /connector

이 명령어를 입력하면 자동으로 연결 테스트가 시작됩니다.

---

### 연결 테스트

이 단계에 진입하는 경로는 2가지다:

1. **케이스 B (installing → 재시작 후 /connector 재진입)**: state.json의 `pending_services`에 있는 서비스만 테스트한다.
2. **일반 흐름 (재시작 불필요 또는 같은 세션 내)**: 선택한 서비스를 **모두** 테스트한다.

각 서비스의 연결 테스트 방법은 해당 가이드를 참조한다.

연결 실패 시 트러블슈팅 → [references/troubleshoot.md](references/troubleshoot.md)

### 완료 리포트

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
- **API Docs**: "결제 관련 API 명세 찾아줘", "사용자 인증 API 스펙 알려줘"
- **GitLab**: "내 MR 목록 보여줘", "feature 브랜치에서 MR 만들어줘"
- **Amplitude**: "이번 주 DAU 차트 보여줘", "회원가입 퍼널 분석해줘"

연결 실패한 서비스가 있으면 트러블슈팅 안내를 함께 표시한다.

> 팁: 토큰이 만료되거나 연결에 문제가 생기면 "커넥터 설정해줘" 한 마디로 이 스킬을 다시 실행할 수 있습니다.

> ⚠️ 완료 리포트를 출력한 후 여기서 멈추지 않는다. 반드시 아래 "자동화 스킬 제안" 섹션을 이어서 실행한다.

### 자동화 스킬 제안 (첫 실행 시만)

완료 리포트를 출력한 후, 첫 실행 여부를 확인한다.

**확인 방법:**
state.json의 `connector.first_completed_at` 값으로 판단한다.
- `first_completed_at`이 없다 → 첫 실행 (한 번도 completed된 적 없음)
- `first_completed_at`이 있다 → 재실행 (이전에 이미 completed를 거침)
- **하위 호환**: 레거시 형식(`completed: true`만 있고 `first_completed_at`이 없는 경우)은 **재실행**으로 취급한다. 이미 한 번 완료한 사용자이기 때문이다.

> 이 기준을 사용하면, 케이스 C(completed) → 추가 서비스 설치 → 케이스 B(installing)로 돌아온 사용자에게 자동화를 다시 제안하지 않는다.

**케이스 1: 첫 실행 + 연결 성공 서비스 1개 이상**

1. state.json의 connector를 completed 상태로 업데이트한다:
   ```json
   {
     "connector": {
       "status": "completed",
       "connected_services": ["{연결 성공한 전체 서비스 목록}"],
       "first_completed_at": "{현재 날짜}",
       "last_updated_at": "{현재 ISO 8601 시각}"
     }
   }
   ```
   - `pending_services`는 제거한다 (더 이상 대기 중인 서비스가 없으므로).
   - `connected_services`에는 이번에 테스트 성공한 서비스 + 기존에 이미 연결되어 있던 서비스를 모두 포함하고 **중복을 제거**한다.
   - 파일이 이미 존재하면 기존 내용을 보존하고 `connector` 키만 업데이트한다.

2. 출력 스타일 전환을 안내한다:

   ```
   🎓 자동화를 시작하기 전에, 더 자세한 설명과 함께 진행할 수 있도록
   출력 스타일을 변경해볼까요?

   👉 /output-style 을 입력하고 Explanatory를 선택하세요!
   ```

3. 자동화 스킬을 제안한다:

   ```
   💡 연결된 도구로 바로 업무 자동화를 만들어볼까요?

   반복되는 업무(주간보고, 회의록, 발표자료 등)를
   대화만으로 자동화 스킬로 만들 수 있어요.

   ① 좋아요, 만들어볼래요 → automation 스킬의 Phase 1부터 시작
   ② 나중에 할게요 → 종료 (나중에 "자동화 만들기"로 실행 가능)
   ```

   AskUserQuestion으로 선택을 받는다.
   - ① 선택 시: automation 스킬의 Phase 1부터 실행 (Phase 0 건너뜀, 연결 상태를 이미 알고 있으므로)
   - ② 선택 시: "나중에 만들고 싶으면 '자동화 만들기'라고 말하면 돼요!" 안내 후 종료

**케이스 2: 첫 실행 + 연결 성공 서비스 0개**

state.json에서 `connector` 키를 **제거**한다. 이렇게 하면 다음 `/connector` 실행 시 케이스 A(신규)로 돌아가서 처음부터 진행할 수 있다. `connector` 키를 제거하지 않으면, `installing` 상태가 유지되어 케이스 B가 반복되는 무한 루프가 발생한다.

완료 리포트와 트러블슈팅 안내를 출력하고 종료한다.

**케이스 3: 재실행 (케이스 C에서 진입)**

state.json을 업데이트하고, 자동화 제안은 건너뛴다. 완료 리포트만 출력하고 종료한다.

**케이스 3-a: 연결 성공 서비스 1개 이상**

state.json의 connector를 아래와 같이 업데이트한다:
```json
{
  "connector": {
    "status": "completed",
    "connected_services": ["{연결 성공한 전체 서비스 목록 (중복 제거)}"],
    "first_completed_at": "{기존 값 반드시 보존}",
    "last_updated_at": "{현재 ISO 8601 시각}"
  }
}
```
- `connected_services`: 이번에 테스트 성공한 서비스 + 기존에 이미 연결되어 있던 서비스를 합친 뒤 **중복을 제거**한다.
- `first_completed_at`: 기존 state.json의 값을 **반드시 보존**한다. 이 값이 변경되면 첫 실행/재실행 판단이 깨진다.

**케이스 3-b: 연결 성공 서비스 0개 (전체 실패)**

state.json을 업데이트하되, `connected_services`는 기존 값을 유지한다 (이번 테스트에서 실패했더라도 이전에 연결했던 서비스 목록은 보존). `status`는 `"completed"`를 유지한다. 트러블슈팅을 안내하고 종료한다.

## MCP 설정 위치

MCP는 항상 **전역 설정 파일**에 추가한다. OS마다 경로가 다르다:

| OS | 설정 파일 경로 |
|------|------|
| Mac / Linux | `~/.claude.json` |
| Windows | `%USERPROFILE%\.claude.json` (예: `C:\Users\hyuntkim\.claude.json`) |

> 전역 설정 파일에 저장하면 모든 프로젝트에서 사용할 수 있다.
> Git 저장소 밖에 있으므로 토큰이 실수로 커밋될 위험이 없다.
