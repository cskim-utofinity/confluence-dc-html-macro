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
  - JS를 쓰더라도 **구조는 HTML에 그대로 두고 JS는 값만 주입**해 script가 막혀도 레이아웃은 보이게 한다(아래 "설정값 상단 변수화" 패턴). 도입 시 "매크로가 script를 제거하면 동작 안 함 → 한 항목만 먼저 저장해 확인"을 함께 안내한다.

## 반드시 지키는 규칙

1. **CSS는 고유 클래스로 스코프한다.** HTML 매크로의 `<style>`은 페이지 전역에 주입되므로, 일반 태그 셀렉터(`table`, `td`, `h3` 등)를 그대로 쓰면 **페이지의 다른 영역까지 오염**된다.
   - 모든 규칙을 고유 래퍼 클래스 하위로 한정: `.cf-callout { ... }`, `.cf-callout .title { ... }`
   - 접두사 `cf-`(confluence) + 컴포넌트명 사용. 흔한 단어(`.box`, `.card`) 단독 금지.

2. **자기완결형(self-contained)으로 작성한다.** 하나의 래퍼 `<div>` 안에 `<style>` + 마크업을 함께 넣어 복붙 한 번으로 끝나게 한다.

3. **단일 컴포넌트면 인라인 `style`도 고려한다.** 재사용/반복이 없고 짧으면, 충돌 위험이 0인 인라인 스타일이 더 안전하다. 표처럼 셀이 많아 반복되면 `<style>` + class가 낫다.

4. **Confluence가 깨뜨리는 것들을 피한다:**
   - 가능하면 모든 태그를 닫는다(`<br/>`, `<hr/>`). 에디터가 XHTML로 재정규화할 때 안전.
   - `position: fixed`, 매우 큰 `z-index`, 뷰포트 단위(`vw/vh`)는 Confluence 레이아웃과 충돌하니 피한다. (단 floating 팝업·PIP·토스트처럼 화면에 떠야 하는 컴포넌트는 `position:fixed` + **중간 수준 `z-index`(예: 200)** 로 허용. 크기는 `vw/vh` 대신 `px` + `max-width:calc(100% - 40px)` 로 잡는다.)
   - 외부 리소스(`<link>` 폰트/CSS, 외부 이미지 host)는 사내망/프록시 정책으로 막힐 수 있으니 기본은 시스템 폰트·인라인 SVG·data URI를 우선한다.
   - 색상은 Confluence/ADG 팔레트와 어울리게: 기본 텍스트 `#172B4D`, 보조 `#5E6C84`, 보더 `#DFE1E6`, 배경 강조 `#F4F5F7`, 파랑 `#0052CC`, 초록 `#36B37E`, 빨강 `#DE350B`, 노랑 `#FFAB00`.

5. **폰트는 시스템 스택**을 기본으로: `-apple-system, BlinkMacSystemFont, "Segoe UI", "Malgun Gothic", Roboto, sans-serif` (한글 환경 고려).

6. **미디어·첨부·임베드 함정을 안다 (실측):**
   - **`.html` 첨부파일은 iframe으로 못 띄운다.** Confluence는 보안상 `.html` 첨부를 `Content-Disposition: attachment` 로 내려주므로, iframe `src`에 `.html` 첨부 URL을 넣으면 렌더링이 아니라 **파일 다운로드**가 된다. 이건 서버 헤더 문제라 `loading` 속성이나 `<script>`로도 못 바꾼다. 대안 3가지:
     1. (가장 확실) HTML 매크로 안이므로 **그 HTML 내용을 팝업/본문 div에 직접 삽입**한다. 단 완전한 문서면 `<html>/<head>/<body>` 껍데기는 빼고 `<body>` 안쪽만 넣고, 삽입할 HTML의 `<style>`도 고유 클래스로 스코프돼 있는지 확인(없으면 페이지 오염).
     2. 내용을 **별도 Confluence 페이지**로 만들고 그 **페이지 URL**을 iframe `src`로 쓴다(같은 호스트라 SAMEORIGIN 임베드 허용, 첨부와 달리 inline 렌더). 다만 iframe 안에 Confluence 헤더/사이드바가 같이 보일 수 있음.
     3. 짧으면 iframe `srcdoc="...escape된 HTML..."` 사용.
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
   - 진단 팁: F12 Elements 탭에서 사라진 콘텐츠가 **매크로 래퍼 div의 자식으로 들어가 있으면** ①·③(태그 미닫힘), 페이지 곳곳 스타일이 바뀌었으면 ④(CSS 누수).

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
