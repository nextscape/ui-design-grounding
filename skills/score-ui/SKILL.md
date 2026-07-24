---
name: score-ui
description: UXヒューリスティクス評価を実施する。ニールセンの10ヒューリスティクスで採点し、ペルソナベースのレッドフラグテストを行い、UX品質を構造的に評価する。UXレビュー・ヒューリスティクス評価・ユーザビリティ評価を依頼されたときに使用する。
user-invocable: true
argument-hint: "[対象 (画面、コンポーネント、機能...)]"
---

# score-ui

## 準備（MANDATORY PREPARATION）

ui-design-grounding スキルを呼び出し、以下のリファレンスを読み込む:

- `ui-design-grounding/reference/usability.md`
- `ui-design-grounding/reference/cognitive.md`
- `ui-design-grounding/reference/information-arch.md`
- `ui-design-grounding/reference/interaction.md`
- `ui-design-grounding/reference/wording.md`
- `ui-design-grounding/reference/typography.md`
- `ui-design-grounding/reference/color-system.md`
- `ui-design-grounding/reference/anti-patterns.md`
- `ui-design-grounding/reference/playwright.md`
- `ui-design-grounding/reference/ui-report.md`
- `ui-design-grounding/reference/design-md-gate.md`

## 手順

### 0. 基準の確認（DESIGN.md 前段ゲート）

`design-md-gate.md` の **前段ゲート** を実施する。DESIGN.md があれば front matter のトークンと本文の指針・用語規約を**基準として読み込み**（リファレンスより優先）、以降のヒューリスティクス評価（特に H4 一貫性・用語の整合）はこの基準に照らして行う。無ければ未整備として扱い `/init-design`（外部 URL からは `/scan-ui`）を提案する。評価専用のため後段ゲートは不要。

1. **コンテキスト確認と実地観察**: 対象UIの目的・想定ユーザー・利用状況を確認する。`playwright.md` の**準備（共通）**を実施し、対象画面を Playwright MCP で開く。**採点・レッドフラグ判定は実際に操作した結果を根拠にする**（状態視認性・エラー防止・エラー回復は再現して初めて評価できる）。MCP が使えなければその旨を明示し、操作を要する項目は「未検証」として扱う。特に以下は `playwright.md` の**状態のトリガ**に従い実際に発火させる: H1（状態の視認性）→ 処理を発火してフィードバックを観察 / H5・H9（エラー防止・回復）→ 不正入力・失敗を実際に起こす / H6（記憶より認識）→ 初回導線を実際に踏む。
2. **ニールセン10ヒューリスティクス採点**（各0-4点、合計/40点）:
   - H1. システム状態の視認性
   - H2. システムと現実世界の一致
   - H3. ユーザーの制御と自由
   - H4. 一貫性と標準
   - H5. エラー防止
   - H6. 記憶よりも認識
   - H7. 柔軟性と効率性
   - H8. 美的でミニマルな設計
   - H9. エラーの認識・診断・回復の支援
   - H10. ヘルプとドキュメント
3. **評価帯の判定**:
   - 36-40: Excellent
   - 28-35: Good
   - 20-27: Acceptable
   - 12-19: Needs Improvement
   - 0-11: Critical
4. **ペルソナベーステスト**（対象に合わせて2-3人選択）: 各ペルソナの導線を**実際に Playwright MCP で歩いて**レッドフラグを検出する（思考実験で済ませない）。
   - 初回利用者（Jordan）: ガイダンス、ラベルの明確さ、初回タスク完了 → 予備知識なしで主要タスクを実際に完了できるか操作する
   - 熟練者（Alex）: 効率性、ショートカット、バルクアクション → 反復操作・一括操作の手数を実測
   - アクセシビリティ依存（Sam）: ARIA、コントラスト、キーボード、フォーカス → `browser_snapshot` で ARIA を確認、**一括監査スイープ**でコントラスト等を回収し、`browser_press_key`（Tab）で全導線を辿ってフォーカス可視性を確認
   - モバイル専用（Casey）: タッチターゲット、サムゾーン、低速接続 → `browser_resize`（320/768px）でタッチターゲット・オーバーフローを確認
   - ストレステスター（Riley）: エッジケース、並行操作、エラー回復 → 不正入力・連打・失敗を実際に起こしてエラー回復を確認
5. **認知負荷アセスメント**: cognitive.md の違反パターン（選択肢の壁、メモリブリッジ等）を検出
6. **課題の分類**: P0-P3の重篤度で整理
7. **推奨アクション**: 検出した課題を解決できるコマンドスキルを、優先度順に紐付けて提示する。各項目に対象スキル名・対応する課題件数・代表的な課題を含める
8. **レポート保存**: `ui-report.md` に従い、評価対象プロジェクトの `.design/reports/YYYY-MM-DD/HHmmss-score-ui.md` に詳細レポートを保存する。スクリーンショットを取得した場合は `.design/reports/YYYY-MM-DD/screenshots/` に保存し、レポート本文から相対リンクする。会話内の最終応答では、要約・最優先アクション・保存先を短く示す
9. **フォローアップ**: `ui-report.md` の「指摘のフォローアップ（タスク管理への接続）」を必ず実施する — この実行内で修正しない P0/P1 指摘は、プロジェクトのタスク管理へ登録するか、最終応答で登録を提案する

## 出力フォーマット

```markdown
# score-ui レポート: <対象>

| 項目 | 内容 |
|---|---|
| スキル | `score-ui` |
| 対象 | <画面・コンポーネント・機能> |
| 実施日時 | <ISO 8601形式のローカル日時> |
| DESIGN.md | あり / なし / 未確認 |
| 観察方法 | Playwright MCP / 未実施（理由） |
| レポート保存先 | `.design/reports/YYYY-MM-DD/HHmmss-score-ui.md` |

## スクリーンショット

| # | 内容 | パス |
|---|---|---|
| 1 | <画面・状態・幅など> | [screenshots/HHmmss-score-ui-01.png](screenshots/HHmmss-score-ui-01.png) |

<!-- 取得していない場合は「なし」と書く。 -->

## UX評価サマリー

| 項目 | 内容 |
|---|---|
| 評価帯 | Excellent / Good / Acceptable / Needs Improvement / Critical |
| 合計スコア | X/40 |
| 主要UXリスク | <最も影響が大きいUXリスクを1文で書く> |
| P0/P1件数 | P0: X件 / P1: X件 |
| 次にやること | <最優先で実行すべき対応を1文で書く> |

## ヒューリスティクス評価

| # | ヒューリスティクス | スコア(/4) | 所見 |
|---|------------------|-----------|------|
| H1 | システム状態の視認性 | X | ... |
| H2 | システムと現実世界の一致 | X | ... |
| ... | ... | ... | ... |
| **合計** | | **X/40** | **評価帯** |

## ペルソナテスト結果

### [ペルソナ名]
- 観察した導線: ...
- レッドフラグ: ...
- 影響: ...
- 推奨改善: ...
- 関連スクリーンショット: [screenshots/HHmmss-score-ui-01.png](screenshots/HHmmss-score-ui-01.png) / なし

## 認知負荷の問題
- **<問題名>**
  - 根拠: ...
  - 影響: ...
  - 推奨対応: ...

## 課題一覧

### P0（Blocking）
- **<課題名>**
  - 根拠: <観察・操作結果・スクリーンショット・コード位置など>
  - 影響: <ユーザーまたはUXへの影響>
  - 推奨対応: <対応方針>
  - 関連スクリーンショット: [screenshots/HHmmss-score-ui-01.png](screenshots/HHmmss-score-ui-01.png) / なし

### P1（Major）
- **<課題名>**
  - 根拠: ...
  - 影響: ...
  - 推奨対応: ...
  - 関連スクリーンショット: ...

### P2（Minor）
- **<課題名>**
  - 根拠: ...
  - 影響: ...
  - 推奨対応: ...
  - 関連スクリーンショット: ...

### P3（Polish）
- **<課題名>**
  - 根拠: ...
  - 影響: ...
  - 推奨対応: ...
  - 関連スクリーンショット: ...

## 推奨アクション（優先度順）

<!-- 検出した課題から対応スキルを自動マッピングする。
     課題がないカテゴリのスキルは記載しない。
     マッピング参照:
       H1 システム状態の視認性 → /animate-ui (フィードバック), /clarify-ui (状態表示文言)
       H2 現実世界との一致    → /clarify-ui (用語・ラベル)
       H3 制御と自由          → /guard-ui (取り消し・エスケープ)
       H4 一貫性と標準        → /extract-ui (トークン統一), /arrange-ui (レイアウト統一)
       H5 エラー防止          → /guard-ui (バリデーション), /clarify-ui (ガイダンス)
       H6 記憶よりも認識      → /arrange-ui (視覚階層), /slim-ui (情報量削減)
       H7 柔軟性と効率性      → /adapt-ui (マルチデバイス), /optimize-ui (パフォーマンス)
       H8 美的でミニマルな設計 → /slim-ui (削ぎ落とし), /calm-ui (ノイズ低減), /typeset-ui (タイポグラフィ)
       H9 エラー回復の支援    → /clarify-ui (エラーメッセージ), /guard-ui (リカバリーパス)
       H10 ヘルプ             → /clarify-ui (ガイダンス・空状態)
       認知負荷の問題         → /slim-ui (選択肢の壁), /arrange-ui (グルーピング)
-->

1. `/[スキル名]` — [対応する課題N件]（[代表的な課題の要約]）
2. `/[スキル名]` — [対応する課題N件]（[代表的な課題の要約]）
3. ...

> 上から順に実行することを推奨。「N番やって」で該当スキルを実行できます。

## 未検証・制約

- <未検証項目または制約。なければ「なし」>
```
