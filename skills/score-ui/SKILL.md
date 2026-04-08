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

## 手順

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
7. **推奨アクション**: 次に実行すべきコマンドスキルを提示

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

## 推奨アクション
- ...
```
