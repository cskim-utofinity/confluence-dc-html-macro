# confluence-dc-html-macro

Confluence Data Center / Server의 "HTML 매크로"에 붙여넣을 HTML/CSS 코드를 작성해주는
[Claude Code](https://claude.com/claude-code) 스킬입니다. 콜아웃 박스, 스타일 표, 카드, 배지,
버튼, 배너, 레이아웃 등을 만들어 주는데, Confluence 렌더링 특이사항에 걸리지 않고 스타일이
페이지 나머지 영역으로 새지 않습니다.

스킬 본문([`confluence-dc-html-macro/SKILL.md`](confluence-dc-html-macro/SKILL.md))은 한국어
Confluence DC/Server 환경을 대상으로 하기 때문에 한국어로 쓰여 있습니다. 다만 여기 담긴
패턴과 함정(CSS 스코핑, EHP 내비게이션 셀렉터, `.ia-splitter` 함정, 링크 스타일 강제 등)은
HTML 매크로가 켜진 Confluence Data Center/Server 환경이라면 어디에나 적용됩니다.

## 왜 이름에 "dc"가 붙었나

이 스킬은 `.ia-fixed-sidebar`, `.ia-splitter`, Enhanced Header Plugin(EHP) 등 Confluence
Data Center / Server 전용 DOM 구조와 플러그인에 크게 의존합니다. HTML 매크로 자체도 Confluence
Cloud에는 없습니다. `dc` 접미사는 Cloud 사용자가 잘못 적용하지 말라는 표시입니다.

## 설치

Claude Code는 `~/.claude/skills/<name>/SKILL.md` 경로에서 스킬을 읽습니다. 리포를 클론한 뒤
스킬 폴더를 복사하거나 심볼릭 링크/주니션으로 연결하면 됩니다.

```sh
git clone https://github.com/<your-username>/confluence-dc-html-macro.git
cp -r confluence-dc-html-macro/confluence-dc-html-macro ~/.claude/skills/confluence-dc-html-macro
```

Claude Code를 재시작하거나 새 세션을 열면 Confluence 페이지용 HTML 매크로 마크업을 요청할
때 이 스킬이 자동으로 인식됩니다.

## 라이선스

MIT. 자세한 내용은 [LICENSE](LICENSE) 참고.
