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
- `ui-design-grounding/reference/color-system.md`
- `ui-design-grounding/reference/design-tokens.md`
- `ui-design-grounding/reference/responsive-design.md`
- `ui-design-grounding/reference/anti-patterns.md`
- `ui-design-grounding/reference/playwright.md`
- `ui-design-grounding/reference/ui-report.md`
- `ui-design-grounding/reference/design-md-gate.md`

## 手順

### 0. 基準の確認（DESIGN.md 前段ゲート）

`design-md-gate.md` の **前段ゲート** を実施する。DESIGN.md があれば front matter のトークンと本文の指針を**基準（source of truth）として読み込み**（リファレンスより優先）、以降の監査（特にテーミング次元の準拠度）はこの基準に照らして測る。無ければ未整備として扱い `/init-design`（外部 URL からは `/scan-ui`）を提案し、「基準が未整備」と明示する（基準なしに準拠度を断定しない）。評価専用のため後段ゲートは不要。

1. **対象の確認と実地観察**: 監査対象のUI・技術スタック・制約を把握する。`playwright.md` の**準備（共通）**を実施し、対象画面を Playwright MCP で開く。**まず `playwright.md` の一括監査スイープを1回流して機械的な指摘候補（コントラスト・タッチターゲット・オーバーフロー・未ラベル・レイアウトアニメ）を回収し、screenshot は要確認箇所だけに絞る**（→ 効率化のトリアージ）。スコアリングは観察・実測を根拠にし、ソース読解のみで断定しない。MCP が使えなければその旨を明示し、実測を要する項目は「未検証」として扱う。
2. **5次元スコアリング**（各0-4点、合計/20点）: スイープの候補を各次元へ振り分け、足りない分だけ追加観察する。
   - **アクセシビリティ**: コントラスト比・タッチターゲット・未ラベルは**一括監査スイープ**の結果を使う。フォーカス可視性は `browser_press_key`（Tab）で当ててから確認、キーボード操作も同様に実際に辿り、`prefers-reduced-motion` はエミュレートして確認。ARIA属性・セマンティックHTMLは `browser_snapshot` で確認
   - **パフォーマンス**: アニメーション対象プロパティ（transform/opacityのみか）はスイープの `layoutAnim` で検出、遅延読み込み・不要な再レンダリング・Core Web Vitals
   - **テーミング**: デザイントークン使用率、ハードコード値の有無、ダークモード対応（`prefers-color-scheme` エミュレートで確認）、セマンティックトークンの適切さ
   - **レスポンシブ**: `browser_resize` で 320/768/1024/1280px を巡回し、固定幅・オーバーフロー・タッチターゲット（44px最小）・入力方式対応（pointer/hover）を各幅で確認
   - **アンチパターン**: AI slop指標の該当数、横断的アンチパターンの該当数
3. **問題の分類**: 各問題をP0-P3の重篤度で分類する
   - P0（Blocking）: タスク完了不能、重大なアクセシビリティ違反
   - P1（Major）: WCAG AA違反、重大な摩擦
   - P2（Minor）: 回避策あり、軽微な品質問題
   - P3（Polish）: 外観上の改善余地
4. **システム的問題の特定**: 複数箇所に共通する根本原因を洗い出す
5. **ポジティブな発見**: 良い実装・パターンも記録する
6. **推奨アクション**: 検出した問題を解決できるコマンドスキルを、優先度順に紐付けて提示する。各項目に対象スキル名・対応する問題件数・代表的な問題を含める
7. **レポート保存**: `ui-report.md` に従い、評価対象プロジェクトの `.design/reports/YYYY-MM-DD/HHmmss-audit-ui.md` に詳細レポートを保存する。スクリーンショットを取得した場合は `.design/reports/YYYY-MM-DD/screenshots/` に保存し、レポート本文から相対リンクする。会話内の最終応答では、要約・最優先アクション・保存先を短く示す

## 出力フォーマット

```markdown
# audit-ui レポート: <対象>

| 項目 | 内容 |
|---|---|
| スキル | `audit-ui` |
| 対象 | <画面・コンポーネント・機能> |
| 実施日時 | <ISO 8601形式のローカル日時> |
| DESIGN.md | あり / なし / 未確認 |
| 観察方法 | Playwright MCP / 未実施（理由） |
| レポート保存先 | `.design/reports/YYYY-MM-DD/HHmmss-audit-ui.md` |

## スクリーンショット

| # | 内容 | パス |
|---|---|---|
| 1 | <画面・状態・幅など> | [screenshots/HHmmss-audit-ui-01.png](screenshots/HHmmss-audit-ui-01.png) |

<!-- 取得していない場合は「なし」と書く。 -->

## 監査サマリー

| 項目 | 内容 |
|---|---|
| 総合判定 | Critical / Needs Improvement / Acceptable / Good |
| 合計スコア | X/20 |
| P0/P1件数 | P0: X件 / P1: X件 |
| 次にやること | <最優先で実行すべき対応を1文で書く> |

## 5次元スコア

| 次元 | スコア(/4) | 主要な問題 |
|------|-----------|-----------|
| アクセシビリティ | X | ... |
| パフォーマンス | X | ... |
| テーミング | X | ... |
| レスポンシブ | X | ... |
| アンチパターン | X | ... |
| **合計** | **X/20** | |

## 問題一覧

### P0（Blocking）
- **<問題名>**
  - 根拠: <観察・実測・スクリーンショット・コード位置など>
  - 影響: <ユーザーまたは品質への影響>
  - 推奨対応: <対応方針>
  - 関連スクリーンショット: [screenshots/HHmmss-audit-ui-01.png](screenshots/HHmmss-audit-ui-01.png) / なし

### P1（Major）
- **<問題名>**
  - 根拠: ...
  - 影響: ...
  - 推奨対応: ...
  - 関連スクリーンショット: ...

### P2（Minor）
- **<問題名>**
  - 根拠: ...
  - 影響: ...
  - 推奨対応: ...
  - 関連スクリーンショット: ...

### P3（Polish）
- **<問題名>**
  - 根拠: ...
  - 影響: ...
  - 推奨対応: ...
  - 関連スクリーンショット: ...

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

## 未検証・制約

- <未検証項目または制約。なければ「なし」>
```
