# Decision Ledger

チーム共有用の決定一覧。詳細は各ADRを参照。
設計書本体（`smart_shrimp_pond_system_design_outline.md`）の記述と矛盾が出た場合、**ADRを正とする**。

## Decided

| ID | 決定 | 日付 | 影響範囲 |
|---|---|---|---|
| [ADR-001](decisions/ADR-001-controlled-variable.md) | 水位を唯一のPID制御対象とする。水質は判断材料 | 2026-09-09 | Control, Agent, ML, Demo |
| [ADR-002](decisions/ADR-002-pid-execution-location.md) | PIDループはEdge（ESP32）で実行する | 2026-09-09 | IoT, Backend, Safety |
| [ADR-003](decisions/ADR-003-ml-scope.md) | 予測層はIF定義のみ。Phase 1はルールベース | 2026-09-09 | ML, Roadmap, 提案書 |
| [ADR-004](decisions/ADR-004-agentic-sdlc.md) | 運用系と同一の原則をSDLCに適用する | 2026-09-09 | 開発プロセス全体 |

## 前提条件（今日確定）

| 項目 | 決定 |
|---|---|
| 実環境 | 実際の養殖池なし。**センサー実機のみ** → ベンチスケールで検証 |
| アクチュエータ | 給排水ポンプ（水位・水交換） |
| 実データ | 不明・実質なし前提 → MLは後回し（ADR-003） |
| センサー構成 | 5センサー（pH / 温度 / TDS / 濁度 / 水位）を維持。DOはPhase 2 |

## Open Questions（未決）

| 項目 | 状態 | 必要な時期 |
|---|---|---|
| ポンプ駆動方式（ON/OFF or 流量可変） | 実機仕様の確認待ち | Week 2 |
| センサー型番・精度 | 実機が手元にあるため転記のみ | Week 1 |
| Domain（種・池サイズ・密度） | 文献デフォルトで仮置き `DRAFT` | 提案時は仮置きで可 |
| Business（ROI・導入コスト） | 提案段階では確定しない（§39） | Phase 2以降 |

## Status 定義

`DRAFT` 仮説 / `DISCUSSION` 議論中 / `DECIDED` 合意済 / `IMPLEMENTED` 実装済 / `VALIDATED` 検証済 / `DEPRECATED` 廃止

**Development Agent は `DECIDED` 以上の項目のみ実装してよい。** 詳細は [12_agentic-sdlc.md](12_agentic-sdlc.md) §12.4
