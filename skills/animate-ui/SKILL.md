---
name: animate-ui
description: UIにモーション・アニメーションを追加する。エントランスアニメーション・マイクロインタラクション・状態遷移・ナビゲーション効果を目的を持って追加する。アニメーション追加・モーション設計・トランジション改善を依頼されたときに使用する。
user-invocable: true
argument-hint: "[対象 (画面、コンポーネント、機能...)]"
---

# animate-ui

## 準備（MANDATORY PREPARATION）

ui-design-grounding スキルを呼び出し、以下のリファレンスを読み込む:

- `ui-design-grounding/reference/motion-design.md`
- `ui-design-grounding/reference/interaction.md`
- `ui-design-grounding/reference/accessibility.md`

## 手順

### 1. アニメーション機会の特定

以下の観点で、アニメーションが効果的な箇所を洗い出す:

- 静的でフィードバックのない領域（クリックしても視覚的反応がない）
- 唐突な状態変化・画面遷移（パッと切り替わる）
- 要素間の関係性が不明確な箇所（どこから来たか分からない）
- ユーザーアクション後のフィードバック不足

### 2. アニメーション戦略の策定

- **シグネチャアニメーション**: 印象に残る1つの動きを決定
- **レイヤードアプローチ**:
  1. Hero: 最も目立つアニメーション（エントランス等）
  2. Feedback: ユーザー操作への応答（ホバー、クリック）
  3. Transition: 状態・画面の遷移
  4. Delight: 演出・楽しさ（控えめに）

### 3. カテゴリ別に実装

**エントランス**:
- フェードイン + スライド（500-800ms、ease-out-quint）
- スタガリング: `animation-delay: calc(var(--i) * 50ms)`

**マイクロインタラクション**:
- ボタンホバー: scale(1.02) + 色変化（100-150ms）
- トグル・チェック: scale + opacity（200-300ms）
- アイコンアニメーション: rotate, scale

**状態遷移**:
- タブ切り替え: クロスフェード（200-300ms）
- アコーディオン: `grid-template-rows: 0fr → 1fr`（300-500ms）
- モーダル: scale(0.95→1) + opacity（200-300ms）

**ナビゲーション効果**:
- ページ遷移: View Transitions API の検討
- パネル開閉: translate + opacity

**フィードバック**:
- 成功: チェックマークアニメーション
- エラー: シェイク（控えめに）
- ローディング: スケルトンシマー or スピナー

### 4. デュレーション・イージングの適用

motion-design.md の体系に従う:
- **イージング**: ease-out（入場）、ease-in（退出）、ease-in-out（トグル）
- **指数関数カーブ**: `cubic-bezier(0.22, 1, 0.36, 1)` 等
- バウンス/エラスティックは使わない

### 5. 検証

- [ ] `prefers-reduced-motion` 対応（必須）: クロスフェードで代替
- [ ] 60fps 性能: `transform` と `opacity` のみアニメーション
- [ ] 各アニメーションに明確な目的があるか
- [ ] 過剰になっていないか（全てが動いていないか）

## 推奨される次のステップ

アニメーション追加後、以下のコマンドスキルの実行を検討する:
- `/optimize-ui` — アニメーションのパフォーマンス影響を確認
- `/audit-ui` — アクセシビリティ（reduced-motion対応）を含む技術品質を監査
