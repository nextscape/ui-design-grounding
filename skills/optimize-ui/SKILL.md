---
name: optimize-ui
description: UIのパフォーマンスを最適化する。Core Web Vitals・レンダリング性能・アニメーション性能・バンドルサイズ・画像最適化を分析し改善する。パフォーマンス改善・速度向上・表示最適化を依頼されたときに使用する。
user-invocable: true
argument-hint: "[対象 (画面、コンポーネント、機能...)]"
---

# optimize-ui

## 準備（MANDATORY PREPARATION）

ui-design-grounding スキルを呼び出し、以下のリファレンスを読み込む:

- `ui-design-grounding/reference/implementation.md`
- `ui-design-grounding/reference/motion-design.md`
- `ui-design-grounding/reference/responsive-design.md`
- `ui-design-grounding/reference/typography.md`
- `ui-design-grounding/reference/design-md-gate.md`

## 手順

### 0. 基準の確認（DESIGN.md 前段ゲート）

`design-md-gate.md` の **前段ゲート** を実施する。DESIGN.md があればフォント・モーション等のトークン・規約を基準として読み込み（リファレンスより優先）、なければ未整備として扱い `/init-design` を提案する。基準なしに「問題なし」と判断しない。

### 1. 現状の確認

対象UIの技術スタック、既知のパフォーマンス問題、計測値（あれば）を把握する。

### 2. レンダリング性能

- 不要な再レンダリングの特定と修正
- コンポーネントのメモ化（React.memo、useMemo、useCallback）の検討
- 仮想スクロールの必要性判断（大量リスト）
- DOM要素数の削減

### 3. アニメーション性能

- `transform`/`opacity` 以外のアニメーションプロパティを特定し修正
- `will-change` の不適切な先行使用を特定し除去
- 高さアニメーション → `grid-template-rows: 0fr → 1fr` への変更
- 60fps維持の確認

### 4. 画像・アセット最適化

- 適切なフォーマット選択: WebP、AVIF（フォールバック付き）
- レスポンシブ画像: `srcset` + `sizes` の実装
- 遅延読み込み: `loading="lazy"`（ビューポート外の画像）
- 画像サイズの最適化（不要に大きい画像の特定）

### 5. フォント最適化

- `font-display: swap` の設定
- メトリクスマッチング: `size-adjust`、`ascent-override` でレイアウトシフト防止
- 使用しないウェイト・文字セットの除去
- システムフォントの検討（性能最優先の場合）

### 6. バンドル・ローディング

- コード分割の機会特定（ルートベース、コンポーネントベース）
- クリティカルCSSのインライン化
- 不要なCSS/JSの除去
- プリロード・プリフェッチの適用

### 7. Core Web Vitals 改善提案

- **LCP**（Largest Contentful Paint）: ヒーロー画像の最適化、フォント読み込み、サーバー応答時間
- **INP**（Interaction to Next Paint）: イベントハンドラの最適化、長いタスクの分割
- **CLS**（Cumulative Layout Shift）: 画像の寸法指定、フォントのメトリクスマッチング、動的コンテンツの領域確保

## 出力フォーマット

```markdown
## 最適化結果

### 改善項目
| 項目 | 対象 | 改善内容 | 影響度 |
|------|------|---------|-------|
| ... | ... | ... | 高/中/低 |

### Core Web Vitals への影響
- LCP: ...
- INP: ...
- CLS: ...

### 追加で検討すべき最適化
- ...
```

## DESIGN.md 後段ゲート

`design-md-gate.md` の **後段ゲート** を実施する。今回の最適化が DESIGN.md のトークン水準・規約（フォント・モーション等）を変えた場合は乖離を明示し、`/init-design` へ誘導する（DESIGN.md は自動では書き換えない）。

## 推奨される次のステップ

パフォーマンス最適化後、以下のコマンドスキルの実行を検討する:
- `/audit-ui` — 最適化後の技術品質を総合監査
- `/adapt-ui` — 最適化がモバイル等の各デバイスで有効か確認
