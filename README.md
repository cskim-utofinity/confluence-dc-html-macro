# Confluence DC HTML 매크로 스킬

`confluence-dc-html-macro`는 Confluence Data Center / Server의 "HTML 매크로"에 붙여넣을 HTML/CSS 코드를 작성해주는
[Claude Code](https://claude.com/claude-code) 스킬입니다. 콜아웃 박스, 스타일 표, 카드, 배지,
버튼, 배너, 레이아웃 등을 만들어 주는데, Confluence 렌더링 특이사항에 걸리지 않고 스타일이
페이지 나머지 영역으로 새지 않습니다.

스킬 본문([`skills/confluence-dc-html-macro/SKILL.md`](skills/confluence-dc-html-macro/SKILL.md))은 한국어
Confluence DC/Server 환경을 대상으로 하기 때문에 한국어로 쓰여 있습니다. 다만 여기 담긴
패턴과 함정(CSS 스코핑, EHP 내비게이션 셀렉터, `.ia-splitter` 함정, 링크 스타일 강제 등)은
HTML 매크로가 켜진 Confluence Data Center/Server 환경이라면 어디에나 적용됩니다.

## 이 스킬을 쓰는 이유

Confluence HTML 매크로에 익숙하지 않은 채로 AI에게 마크업을 맡기면 흔히 이런 문제가 생깁니다.

- `<style>` 규칙이 스코프 없이 `table`, `h3` 같은 태그 셀렉터를 그대로 써서 매크로 밖 페이지 전체가 오염된다.
- 태그가 하나라도 안 닫히면 매크로 아래 본문이 통째로 사라진다.
- 목차·카드용 링크가 Confluence 기본 링크 스타일(파란색+밑줄)에 덮여 의도한 색이 안 나온다.
- `<ul>`로 짠 목록에 Confluence 기본 불릿이 하나 더 붙는다.
- `.html` 첨부파일을 iframe에 넣으면 렌더링 대신 다운로드가 된다.

이 스킬은 실제로 겪은 위 함정들을 규칙으로 미리 담아 두고, 처음부터 스코프된 클래스·닫힌
태그·Confluence 색상 팔레트로 코드를 뽑아 줍니다.

## 왜 이름에 "dc"가 붙었나

이 스킬은 `.ia-fixed-sidebar`, `.ia-splitter`, Enhanced Header Plugin(EHP) 등 Confluence
Data Center / Server 전용 DOM 구조와 플러그인에 크게 의존합니다. HTML 매크로 자체도 Confluence
Cloud에는 없습니다. `dc` 접미사는 Cloud 사용자가 잘못 적용하지 말라는 표시입니다.

## 설치

harness마다 설치 방법이 다릅니다.

### Claude Code

#### 방법 ① 플러그인 마켓플레이스 — git 불필요 (권장)

Claude Code 세션에서 다음 두 줄을 실행합니다.

```
/plugin marketplace add cskim-utofinity/confluence-dc-html-macro
/plugin install confluence-dc-html-macro@confluence-dc-html-macro
```

- 업데이트: `/plugin marketplace update confluence-dc-html-macro` 후 `/plugin update confluence-dc-html-macro`
- 제거: `/plugin uninstall confluence-dc-html-macro`

#### 방법 ② 클론

Claude Code는 `~/.claude/skills/<name>/SKILL.md` 경로에서 스킬을 읽습니다. 리포를 클론한 뒤
스킬 폴더를 복사하거나 심볼릭 링크/주니션으로 연결하면 됩니다.

```sh
git clone https://github.com/cskim-utofinity/confluence-dc-html-macro.git
cp -r confluence-dc-html-macro/skills/confluence-dc-html-macro ~/.claude/skills/confluence-dc-html-macro
```

어느 방법이든 Claude Code를 재시작하거나 새 세션을 열면 Confluence 페이지용 HTML 매크로
마크업을 요청할 때 이 스킬이 자동으로 인식됩니다.

### Codex CLI

Codex의 공식 플러그인 마켓플레이스([openai/plugins](https://github.com/openai/plugins))는
등록·심사를 거친 플러그인만 올라가는 큐레이션 목록이라, 이 리포처럼 그 목록에 없는 개인
스킬은 `/plugins` 검색으로 설치할 수 없습니다. 대신 클론한 스킬 폴더를 Codex가 읽는
경로에 직접 연결합니다.

```sh
git clone https://github.com/cskim-utofinity/confluence-dc-html-macro.git
cp -r confluence-dc-html-macro/skills/confluence-dc-html-macro ~/.codex/skills/confluence-dc-html-macro
```

새 세션을 열면 `/skills` 메뉴에서 확인할 수 있습니다.

### Antigravity

사전 등록 없이 리포 URL로 바로 설치합니다.

```sh
agy plugin install https://github.com/cskim-utofinity/confluence-dc-html-macro
```

업데이트도 같은 명령을 다시 실행하면 됩니다.

## 라이선스

MIT. 자세한 내용은 [LICENSE](LICENSE) 참고.

---

Atlassian 및 Confluence와 무관한 비공식 커뮤니티 프로젝트입니다.
