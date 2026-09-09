# 発表資料 — Smart Shrimp Pond Management System

**Status:** `DECIDED`
**想定時間:** 15〜18分（20枚 / 1枚あたり約45秒）
**関連:** 設計書 §33, [DECISIONS.md](DECISIONS.md), [12_agentic-sdlc.md](12_agentic-sdlc.md)
**スライド:** https://claude.ai/code/artifact/266fd716-7359-496b-89e4-cdbf5f17df75

> このファイルが発表内容の**正**。スライドはここから生成する。
> 操作：`←` `→` 移動 / `N` 発表者ノート / `O` スライド一覧 / `Esc` 閉じる

---

## Slide 1 — Title

**Smart Shrimp Pond Management System**

From Monitoring to Intelligent Aquaculture
「見る養殖」から「予測し、考え、動く養殖」へ

チーム2名 / 2026-09-09

> **Note:** 提案するのは完成品ではなく、2か月で実現可能な設計であることを最初に明示する。

---

## Slide 2 — Background

- エビ養殖は水質が生存率・成長率・生産量を直接左右する
- 養殖環境は時間とともに連続的に変化する
- IoTによる計測の導入は進んでいる

**しかし、多くのシステムは「見る」で止まっている。**

> **Note:** 敵は「IoTがないこと」ではなく「IoTがあっても人の判断が減らないこと」。ここで問題の焦点をずらす。

---

## Slide 3 — Shrimp Farming Problem

| 課題 | 内容 |
|---|---|
| 常時監視 | 池の状態は24時間変化し続ける |
| 発見の遅れ | 異常に気づいた時には手遅れのことがある |
| 属人性 | 判断が経験に依存し、継承・再現できない |
| 予測不能 | 「これからどうなるか」が分からない |

> これは **Monitoring Problem ではなく、Decision & Control Problem である。**

> **Note:** この一行が提案全体の軸。ここだけは必ず言い切る。

---

## Slide 4 — Current Workflow

```text
養殖池 → センサー → 養殖者 → 経験・判断 → ポンプ操作 → 養殖池
                      ▲
                 人がループの中心
```

**人がループの中に常駐している限り、人がボトルネックになる。**

> **Note:** 図の「人」を指しながら話す。次のスライドで人の位置を動かすことを予告する。

---

## Slide 5 — Our Goal

```text
高品質  ×  大量生産  ×  養殖者の負担軽減
```

**人を「監視者」から「監督者」へ移す。**

- Primary: 高品質なエビを安定して大量生産できる養殖環境
- Secondary: 監視・分析・判断・操作の自動化と支援

> **Note:** 「人を排除する」ではなく「人の位置を変える」。ここを誤解されると安全性の質問が来る。

---

## Slide 6 — Proposed Concept

```text
Observe → Collect → Understand → Predict → Decide → Control → Observe ↺
```

観測して終わりではなく、**理解し、予測し、判断し、動き、結果を再び観測する**閉じたループ。

> **Note:** ここで初めてシステムの全体像の「形」を出す。詳細は後続スライド。

---

## Slide 7 — IoT Sensors

### 今回のスコープ

| Parameter | Sensor | 用途 |
|---|---|---|
| pH | pH Sensor | 水質状態 |
| Temperature | Temperature Sensor | 成長環境 |
| TDS | TDS Sensor | 溶解物質 |
| Turbidity | Turbidity Sensor | 濁度 |
| **Water Level** | **Ultrasonic** | **制御対象** |

### Phase 2

DO（溶存酸素）/ Salinity / ORP / Ammonia / Weather / Camera

> **Note:** DOが今回入っていないことを**こちらから先に言う。** 「斃死に直結する主要因子であり、Phase 2で追加する」と明言すれば、質問ではなく設計判断になる。

---

## Slide 8 — System Architecture

```text
              Sensors
                 ↓
           Edge / Gateway  ← PIDはここで動く
                 ↓
            Data Platform
                 ↓
    ┌────────────┼────────────┐
    ▼            ▼            ▼
Dashboard   Detection      Agent
                 │            │
                 └─────┬──────┘
                       ▼
                  Safety Layer
                       ▼
                   PID → Pump → Pond
```

> **Note:** Safety Layer が Agent の**下**にあることを強調。これが後半の安全性の論拠になる。

---

## Slide 9 — Data Flow

```text
Sensor → MQTT → Ingestion → Validation → DB → API → Dashboard / Detection / Agent
```

### サンプリングは2階層

| 用途 | 頻度 | 実行場所 |
|---|---|---|
| 制御用（水位） | 1 Hz | Edge内で完結 |
| 記録用（水質） | 1分 | 全件をDBへ |

> **Note:** 「なぜ2階層か」＝制御と記録で必要な時間分解能が2桁違うから。技術的な詰めができている印象を与える箇所。

---

## Slide 10 — Machine Learning

### 現状

**学習に使える実データが存在しない。**

### 判断

> データがない段階で、学習済みモデルの精度を主張しない。

- 予測・検知層は**差し替え可能なインターフェース**として設計
- **Phase 1（今回）：ルールベース／統計**
  レンジ・移動平均±3σ・変化率・stuck値・欠損検出
- Phase 2：環境予測モデル（データ蓄積後にIFを変えず差し替え）
- Phase 3：成長・生産予測

> **Note:** ここは弱点ではなく**誠実さの提示**として話す。「MLをやります（実体なし）」より強い。ADR-003。

---

## Slide 11 — PID Control

### 制御対象は水位のみ

| Parameter | ポンプで動くか | 応答 | PID適性 |
|---|---|---|---|
| **水位** | ◎ 直接・単調 | 分オーダー | **◎** |
| pH / TDS / 濁度 | △ 水交換で間接 | 時間オーダー | ✕ |
| 温度 | ✕ 外気・日射が支配 | — | ✕ |

| 項目 | 値 |
|---|---|
| Input | 水位 [cm] |
| Output | ポンプduty [-100〜+100%]（正＝給水 / 負＝排水） |
| Setpoint | 目標水位 [cm] |

**PIDはEdgeで実行する** → 通信断でも制御が継続する

> **Note:** 「なぜpHをPIDで制御しないのか」への回答を先に置いている。ADR-001 / ADR-002。

---

## Slide 12 — AI Agent

**複数の情報源とツールを使い、状況を理解し、次の行動を計画する意思決定オーケストレーター**

```text
get_current_sensor_data()   get_prediction()
get_historical_data()       get_alerts()
get_pump_status()           set_pump_target()  ← 要承認
```

### 自律レベル

| Level | 内容 | 今回 |
|---|---|---|
| 1 | Monitoring | |
| 2 | Recommendation | ✅ |
| 3 | Supervised Control | ✅ |
| 4 | Autonomous Control | 将来 |

> **Note:** Agentが送れるのは**setpointだけ**。ポンプ出力そのものには到達できない。

---

## Slide 13 — Closed-loop Architecture

```text
Agent → Safety Layer → PID → Pump → Pond → Sensor
  ▲     ├ Range Check                          │
  │     ├ Rate Limit                           │
  │     ├ Runtime Limit                        │
  │     ├ Emergency Stop                       │
  │     └ Human Override                       │
  └───────────────────────────────────────────┘
```

**Agentが誤った判断をしても、ポンプを危険な状態にできない構造。**

> **Note:** 安全性の質問はここで回収する。「Agentが暴走したら？」→ Safety Layerは Agent の下位にあり、Agentは迂回できない。

---

## Slide 14 — Human + AI

| 層 | 責務 |
|---|---|
| Sensor | Observe |
| Data Platform | Remember |
| Detection / ML | Predict |
| Agent | Decide |
| Safety Layer | Bound |
| PID | Control |
| Pump | Act |
| **Farmer** | **Supervise** |

> 人間が池を常に監視するのではなく、システムが継続的に観測・理解・予測し、
> **人間には重要な判断だけを求める。**

> **Note:** Slide 4 の「人がループの中心」と対比させる。人の位置が変わったことを示す。

---

## Slide 15 — Agentic SDLC ★

**運用系と開発系に、同一のアーキテクチャ原則を適用する。**

```text
【運用】 Farmer → Agent → Safety Layer      → PID → Pump  → Pond
【開発】 Human  → Agent → Verification Gate → CI  → Merge → Codebase
```

| 原則 | 運用系 | 開発系 |
|---|---|---|
| 直接操作させない | ポンプを直接叩かせない | mainに直接コミットさせない |
| 境界を必ず通す | Safety Layer | テスト・Lint・ADR整合 |
| Human Override | 手動運転・緊急停止 | レビュー差し戻し・Revert |
| 段階的自律 | Level 1→4 | Level 1→4 |

> Agentic SDLCを採用する理由は生産性ではなく、**プロダクトの設計思想との一貫性**である。

> **Note:** 本提案の中核。ここに最も時間を使う（90秒）。「流行だから使う」ではないことを言い切る。

---

## Slide 16 — Development Agents

| Agent | 変更してよい | 変更してはいけない |
|---|---|---|
| Architecture | docs・ADR | 実装コード |
| IoT | firmware・MQTT | Backend内部・Safety |
| Backend | API・DB | 制御ループ |
| Frontend | UI | Backend内部 |
| ML | 検知ロジック | 制御系 |
| Control | PID・シミュレータ | **安全境界の定数** |
| QA | テスト | **実装コード** |

### 2つの絶対境界

1. **安全境界の定数はHumanのみ変更できる**
2. **QA Agentは実装コードを変更できない** — テストを通すために実装を歪める失敗を構造的に排除

### Statusが実装権限を決める

`DRAFT` / `DISCUSSION` → 実装禁止　|　`DECIDED` → 実装可能

> **Note:** 「AIに書かせる」ではなく「AIに権限境界を設ける」という話。ここが他チームとの差になる。

---

## Slide 17 — 2-Month Roadmap

| Week | 内容 |
|---|---|
| 1 | Research / Architecture / ADR |
| 2 | IoT: センサー・ESP32・MQTT |
| 3 | Backend: DB・API・Ingestion |
| 4 | Frontend: Dashboard |
| 5 | Anomaly Detection Service（IF確定＋ルールベース） |
| 6 | Control: PID・Safety・シミュレーション |
| 7 | Agent: ツール・判断・承認フロー |
| 8 | 統合・障害試験・評価・発表 |

> **Note:** Week 5 が「MLモデル構築」でないことに触れる。ADR-003の結果であり、空いた工数はWeek 7-8とバッファへ。

---

## Slide 18 — MVP

### 検証環境：ベンチスケール実機

実際の養殖池は用いず、**センサー実機＋給排水ポンプ**で検証する。

```text
実センサー → MQTT → Backend → DB → Agent → Safety → PID(Edge) → 実ポンプ
     ▲                                                             │
     └──────────────── 実際に水位が変化する ─────────────────┘
```

> 動かない大きな構想ではなく、**小さくても実機で1周するループ**を成果物とする。

デモ：外乱（手動で水を抜く）→ 検知 → Agent提案 → 承認 → PID復帰

> **Note:** 「実池がない」を弱点として言わず、「実機で閉ループが完結する範囲を選んだ」と説明する。デモで外乱を入れると閉ループが動いていることが目で分かる。

---

## Slide 19 — Evaluation

| 層 | 指標 |
|---|---|
| **技術** | センサー精度・データ欠損率・APIレイテンシ・制御安定性 |
| **運用** | 監視時間・手動操作回数・異常の見逃し |
| **開発プロセス** | Gate初回通過率・Human修正率・レビュー時間 |

> 運用系KPIが「養殖者の監視時間削減」であるのと対応して、
> 開発系KPIは「開発者のレビュー時間削減」となる。
> **同じ主張を2つの層で検証する。**

> **Note:** Agentic SDLCを「やった」で終わらせず測定対象にしている点を強調。

---

## Slide 20 — Future Vision

```text
Phase 1  Monitoring
Phase 2  Prediction
Phase 3  Decision Support     ← 2か月の到達点
Phase 4  Supervised Automation
Phase 5  Autonomous Aquaculture
Phase 6  Multi-Pond Optimization
Phase 7  Farm-level Optimization
```

> **人間が養殖池を常に監視するのではなく、**
> **システムが継続的に観測・理解・予測し、人間には重要な判断だけを求める。**

> **Note:** 締め。Slide 5 のGoalに戻って終わる。

---

## 想定Q&A

| 質問 | 回答 |
|---|---|
| DOがないのはなぜ？ | 斃死の主要因子と認識している。今回は制御対象（水位）と観測範囲を絞り、Phase 2で追加する。§29に限界として明記済み |
| MLは結局作らないのか？ | 学習データがないため精度を主張しない。IFを固定しPhase 1はルールベース。データ取得後にIFを変えず差し替える |
| Agentが暴走したら？ | Safety LayerはAgentの下位にあり迂回できない。Agentが送れるのはsetpointのみ。PIDはEdgeで動き通信断でも継続 |
| なぜpHを制御しない？ | 給排水ポンプで直接・単調・高速に応答するのは水位のみ。pHは水交換を介した間接影響で時間オーダー。Agentの判断材料として扱う |
| 実池がなくて評価できるのか？ | 水位制御に関しては実機で閉ループが完結する。実池特有の条件はPhase 2 |
| Agentic SDLCの効果は？ | Gate初回通過率・Human修正率・レビュー時間を計測する（Slide 19） |
| 2人・2か月で終わるのか？ | MVPをベンチスケール閉ループに限定し、ML学習をPhase 2へ送ることで工数を確保した |
