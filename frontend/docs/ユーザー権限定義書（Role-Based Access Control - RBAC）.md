# ユーザー権限定義書

**Role-Based Access Control / RBAC**

## 1. 目的

Smart Shrimp Pond Management Systemにおける、ユーザーRoleと基本的な権限範囲を定義する。

本書では以下を定義しない。

- 業務フロー
- 画面構成
- UI/UX
- KPI・データ項目の詳細
- AIの分析・予測ロジック
- システム内部の実装

これらは後続のドキュメントで定義する。

---

## 2. ユーザーRole

本システムでは、以下の3 Roleを業務ユーザーとして定義する。

| Role                  | 役割                                                   |
| --------------------- | ------------------------------------------------------ |
| **Farms Manager**     | 複数のFarmを統括し、養殖事業全体の経営・生産判断を行う |
| **Technical Manager** | 担当Farm・Pondの状態を管理し、技術・運用上の判断を行う |
| **Field Operator**    | Farm・Pondで必要な現場作業を実行する                   |

---

## 3. 権限

ユーザー権限は以下の4種類とする。

| Permission      | 意味                           |
| --------------- | ------------------------------ |
| **View**        | 情報を閲覧する                 |
| **Acknowledge** | Alert / Notificationを確認する |
| **Decide**      | 業務上の判断を行う             |
| **Execute**     | 許可された操作・作業を実行する |

---

## 4. Role × Permission

| 対象                 | Farms Manager                | Technical Manager           | Field Operator |
| -------------------- | ---------------------------- | --------------------------- | -------------- |
| Company情報          | View / Decide                | -                           | -              |
| Farm情報             | View / Decide                | View                        | -              |
| Pond情報             | View                         | View / Decide               | -              |
| IoT情報              | View（集約値・分析結果のみ） | View（担当Farmの詳細値）    | -              |
| Alert / Notification | Acknowledge                  | View / Acknowledge / Decide | -              |
| AI Recommendation    | View / Decide                | View / Decide               | -              |
| 生産・収穫情報       | View / Decide                | View / Decide               | -              |
| 在庫・供給情報       | View / Decide                | View                        | -              |
| 現場作業             | -                            | Decide                      | Execute        |
| Actuator操作         | -                            | Execute\*                   | -              |

\* Technical ManagerのActuator操作は、Safety Layer等による許可範囲内に限定する。

---

## 5. データアクセス範囲

### Farms Manager

- Company全体の情報を閲覧できる
- 管轄する複数Farmの情報を閲覧できる
- FarmからPondへ集約KPI、分析根拠、トレンド要約を参照できる
- 生センサー値・詳細な現場入力値は閲覧できない
- Pondの現場操作は行わない

### Technical Manager

- 担当Farmの情報を閲覧・管理できる
- 担当Farm内のPond情報を閲覧できる
- 担当Farm内のPondについて、IoT詳細値、日次・週次入力、アラート詳細を閲覧できる
- Pondの技術・運用上の判断を行う
- 許可された範囲でActuatorを操作できる

### Field Operator

- 担当する現場作業を実行する
- 経営・分析情報へのアクセスは原則として必要としない

---

## 6. AIに関する権限

AIは**意思決定を支援するRecommendationを提供する**。

AIが直接Actuatorを操作する権限は持たない。

基本的な責任分担は以下とする。

```text
AI
  ↓
Recommendation
  ↓
Human Decision
  ↓
Technical Manager
  ↓
Safety Layer / PID
  ↓
Actuator
```

したがって、

- AI：Recommendation
- Farms Manager：経営・生産上のDecision
- Technical Manager：技術・運用上のDecision / 許可されたExecute
- Field Operator：現場作業のExecute

とする。

---

## 7. 権限設計原則

1. **最小権限**を原則とする
2. Roleごとに必要な情報・操作のみを提供する
3. 情報の閲覧と操作の権限を分離する
4. AIのRecommendationと人間によるDecisionを分離する
5. Actuator操作は安全機構の管理下に置く
6. 詳細な業務フロー・画面・UI/UXは本書では定義しない

---

## 8. 今後決定する事項

以下は後続の要件定義で決定する。

- Farms Managerの最終承認範囲
- Technical Managerが操作可能なActuatorの範囲
- Human Overrideの具体的な権限
- Field Operatorに提供するシステム機能
- Farm / Pondの担当者割当ルール
- AI Recommendationに対するDecisionの記録方法
