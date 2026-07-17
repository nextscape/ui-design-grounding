---
name: design-ui
description: 要件・仕様からUI/UXの構造と設計方針を整理し、機能単位の設計文書（.design/<feature-slug>/FEATURE_DESIGN.md）として保存する。要件が曖昧なときはインタビューで明確化してから設計する。新規UI設計・画面設計・UI構造の検討・機能設計の作成・実装前の要件整理を依頼されたときに使用する。
user-invocable: true
argument-hint: "[要件、課題、シナリオ...]"
---

# design-ui

## 準備（MANDATORY PREPARATION）

ui-design-grounding スキルを呼び出し、以下のリファレンスを読み込む:

- `ui-design-grounding/reference/interview.md`
- `ui-design-grounding/reference/feature-design.md`
- `ui-design-grounding/reference/information-arch.md`
- `ui-design-grounding/reference/wording.md`
- `ui-design-grounding/reference/design-system.md`
- `ui-design-grounding/reference/design-tokens.md`
- `ui-design-grounding/reference/color-system.md`
- `ui-design-grounding/reference/accessibility.md`
- `ui-design-grounding/reference/typography.md`
- `ui-design-grounding/reference/spatial-layout.md`
- `ui-design-grounding/reference/responsive-design.md`
- `ui-design-grounding/reference/design-md-gate.md`

## 入力

- 要望・課題
- 利用シナリオ
- 対象ユーザー
- 制約条件

## 手順

### 0. 基準の確認（DESIGN.md 前段ゲート）

`design-md-gate.md` の **前段ゲート** を実施する。DESIGN.md があれば front matter のトークンと本文の指針を**制約として読み込み**（リファレンスより優先）、構造設計はその視覚基準の中で行う。

**DESIGN.md が無い場合は提案に留めず、`/init-design` へ委譲して基準を先に作る**:

- 「これから基準づくりの質問を数問する」と宣言してから init-design を開始する。
- ユーザーが急ぐ場合は最小構成 DESIGN.md（Summary + 主要トークン + 聞けた範囲の意図）で切り上げてよい。
- DESIGN.md ができたら本スキルへ復帰する。

**本スキルは構造（画面・遷移・情報設計）を設計するもので、DESIGN.md（視覚的憲法）は書き換えない。**

### 1. 要件の明確化判定とインタビュー

要件の明確化状況を判定する: 対象画面・主要ユーザー・成功条件が入力から特定できるか。

- **特定できる** → インタビューを省略し、把握した前提を一度だけ要約して確認する。
- **曖昧・複数解釈がある** → `interview.md` の5原則でインタビューを実施する。質問は機能単位（主要ユーザーと JTBD / 成功の定義 / コンテンツ / 機能固有の制約 / スコープ外）。**DESIGN.md・コードベースが答えを持つ質問は聞かずに調べる。**

インタビュー中に DESIGN.md 級の話題（トーン・参照・新トークン）が出たら、機能設計に記録して手順8の昇格検出に回す。

2. **UX目的の言語化**: この UI が達成すべきユーザー体験を明文化する
3. **画面・状態の整理**: 必要な画面と各画面の状態（初期/ローディング/成功/エラー/空）を洗い出す
4. **情報構造と画面遷移**: 情報の階層、グルーピング、ナビゲーション、画面間の遷移を設計する
5. **既存資産の確認**: 既存のコンポーネント・CSS・トークン資産を確認し、再利用可能なものを特定する
6. **ワーディング方向性**: ラベル、メッセージ、ガイダンスの方向性を検討する
7. **実装を見据えた構造提案**: レスポンシブ、アクセシビリティ、パフォーマンスを考慮した構造を提案する

### 8. 機能設計の保存と昇格検出

- 設計結果を `feature-design.md` のテンプレートに従い `.design/<feature-slug>/FEATURE_DESIGN.md` に保存する（`<feature-slug>` は機能・画面から導いた小文字ハイフン区切り）。
- DESIGN.md 級の恒久的決定（トーンの明確化・新トークン候補・画面横断の新規約）が生まれていれば「DESIGN.md へ昇格すべき決定」として列挙し、`/init-design` を提案する（本スキルからは書き換えない）。

### 9. 実装への受け渡し

固有の実装ワークフロー（CLAUDE.md / AGENTS.md の記載・ユーザー指定・導入済みの実装プロセス系スキル。スキルは明記が無くても暗黙に期待されうる — 迷えばユーザーに確認）の有無で分岐する:

- **ある / 実装が UI に閉じない** → FEATURE_DESIGN.md と DESIGN.md を設計入力として渡す。UI 実装フェーズで `/implement-ui` を部品として使える。
- **無く、UI に閉じる** → `/implement-ui` へ。

## 出力フォーマット

FEATURE_DESIGN.md（`feature-design.md` のテンプレート準拠）を保存したうえで、会話では要約を示す:

```markdown
## 設計サマリ
- 機能設計の保存先: `.design/<feature-slug>/FEATURE_DESIGN.md`
- UX目的: ...
- 画面構成: [画面一覧と遷移の要点]
- 体験原則: [最大3つ]

## DESIGN.md へ昇格すべき決定（あれば）
- [決定]: → `/init-design`

## 要確認事項（あれば）
- [インタビューで確定しなかった判断・推奨で埋めた判断]

## 推奨される次のステップ
- [手順9の判定結果: 固有ワークフローへの受け渡し、または `/implement-ui`]
```

## 注意

- 見た目の細部を断定しない（色、フォント等の具体値は参考程度。視覚基準は DESIGN.md に委ねる）
- 実装可能性を無視しない
- 画面デザインの完成ではなく、「考え方」と「構造」を明確にし、実装に受け渡せる形（FEATURE_DESIGN.md）で残すことが目的
- DESIGN.md は制約として読むが、本スキルは DESIGN.md を書き換えない（視覚的憲法の定義・更新は `/init-design`）
