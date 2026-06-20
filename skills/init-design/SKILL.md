---
name: init-design
description: プロジェクトルートに DESIGN.md（リポジトリの視覚的アイデンティティ定義）を作成・更新する。google-labs-code/design.md 仕様に準拠し、YAML front matter の機械可読デザイントークン（色・タイポグラフィ・余白・コンポーネント）と散文のデザイン指針を、既存コード・CSS・トークンの分析から生成する。DESIGN.md作成・デザインシステム定義・ガイドライン策定・デザイントークン整理・ブランド定義を依頼されたとき、またデザインガイドラインが不明確なときに使用する。
user-invocable: true
argument-hint: "[プロダクト情報、既存デザイン、ブランド方針...]"
---

# init-design

## 概要

プロジェクトルートに **DESIGN.md** を作成・メンテナンスする。

DESIGN.md は「リポジトリの視覚的アイデンティティの憲法」であり、AI コーディングエージェント（Claude Code, Cursor, Copilot 等）がスキル呼び出しなしで自動参照できるデザインシステム文書である。**google-labs-code/design.md** フォーマット（`version: alpha`）に準拠する。

**二層構造**:
- **YAML front matter** — 機械可読のデザイントークン（`colors` / `typography` / `rounded` / `spacing` / `components`）。コントラスト検証や自動処理の対象。
- **Markdown 本文** — 人間（とエージェント）が読むデザインの根拠・適用指針。8セクション固定。

**DESIGN.md の価値**:
- **ツール横断**: 特定の AI ツールに依存しない（Markdown 1枚でどのエージェントも参照可能）
- **自動参照**: プロジェクトルートに置くだけでエージェントが読み込む
- **プロンプト削減**: デザイン指示を毎回書く必要がなくなる
- **一元管理**: トークンの正規値を1ファイルに集約、Git で履歴追跡可能

厚くしすぎないこと。10分以内に全体を読める分量が目安。

## 設計思想（design.md の哲学）

**重要 — design.md は具体値の網羅よりも散文の質を最優先する。**

- **散文 > 精密な値**: 生成される UI の品質は、値の精度より「意図がどれだけ明確に記述されているか」で決まる。
- **具体的な参照 > 汎用形容詞**: 「モダンでクリーン、信頼感」より「1970年代の大学講義ハンドアウト」のような**具体的な参照**のほうが情報量を持つ。本文には固有名詞・質感・比喩を盛り込む。
- **強い参照は禁止事項を内包する**: 講義ハンドアウトには gradient も glow もヒーローも存在しない。禁止リストを延々と並べるより、鮮明な参照を1つ置くほうが設計空間を自然に制約する。
- **トークンは指示でなく文脈**: front matter の値は人間理解のための参照点であって、硬直したレンダリング命令ではない。`why`（本文）が `what`（トークン）を導く。

## 準備（MANDATORY PREPARATION）

ui-design-grounding スキルを呼び出し、以下のリファレンスを読み込む:

- `ui-design-grounding/reference/design-tokens.md`
- `ui-design-grounding/reference/color-system.md`
- `ui-design-grounding/reference/typography.md`
- `ui-design-grounding/reference/spatial-layout.md`
- `ui-design-grounding/reference/interaction.md`
- `ui-design-grounding/reference/responsive-design.md`
- `ui-design-grounding/reference/anti-patterns.md`

## フォーマット仕様

### YAML front matter

```yaml
---
version: alpha          # 任意（現行は "alpha"）
name: <string>          # 必須 — デザインシステム名
description: <string>   # 任意

colors:                 # 2層構造（primitive → semantic）を標準とする
  # --- Tier 1: primitive（原色パレット） --- 単一の源色を literal で定義。ヒュー×階調で命名
  blue-400: "#7c9eff"   # hex / 名前付き / rgb() / hsl() / oklch() 等。内部で sRGB 変換し WCAG コントラスト検証
  ink-900: "#0a0e1a"
  neutral-0: "#0e1018"
  neutral-900: "#e6e8f0"
  # ramp は使う範囲だけ定義（例 blue-50..900 / neutral-0..1000）。M3 流の tonal 命名 primary-40 等も可
  # --- Tier 2: semantic（役割） --- primitive を {} で参照する（重複なし＝単一情報源）。参照先は leaf の primitive（group 不可）
  primary: "{colors.blue-400}"        # primary は最低限必須
  on-primary: "{colors.ink-900}"
  surface: "{colors.neutral-0}"
  on-surface: "{colors.neutral-900}"
  # semantic 命名は Material 3 系を推奨: primary / on-primary / primary-container / on-primary-container /
  #   secondary… / tertiary… / error… / surface / surface-container(-low/-high…) /
  #   on-surface / on-surface-variant / outline / outline-variant / background / on-background
  # components 層は semantic を {colors.primary} で参照する

typography:             # 9〜15 レベル推奨。命名は headline-* / body-* / label-*（xl/lg/md/sm）
  <token>:
    fontFamily: <string>
    fontSize: <dimension>          # px / rem / em
    fontWeight: <number>           # "400" / "700" 等
    lineHeight: <dimension|number> # 寸法 or 無単位倍率
    letterSpacing: <dimension>     # 例 -0.02em
    fontFeature: <string>          # 任意 — OpenType feature
    fontVariation: <string>        # 任意 — variable font 設定

rounded:                # 角丸スケール
  sm: <dimension>
  DEFAULT: <dimension>
  md: <dimension>
  lg: <dimension>
  xl: <dimension>
  full: 9999px

spacing:                # 余白・寸法スケール
  unit: <dimension>           # ベースユニット（例 8px）
  container-max: <dimension>
  gutter: <dimension>
  margin-mobile: <dimension>
  margin-desktop: <dimension>

components:             # アトムのスタイル。バリアントは <name>-<variant> キーで表現
  <component-name>:
    backgroundColor: <color | {参照}>
    textColor: <color | {参照}>
    typography: "{typography.<token>}"
    rounded: "{rounded.<level>}"
    padding: <dimension>
    height: <dimension>        # size / width も可
  <component-name>-hover:      # 状態違いは別キー（例 button-primary / button-primary-hover）
    backgroundColor: <color | {参照}>
---
```

**トークン参照構文**: 値の重複を避けるため、波括弧 + ドット記法で他トークンを参照する。例: `{colors.primary}` / `{typography.label-md}` / `{rounded.lg}` / `{spacing.gutter}`。`colors` を含むどのグループの値でも使える（color の semantic → primitive 参照も可）。参照先は **leaf のトークン**であること（`colors` のようなグループ自体は指せない）。

**処理規則（CLI 検証の挙動）**:
- 未知セクション・未知トークン名は、値が妥当なら**保持**される（拡張可能）
- **重複見出しはエラー**になる
- 未知のコンポーネントプロパティは**警告付きで許容**
- 色は全て sRGB に変換され WCAG コントラスト検証の対象になる

### 本文8セクション（順序固定）

`##` 見出しで以下の順に記述する。各セクションは**散文中心**で、上の哲学に従い具体的な参照・質感を込める。

| # | セクション | 内容 |
|---|-----------|------|
| 1 | **Overview** | ブランドの人格・ターゲット・UI が喚起すべき感情。look & feel の全体像。**具体的な参照**で世界観を1〜2段落。 |
| 2 | **Colors** | カラーパレットと各ロールの意味。最低限 primary を定義。どの primitive をどの semantic ロールに割り当てたかの意図と、60-30-10 や明暗の物語を散文で。 |
| 3 | **Typography** | タイポグラフィレベル（9〜15）。フォント選定理由、ウェイト・字間の使い分け方針。 |
| 4 | **Layout** | グリッドモデルと余白戦略。ベースユニット、コンテナ幅、グルーピング原則。**レスポンシブのブレイクポイント・モバイル/デスクトップの変形もここに統合**する。 |
| 5 | **Elevation & Depth** | 視覚階層の表現方法。ドロップシャドウか、ボーダー／色コントラスト／glassmorphism 等の代替か。 |
| 6 | **Shapes** | 角丸の方針。どのコンポーネントにどのスケールを当てるか、形の言語。 |
| 7 | **Components** | アトム（buttons, inputs, chips, lists, tooltips, checkboxes, radios 等）のスタイル指針。front matter の `components` を散文で補足する。 |
| 8 | **Do's and Don'ts** | 実践的な指針と典型的な落とし穴。設計時のガードレール。**タッチターゲット最小44px 等のレスポンシブ・アクセシビリティ要件もここに**。 |

> 本文の見出しは canonical 名（上記）を推奨。プロダクト固有の追加セクション（`## Motion` 等）は順序の最後に置けば保持される。

## 動作モード

DESIGN.md の有無で分岐する:

- **A. 新規作成** — DESIGN.md が存在しない。入力収集 → 既存コード分析 → 生成。
- **B. 更新** — 既存 DESIGN.md を読み込み、差分更新。front matter のトークンと本文の整合を保つ。
- **C. 抽出（リバース生成）** — 既存 CSS / トークン / コンポーネントを分析し、値を front matter に、根拠を本文に起こす。

## 手順（新規作成）

### Step 1: 入力の収集

不明な点は仮置き + 明示。
- プロダクトの性質（toB / toC / 社内）・ターゲット・喚起したい感情
- ブランドの視覚的基調 — **具体的な参照**を引き出す（「何に似ているか」を1つ）
- 既存デザイン資産（Figma、CSS、コンポーネントライブラリ）・技術制約

### Step 2: 既存コードの分析

- CSS / SCSS / Tailwind 設定 — 色、フォント、余白、シャドウ、角丸
- デザイントークンファイル・テーマ定義
- コンポーネント実装 — ボタン、カード、入力等のスタイルパターン
- package.json — フォント依存、UI フレームワーク

### Step 3: front matter（トークン）の構築

抽出・決定した値を `colors` / `typography` / `rounded` / `spacing` / `components` に落とす。
- `colors`: **primitive → semantic の2層**で構成する。primitive（`blue-400` 等のパレット）を literal で定義し、semantic（`primary` `on-primary` `surface` `on-surface` …）は `{colors.blue-400}` のように **primitive を `{}` 参照**する（値の重複なし＝単一情報源）。参照先は leaf の primitive とし `colors` グループ自体は指さない。primary は最低限必須、Material 3 系命名で on-color ペアを揃える
- `typography`: 9〜15 レベル
- `components`: 値は `{参照}` で semantic（必要なら primitive）トークンを指す

### Step 4: 本文8セクションの記述

哲学に従い**散文中心**で記述する。汎用形容詞を避け、具体的な参照・質感・比喩を込める。レスポンシブ観点は Layout / Do's and Don'ts に統合する。

### Step 5: ファイル配置

```
project-root/
├── DESIGN.md          ← 生成したファイル
├── CLAUDE.md          ← 既存の場合、DESIGN.md 参照の一文を追記（要ユーザー確認）
└── assets/            ← ロゴ等のバイナリ（必要に応じて）
```

CLAUDE.md が存在する場合、追記前にユーザーへ確認した上で以下を加える:

```markdown
UI生成・修正時は DESIGN.md の値（front matter のトークンと本文の指針）のみを使用すること。
```

## 出力例（抜粋）

```markdown
---
version: alpha
name: Aurora Notes Design System
description: 夜の書斎のような、静かで集中できるノートアプリ
colors:
  # Tier 1: primitive（原色パレット）— literal
  blue-300: "#9bb4ff"
  blue-400: "#7c9eff"
  ink-900: "#0a0e1a"
  neutral-0: "#0e1018"
  neutral-50: "#171a24"
  neutral-300: "#3a3f52"
  neutral-500: "#a3a7b8"
  neutral-900: "#e6e8f0"
  red-300: "#ff8a8a"
  red-950: "#1a0606"
  # Tier 2: semantic（役割）— primitive を {} で参照（単一情報源）
  primary: "{colors.blue-400}"
  on-primary: "{colors.ink-900}"
  surface: "{colors.neutral-0}"
  surface-container: "{colors.neutral-50}"
  on-surface: "{colors.neutral-900}"
  on-surface-variant: "{colors.neutral-500}"
  outline: "{colors.neutral-300}"
  error: "{colors.red-300}"
  on-error: "{colors.red-950}"
typography:
  headline-lg:
    fontFamily: Newsreader
    fontSize: 40px
    fontWeight: "600"
    lineHeight: 48px
    letterSpacing: -0.02em
  body-md:
    fontFamily: Inter
    fontSize: 16px
    fontWeight: "400"
    lineHeight: 26px
    letterSpacing: 0em
  label-sm:
    fontFamily: Inter
    fontSize: 13px
    fontWeight: "500"
    lineHeight: 18px
    letterSpacing: 0.04em
rounded:
  md: 0.5rem
  lg: 0.75rem
  full: 9999px
spacing:
  unit: 8px
  container-max: 760px
  gutter: 24px
components:
  button-primary:
    backgroundColor: "{colors.primary}"
    textColor: "{colors.on-primary}"
    typography: "{typography.label-sm}"
    rounded: "{rounded.lg}"
    padding: 10px
    height: 44px
  button-primary-hover:
    backgroundColor: "{colors.blue-300}"   # hover の明色は primitive を直接参照してもよい
  input-field:
    backgroundColor: "{colors.surface-container}"
    textColor: "{colors.on-surface}"
    typography: "{typography.body-md}"
    rounded: "{rounded.md}"
    padding: 12px
---

## Overview

Aurora Notes は深夜の書斎の机上を思わせる。明かりを落とした部屋で1冊のノートだけが
照らされているような、静かで集中を妨げないインターフェースを目指す。装飾は最小限で、
読み書きそのものが主役になる。toC の個人向けツールで、ターゲットは長文を書く人。

## Colors

パレットは藍と淡青、オフホワイトの3系統（primitive）を、役割（semantic）に割り当てて構成する。
藍色に沈んだ背景 `surface`（源色 `neutral-0`）を基調に、文字 `on-surface`（`neutral-900`）は純白を避けた
わずかに青みのあるオフホワイト。アクセントの `primary`（`blue-400`）はオーロラの淡い青で、保存やリンクなど
能動的な操作にだけ使う。最も明るい青 `blue-300` は hover の浮き上がりにのみ充てる。彩度の高い色は画面に1点まで。

## Typography

見出しはセリフの Newsreader で「本」の質感を、本文は Inter で可読性を担保する二書体構成。
本文 body-md は line-height 26px と広めにとり、長文でも目が疲れない読書リズムをつくる。

## Layout

760px の単一カラムを中央に置く。8px ベースの余白スケールで、段落間は gutter (24px) を基準に
呼吸をもたせる。768px 未満ではコンテナ左右マージンを 16px に詰め、ナビは下部固定タブへ。

## Elevation & Depth

影は使わない。深度はすべて背景色の段差（surface → surface-container）で表現する。
フォーカス時のみ primary の 1px ボーダーで浮き上がりを示す。

## Shapes

角は控えめ。入力欄は rounded-md (8px)、ボタンは rounded-lg (12px)。バッジのみ full。
鋭すぎず丸すぎない、ノートの罫線のような落ち着いた形。

## Components

button-primary は primary 背景に on-primary の文字、hover でわずかに明るい青へ。
input-field は surface-container に沈め、入力中の文字が背景から自然に浮くコントラストを保つ。

## Do's and Don'ts

- **Do**: アクセントの青は1画面1箇所に絞る — 静けさがブランドの核
- **Do**: タッチターゲットは最小 44×44px を確保する
- **Don't**: 純黒 `#000000` を背景に使わない — 藍を含んだ `#0e1018` が世界観
- **Don't**: ドロップシャドウで階層を作らない — 深度は背景の段差で表現する
```

## 出力品質チェックリスト

- [ ] **front matter** が有効な YAML で、`name` を持ち `colors.primary` が定義されている
- [ ] `colors` が **primitive（literal パレット）→ semantic（`{}` で primitive を参照）の2層**で、semantic は Material 3 系命名・on-color ペアが揃い、参照先が leaf の primitive になっている
- [ ] `typography` が 9〜15 レベル（小規模なら最低限でも可、その旨を明示）
- [ ] `components` の値が可能な範囲で `{参照}` を使い、値の重複がない
- [ ] 本文が **8セクションを順序通り**に持つ（Overview → … → Do's and Don'ts）
- [ ] **見出しの重複がない**（重複は CLI でエラー）
- [ ] Overview に**具体的な参照**があり、汎用形容詞の羅列になっていない
- [ ] レスポンシブ観点が Layout / Do's and Don'ts に織り込まれている
- [ ] front matter のトークンと本文の記述に矛盾がない
- [ ] `??` や `[プレースホルダー]` が残っていない（不明値は `/* TODO: 確定待ち */` で明示）
- [ ] 10分以内に全体を読める分量

## 出力フォーマット

### 新規作成時

1. `DESIGN.md` をプロジェクトルートに作成
2. CLAUDE.md への参照追記（存在する場合・要確認）
3. 生成サマリーを提示:

```markdown
## DESIGN.md 生成結果

- 準拠: design.md (version alpha)
- 色トークン: primitive N色 / semantic N色（2層）
- タイポグラフィ: N レベル（フォント: [ファミリー名]）
- spacing: ベース [unit]px / container-max [幅]px
- rounded: N スケール
- components: N 定義（うち hover 等の状態 N）
- 本文: 8/8 セクション
- 抽出元: [CSS / トークンファイル / Figma / 手動入力]

### 要確認事項
- [人間の判断が必要な項目]

### 推奨される次のステップ
- `/extract-ui`（既存UIからコンポーネント・トークンを抽出し、front matter と整合させる）
- `/audit-ui`（既存コードのトークン準拠率を監査する）
```

### 更新時

```markdown
## DESIGN.md 更新結果

### 変更箇所
- [front matter トークン / セクション]: [変更内容]

### 追加箇所
- [トークン / セクション]: [追加内容]

### 要確認事項
- [人間の判断が必要な項目]
```

## 注意

- **CLAUDE.md への副作用**: CLAUDE.md が存在する場合、「UI生成・修正時は DESIGN.md の値のみを使用すること。」の追記はユーザーのプロジェクトファイルを変更する操作。追記前にユーザーに確認すること
- DESIGN.md は**たたき台**として生成する。最終判断は人間に委ねる
- DESIGN.md は「完成させるもの」ではなく、**使われながら育つもの**（仕様自体が `alpha`）
- 既存のデザイン資産がある場合は**既存値を尊重**し勝手に変更しない。ブランドガイドラインがあればそれを正とする
- テキスト中心設計: DESIGN.md は Markdown のみ。ロゴ等バイナリは `assets/` に分離
- 不明な値は仮置きし `/* TODO: 確定待ち */` で明示する
- 更新は Git で履歴管理し、変更理由をコミットメッセージに残すことを推奨する
- **YAMLコメント（`#`）やテンプレート内の説明書きは、最終出力に不要なものは含めない**
