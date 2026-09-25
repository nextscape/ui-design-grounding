---
name: adapt-ui
description: UIをマルチデバイス・レスポンシブに提案と承認を経て適応させる。ブレイクポイント設計・入力方式対応・セーフエリア・レイアウト変形を実施する。レスポンシブ対応・モバイル対応・マルチデバイス対応を依頼されたときに使用する。
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
- `ui-design-grounding/reference/playwright.md`
- `ui-design-grounding/reference/design-md-gate.md`
- `ui-design-grounding/reference/change-gate.md`

## 手順

### 0. 基準の確認（DESIGN.md 前段ゲート）

`design-md-gate.md` の **前段ゲート** を実施する。DESIGN.md があれば `breakpoints`・スペーシング等のトークン・規約を基準として読み込み（リファレンスより優先）、なければ未整備として扱い `/init-design` を提案する。基準なしに「問題なし」と判断しない。

### 0.5 実地観察の方針

`playwright.md` の準備を実施する。レスポンシブは**幅を実際に変えて初めて崩れが見える**ため、`playwright.md`「修正系」の型と**ビューポート/メディア変種**に従い、`browser_resize` で 320/768/1024/1280px を巡回し、各幅で**一括監査スイープを流す（安い）→ `clippedX`/`overflowX`/`smallTargets` が出た幅だけ `browser_take_screenshot` で崩れを確認する**（**検出 → 変更承認ゲート → 修正 → 各幅で再観察**）。入力方式（`pointer`/`hover`）・セーフエリアの再現も同節に従う。MCP が使えなければその旨を明示する。

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

### 2.5 変更承認ゲート

`change-gate.md` に従い、ここまでの診断と手順2の設計を改善提案（変更点・対象・規模判定）としてユーザーに提示し、承認を得てから次の手順に進む。規模が小なら実装計画を会話内で示して承認を得る。大なら設計（`.design/specs/`）と実装計画（`.design/plans/`）をそれぞれ保存して承認を得る。以降の手順は承認された項目だけを実施する。

第1層（`refine-ui` / `implement-ui`）から承認済みの実装計画を受け取っている場合は、このゲートを省略し、計画のうち自分に割り当てられたタスクの範囲だけを実施する。範囲外の変更が必要になったら手を止め、差分を提案する。

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

`playwright.md` の**ビューポート/メディア変種**に従い、`browser_resize` で各ブレイクポイントを巡回して以下を確認する（修正後は各幅で再観察）。

- [ ] 主要ブレイクポイント（320/768/1024/1280px）での動作確認
- [ ] 横向き/縦向き両方の確認
- [ ] タッチターゲット 44px の確認（スイープの `smallTargets` で検出）
- [ ] hover に依存した機能がないか
- [ ] オーバーフロー（横スクロール）がないか
- [ ] 最終確認は可能なら実機で（エミュレートで取り切れないセーフエリア・実機固有挙動がある）

## DESIGN.md 後段ゲート

`design-md-gate.md` の **後段ゲート** を実施する。今回の適応が DESIGN.md のトークン水準・規約（ブレイクポイント・スペーシング等）を変えた・新設した場合は乖離を明示し、`/init-design` へ誘導する（DESIGN.md は自動では書き換えない）。

## 推奨される次のステップ

レスポンシブ適応後、以下のコマンドスキルの実行を検討する:
- `/guard-ui` — デバイス固有のエッジケースを堅牢化
- `/optimize-ui` — レスポンシブ画像・パフォーマンスを最適化
