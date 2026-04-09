---
name: polish-ui
description: UIのリリース前最終仕上げを行う。視覚的アラインメント・タイポグラフィ・色・インタラクション状態・マイクロインタラクション・コンテンツ・フォーム・エッジケース・レスポンシブ・パフォーマンスをチェックリスト駆動で確認し、問題を実際に修正する。仕上げ・ポリッシュ・最終チェック・リリース前確認を依頼されたときに使用する。
user-invocable: true
argument-hint: "[対象 (画面、コンポーネント、機能...)]"
---

# polish-ui

## 準備（MANDATORY PREPARATION）

ui-design-grounding スキルを呼び出し、以下のリファレンスを読み込む:

- `ui-design-grounding/reference/interaction.md`
- `ui-design-grounding/reference/typography.md`
- `ui-design-grounding/reference/color-system.md`
- `ui-design-grounding/reference/spatial-layout.md`
- `ui-design-grounding/reference/anti-patterns.md`
- `ui-design-grounding/reference/accessibility.md`
- `ui-design-grounding/reference/responsive-design.md`
- `ui-design-grounding/reference/wording.md`
- `ui-design-grounding/reference/design-tokens.md`
- `ui-design-grounding/reference/motion-design.md`

## 注意

- **polish は最終ステップ**。機能が完成していない場合は先に完成させる
- review-ui（評価のみ）との違い: **polish-ui は問題を発見次第、実際に修正する**

## 手順

以下のチェックリストを順に確認し、問題があれば即座に修正する。

### 1. 視覚的アラインメント
- [ ] 要素の揃いが正確か（グリッド準拠）
- [ ] 余白の一貫性（4ptグリッド）
- [ ] 光学的調整（視覚的中央 vs 数学的中央）

### 2. タイポグラフィ
- [ ] フォント選定が適切か（使い古されたフォントでないか）
- [ ] 階層が明確か（5段階モジュラースケール）
- [ ] 垂直リズムが維持されているか
- [ ] 可読性（行長 65ch以内、適切な行間）

### 3. 色・コントラスト
- [ ] デザイントークンが使用されているか
- [ ] コントラスト比（テキスト 4.5:1、大テキスト 3:1、UI 3:1）
- [ ] ダークモード対応（該当する場合）
- [ ] 色背景にグレーテキストを使っていないか

### 4. インタラクション状態（8状態すべて）
- [ ] Default: 基本状態が明確
- [ ] Hover: カーソルホバー時の変化
- [ ] Focus: `:focus-visible` スタイル（2-3px、3:1コントラスト）
- [ ] Active: 押下時の視覚フィードバック
- [ ] Disabled: 操作不可の明示
- [ ] Loading: 処理中の表示（スケルトン or スピナー）
- [ ] Error: エラー状態の表示
- [ ] Success: 成功状態の表示

### 5. マイクロインタラクション
- [ ] ホバー・フォーカスのトランジション
- [ ] 状態変化のアニメーション（transform/opacityのみ）
- [ ] prefers-reduced-motion 対応

### 6. コンテンツ・コピー
- [ ] ボタンラベル（動詞+目的語、"OK"/"送信"を避ける）
- [ ] エラーメッセージ（何が起きたか→なぜ→どう直すか）
- [ ] 空状態のメッセージ
- [ ] 用語の一貫性

### 7. フォーム・入力
- [ ] ラベルが明確（プレースホルダーをラベル代わりにしていない）
- [ ] バリデーション（blur時）
- [ ] エラー表示（フィールド直下、aria-describedby）

### 8. エッジケース
- [ ] 長文テキストの処理（オーバーフロー、省略）
- [ ] 空データの表示
- [ ] 大量データの動作
- [ ] エラー時の動作

### 9. レスポンシブ
- [ ] 主要ブレイクポイントでの動作
- [ ] タッチターゲット（44px最小）
- [ ] オーバーフローなし

### 10. パフォーマンス
- [ ] アニメーションが transform/opacity のみか
- [ ] 画像の最適化（lazy loading、適切フォーマット）
- [ ] 不要なスタイル・コードの除去

## 出力フォーマット

修正を実施しつつ、以下のサマリーを提供する:

```markdown
## Polish 結果

### 修正済み
- [箇所]: [修正内容]

### 確認済み（問題なし）
- [チェック項目]

### 要検討（人間の判断が必要）
- [箇所]: [選択肢とトレードオフ]

### 推奨される次のステップ
- `/score-ui`（仕上げ後のUXヒューリスティクス最終評価 — ユーザビリティに悪影響がないか確認）
```
