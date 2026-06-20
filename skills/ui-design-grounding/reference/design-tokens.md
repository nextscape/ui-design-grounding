# Design Tokens Reference

## このドキュメントの目的
デザイントークンを導入・運用する際の判断基準とパターンを提供する。文脈・制約・目的と合わせて参照する。

## デザイントークンとは
UIの見た目（色・余白・フォント・角丸など）を「値」ではなく「意味（役割）」で管理する仕組み。目的：

- 一貫性（UIの統一感）
- 変更耐性（修正が局所ではなく全体に効く）
- コミュニケーション（デザイン意図の共有）
- テーマ対応（ダークモード、ブランド差分）

## トークンの種類と使い分け

### Primitive Tokens（値の集合）
例（ヒュー×階調 / スケールで命名）：
- neutral-900 = #111111 / blue-400 = #7c9eff
- spacing.unit = 8px（派生スケールは unit の倍数）

用途：基本のカラーパレットやスケール（余白・フォントサイズ）を表す。
注意：そのままUIに直結させると「意味が読めないCSS」になりやすい。

### Semantic Tokens（意味・役割）
例（正規の役割語彙 = Material 3 系）：
- primary = blue-400（primitive を参照）
- on-primary = ink-900
- surface = neutral-0 / surface-container = neutral-50
- on-surface = neutral-900 / on-surface-variant = neutral-500

用途：
- UI上の役割（主要アクション、その上に乗る文字、背景面、境界線）を表す
- 原則としてUIでは semantic を使う

> **本ドキュメントが「トークン階層モデル」と「正規の役割語彙」の単一所有者である。** ナレッジベース全体は Material 3 系の役割名（`primary` / `on-primary` / `surface` / `surface-container` / `on-surface` / `on-surface-variant` / `outline` / `error` / `on-error` …）を唯一の語彙として用いる。DESIGN.md（`design-md-spec.md`）はこの語彙を**フラットなキー**として front matter に採用し、`{}` 参照構文などの**書式だけ**を定義する（語彙そのものは再定義しない）。CSS 実装では同じ役割を機械的に変数化する（役割 `primary` → `--color-primary`）。

**primitive → semantic の2層構造**が本理論の中核：primitive（`blue-400` 等）を literal で1度だけ定義し、semantic は primitive を参照する（値の重複なし＝単一情報源）。

### Component Tokens（特定コンポーネント専用）
例：
- button.primary.bg
- card.shadow

用途：コンポーネントが**新しい値**を必要とする場合に限定（semantic で表現できないとき）。

注意：
- 乱用するとデザインシステムが崩れる。semantic で表現できないか先に検討する
- **DESIGN.md の `components` セクションとは別物**：あちらは各コンポーネントが*どの semantic を使うか*を `{colors.primary}` で参照する**適用仕様**であり、新しい値を持つ component トークン（本 tier）ではない。適用仕様は常設してよく、component トークンは例外に留める

## 命名規則
役割語彙は上記 Semantic Tokens の Material 3 系を正規とする。カテゴリ別のキー構成は以下：

- **colors**（フラットな役割名）
  - primary / on-primary（必要なら primary-container / on-primary-container）
  - secondary… / tertiary… / error / on-error
  - surface / surface-container(-low/-high) / background / on-background
  - on-surface / on-surface-variant / outline / outline-variant
- **typography**（レベル名）: headline-* / body-* / label-*（xl/lg/md/sm）
- **rounded**: sm / DEFAULT / md / lg / xl / full
- **spacing**: unit / container-max / gutter / margin-mobile / margin-desktop（DESIGN.md のアンカー集合）。日常の余白スケールは同じ `spacing.*` 接頭辞のTシャツ段階（`spacing.2xs`…`spacing.4xl`）で、`spatial-layout.md` が詳述する

CSS へのレンダリング規則：役割名をそのまま変数化する（`primary` → `--color-primary`、`surface-container` → `--color-surface-container`、`body-md` → `--font-body-md`）。

原則：
- UIの意図が読めること
- 状態（hover/active/disabled）は必要な場合だけ増やす

## Tailwind への落とし込み（基本パターン）
推奨：CSS Variables と Tailwind theme の併用。

- 変数（semantic）を :root で定義し、Tailwind theme には var(--token) を割り当てる
- 狙い：Tailwind utility を維持しつつ意味による統一を実現し、テーマ切り替えも容易にする

## プレーンCSS / SCSS への落とし込み（基本パターン）
- tokens.css（または _tokens.scss）を作成し、:root に定義
- 既存CSSの直書き値を段階的に置換
- テーマが必要なら data-theme="dark" 等で上書きブロックを作る

## 段階導入（壊さない移行）
1) まず tokens を追加（見た目は変えない）
2) 影響が小さいカテゴリから置換（余白 → ボーダー → 背景 → テキスト → 状態色）
3) 画面単位・コンポーネント単位で差分確認
4) ルール化（直書き禁止やLint等）は後半で導入

注意：「同じ値」でも「同じ意味」とは限らない。semantic を起点に置換するのが安全。

## 最小トークンセット（導入初期の例）
- colors:
  - primary / on-primary
  - surface / surface-container
  - on-surface / on-surface-variant
  - outline
  - error / on-error
- spacing:
  - unit / gutter
- rounded:
  - sm / md
- typography:
  - body-md / label-sm / headline-lg

## このreferenceの使い方

以下のコマンドスキルから参照される:
- `audit-ui` — 技術品質監査のテーミング（トークン使用率）評価時に参照
- `init-design` — DESIGN.md 生成時のトークン設計方針に参照
- `extract-ui` — コンポーネント・トークン抽出時の命名・構造設計に参照
- `implement-ui` — UI実装時のトークン実装方針に参照
- `design-ui` — UI設計時のトークン体系検討に参照
**関連リファレンス:**
- `color-system.md` — 色のセマンティックトークン設計・ダークモード対応
- `design-system.md` — デザインシステム全体の構成とトークンの位置づけ
- `design-md-spec.md` — DESIGN.md における2層トークンの具体的な書式（front matter / `{}` 参照構文）
- `spatial-layout.md` — スペーシングトークンのグリッドシステムとの連携
