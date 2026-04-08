---
name: adapt-ui
description: UIをマルチデバイス・レスポンシブに適応させる。ブレイクポイント設計・入力方式対応・セーフエリア・レイアウト変形を実施する。レスポンシブ対応・モバイル対応・マルチデバイス対応を依頼されたときに使用する。
user-invocable: true
argument-hint: "[対象 (画面、コンポーネント、機能...)]"
---

# adapt-ui

## 準備（MANDATORY PREPARATION）

ui-design-grounding スキルを呼び出し、以下のリファレンスを読み込む:

- `ui-design-grounding/reference/responsive-design.md`
- `ui-design-grounding/reference/spatial-layout.md`
- `ui-design-grounding/reference/accessibility.md`
- `ui-design-grounding/reference/interaction.md`

## 手順

### 1. 現状分析

- ビューポート前提: 固定幅、特定デバイスサイズ依存がないか
- メディアクエリ: `min-width`（モバイルファースト）か `max-width`（デスクトップファースト）か
- 入力方式: hover依存の機能がないか
- セーフエリア: ノッチ・丸角への対応状況
- タッチターゲット: 44px最小を満たしているか

### 2. ブレイクポイント設計

- コンテンツ駆動: デバイスサイズではなく、コンテンツが壊れる地点で設定
- 通常3つで十分: ~640px, ~768px, ~1024px
- `clamp()` でブレイクポイント間のフルードスケーリング
- モバイルファースト（`min-width`）への統一

### 3. レイアウト適応

**ナビゲーション**:
- モバイル: ハンバーガーメニュー or ボトムナビ
- タブレット: コンパクトナビ
- デスクトップ: フルナビゲーションバー

**コンテンツ**:
- テーブル → モバイルではカード形式に変換
- サイドバー → モバイルではドロワーまたは折りたたみ
- 段階的開示: `<details>/<summary>` で補助情報を折りたたみ
- 画像: `srcset` + `sizes` でレスポンシブ配信

**グリッド**:
- 自己調整: `repeat(auto-fit, minmax(280px, 1fr))`
- コンテナクエリ: コンポーネントレベルの適応（`@container`）

### 4. 入力方式への対応

```css
/* タッチデバイス: 大きなタッチターゲット */
@media (pointer: coarse) {
  .interactive { min-height: 44px; padding: 12px; }
}

/* hover不可: hover依存UIの代替 */
@media (hover: none) {
  .tooltip-trigger { /* タップで表示に変更 */ }
}
```

### 5. セーフエリア対応

```css
.bottom-bar {
  padding-bottom: env(safe-area-inset-bottom);
}
```

- `<meta name="viewport" content="viewport-fit=cover">` の設定
- 固定配置要素（ヘッダー、フッター、FAB）での対応

### 6. 検証

- [ ] 主要ブレイクポイントでの動作確認
- [ ] 横向き/縦向き両方の確認
- [ ] タッチターゲット 44px の確認
- [ ] hover に依存した機能がないか
- [ ] オーバーフロー（横スクロール）がないか
- [ ] 実機テスト推奨（DevTools だけでは不十分）

## 推奨される次のステップ

レスポンシブ適応後、以下のコマンドスキルの実行を検討する:
- `/guard-ui` — デバイス固有のエッジケースを堅牢化
- `/optimize-ui` — レスポンシブ画像・パフォーマンスを最適化
