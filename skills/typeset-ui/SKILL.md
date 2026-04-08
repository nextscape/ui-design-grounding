---
name: typeset-ui
description: UIのタイポグラフィを修正・改善する。フォント選定・階層設計・サイズスケール・垂直リズム・可読性を最適化する。文字が読みにくい・フォントが合わない・テキスト階層が不明確なときに使用する。
user-invocable: true
argument-hint: "[対象 (画面、コンポーネント、機能...)]"
---

# typeset-ui

## 準備（MANDATORY PREPARATION）

ui-design-grounding スキルを呼び出し、以下のリファレンスを読み込む:

- `ui-design-grounding/reference/typography.md`
- `ui-design-grounding/reference/accessibility.md`
- `ui-design-grounding/reference/spatial-layout.md`

## 手順

### 1. 現在のタイポグラフィを診断

- 使用フォントの一覧と評価（使い古されたフォントでないか）
- サイズスケール: 段階数、比率、最小値/最大値
- ウェイトの使い分け: 何種類使われているか
- 行間（line-height）: 適切か、本文とダーク背景で異なるか
- 行長（max-width）: 65ch以内に収まっているか
- 垂直リズム: ベースライン単位が統一されているか

### 2. 問題の特定

- 使い古されたフォント（Inter, Roboto, Arial, Open Sans）
- サイズ段階が多すぎる（5段階を超える）or 差が小さすぎる
- 垂直リズムの破綻（余白がバラバラ）
- 可読性の問題（行が長すぎる、行間が狭すぎる）
- ウェイトの使い分けが不明確
- モノスペースの安易な使用

### 3. 改善の実施

**フォント選定**:
- typography.md の推奨リストから代替を提案
- ペアリング: コントラスト複数軸（serif+sans 等）
- 1ファミリーのウェイト違いで十分か検討
- システムフォントの検討（性能重視の場合）

**スケール**:
- 5段階モジュラースケールの適用
- 比率選択: 1.25（控えめ）/ 1.333（標準）/ 1.5（大胆）
- Fluid Type: アプリUI = 固定rem、マーケティング = `clamp()`

**垂直リズム**:
- ベースライン単位の決定（例: 24px line-height → 余白は24の倍数）
- セクション間、要素間の余白をベースライン単位に統一

**可読性**:
- `max-width: 65ch` の適用
- 行間の最適化（本文: 1.5-1.7、見出し: 1.1-1.3）
- ダーク背景: line-height を +0.05-0.1

**OpenType機能**:
- データテーブル: `font-variant-numeric: tabular-nums`
- 略語: `font-variant-caps: all-small-caps`

### 4. Webフォント性能の確認

- `font-display: swap` が設定されているか
- メトリクスマッチング（`size-adjust` 等）でレイアウトシフトを防いでいるか
- 不要なウェイト・文字セットをロードしていないか

### 5. アクセシビリティ確認

- [ ] フォントサイズが rem/em 単位か（px ではなく）
- [ ] 本文が最小 16px 以上か
- [ ] `user-scalable=no` を使っていないか
- [ ] タップターゲットが 44px 以上か

## 推奨される次のステップ

タイポグラフィ修正後、以下のコマンドスキルの実行を検討する:
- `/arrange-ui` — タイポグラフィ変更に伴う余白・レイアウトの再調整
- `/audit-ui` — 変更後の技術品質を監査
