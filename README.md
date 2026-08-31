# confluence-dc-html-macro

A [Claude Code](https://claude.com/claude-code) skill for writing HTML/CSS to paste into the
**Confluence Data Center / Server "HTML macro"** — callout boxes, styled tables, cards, badges,
buttons, banners, and layouts that survive Confluence's rendering quirks without leaking styles
into the rest of the page.

The skill content ([`confluence-dc-html-macro/SKILL.md`](confluence-dc-html-macro/SKILL.md)) is
written in Korean, since it targets Korean-language Confluence DC/Server environments, but the
patterns and pitfalls it documents (CSS scoping, EHP navigation selectors, `.ia-splitter` traps,
link-style overrides, etc.) apply to any Confluence Data Center/Server instance with the HTML
macro enabled.

## Why "dc"

This skill relies heavily on DOM structure and plugins specific to **Confluence Data Center /
Server** (`.ia-fixed-sidebar`, `.ia-splitter`, the Enhanced Header Plugin, etc.), and the HTML
macro itself doesn't exist on Confluence **Cloud**. The `dc` suffix is there to stop Cloud users
from reaching for it by mistake.

## Install

Claude Code loads skills from `~/.claude/skills/<name>/SKILL.md`. Clone this repo and copy (or
symlink/junction) the skill folder in:

```sh
git clone https://github.com/<your-username>/confluence-dc-html-macro.git
cp -r confluence-dc-html-macro/confluence-dc-html-macro ~/.claude/skills/confluence-dc-html-macro
```

Restart Claude Code (or start a new session) and the skill will be picked up automatically when
you ask it to build HTML-macro markup for a Confluence page.

## License

MIT — see [LICENSE](LICENSE).
