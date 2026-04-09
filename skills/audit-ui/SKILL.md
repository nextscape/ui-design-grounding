---
name: audit-ui
description: UIの技術品質を監査する。アクセシビリティ・パフォーマンス・テーミング（トークン）・レスポンシブ・アンチパターンの5次元でスコアリングし、具体的な改善箇所を特定する。UI監査・品質チェック・技術レビューを依頼されたときに使用する。
user-invocable: true
argument-hint: "[対象 (画面、コンポーネント、機能...)]"
---

# audit-ui

## 準備（MANDATORY PREPARATION）

ui-design-grounding スキルを呼び出し、以下のリファレンスを読み込む:

- `ui-design-grounding/reference/accessibility.md`
- `ui-design-grounding/reference/implementation.md`
- `ui-design-grounding/reference/responsive-design.md`
- `ui-design-grounding/reference/color-system.md`
- `ui-design-grounding/reference/design-tokens.md`
- `ui-design-grounding/reference/anti-patterns.md`

## 手順

1. **対象の確認**: 監査対象のUI・技術スタック・制約を把握する
2. **5次元スコアリング**（各0-4点、合計/20点）:
   - **アクセシビリティ**: コントラスト比、ARIA属性、キーボード操作、セマンティックHTML、フォーカス管理、`prefers-reduced-motion`対応
   - **パフォーマンス**: アニメーション対象プロパティ（transform/opacityのみか）、遅延読み込み、不要な再レンダリング、Core Web Vitals
   - **テーミング**: デザイントークン使用率、ハードコード値の有無、ダークモード対応、セマンティックトークンの適切さ
   - **レスポンシブ**: 固定幅の有無、タッチターゲットサイズ（44px最小）、オーバーフロー、入力方式対応（pointer/hover）
   - **アンチパターン**: AI slop指標の該当数、横断的アンチパターンの該当数
3. **問題の分類**: 各問題をP0-P3の重篤度で分類する
   - P0（Blocking）: タスク完了不能、重大なアクセシビリティ違反
   - P1（Major）: WCAG AA違反、重大な摩擦
   - P2（Minor）: 回避策あり、軽微な品質問題
   - P3（Polish）: 外観上の改善余地
4. **システム的問題の特定**: 複数箇所に共通する根本原因を洗い出す
5. **ポジティブな発見**: 良い実装・パターンも記録する
6. **推奨アクション**: 検出した問題を解決できるコマンドスキルを、優先度順に紐付けて提示する。各項目に対象スキル名・対応する問題件数・代表的な問題を含める

## 出力フォーマット

```markdown
## 監査結果サマリー

| 次元 | スコア(/4) | 主要な問題 |
|------|-----------|-----------|
| アクセシビリティ | X | ... |
| パフォーマンス | X | ... |
| テーミング | X | ... |
| レスポンシブ | X | ... |
| アンチパターン | X | ... |
| **合計** | **X/20** | |

## 問題一覧（重篤度順）

### P0（Blocking）
- ...

### P1（Major）
- ...

### P2（Minor）
- ...

### P3（Polish）
- ...

## システム的な問題
- ...

## ポジティブな発見
- ...

## 推奨アクション（優先度順）

<!-- 検出した問題から対応スキルを自動マッピングする。
     問題がないカテゴリのスキルは記載しない。
     マッピング参照:
       アクセシビリティ → /guard-ui (a11y), /arrange-ui (コントラスト・階層)
       パフォーマンス   → /optimize-ui
       テーミング       → /extract-ui (トークン抽出), /init-design (DESIGN.md未整備の場合)
       レスポンシブ     → /adapt-ui
       アンチパターン   → /slim-ui (過剰装飾), /calm-ui (ノイズ), /clarify-ui (文言)
       レイアウト問題   → /arrange-ui
       タイポグラフィ   → /typeset-ui
-->

1. `/[スキル名]` — [対応する問題N件]（[代表的な問題の要約]）
2. `/[スキル名]` — [対応する問題N件]（[代表的な問題の要約]）
3. ...

> 上から順に実行することを推奨。「N番やって」で該当スキルを実行できます。
```
