---
name: guard-ui
description: UIのエッジケース・堅牢性を強化する。テキストオーバーフロー・国際化対応・エラー耐性・境界値・アクセシビリティエッジケースを体系的にチェックし修正する。堅牢化・エッジケース対応・i18n対応・エラーハンドリング強化を依頼されたときに使用する。
user-invocable: true
argument-hint: "[対象 (画面、コンポーネント、機能...)]"
---

# guard-ui

## 準備（MANDATORY PREPARATION）

ui-design-grounding スキルを呼び出し、以下のリファレンスを読み込む:

- `ui-design-grounding/reference/interaction.md`
- `ui-design-grounding/reference/accessibility.md`
- `ui-design-grounding/reference/wording.md`
- `ui-design-grounding/reference/responsive-design.md`
- `ui-design-grounding/reference/implementation.md`
- `ui-design-grounding/reference/design-md-gate.md` — DESIGN.md ゲート（前段・後段）の手順

## 手順

### 0. 基準の確認（DESIGN.md 前段ゲート）

`design-md-gate.md` の **前段ゲート** を実施する。DESIGN.md があればそのトークン・規約を基準として読み込み（リファレンスより優先）、なければ未整備として扱い `/init-design` を提案する。基準なしに「問題なし」と判断しない。

### 1. テキストオーバーフロー処理

- 単一行の省略: `text-overflow: ellipsis` + `overflow: hidden` + `white-space: nowrap`
- 複数行の省略: `-webkit-line-clamp` + `display: -webkit-box`
- Flex内のテキスト: `min-width: 0` を親要素に設定
- 長い単語（URL等）: `overflow-wrap: break-word`
- テーブルセル: `table-layout: fixed` の検討

### 2. 国際化準備

- **翻訳膨張率**: ドイツ語+30%、フランス語+20%、フィンランド語+30-40%、中国語-30%
- 固定幅レイアウトの排除（テキスト量増加に耐えるか）
- CSS論理プロパティの使用: `margin-inline-start`、`padding-block-end` 等
- 数値・日付のローカライゼーション: `Intl` API の活用
- 文字列の外部化（ハードコードされたテキストの特定）

### 3. エラー耐性

- ネットワーク障害時: オフラインメッセージ、リトライUI
- APIエラー時: フォールバック表示、エラーバウンダリ
- バリデーション境界値: 空文字、最大長、特殊文字、スクリプトインジェクション
- 権限エラー時: 適切なメッセージと代替導線
- タイムアウト: ローディング状態の遷移とキャンセル手段

### 4. エッジケース

- **空状態**: データなし時の表示（説明+アクション誘導）
- **ローディング**: 初回読み込み、追加読み込み、リフレッシュの区別
- **大量データ**: 仮想スクロール、ページネーション、件数表示
- **並行操作**: 楽観的更新の競合、連打防止、重複送信ガード
- **戻る/進む**: ブラウザ履歴操作時のデータ保持

### 5. アクセシビリティエッジケース

- キーボードナビゲーション: Tab順序に抜け道がないか
- スクリーンリーダー: 動的コンテンツ追加時の `aria-live` 通知
- `prefers-reduced-motion`: クロスフェードで代替されているか
- `prefers-color-scheme`: ダークモード対応
- High Contrast モード: `forced-colors` メディアクエリ対応

## 出力フォーマット

```markdown
## 堅牢化結果

### テキストオーバーフロー
- [箇所]: [対処内容]

### 国際化
- [箇所]: [対処内容]

### エラー耐性
- [箇所]: [対処内容]

### エッジケース
- [箇所]: [対処内容]

### アクセシビリティ
- [箇所]: [対処内容]
```

## DESIGN.md 後段ゲート

`design-md-gate.md` の **後段ゲート** を実施する。今回の堅牢化が DESIGN.md のトークン水準・規約を変えた場合は乖離を明示し、**色は `/recolor-ui`、その他は `/init-design`** へ誘導する（DESIGN.md は自動では書き換えない）。

## 推奨される次のステップ

堅牢化後、以下のコマンドスキルの実行を検討する:
- `/audit-ui` — 堅牢化後の技術品質を総合監査
- `/polish-ui` — 堅牢化で追加した要素の仕上げ
