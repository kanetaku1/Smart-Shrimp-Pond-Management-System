# 12. Agentic SDLC

**Status:** `DECIDED`
**更新日:** 2026-09-09
**関連:** [ADR-004](decisions/ADR-004-agentic-sdlc.md), 設計書 §20, §21, §22, §36, §41

---

## 12.1 中核アイデア — 同じ構造を2回使う

このプロジェクトの提案の核心は、**運用系と開発系に同一のアーキテクチャ原則を適用する**ことにある。

養殖池をAgentが監督する構造と、コードベースをAgentが生成する構造を、意図的に同じ形にする。

```text
【運用系】 Farmer  → Agent → Safety Layer      → PID  → Pump  → Pond
                                (境界を強制)

【開発系】 Human   → Agent → Verification Gate → CI   → Merge → Codebase
                                (境界を強制)
```

### 対応関係

| 運用系（プロダクト） | 開発系（SDLC） | 共通の役割 |
|---|---|---|
| Sensor | Repo / docs / テスト結果 | Observe（現状を観測する） |
| Data Platform | docs/ + ADR | Remember（決定と履歴を保持する） |
| ML | — （Phase 2） | Predict |
| Farm Agent | Development Agent | Decide（次の行動を計画する） |
| **Safety Layer** | **Verification Gate** | **境界を強制する** |
| PID | CI / 自動テスト | Control（検証済みの手続きで実行） |
| Pump | Merge | Act（実世界を変更する） |
| Farmer | Human Reviewer | Supervise（重要判断のみ行う） |

### 共通原則

設計書 §36 のチームルールが、そのまま両方に適用される。

| 原則 | 運用系での意味 | 開発系での意味 |
|---|---|---|
| Agentに直接ハードウェアを操作させない | ポンプを直接叩かせない | `main` に直接コミットさせない |
| Safety Layerを必ず通す | 範囲・レート制限 | テスト・Lint・ADR整合チェック |
| Human Override | 手動運転・緊急停止 | レビュー差し戻し・Revert |
| 段階的自律 | Level 1 → 4 | Level 1 → 4 |
| 実データがない部分はSimulationで検証 | 仮想池 | テスト・シミュレーション環境 |

> **この一貫性が、本提案における Agentic SDLC の位置づけである。**
> 流行だから採用するのではなく、**プロダクトの設計思想をそのまま開発プロセスに適用した結果**として採用する。

---

## 12.2 開発Agentの自律レベル

運用系の自律レベル（設計書 §17.4）と対応させて定義する。

| Level | 名称 | Agentの役割 | Humanの役割 |
|---|---|---|---|
| 1 | Assist | 調査・要約・ドキュメント草案 | 全てを自分で書く |
| 2 | Generate + Review | コード・テストを生成 | 全差分をレビューして承認 |
| 3 | Supervised Autonomy | 実装・テスト・修正まで完遂 | Gate結果と差分の要点をレビュー |
| 4 | Autonomous | Issue → PR → Merge | 例外のみ介入 |

### 本プロジェクトでの採用方針

- **通常領域：Level 2〜3**（運用系の目標と同一）
- **Safety領域：Level 1〜2 に固定**（安全境界の定数、PIDパラメータ、緊急停止ロジック）
- **Level 4 を試すのは非Safety領域のみ**（ドキュメント整形、テスト追加、機械的リファクタリング）

> 運用系で「2か月ではLevel 2〜3が現実的」と判断したのと同じ根拠を、開発系にも適用する。

---

## 12.3 開発ループの具体化

設計書 §20.2 のループを、**入力・出力・通過条件**まで定義する。

```text
GUIDE → GENERATE → VERIFY → SOLVE → REVIEW → ITERATE
```

| Stage | 実施者 | 入力 | 出力 | 通過条件（Gate） |
|---|---|---|---|---|
| **GUIDE** | Human | 目的・制約 | Issue（仕様＋受入条件） | 受入条件が**検証可能な形**で書かれている |
| **GENERATE** | Agent | Issue + `docs/` + ADR | 実装 **＋ テスト** | 差分がIssueのスコープ内に収まっている |
| **VERIFY** | Agent / CI | 実装 | テスト結果・Lint・型検査 | 全てgreen、既存テストの退行なし |
| **SOLVE** | Agent | 失敗ログ | 修正 | **3回失敗したらHumanへエスカレーション** |
| **REVIEW** | Human | PR | 承認 / 差し戻し | ADRと矛盾しない・安全境界を侵していない |
| **ITERATE** | Both | 得られた知見 | `docs/` と ADR の更新 | 新しい決定がADRに記録された |

### 重要な設計判断

1. **GENERATE は必ず実装とテストをセットで出す。**
   テストのないコードは Verification Gate を通れない。Agentのhallucination（§29）に対する一次防御。

2. **SOLVE には試行回数の上限を置く。**
   Agentが同じ失敗を繰り返してコンテキストを浪費する事故を防ぐ。3回でHumanにエスカレーション。

3. **REVIEW では「動くか」ではなく「決定と整合するか」を見る。**
   動作確認は VERIFY の責務。Humanは設計判断の一貫性のみを見る。これがHumanの時間を最も節約する。

---

## 12.4 Context Engineering — `docs/` をAgentの記憶にする

Agentic SDLC の実体は、プロンプトの工夫ではなく **コンテキスト設計**である。

### 原則：ドキュメントは二重の読者を持つ

`docs/` は、人間向けの設計書であると同時に、**Development Agent への入力コンテキスト**である。
両者を別々に持つと必ず乖離するため、**単一のドキュメントが両方を兼ねる**。

```text
docs/          … 人間が読む設計書  ＝  Agentが読む仕様
docs/decisions/… 人間が読む決定履歴 ＝  Agentが参照する制約
CLAUDE.md      … 領域ごとの規約     ＝  Agentの行動規則
```

### Status タグが Agent の行動を制御する

設計書 §41 の Living Document ステータスに、**実行可能な意味**を与える。

| Status | Development Agent の許可 |
|---|---|
| `DRAFT` | **実装禁止。** 調査・議論・草案作成のみ |
| `DISCUSSION` | **実装禁止。** 選択肢の比較検討のみ |
| `DECIDED` | **実装可能。** |
| `IMPLEMENTED` | 変更にはADRの更新が必要 |
| `VALIDATED` | 変更にはADR更新＋再評価が必要 |
| `DEPRECATED` | 参照禁止 |

> これにより「仮説の段階なのにAgentが実装を始めてしまう」事故を防ぐ。
> §36 Rule 8「できることと仮説を明確に分ける」の実装形態である。

### ADR は Agent が「なぜ」を読む場所

Agentはコードから *what* は読めるが、*why* は読めない。
ADRがないと、Agentは過去の決定を悪気なく覆す。

**ルール：`Consequences` に影響範囲を明記し、REVIEW時にADR整合を確認する。**

---

## 12.5 Development Agent の構成と権限境界

設計書 §21 のAgent構成に、**触ってよい範囲**を定義する。運用系の Agent Permissions と同じ考え方を使う。

| Agent | 主担当ディレクトリ | 変更してよい | 変更してはいけない |
|---|---|---|---|
| **Architecture** | `docs/` | 設計書・ADR・IF定義 | 実装コード全般 |
| **IoT** | `iot/` | ファームウェア・センサー・MQTT | Backend内部・Safety |
| **Backend** | `backend/` | API・DB・Ingestion | 制御ループ・Safety |
| **Frontend** | `frontend/` | UI・可視化 | Backend内部実装 |
| **ML** | `ml/` | 前処理・検知ロジック・評価 | 制御系全般 |
| **Control** | `control/`, `simulation/` | PIDロジック・シミュレータ | **安全境界の定数** |
| **QA** | `tests/` | テスト全般 | **実装コード** |

### 2つの絶対境界

1. **安全境界の定数はHumanのみが変更できる。**
   最大ポンプ出力、水位のhard limit、最大連続運転時間、緊急停止条件。
   → §36 Rule 6「Safety Layerを必ず通す」の実装。

2. **QA Agent は実装コードを変更できない。**
   テストを通すために実装を歪める、という最も検出しにくい失敗モードを構造的に排除する。

---

## 12.6 2人 × N Agent の協働モデル

### 分担の考え方

Humanは**コードを書く人**ではなく、**Agentを運用しレビューする人**として配置する。

| | Member A | Member B |
|---|---|---|
| 担当領域 | System / IoT / Control | Intelligence / Application |
| 運用するAgent | Architecture, IoT, Backend, Control | ML, Agent, Frontend, QA |
| 責任 | 実世界に触れる層の安全性 | 判断・提示層の妥当性 |

### 衝突を防ぐ構造

```text
領域  =  ディレクトリ  =  ブランチ  =  担当Agent
```

境界面（API・データモデル・IF）は、**コード生成の前に `docs/` で合意する**（§36 Rule 3「コードより先にInterfaceを決める」）。
Interface-first であるため、2人のAgentが並行生成しても統合時に破綻しない。

### 共有領域（両者レビュー必須・Agent単独生成を禁止）

- Requirements / Architecture / Data Model / API
- Agent権限定義 / Safety
- 最終提案・発表資料

---

## 12.7 Agentic SDLC の評価

「Agentic SDLCを採用した」という主張は、測定できなければ提案として成立しない。
以下を計測し、設計書 §31 Evaluation に組み込む。

| 指標 | 意味 |
|---|---|
| Gate初回通過率 | Agent生成物がVERIFYを一度で通った割合 |
| Human修正率 | Agent生成行のうち人が書き換えた割合 |
| SOLVEエスカレーション率 | 3回失敗してHumanに戻った割合 |
| Issue→PR リードタイム | 1タスクの所要時間 |
| ADR整合違反の検出数 | 過去の決定を覆しかけた回数 |
| Humanレビュー時間 | 人的コストの実測 |

> 運用系のKPIが「養殖者の監視時間削減」であるのと対応して、
> 開発系のKPIは「開発者のレビュー時間削減」となる。**同じ主張を2つの層で検証する。**

---

## 12.8 リスクと対策

| リスク | 対策 |
|---|---|
| Agentのhallucination | 生成物は必ずテストとセット。Verification Gate必須 |
| 過去の決定からの逸脱 | ADR参照を義務化。REVIEWでADR整合を確認 |
| コンテキストの肥大 | `docs/` を領域分割し、領域ごとに `CLAUDE.md` を置く |
| 2人の認識差 | ADR ＋ 共有領域の両者レビュー |
| Agentへの過信 | 安全境界の定数と安全判断はHuman専有 |
| Agentが同じ失敗を反復 | SOLVE試行回数を3回に制限 |
| テストの形骸化 | QA Agentから実装コードへの書き込み権限を剥奪 |

---

## 12.9 提案としての主張

```text
このプロジェクトは、
「Agentが人間の監督下で現実世界を操作するシステム」を、
「Agentが人間の監督下でコードベースを構築するプロセス」によって開発する。

Observe → Decide → Verify → Act → Observe

というループを、養殖池とコードベースの両方に適用する。
```

**プロダクトと開発プロセスが同一の設計思想で貫かれていること**が、本提案の独自性である。
