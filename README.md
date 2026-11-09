# arabic-english-bidi

A Claude plugin that fixes **bidirectional (RTL/LTR) text ordering** when Claude
writes mixed Arabic and English content.

## What it fixes

When Arabic (right-to-left) and English/numbers (left-to-right) appear on the
same line, the Unicode Bidirectional Algorithm often misplaces the embedded
runs: version numbers reverse, English words jump sides, and trailing `?` / `:`
land on the wrong end. This plugin's skill teaches Claude to wrap embedded runs
in invisible Unicode bidi-control characters (isolates and marks) so everything
renders in the right visual position.

## What it does NOT fix

Block-level alignment (right-aligning a whole Arabic paragraph) is a UI/CSS
concern (`dir="rtl"`), which an inline text reply cannot set. The skill is
honest about this limit. For full RTL layout in your Angular apps, use the
`dir` attribute / CSS in the frontend.

## Structure

```
arabic-english-bidi/
├── .claude-plugin/
│   └── plugin.json              # plugin manifest
├── skills/
│   └── bidi-direction/
│       └── SKILL.md             # the bidi-handling instructions
└── README.md
```

## Reliability note

The skill is a *model-triggered* capability: it fires when Claude judges the
reply to contain Arabic/mixed content. For a guarantee on **every** turn in the
chat app, also put the same rules in your **Custom Style** or **Preferences**.
The plugin is the shareable / Claude Code route.

## Install (Claude Code)

Place this folder in a marketplace repo or point your plugin config at it, then
enable `arabic-english-bidi`. The `bidi-direction` skill loads automatically when
relevant.
