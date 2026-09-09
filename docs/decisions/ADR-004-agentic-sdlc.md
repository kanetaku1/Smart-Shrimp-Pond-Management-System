# ADR-004: 運用系と同一のアーキテクチャ原則をSDLCに適用する

**Status:** `DECIDED`
**日付:** 2026-09-09

## Context
Agentic SDLC を本提案の中核に据えるにあたり、
「AIでコードを書く」以上の設計的な根拠を定義する必要がある。

## Options
- A: 開発効率化の手段としてAIコーディングを利用する（根拠は生産性のみ）
- B: 運用系のアーキテクチャ原則をそのままSDLCに適用する

## Decision
**B を採用。** 運用系と開発系に同一の構造を適用する。

```text
【運用】 Farmer → Agent → Safety Layer      → PID → Pump  → Pond
【開発】 Human  → Agent → Verification Gate → CI  → Merge → Codebase
```

## Reason
- Agentic SDLC の採用理由が「流行」ではなく「設計思想の一貫性」になる
- 運用系で定めた安全原則（§36）が、そのまま開発プロセスの規律として再利用できる
- Living Documentのステータス（§41）に、Agentの行動を制御する実行可能な意味を与えられる
- 提案としての独自性が、プロダクト単体ではなく**プロダクトと開発プロセスの一貫性**に立つ

## Consequences
- 開発Agentにも自律レベル（Level 1〜4）を定義し、2か月ではLevel 2〜3を採用する
- 安全境界の定数はHumanのみが変更できる
- QA Agentは実装コードを変更できない
- `docs/` は人間向け設計書とAgentのコンテキストを兼ねる（二重管理を禁止）
- Agentic SDLC自体の評価指標を §31 Evaluation に追加する
- 詳細設計は [docs/12_agentic-sdlc.md](../12_agentic-sdlc.md)

## Open Questions
- Level 4 を試行する非Safety領域の具体的な範囲
