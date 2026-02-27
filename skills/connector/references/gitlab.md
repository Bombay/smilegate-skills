# GitLab CLI 설정 가이드

사내 GitLab(git.sginfra.net) CLI 도구(glab) 설치 및 인증 방법을 안내한다.

> ⚠️ 사내 GitLab은 **사내망 또는 VPN 연결이 필요**합니다. 연결 상태를 먼저 확인해주세요.

## glab 설치

Bash로 `uname -s` 실행하여 OS를 감지한 뒤, 적절한 설치 명령을 안내한다.

### macOS

```bash
brew install glab
```

Homebrew가 설치되어 있지 않은 경우 (brew 명령이 없는 경우):
- 먼저 [Homebrew 공식 사이트](https://brew.sh)에서 설치 방법을 안내한다.

### Windows

WinGet (권장):
```bash
winget install -e --id GLab.GLab
```

또는 Scoop:
```bash
scoop install glab
```

### 설치 확인

Bash로 아래 명령을 실행하여 설치를 확인한다:

```bash
glab --version
```

설치 실패 시: 패키지 매니저 문제를 확인하고, 필요하면 [glab Release 페이지](https://gitlab.com/gitlab-org/cli/-/releases)에서 직접 다운로드를 안내한다.

## GitLab PAT 토큰 발급

안내 사항을 코드 블록 없이 마크다운으로 출력한다. URL은 반드시 마크다운 링크 형식으로 표시한다:

① 아래 링크를 브라우저에서 열어주세요:
   👉 [GitLab 토큰 발급 페이지](https://git.sginfra.net/-/user_settings/personal_access_tokens)

② "Add new token" 버튼 클릭 → 이름 입력 (예: "Claude Code 연동용")

③ ⚠️ "만료 날짜" 옵션에서 **자동 만료를 해제**하는 것을 권장합니다

④ **권한(Scopes)** 선택:
   - `api` ✅ (필수)
   - `read_repository` ✅ (필수)
   - `write_repository` ✅ (필수)

⑤ "Create personal access token" 클릭 → ⚠️ 생성된 토큰을 반드시 복사하세요! (다시 볼 수 없습니다)

## 토큰 입력

AskUserQuestion으로 토큰을 입력받는다:
- question: "발급받은 GitLab PAT 토큰을 붙여넣어 주세요"
- options: [
    {label: "발급하고 복사했어요", description: "3번 입력란에 바로 붙여넣어주세요"},
    {label: "발급 절차를 다시 안내해주세요", description: "토큰 발급 방법을 처음부터 안내합니다"}
  ]
- 사용자가 Other(3번)를 선택하여 텍스트를 입력하면, 입력값을 토큰으로 처리한다.
- "발급하고 복사했어요" 선택 시: "토큰을 붙여넣고 엔터를 눌러주세요"라고 안내한다. 다음 사용자 메시지를 토큰으로 처리한다.
- "발급 절차를 다시 안내해주세요" 선택 시: 위의 GitLab 토큰 발급 절차를 스텝별로 다시 안내하고, 완료 후 다시 토큰 입력을 요청한다.

## 인증 설정

Bash로 아래 명령을 실행한다 ({PAT토큰}에 입력받은 토큰을 대입):

```bash
echo "{PAT토큰}" | glab auth login --hostname git.sginfra.net --stdin
```

**중요**: 토큰은 민감정보이므로 대화 내용에 그대로 노출하지 않는다. 명령 실행 시 토큰을 마스킹하여 표시한다.

## 연결 테스트

Bash로 아래 명령을 실행한다:

```bash
glab auth status --hostname git.sginfra.net
```

- 성공하면: 인증 정보 표시 (사용자명, 호스트 등)
- 실패하면: 에러 메시지와 함께 트러블슈팅 안내

## 트러블슈팅

연결 실패 시 확인할 것:
1. glab이 정상 설치되었는지 확인 (`glab --version`)
2. 사내망/VPN에 연결되어 있는지 확인
3. PAT 토큰에 `api`, `read_repository`, `write_repository` 권한이 있는지 확인
4. 토큰이 만료되지 않았는지 확인
5. 호스트명이 `git.sginfra.net`으로 정확한지 확인

트러블슈팅이 진전 없이 막히면 → [트러블슈팅 및 이슈 등록](troubleshoot.md) 절차에 따라 GitHub 이슈 등록을 제안한다.

## 관련 URL

| 항목 | URL |
|------|-----|
| GitLab 토큰 발급 | https://git.sginfra.net/-/user_settings/personal_access_tokens |
| GitLab 프로필 페이지 | https://git.sginfra.net/-/user_settings/profile |
| glab CLI 공식 문서 | https://docs.gitlab.com/cli/ |
| 공식 설치 가이드 | https://wiki.smilegate.net/pages/viewpage.action?pageId=589459355 |
