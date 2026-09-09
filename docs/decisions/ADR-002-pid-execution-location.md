# ADR-002: PID制御ループをEdge（ESP32）で実行する

**Status:** `DECIDED`
**日付:** 2026-09-09

## Context
PID制御ループをEdgeで回すか、クラウド／バックエンドで回すかを決める必要がある。

## Options
- A: Edge（ESP32）でPIDを実行し、Backendはsetpointのみ送る
- B: Backendでレイテンシを含めてPIDを実行する

## Decision
**A を採用。PIDループはEdgeで実行する。Backend / Agentはsetpointのみを送る。**

## Reason
- 通信断が発生しても制御が継続する（設計書 §9.1 Reliability）
- ネットワークレイテンシとジッタが制御性能に直接影響しない
- 「Agentに直接ハードウェアを操作させない」（§36 Rule 5）が構造的に保証される。
  Agentが送れるのはsetpointだけで、ポンプ出力そのものには到達できない

## Consequences
- サンプリングを2階層に分ける
  - 制御用（水位）：1 Hz 程度、Edgeでループ。DBへは間引いて保存
  - 記録用（水質）：1分間隔、全件をDBに保存
- Safety Layer は Edge 側にも二重で置く（通信断時にも境界が効く必要があるため）
- Edgeファームウェアの責務が増える（PID・安全境界・オフラインバッファ）

## Open Questions
- ポンプがON/OFF駆動のみの場合、PID出力を時間比例（time-proportioning）へ変換し、
  デッドバンド＋最小ON/最小OFF時間（ポンプ保護）を設ける必要がある。実機仕様の確認待ち
