# Design Tokens Reference

## このドキュメントの目的
デザイントークンを導入・運用する際の判断基準とパターンを提供する。
単独で結論を出すためのものではなく、文脈・制約・目的と合わせて参照する。

## デザイントークンとは
UIの見た目（色・余白・フォント・角丸など）を
「値」ではなく「意味（役割）」で管理するための仕組み。

目的は以下：
- 一貫性（UIの統一感）
- 変更耐性（修正が局所ではなく全体に効く）
- コミュニケーション（デザイン意図の共有）
- テーマ対応（ダークモード、ブランド差分）

## トークンの種類と使い分け

### Primitive Tokens（値の集合）
例：
- color.gray.900 = #111111
- space.4 = 16px

用途：
- 基本のカラーパレットやスケール（余白・フォントサイズ）を表す

注意：
- そのままUIに直結させると「意味が読めないCSS」になりやすい

### Semantic Tokens（意味・役割）
例：
- color.text.primary = color.gray.900
- color.surface = color.gray.0

用途：
- UI上の役割（本文、背景、境界線、主要アクション）を表す
- 原則としてUIでは semantic を使う

### Component Tokens（特定コンポーネント専用）
例：
- button.primary.bg
- card.shadow

用途：
- コンポーネントが特殊な表現を必要とする場合に限定

注意：
- 乱用するとデザインシステムが崩れる
- semantic で表現できないか先に検討する

## 命名規則（例）
- color.*
  - color.text.primary / muted
  - color.surface / surface.elevated
  - color.border
  - color.action.primary / secondary
  - color.state.error / warning / success
- space.*
  - space.1 / 2 / 3 / 4 / 6 / 8（スケール）
- radius.*
  - radius.sm / md / lg
- font.*
  - font.size.sm / md / lg
  - font.weight.regular / semibold
  - font.lineheight.md

原則：
- UIの意図が読めること
- 状態（hover/active/disabled）は必要な場合だけ増やす

## Tailwind への落とし込み（基本パターン）
推奨：CSS Variables と Tailwind theme の併用

- 変数（semantic）を :root で定義
- Tailwind theme には var(--token) を割り当てる

狙い：
- Tailwind utility は維持しつつ、意味による統一を実現
- テーマ切り替えが容易

## プレーンCSS / SCSS への落とし込み（基本パターン）
- tokens.css（または _tokens.scss）を作成し、:root に定義
- 既存CSSの直書き値を段階的に置換

テーマが必要なら：
- data-theme="dark" 等で上書きブロックを作る

## 段階導入（壊さない移行）
1) まず tokens を追加（見た目は変えない）
2) 影響が小さいカテゴリから置換（余白 → ボーダー → 背景 → テキスト → 状態色）
3) 画面単位・コンポーネント単位で差分確認
4) ルール化（直書き禁止やLint等）は後半で導入

注意：
- 「同じ値」でも「同じ意味」とは限らない
- semantic を起点に置換するのが安全

## 最小トークンセット（導入初期の例）
- color:
  - color.text.primary / muted
  - color.surface / surface.elevated
  - color.border
  - color.action.primary
  - color.state.error
- space:
  - space.1 / 2 / 3 / 4
- radius:
  - radius.sm / md
- font:
  - font.size.sm / md / lg

## このreferenceの使い方

以下のコマンドスキルから参照される:
- `audit-ui` — 技術品質監査のテーミング（トークン使用率）評価時に参照
- `init-design` — DESIGN.md 生成時のトークン設計方針に参照
- `extract-ui` — コンポーネント・トークン抽出時の命名・構造設計に参照
- `implement-ui` — UI実装時のトークン実装方針に参照
- `design-ui` — UI設計時のトークン体系検討に参照
- 色の意味設計は `reference/color-system.md` を併せて参照する
