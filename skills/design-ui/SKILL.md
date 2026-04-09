---
name: design-ui
description: 要件・仕様からUI/UXの構造と設計方針を整理する。画面構成・遷移・情報設計・ワーディング方向性を、実装を見据えた形で提案する。新規UI設計・画面設計・UI構造の検討を依頼されたときに使用する。
user-invocable: true
argument-hint: "[要件、課題、シナリオ...]"
---

# design-ui

## 準備（MANDATORY PREPARATION）

ui-design-grounding スキルを呼び出し、以下のリファレンスを読み込む:

- `ui-design-grounding/reference/information-arch.md`
- `ui-design-grounding/reference/wording.md`
- `ui-design-grounding/reference/design-system.md`
- `ui-design-grounding/reference/design-tokens.md`
- `ui-design-grounding/reference/color-system.md`
- `ui-design-grounding/reference/accessibility.md`
- `ui-design-grounding/reference/typography.md`
- `ui-design-grounding/reference/spatial-layout.md`
- `ui-design-grounding/reference/responsive-design.md`

## 入力

- 要望・課題
- 利用シナリオ
- 対象ユーザー
- 制約条件

## 手順

1. **UX目的の言語化**: この UI が達成すべきユーザー体験を明文化する
2. **画面・状態の整理**: 必要な画面と各画面の状態（初期/ローディング/成功/エラー/空）を洗い出す
3. **情報構造と画面遷移**: 情報の階層、グルーピング、ナビゲーション、画面間の遷移を設計する
4. **既存資産の確認**: 既存のコンポーネント・CSS・トークン資産を確認し、再利用可能なものを特定する
5. **ワーディング方向性**: ラベル、メッセージ、ガイダンスの方向性を検討する
6. **実装を見据えた構造提案**: レスポンシブ、アクセシビリティ、パフォーマンスを考慮した構造を提案する

## 出力フォーマット

```markdown
## UX目的
- ...

## 画面構成・遷移
- 画面一覧と各画面の役割
- 画面遷移フロー
- 各画面のUI状態

## 情報設計
- 情報の階層とグルーピング
- ナビゲーション構造
- 優先順位の考え方

## 判断軸・トレードオフ
- ...

## 注意点・リスク
- ...

## 推奨される次のステップ
- `/implement-ui`（設計構造をコンポーネント分解・実装プランへ翻訳）
- `/init-design`（デザインシステム未整備の場合、DESIGN.md を生成して設計基盤を定義）
```

## 注意

- 見た目の細部を断定しない（色、フォント等の具体値は参考程度）
- 実装可能性を無視しない
- 画面デザインの完成ではなく、「考え方」と「構造」を明確にすることが目的
