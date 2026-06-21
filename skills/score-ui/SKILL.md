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
- `ui-design-grounding/reference/design-md-gate.md` — DESIGN.md ゲート（前段）の手順

## 手順

### 0. 基準の確認（DESIGN.md 前段ゲート）

`design-md-gate.md` の **前段ゲート** を実施する。DESIGN.md があれば front matter のトークンと本文の指針・用語規約を**基準として読み込み**（リファレンスより優先）、以降のヒューリスティクス評価（特に H4 一貫性・用語の整合）はこの基準に照らして行う。無ければ未整備として扱い `/init-design`（外部 URL からは `/scan-ui`）を提案する。評価専用のため後段ゲートは不要。

1. **コンテキスト確認**: 対象UIの目的・想定ユーザー・利用状況を確認する
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
4. **ペルソナベーステスト**（対象に合わせて2-3人選択）:
   - 初回利用者（Jordan）: ガイダンス、ラベルの明確さ、初回タスク完了
   - 熟練者（Alex）: 効率性、ショートカット、バルクアクション
   - アクセシビリティ依存（Sam）: ARIA、コントラスト、キーボード、フォーカス
   - モバイル専用（Casey）: タッチターゲット、サムゾーン、低速接続
   - ストレステスター（Riley）: エッジケース、並行操作、エラー回復
5. **認知負荷アセスメント**: cognitive.md の違反パターン（選択肢の壁、メモリブリッジ等）を検出
6. **課題の分類**: P0-P3の重篤度で整理
7. **推奨アクション**: 検出した課題を解決できるコマンドスキルを、優先度順に紐付けて提示する。各項目に対象スキル名・対応する課題件数・代表的な課題を含める

## 出力フォーマット

```markdown
## ヒューリスティクス評価

| # | ヒューリスティクス | スコア(/4) | 所見 |
|---|------------------|-----------|------|
| H1 | システム状態の視認性 | X | ... |
| H2 | システムと現実世界の一致 | X | ... |
| ... | ... | ... | ... |
| **合計** | | **X/40** | **評価帯** |

## ペルソナテスト結果

### [ペルソナ名]
- レッドフラグ: ...
- 推奨改善: ...

## 認知負荷の問題
- ...

## 課題一覧（P0-P3）
- ...

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
```
