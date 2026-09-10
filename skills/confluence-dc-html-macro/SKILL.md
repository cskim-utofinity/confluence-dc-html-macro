---
name: confluence-dc-html-macro
description: Write HTML/CSS code to paste into the Confluence Data Center "HTML macro". Use when the user asks to create, style, or fix markup for a Confluence page — callout boxes, styled tables, cards, badges, buttons, banners, layouts, etc. — that will be inserted via the HTML macro (not plain storage-format). Produces self-contained HTML with scoped inline-or-embedded CSS that survives Confluence's rendering and avoids leaking styles into the rest of the page.
---

# Confluence Data Center — HTML Macro 코드 작성

사용자는 **Confluence Data Center**에서 **HTML 매크로가 활성화된** 환경을 사용한다.
이 스킬은 그 HTML 매크로 안에 붙여넣을 HTML/CSS 코드를 만들 때 사용한다.

## 핵심 전제 (먼저 확인하지 말 것)

- 대상은 **HTML 매크로** 다. 따라서 `<style>` 블록, `class` 선택자, 인라인 `style` 모두 사용 가능하다.
- 일반 Storage Format(매크로 없이 본문에 직접)이 필요한 경우에만 별도로 확인한다 — 그때는 **인라인 `style` 속성만** 가능하고 `<style>`/`class`/`<script>`는 제거된다.
- `<script>`는 HTML 매크로에서 동작할 수 있으나 보안/유지보수상 **기본적으로 사용하지 않는다.** 사용자가 명시적으로 요청할 때만 넣는다.
  - 정당한 트리거 예: "링크/값을 **코드 상단에 변수로 모아** 거기만 채우게 해달라", "클릭 시점에 iframe 로드", "드래그 이동" 등 — HTML/CSS만으로 불가능한 요구다(특히 속성값 변수화). 이때만 소량 JS를 제안한다.
  - JS를 쓰더라도 **구조는 HTML에 그대로 두고 JS는 값만 주입**해 script가 막혀도 레이아웃은 보이게 한다(아래 "설정값 상단 변수화" 패턴). 도입 시 "script가 안 돌면 값이 안 채워짐 → 한 항목만 먼저 저장해 확인"을 함께 안내한다. script가 안 도는 원인은 제거보다 **본문 치환**일 때가 많다(규칙 13).
  - 벤더 라이브러리를 얹는 **대용량 JS 앱**(수십 KB급)도 매크로에서 동작한다 — 막히는 지점은 크기가 아니라 **벤더 하나만 조용히 죽는** 쪽이다. 규칙 12 참고.

## 반드시 지키는 규칙

1. **CSS는 고유 클래스로 스코프한다.** HTML 매크로의 `<style>`은 페이지 전역에 주입되므로, 일반 태그 셀렉터(`table`, `td`, `h3` 등)를 그대로 쓰면 **페이지의 다른 영역까지 오염**된다.
   - 모든 규칙을 고유 래퍼 클래스 하위로 한정: `.cf-callout { ... }`, `.cf-callout .title { ... }`
   - 접두사 `cf-`(confluence) + 컴포넌트명 사용. 흔한 단어(`.box`, `.card`) 단독 금지.

2. **자기완결형(self-contained)으로 작성한다.** 하나의 래퍼 `<div>` 안에 `<style>` + 마크업을 함께 넣어 복붙 한 번으로 끝나게 한다.

3. **단일 컴포넌트면 인라인 `style`도 고려한다.** 재사용/반복이 없고 짧으면, 충돌 위험이 0인 인라인 스타일이 더 안전하다. 표처럼 셀이 많아 반복되면 `<style>` + class가 낫다.

4. **Confluence가 깨뜨리는 것들을 피한다:**
   - 가능하면 모든 태그를 닫는다(`<br/>`, `<hr/>`). 에디터가 XHTML로 재정규화할 때 안전.
   - `position: fixed`, 매우 큰 `z-index`, 뷰포트 단위(`vw/vh`)는 Confluence 레이아웃과 충돌하니 피한다. (단 floating 팝업·PIP·토스트처럼 화면에 떠야 하는 컴포넌트는 `position:fixed` + **중간 수준 `z-index`(예: 200)** 로 허용. 크기는 `vw/vh` 대신 `px` + `max-width:calc(100% - 40px)` 로 잡는다.)
   - 외부 리소스(`<link>` 폰트/CSS, 외부 이미지 host)는 폐쇄망/프록시 정책으로 막힐 수 있으니 기본은 시스템 폰트·인라인 SVG·data URI를 우선한다.
   - 색상은 Confluence/ADG 팔레트와 어울리게: 기본 텍스트 `#172B4D`, 보조 `#5E6C84`, 보더 `#DFE1E6`, 배경 강조 `#F4F5F7`, 파랑 `#0052CC`, 초록 `#36B37E`, 빨강 `#DE350B`, 노랑 `#FFAB00`.

5. **폰트는 시스템 스택**을 기본으로: `-apple-system, BlinkMacSystemFont, "Segoe UI", "Malgun Gothic", Roboto, sans-serif` (한글 환경 고려).

6. **미디어·첨부·임베드 함정을 안다 (실측):**
   - **`.html` 첨부파일은 iframe으로 못 띄운다.** Confluence는 보안상 `.html` 첨부를 `Content-Disposition: attachment` 로 내려주므로, iframe `src`에 `.html` 첨부 URL을 넣으면 렌더링이 아니라 **파일 다운로드**가 된다. 이건 서버 헤더 문제라 `loading` 속성이나 `<script>`로도 못 바꾼다. 대안 3가지:
     1. (가장 확실) HTML 매크로 안이므로 **그 HTML 내용을 팝업/본문 div에 직접 삽입**한다. 단 완전한 문서면 `<html>/<head>/<body>` 껍데기는 빼고 `<body>` 안쪽만 넣고, 삽입할 HTML의 `<style>`도 고유 클래스로 스코프돼 있는지 확인(없으면 페이지 오염).
     2. 내용을 **별도 Confluence 페이지**로 만들고 그 **페이지 URL**을 iframe `src`로 쓴다(같은 호스트라 SAMEORIGIN 임베드 허용, 첨부와 달리 inline 렌더). 다만 iframe 안에 Confluence 헤더/사이드바가 같이 보일 수 있음.
     3. iframe `srcdoc="...escape된 HTML..."` 로 감싼다. 길이가 짧을 때만이 아니라 **매크로 본문 치환을 통째로 우회**하고 싶을 때도 쓴다(규칙 13). srcdoc 안쪽은 태그가 아니라 속성값 문자열이라 **태그로 해석되지 않는다.** 속성값이므로 `&` → `&amp;` 를 **먼저** 치환하고 그다음 `<` `>` `"` 순으로 바꾼다(순서를 바꾸면 `&quot;` 의 `&` 가 다시 치환돼 깨진다). 부등호까지 바꾸면 결과물에 `<script` 라는 문자열 자체가 안 남는다. **다만 엔티티는 속성값에서도 풀린다.** `&quot;` 가 속성을 끊어 iframe이 빈 채로 렌더될 수 있으니(규칙 13) 저장 후 화면부터 확인하라.
   - **mp4 등 동영상 첨부는 inline 재생된다.** `<video controls><source src="첨부 URL" type="video/mp4"></video>` 로 정상 동작.
   - **숨김 상태 미디어의 미리 로드 주의.** CSS로 `display:none` 처리해도 `<video>`는 `preload="none"` 을 줘야 재생 전까지 안 받는다. **iframe은 숨겨도 페이지 로드 시 미리 불러올 수 있다**(`loading="lazy"` 로도 완전히 못 막음) — 클릭 시점 로드가 꼭 필요하면 순수 CSS로는 한계가 있고 JS가 필요하다고 알린다.
   - 외부 사이트 iframe은 상대의 `X-Frame-Options`/`CSP frame-ancestors` 로 막히면 빈 화면이 되니, 그 경우 새 창 링크(`<a target="_blank" rel="noopener">`)로 대체한다.

7. **`<style>` 전역 주입은 양날의 검 — Confluence 자체 레이아웃(크롬)도 덮어쓸 수 있다 (실측).** 매크로 `<style>`이 페이지 전역에 적용되는 성질(규칙 1)은 보통 오염 위험이지만, **사이드바·제목·내비를 의도적으로 숨겨 콘텐츠를 전체화면처럼** 만들 때는 이걸 역이용한다. 자주 쓰는 Confluence/플러그인 셀렉터:
   - `#ia-fixed-sidebar` / `.ia-fixed-sidebar` : 좌측 고정 사이드바 패널
   - `.ia-splitter` : 사이드바↔본문 구분선 **(주의: 함정 — 아래 참고)**
   - `#main` : 사이드바 제외 본문 전체 컨테이너 (사이드바 숨기면 `margin-left:0` 으로 당김)
   - `#title-heading` / `#title-text` : 페이지 제목 영역 / 제목 텍스트
   - `.page-metadata` : 작성자·작성일 등 메타정보 줄
   - `#ehp-navigation-wrapper`, `.nav-header-nowrap`, `.ehp-highlight` : EHP(Enhanced Header Plugin) 상단 내비 래퍼 / 내비 항목 / 선택 강조 — `ehp-` 접두사는 모두 이 서드파티 플러그인 소속
   - `div.error.conf-macro.output-block` : **매크로 렌더 실패 시** 나타나는 빨간 오류 박스(3개 클래스 `error`+`conf-macro`+`output-block` 동시 보유). F12에서 이게 보이면 매크로 오류 신호. `display:none` 으로 숨기기도 함.
   - **함정 — `.ia-splitter`에 `display:none` 을 주면 페이지 전체가 사라질 수 있다.** 테마/버전에 따라 `.ia-splitter`가 사이드바와 `#main`(본문)을 **모두 감싸는 부모**라, 숨기면 본문까지 통째로 날아간다. 구분선만 없애려면 `display:none` 대신 `background:#fff;border:none` 로 덮거나, 핸들 자식(`.ia-splitter-handle`)만 `display:none` 한다.
   - 크롬 제어용 규칙은 Confluence 기본 스타일보다 우선해야 하므로 `!important` 가 사실상 필수다.
   - 이런 규칙은 **여러 페이지/공간 레이아웃에 영향**을 주므로(특히 EHP 내비는 전역) "한 페이지에서 먼저 저장해 확인" 후 확대 적용을 안내한다.

8. **매크로 "아래" 콘텐츠가 통째로 사라지면 = 매크로 HTML이 뒤 콘텐츠를 삼킨 것 (실측·중요).** 증상: 매크로 자체(대문 등)는 정상 렌더되는데 **그 아래에 있던 페이지 본문·다른 매크로가 전부 안 보인다.** 원인은 대부분 **DOM 상에서 래퍼 태그가 안 닫혀** 뒤 콘텐츠가 그 안으로 빨려 들어가고 `overflow:hidden` 등에 잘리는 것. 원인·대응 순서:
   - **① `<html>`/`<head>`/`<body>` 태그를 넣지 않는다.** HTML 매크로가 이 골격을 이미 갖고 있어, 포함하면 페이지 구조와 충돌해 뒤 콘텐츠가 깨진다. **붙여넣는 건 최상위 래퍼 `<div>`(+그 안의 `<style>`)까지만.** ([Atlassian KB](https://support.atlassian.com/confluence/kb/html-code-in-confluence-html-macro-is-not-working/)) 완전한 `.html` 문서를 옮길 때 특히 실수하기 쉬움.
   - **② 소스가 태그 균형(open=close)인지 먼저 기계적으로 확인한다.** `<div>`/`</div>` 개수 일치 여부부터 센다. 불일치면 그게 범인.
   - **③ 균형인데도 사라지면 = 편집기 정규화가 닫는 태그를 떨어뜨린 것.** 깊게 중첩된 **순수 인라인 `<div>` 수십 개**(6~7단계)를 붙이면, 리치 에디터가 저장 시 재직렬화하며 `</div>` 하나를 유실해 루트가 열린 채로 남는 사례가 있다. 대응: **인라인 style을 스코프된 `<style>`+class로 옮겨 마크업 길이·중첩을 대폭 줄이고**(문자 수 ↓ → 정규화 오류 확률 ↓), 저장 후 **소스/스토리지 편집기에서 마지막 `</div>`가 살아있는지** 확인한다.
   - **④ 그래도 페이지가 오염되면 CSS 누수를 의심한다.** 매크로 `<style>`이 `body`·`div` 같은 전역 셀렉터를 쓰면 페이지 전체에 샌다(규칙 1). 반드시 `.cf-*` 래퍼 하위로 스코프. ([Atlassian Community](https://community.atlassian.com/forums/Confluence-questions/Custom-HTML-Macro-formatting-issues/qaq-p/883106))
   - 진단 팁: F12 Elements 탭에서 사라진 콘텐츠가 **매크로 래퍼 div의 자식으로 들어가 있으면** ①·③(태그 미닫힘), 페이지 곳곳 스타일이 바뀌었으면 ④(CSS 누수). 개발자도구를 쓸 수 없는 상황이면 규칙 12의 자가 진단 블록으로 같은 정보를 페이지에 찍어 확인한다.

9. **`<a>`에 준 색·밑줄은 Confluence 기본 링크 스타일에 먹힌다 — 링크 스타일만은 `!important`가 필요하다 (실측).** 규칙 1은 "우리 CSS가 페이지로 새는" 방향이지만, 링크는 **반대 방향으로 당한다.** Confluence는 본문 링크를 `.wiki-content a` 같은 자체 규칙으로 칠하는데, 테마/AUI 규칙이 매크로의 클래스 규칙(`.cf-x .cf-y`)보다 우선순위가 높은 경우가 많아 **의도한 색이 무시되고 링크 파랑(방문 후 보라)으로 강제**된다. 목차·카드형 링크처럼 회색/커스텀 색을 쓰려던 `<a>`가 전부 파랗게 보이면 이 현상이다. ([Atlassian Community](https://community.atlassian.com/forums/Confluence-questions/CSS-styling-links/qaq-p/943330))
   - 대응: 링크 관련 선언에만 `!important`를 붙이고, **`:visited`·`:hover`도 함께 고정**한다(방문 후 색 변화 방지).
     ```css
     .cf-x .cf-link,.cf-x .cf-link:visited{color:#42526E !important;text-decoration:none !important;}
     .cf-x .cf-link:hover{color:#0052CC !important;}
     ```
   - `<a>` **안의 `<span>`**은 자기 클래스 규칙이 있으면 보통 안 먹히지만, 상속으로 파랗게 되는 사례가 있어 `.cf-x a:visited .cf-t{...!important}` 식으로 같이 잠가 두면 안전하다.
   - 남용 금지: `!important`는 **링크(색·`text-decoration`)에만** 쓴다. 나머지 스타일까지 붙이면 나중에 조정이 불가능해진다.
   - 본문 속 일반 인라인 링크는 오히려 Confluence 기본 스타일(파랑+밑줄)이 자연스러우니 **그대로 두는 게 낫다.** 덮어쓸 대상은 목차·버튼·카드처럼 **링크처럼 안 보여야 하는 것들**이다.

10. **`<ul>`/`<li>`로 만든 커스텀 목록에는 Confluence 기본 불릿이 덧붙는다 — 목록은 `<div>`로 짠다 (실측).** 규칙 9와 같은 "당하는" 방향의 사고다. `list-style:none`을 줘도 Confluence 본문 규칙(`.wiki-content ul li` 등)이 우선해 **내가 `:before`로 그린 마커 옆에 기본 불릿(•)이 하나 더** 나타난다. 4단 중첩 목록처럼 단계별로 다른 마커를 그리는 경우 특히 티가 난다.
    - **1순위 대응 — 목록 태그를 안 쓴다.** 마커가 붙을 대상 자체를 없애는 게 가장 확실하다. 재발 여지가 없다.
      ```html
      <div class="cf-l2">
        <div class="cf-b2">항목</div>
        <div class="cf-l3"><div class="cf-b3">하위 항목</div></div>
      </div>
      ```
      ```css
      .cf-x .cf-l2{margin:4px 0 8px 10px;}
      .cf-x .cf-b2{position:relative;padding-inline-start:16px;}
      .cf-x .cf-b2:before{content:"";position:absolute;left:2px;top:12px;
        width:6px;height:6px;border-radius:50%;background:#36B37E;}
      ```
      들여쓰기는 부모 `.cf-lN`의 `margin-left`로, 마커는 `.cf-bN:before`로 그린다. 중첩 `.cf-lN`은 `.cf-bN`의 **형제**로 두면 되고, `<li>` 래퍼가 없어 마크업도 짧아진다(규칙 8-③의 정규화 오류 확률도 함께 내려간다).
    - **2순위 — 목록 시맨틱이 꼭 필요할 때만** `<ul>`을 쓰고 `!important`로 잠근다. 단 테마가 `li::before`로 마커를 넣으면 이것도 뚫리니 저장 후 반드시 확인한다.
      ```css
      .cf-x ul{list-style:none !important;padding-left:0 !important;margin:0 !important;}
      .cf-x li{list-style:none !important;}
      .cf-x li::marker{content:"" !important;}
      ```
    - 트레이드오프: `<div>` 목록은 스크린리더의 "목록, 항목 N개" 안내를 잃는다. 읽기용 문서·대시보드는 1순위로, 접근성이 요구되는 문서는 2순위로 간다.
    - 중첩 `<ul>`을 쓸 때는 **반드시 `<li>` 안에** 넣는다(`<ul>` 직속 자식 `<ul>`은 무효 마크업이라 에디터 재직렬화에서 위치가 틀어진다).

11. **표 머리글(`<th>`)의 배경은 Confluence 기본 테이블 스타일에 밀린다 — `background`와 `color` 둘 다 `!important`가 필요하다 (실측).** 규칙 9·10과 같은 "당하는" 방향의 세 번째 사례다. Confluence는 본문 표를 `.wiki-content table th` 같은 자체 규칙으로 칠하는데, 특정성이 매크로의 클래스 규칙(`.cf-x th`)보다 높아 **지정한 배경색이 무시되고 연한 회색으로 강제**된다.
    - 고약한 건 `color`에만 `!important`를 붙였을 때다. 글자색은 살아남고 배경만 밀려서 **흰 글씨가 회색 배경에 묻혀 거의 안 보인다.** 머리글이 회색으로 보이면 이 현상을 먼저 의심한다.
      ```css
      .cf-x th{background:#0052CC !important;color:#fff !important;
        -webkit-text-fill-color:#fff !important;}
      ```
      `-webkit-text-fill-color`까지 같이 잠그는 건 일부 테마가 그걸로 글자색을 덮기 때문이다.
    - 규칙 9의 "`!important`는 링크에만" 원칙에 예외가 둘 더 생긴 셈이다. **링크 색·목록 마커·표 머리글** 셋은 Confluence가 먼저 칠하므로 덮어쓸 수밖에 없다. 나머지 스타일까지 `!important`로 바르지는 말 것.
    - **`<iframe srcdoc>` 안에서는 안 밀린다 (실측).** iframe 안은 별도 문서라 Confluence CSS가 닿지 않아, `!important` 없는 `.t th{background:#0052CC}` 만으로도 그대로 나온다. 즉 이 규칙은 매크로 본문에 직접 넣은 표에만 해당한다.

12. **대용량 JS 앱(벤더 라이브러리 포함)도 매크로에서 동작한다 — 다만 벤더 하나만 조용히 죽는 함정이 있다.** HTML 매크로 출력은 서버에서 페이지 마크업에 그대로 렌더되므로 **인라인 `<script>`는 문서 순서대로 정상 실행**되고, `<script src>`도 순서가 보장된다. 수십~수백 KB 규모에 `<script>` 여러 개를 넣어도 **파일 맨 끝 블록까지 실행된 사례가 있다.** "대용량이라 안 된다"는 통념부터 버리고 시작하라 — 막히는 지점은 대개 크기가 아니다.
    - **벤더 블록 하나만 조용히 실패하면, 아래를 한 번에 하나씩만 바꿔가며 배너로 확인한다.** 여러 개를 동시에 건드리면 증상이 겹쳐 진단이 꼬인다.
      - **AMD 로더 충돌** — UMD 라이브러리는 `define.amd`를 발견하면 전역을 안 만들고 익명 AMD 모듈로 등록해 버린다. Confluence 페이지에는 AMD 로더가 떠 있으므로, 해당 블록만 `(function(define,exports,module){…})(undefined,undefined,undefined)`로 감싸 지역 파라미터로 가리면 브라우저 전역 등록 경로를 탄다.
      - **JS 문법 레벨** — 최신 라이브러리는 구형 엔진에서 **파싱 단계 `SyntaxError`**로 죽는다. 이때 그 블록만 통째로 죽고 나머지 블록은 멀쩡히 돌기 때문에, "벤더 하나만 실패"로 보인다. 아래 프로브로 확인한다.
      - **서버측 템플릿 처리(Velocity)** — 사이트에 따라 "HTML 매크로"가 **유저 매크로**로 구현돼 있고, 그러면 본문이 Velocity를 거친다. Velocity는 `#이름(...)`을 매크로 호출로, `$이름`·`${이름}`을 변수 참조로 **해석해 치환하거나 삼킨다.** JS에는 이 모양이 흔하다 — 최신 문법의 **private 메서드 `this.#name(...)`**, 미니파이 코드의 `$1`·`$&` 치환 패턴, 템플릿 리터럴 `${...}`. 해당 블록만 내용이 바뀌어 조용히 깨진다. 대응은 그 문법을 안 쓰는 빌드를 고르거나, 라이브러리를 첨부 `.js`로 빼는 것이다.
      - **라인 길이·총 바이트** — 편집기 정규화가 긴 줄을 건드릴 여지가 있다(규칙 8-③의 JS 버전). 다만 이것만으로 설명되지 않는 사례가 있으니 **유력한 후보로 앞세우지는 말 것.**
    - **벤더 라이브러리를 넣기 전에 대상 브라우저를 확인한다.** 진단 배너에 `navigator.userAgent` 한 줄을 같이 찍어라. 브라우저가 고정·구형인 환경이 드물지 않고, 최신 문법 라이브러리는 거기서 조용히 죽는다. 문법 지원 여부는 배너에서 직접 잴 수도 있다:
      ```js
      try{ eval("class A{x=1}; (()=>0)(); (null)?.a ?? 1"); s="ES2022 OK"; }
      catch(e){ s="문법 미지원 → 최신 라이브러리 사용 불가: "+e.message; }
      ```
      막히면 **그 라이브러리의 구버전(ES5 빌드)이나 ES5로 작성된 대체 라이브러리**를 쓴다. 트랜스파일 결과물을 넣어도 된다. 버전별 최소 문법 수준은 추측하지 말고 **파서로 재라** — `acorn.parse(src,{ecmaVersion:v})`를 `5→2015→…→2022` 순으로 돌려 통과하는 첫 값이 그 빌드가 요구하는 수준이다.
      - **함정 — 문법이 통과해도 동작한다는 보장은 없다.** 트랜스파일러는 문법만 낮추고 빌트인은 그대로 둔다. ES5 빌드라도 `Promise`·`Set`·`Symbol`·`Object.assign`·`Array.from`·`String.prototype.repeat` 같은 **ES6 런타임 API를 쓰면 아주 오래된 엔진에서는 실행 중에 죽는다.** 다만 이건 파싱 실패와 달리 오류가 잡히므로 진단 배너에 뜬다. 아주 낮은 대상까지 가야 하면 빌트인 사용까지 확인하거나 폴리필을 함께 넣는다.
    - **라이브러리를 페이지 첨부 `.js`로 빼고 `<script src>`로 부르는 것도 고려한다.** 아래 참고. 본문 크기·라인 길이 변수를 통째로 제거한다(단 문법 레벨 문제는 그대로 남는다).
    - **판별은 "수정보다 비쌀 때만" 한다.** 후보를 하나 바꿔 넣고 배너만 보는 게 대개 어떤 정밀 진단보다 싸다. 다만 **한 번에 한 변수만** 바꿔라 — 크기와 문법을 동시에 건드리면 두 증상이 겹쳐 진단이 꼬인다.
    - **"실행 안 됨"과 "내용이 변형됨"을 구분한다.** 무음 실패를 보면 잘림·제거·문법부터 의심하기 쉽지만, **DOM에 멀쩡히 존재하는데 내용만 바뀐** 경우가 있다(위 Velocity 항목이 그렇다). 이건 블록별 `textContent.length`를 **기대값과 대조**하는 것 하나로 갈린다. 짧아졌으면 잘림, **길어졌어도 변형**이다 — 0이 아니면 무조건 신호다. 무음 실패에서는 **길이부터 재라.** 그리고 어긋났으면 **엔티티부터 세라**(규칙 13). 실측 두 건이 모두 그것 하나로 한 글자도 안 틀리고 설명됐다.
    - **진단 코드는 검사 대상보다 "뒤"에 둔다.** 앞에 두면 진단기 자신이 같이 죽었을 때 아무것도 안 남아 한 라운드를 통째로 날린다. 결과를 모아 출력하는 블록은 **항상 마지막 script 블록**에 두고, 앞쪽에는 오류 수집기(`addEventListener('error', …)`)만 최소한으로 남긴다. 그 수집기도 검사 대상과 같은 이유로 죽지 않도록 **ES5로만** 작성한다.
      - **진단기 자신이 오염되지 않게 한다.** 검사하려는 위험 문자열(`#이름(`, `$이름`, `${…}` 등)을 진단 코드가 리터럴로 갖고 있으면 진단기도 같이 변형된다. 기대값·미끼 문자열은 `String.fromCharCode(…)`로 만들어 소스에 그 문자가 남지 않게 하고, **미끼 하나당 `<script>` 하나**로 쪼개 하나가 죽어도 나머지가 보고하게 한다.
    - **원인을 재야 하면, 진단 결과를 콘솔이 아니라 페이지에 찍는다.** 매크로 안의 JS는 이미 돌고 있으므로 콘솔에 넣을 내용을 그대로 화면에 렌더할 수 있다. 스크립트 오류도 `window` 의 `error` 이벤트로 잡아 같이 뿌린다. 재현자가 개발자도구를 못 열거나 다른 사람이 대신 확인해줘야 할 때 특히 이 방법만 남는다 — **결과를 스크린샷 한 장으로 받을 수 있다는 게 핵심.**
      ```html
      <div class="cf-diag"><style>.cf-diag{font:12px/1.5 ui-monospace,Consolas,monospace;
        white-space:pre-wrap;word-break:break-all;background:#F4F5F7;border:1px solid #DFE1E6;
        padding:10px;margin:8px 0;color:#172B4D;}</style>
      <script>window.CF_ERR=[];addEventListener('error',function(e){
        CF_ERR.push(e.message+' @'+(e.lineno||'?')+'행');});</script>

      <!-- ▼ 검사 대상 script 블록들을 여기 그대로 둔다 (벤더 라이브러리 등) -->
      <!-- ▲ -->

      <script>(function(){
        var d=document.querySelector('.cf-diag'),o='';
        [].forEach.call(d.querySelectorAll('script'),function(x,i){
          var t=x.textContent||'';
          o+='['+i+'] '+(x.src?'src='+x.src
            :t.length+'자 | 끝40자: '+t.slice(-40).replace(/\s+/g,' '))+'\n';
        });
        o+='\n전역: '+['marked','TurndownService'].map(function(n){
          return n+'='+(typeof window[n]);}).join(' / ');
        try{ eval("class A{x=1}; (()=>0)(); (null)?.a ?? 1"); o+='\n문법: ES2022 OK'; }
        catch(e){ o+='\n문법 미지원 → 최신 라이브러리 사용 불가: '+e.message; }
        o+='\nUA: '+navigator.userAgent;
        o+='\n\n에러('+CF_ERR.length+'):\n'+(CF_ERR.join('\n')||'없음');
        d.appendChild(document.createTextNode(o));
      })();</script></div>
      ```
      읽는 법: 각 블록의 **글자 수를 원본 파일과 대조**한다. 세 갈래로 갈린다.
      - **정확히 일치** → 내용은 무사히 도착했으니 **실행/문법 문제**다(에러 줄에 `SyntaxError`가 뜬다).
      - **짧다** → 잘렸다. `끝40자`로 어디서 끊겼는지 보인다. 순수 라인 길이 한계를 재려면 더미 블록(`<script>window.CF_LINELEN="AAAA…4만 자…".length;</script>`)으로 길이를 바꿔가며 이분 탐색한다.
      - **길다, 또는 조금 어긋난다** → **서버측에서 내용이 치환된 것**이다. **엔티티부터 세라(규칙 13).** 실측 두 건이 모두 그것이었고 계산이 한 글자도 안 틀린다. 그걸로 안 맞으면 위 Velocity 항목을 의심하고, 정상 블록에는 없고 실패 블록에만 있는 문자열을 찾아라. 이때 **눈에 띄는 토큰 말고 문자 단위로 세라.** 엔티티는 `<` 나 `$` 처럼 튀지 않아 육안으로는 안 보이고, 실제로 그것 때문에 후보 목록에 못 올라가 한참을 돌아간 사례가 있다.
    - **`.js` 첨부를 `<script src>`로 로드하는 건 규칙 6의 `.html` iframe 차단과 다른 문제다 (미검증 — 사이트마다 확인 필요).** `.html`이 막히는 원인은 `Content-Disposition: attachment`인데, **브라우저는 script/img/css 같은 서브리소스 요청에서 `Content-Disposition`을 무시한다**(내비게이션에만 적용). 따라서 원리상 `.js` 첨부는 로드된다. 실제 차단 변수는 **MIME 타입 + `X-Content-Type-Options: nosniff` 조합**이다 — Confluence가 첨부를 `application/octet-stream`/`text/plain`으로 내려주면서 nosniff를 붙이면 브라우저가 실행을 거부한다(`Refused to execute script … MIME type … is not executable`).
      **확인법 — 헤더를 보지 말고 그냥 시켜본다.** `window.CF_ATTACH=1` 한 줄짜리 `probe.js`를 페이지에 첨부하고, 매크로에 `<script src="/download/attachments/<pageId>/probe.js"></script>` + 위 진단 블록을 넣어 `CF_ATTACH`가 잡히는지 본다. 응답 헤더를 직접 볼 수 있는 환경이면 `Content-Type`이 `text/javascript`·`application/javascript`인지, `X-Content-Type-Options: nosniff`가 붙는지 확인하는 것도 같은 답을 준다. **되면 본문 크기·라인 길이 문제가 통째로 사라지므로 대용량 앱은 이 경로를 먼저 타진한다.**
    - **매크로에 JS를 넣을 땐 진단 배너를 함께 넣는다 (무음 실패 방지).** 매크로 안의 JS는 실패해도 화면이 그냥 "반응 없음"이라 원인 범위가 안 좁혀진다. 단계별 배너 한 줄이 이 사례에서 원인을 한 번에 특정했다.
      ```html
      <div class="cf-boot">❌ 0단계 — 스크립트가 안 돌았습니다(제거됐거나 본문이 치환돼 죽었습니다).</div>
      <script>/* 맨 위 */ var b=document.querySelector('.cf-x .cf-boot');
        if(b) b.textContent='⏳ 1단계 — 스크립트 실행됨. 라이브러리 로딩 중…';</script>
      <!-- …벤더 script들… -->
      <script>(function(){ var b=document.querySelector('.cf-x .cf-boot');
        function fail(m){ if(b) b.textContent='⚠️ '+m; }
        try{
          var miss=[]; ['marked','TurndownService'].forEach(function(n){
            if(typeof window[n]==='undefined') miss.push(n); });
          if(miss.length) return fail('2단계 — 라이브러리 미로딩: '+miss.join(', '));
          /* 3단계: DOM 요소 확인 → 초기화 → 성공 시 b.remove() */
        }catch(e){ fail('초기화 예외: '+e.message); }
      })();</script>
      ```
      포인트: 배너 초기 텍스트는 **0단계(=script 자체가 안 돎)**로 둬서 script가 안 돌면 그 상태가 그대로 보이게 한다. 다만 이 배너만으로는 **제거와 치환을 구분하지 못한다** — 둘 다 똑같이 0단계로 보인다. 여기서 "제거됐다"로 건너뛰면 엉뚱한 우회로 몇 라운드를 날린다. 갈라야 하면 규칙 13의 길이 대조를 같이 쓴다. 누락 라이브러리는 **첫 개만 말고 전부 모아** 출력해야 "하나만 실패"인지 "전부 실패"인지 구분된다. 정상 초기화되면 배너를 지운다.
    - **id를 쓰지 말고 class로 찾는다.** Confluence가 id를 정리하거나 페이지 내 다른 요소와 충돌할 수 있다. 매크로가 같은 페이지에 여러 번 삽입돼도 전역 `const` 재선언 오류가 안 나도록 초기화 코드는 IIFE로 감싼다.
    - **위 후보를 다 배제했는데도 재현되면, 그 사이트 고유 설정을 의심한다.** 보안·새니타이저 플러그인, "HTML 매크로"가 실은 Velocity 유저 매크로인 경우, 프록시·WAF의 응답 재작성, 페이지 상태(매크로 중복·붙여넣기 절단) 등은 인스턴스마다 다르므로 **일반 규칙으로 일반화하지 말고 그 사이트의 지식으로 따로 남긴다.** 다음 값들도 사이트별로 달라 여기 못 박아두지 않는다: 라인 길이 한계 / 본문 총 바이트 상한 / `.js` 첨부의 `Content-Type`·nosniff / 외부 CDN 접근 가능 여부.

13. **매크로 본문의 HTML 엔티티가 디코딩된다 — `<script>` 안의 `"&quot;"` 가 `"""` 가 되어 블록이 통째로 죽는다 (실측).** `&amp;` `&lt;` `&gt;` `&quot;` `&#39;` 같은 문자 참조가 매크로를 거치면서 원래 문자 한 개로 풀린다. HTML 텍스트라면 맞는 동작이지만 `<script>` 안에서는 **소스 코드가 바뀌는 것**이라 문자열이 끊기고 `SyntaxError`로 그 블록만 죽는다. 화면에는 아무 표시도 안 나므로 규칙 12의 무음 실패로 보인다.
    - 전형적인 희생자는 HTML 이스케이프 함수다. 아래가 그대로 도착하면 `"""` 에서 문자열이 끊겨 `replace(` 의 괄호가 안 닫히고, V8은 `missing ) after argument list` 를 뱉는다.
      ```js
      .replace(/&/g,"&amp;").replace(/</g,"&lt;").replace(/>/g,"&gt;").replace(/"/g,"&quot;")
      ```
    - **대응은 소스에 `&` 글자를 안 남기는 것이다.** `\x26` 으로 쓰면 디코더가 잡을 게 없다. 동작은 같고 매크로를 거쳐도 길이가 안 변한다.
      ```js
      .replace(/&/g,"\x26amp;").replace(/</g,"\x26lt;")
        .replace(/>/g,"\x26gt;").replace(/"/g,"\x26quot;")
      ```
    - **주석 안에도 쓰지 않는다.** 주석이면 문법은 안 깨지고 길이만 어긋나서, 오히려 진단이 더 헷갈려진다. 수정 이유를 설명하려고 주석에 엔티티를 적었다가 다시 5자가 어긋난 사례가 있다.
    - **검산이 정확해서 가설 검증이 쉽다.** 블록마다 엔티티를 세어 `각 엔티티 길이 − 1` 의 합을 기대값에서 빼면 도착 길이가 그대로 나온다. 실측 두 건 다 한 글자도 안 틀렸다. 앱 블록 17,620 → 17,605(엔티티 4개, −15), 벤더 블록 101,048 → 101,029(엔티티 5개, `&#39;` 포함, −19). 같은 파일에서 엔티티가 0개인 블록은 전부 길이가 정확했다. **예외가 하나도 없으면 그때 인과로 본다.**
    - **`<script>` 가 제거된 것으로 오인하기 쉽다.** CSS는 멀쩡히 적용되고 JS만 안 도는 그림이라 "매크로가 script를 걸러낸다"로 건너뛰기 쉽다. 빈 페이지에 두 줄만 붙이면 30초에 갈린다.
      ```html
      <div class="cf-t">❌ script 미실행</div>
      <script>document.querySelector('.cf-t').textContent='✅ script 실행됨';</script>
      ```
    - **텍스트뿐 아니라 속성값에서도 풀린다 (실측).** `srcdoc` 안에 `class=&quot;t&quot;` 를 넣으면 `&quot;` 가 `"` 로 풀리면서 **속성값이 거기서 끊겨** iframe이 빈 채로 렌더된다(하얀 네모). 같은 코드에서 따옴표를 없앤 `class=t` 로 바꾸면 정상 동작한다. 두 판본의 차이가 그것뿐이라 원인은 확실하다.
      - 대응은 **속성값에 따옴표를 안 쓰는 것**이다. HTML은 공백·따옴표·`=`·`<`·`>`·백틱이 없는 단순한 값이면 따옴표를 생략할 수 있다.
      - 다만 `srcdoc` 으로 완전한 문서를 감쌀 때는 따옴표를 피할 수 없다. 그런 코드가 실제로 동작하는 사례도 있어 **어떤 조건에서 살아남는지는 아직 못 밝혔다.** srcdoc 래퍼를 쓸 거면 저장 후 화면부터 확인하라.
    - **규칙 6의 `<iframe srcdoc>` 이 이 문제를 우회하는 이유가 여기 있다.** srcdoc으로 감쌀 때 `&` 를 `&amp;` 로 선치환하는데, 매크로의 디코딩이 그걸 원래대로 되돌려 정확히 상쇄된다. 원인을 모르면 "srcdoc이 script 필터를 우회한다"고 오해하기 쉽지만 우회 대상은 필터가 아니라 이 디코딩이다. 알고 나면 래퍼 없이 `\x26` 만으로 충분하고, 래퍼와 빌드 단계를 통째로 지울 수 있다.

## 출력 형식

- 바로 복사할 수 있게 **하나의 코드 블록**으로 완성 코드를 제공한다.
- 코드 위/아래에 "어디에 붙여넣는지(HTML 매크로)"와 **수정 포인트**(색·문구·열 수 등)를 한두 줄로 안내한다.
- 사용자가 색/문구 등을 바꿀 가능성이 높은 값은 `<style>` 상단에 모아두거나 주석으로 표시한다.

## 자주 쓰는 패턴 (요청 시 변형)

### 콜아웃/안내 박스 (info·warning·success)
```html
<div class="cf-callout cf-callout--info">
  <style>
    .cf-callout{display:flex;gap:12px;padding:12px 16px;border-radius:4px;
      font-family:-apple-system,BlinkMacSystemFont,"Segoe UI","Malgun Gothic",Roboto,sans-serif;
      font-size:14px;line-height:1.5;color:#172B4D;margin:8px 0;}
    .cf-callout--info{background:#DEEBFF;border-left:4px solid #0052CC;}
    .cf-callout--warning{background:#FFFAE6;border-left:4px solid #FFAB00;}
    .cf-callout--success{background:#E3FCEF;border-left:4px solid #36B37E;}
    .cf-callout .cf-ico{font-weight:700;flex:0 0 auto;}
    .cf-callout .cf-body{flex:1 1 auto;}
    .cf-callout .cf-body b{display:block;margin-bottom:2px;}
  </style>
  <div class="cf-ico">ℹ️</div>
  <div class="cf-body"><b>제목</b>본문 내용을 여기에 작성합니다.</div>
</div>
```

### 스타일 표
```html
<div class="cf-table-wrap">
  <style>
    .cf-table-wrap{font-family:-apple-system,BlinkMacSystemFont,"Segoe UI","Malgun Gothic",Roboto,sans-serif;margin:8px 0;}
    .cf-table-wrap table{border-collapse:collapse;width:100%;font-size:14px;color:#172B4D;}
    .cf-table-wrap th{background:#0052CC;color:#fff;text-align:left;padding:10px 12px;}
    .cf-table-wrap td{border-bottom:1px solid #DFE1E6;padding:10px 12px;}
    .cf-table-wrap tr:nth-child(even) td{background:#F4F5F7;}
  </style>
  <table>
    <thead><tr><th>항목</th><th>설명</th></tr></thead>
    <tbody>
      <tr><td>A</td><td>설명 A</td></tr>
      <tr><td>B</td><td>설명 B</td></tr>
    </tbody>
  </table>
</div>
```

### 배지 / 상태 라벨 (인라인 style — 단발성이라 클래스 불필요)
```html
<span style="display:inline-block;padding:2px 8px;border-radius:3px;font-size:12px;
  font-weight:700;color:#fff;background:#36B37E;font-family:-apple-system,'Segoe UI','Malgun Gothic',sans-serif;">완료</span>
```

### 버튼형 링크
```html
<a href="https://example.com" style="display:inline-block;padding:8px 16px;background:#0052CC;
  color:#fff;text-decoration:none;border-radius:3px;font-size:14px;font-weight:600;
  font-family:-apple-system,'Segoe UI','Malgun Gothic',sans-serif;">바로가기 →</a>
```

### 순수 CSS 토글 팝업 / 모달 / PIP (스크립트 없이)
`<script>` 없이 클릭 토글이 필요할 때 **라디오(체크박스) 핵**을 쓴다. 핵심:
- 숨긴 라디오 N개 + **기본 선택 `none` 라디오 1개**(= 닫힘 상태). 같은 `name` 그룹이라 한 번에 하나만 열린다.
- 여는 트리거는 `<label for="...">`, **닫기 버튼은 `<label for="(none 라디오)">`**. (라디오는 다시 클릭해도 안 꺼지므로 닫기는 반드시 `none` 라디오로 보낸다.)
- 라디오는 `display:none` 대신 `position:absolute;opacity:0;width:0;height:0` 로 숨겨야 label 토글·`:checked`가 안전하게 동작.
- 라디오와 팝업 컨테이너가 **형제**여야 일반형제 결합자(`~`)로 표시 제어 가능.

```html
<div class="cf-pop-demo">
  <style>
    .cf-pop-demo>input.cf-r{position:absolute;opacity:0;width:0;height:0;pointer-events:none;}
    .cf-pop-demo .cf-link{display:inline-block;padding:3px 12px;border:1px solid #DFE1E6;border-radius:14px;
      font-size:12.5px;font-weight:600;color:#0052CC;cursor:pointer;user-select:none;
      font-family:-apple-system,"Segoe UI","Malgun Gothic",sans-serif;}
    .cf-pop-demo .cf-link:hover{background:#DEEBFF;}
    /* PIP: 화면 우측 하단에 떠서 보임 */
    .cf-pop-demo .cf-pip{display:none;position:fixed;right:20px;bottom:20px;z-index:200;
      width:520px;max-width:calc(100% - 40px);background:#fff;border:1px solid #DFE1E6;
      border-radius:8px;overflow:hidden;box-shadow:0 8px 24px rgba(9,30,66,.25);}
    .cf-pop-demo .cf-pip-head{display:flex;justify-content:space-between;align-items:center;
      background:#253858;color:#fff;padding:8px 12px;font-size:13px;font-weight:600;}
    .cf-pop-demo .cf-x{cursor:pointer;padding:2px 6px;border-radius:4px;user-select:none;}
    .cf-pop-demo #cf-p1:checked ~ .cf-stage .pip-1{display:block;}
  </style>
  <input class="cf-r" type="radio" name="cf-pop" id="cf-none" checked/>
  <input class="cf-r" type="radio" name="cf-pop" id="cf-p1"/>
  <label class="cf-link" for="cf-p1">▶ 열기</label>
  <div class="cf-stage">
    <div class="cf-pip pip-1">
      <div class="cf-pip-head"><span>제목</span><label class="cf-x" for="cf-none">✕</label></div>
      <div style="padding:14px;">팝업 본문(동영상·이미지·내용 등).</div>
    </div>
  </div>
</div>
```
변형: 가운데 모달이면 `.cf-pip` 를 `position:fixed;inset:0;margin:auto` + 반투명 배경 오버레이(역시 `none` 라디오로 닫기)로 바꾼다. 항목이 많아지면 `#cf-pN:checked ~ .cf-stage .pip-N{display:block;}` 줄만 추가한다.
**한계:** 순수 CSS라 드래그 이동·클릭 시점 iframe 로드는 불가 — 필요하면 소량 JS를 제안한다.

### 설정값 상단 변수화 (소량 JS — 속성값을 한 곳에서 채우기)
"부서 링크/URL을 코드 위쪽에 변수로 빼고 거기만 채우게" 같은 요청. HTML/CSS엔 `href`/`src` 변수가 없으므로 **상단 config 객체 + 하단 주입 스크립트**로 푼다. 핵심 원칙: **마크업(구조/이름/계층)은 HTML에 그대로 두고**, JS는 `data-*` 키로 매칭해 **값만 주입** → script가 막혀도 레이아웃은 보인다.
```html
<div class="cf-cfg">
  <!-- ▼ 여기만 채움 -->
  <script>
  window.CF_CFG = {
    "a": { home:"", video:"" },   // 항목 A
    "b": { home:"", video:"" }    // 항목 B
  };
  </script>
  <!-- ▲ -->

  <a class="cf-link" data-home="a" href="#" target="_blank" rel="noopener">A 바로가기</a>
  <video data-vid="a" controls preload="none"><source src="" type="video/mp4"/></video>

  <!-- 주입기 (수정 불필요) -->
  <script>
  (function(){
    var C = window.CF_CFG || {};
    var root = (document.currentScript && document.currentScript.closest('.cf-cfg')) || document.querySelector('.cf-cfg');
    if(!root) return;
    root.querySelectorAll('a[data-home]').forEach(function(a){
      var u=(C[a.getAttribute('data-home')]||{}).home;
      if(u){a.href=u;} else {a.removeAttribute('href');a.style.opacity='.45';}   // 미설정은 비활성 표시
    });
    root.querySelectorAll('video[data-vid]').forEach(function(v){
      var u=(C[v.getAttribute('data-vid')]||{}).video;
      if(u){var s=v.querySelector('source');s.src=u;s.type='video/mp4';v.load();} // load() 필수
    });
  })();
  </script>
</div>
```
포인트: config `<script>`는 **맨 위**(편집 대상), 주입 `<script>`는 **마크업 뒤**(DOM 파싱 후 실행)에 둔다. `document.currentScript.closest()`로 자기 래퍼만 스코프. `<source>.src`를 바꾼 뒤엔 `video.load()` 호출해야 반영. `window.CF_*` 전역은 같은 페이지에 매크로가 여럿이면 이름 충돌하니 고유화한다.

## 작업 순서

1. 사용자가 만들려는 컴포넌트와 내용/색/구조를 파악한다(불명확하면 1~2개만 간단히 질문).
2. 위 규칙대로 자기완결형 코드 작성 — 반복 요소면 스코프된 `<style>`+class, 단발성이면 인라인 style.
3. 코드 블록 + 붙여넣기 위치 + 수정 포인트를 함께 제시한다.
4. 변경 이력을 마크다운에 남길 경우 날짜는 **KST(한국 기준)** 로 표기한다.
