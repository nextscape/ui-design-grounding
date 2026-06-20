# DESIGN.md Spec Reference

## このドキュメントの役割

**DESIGN.md** のフォーマット仕様（出力契約）と設計思想を定義する単一情報源。

DESIGN.md は「リポジトリの視覚的アイデンティティの憲法」であり、AI コーディングエージェント（Claude Code, Cursor, Copilot 等）がスキル呼び出しなしで自動参照できるデザインシステム文書。**google-labs-code/design.md** フォーマット（`version: alpha`）に準拠する。

出力元は `/init-design` だけでなく `/extract-ui`（反映）や `/audit-ui`（準拠率確認）等も触れうるため、正規定義を1箇所に集約する。本リファレンスは DESIGN.md を**生成・更新・監査するすべてのスキル**から参照される。

**役割分担（重複させないための線引き）**:
- 本リファレンス = DESIGN.md の**出力契約（front matter 構造・`{}` 参照構文・CLI 処理規則・本文8セクション）**。語彙は再定義しない。
- `design-tokens.md` = トークンの**理論**（primitive / semantic / component の仕組み・段階導入）かつ**正規の役割語彙（Material 3 系）の単一所有者**。ツール非依存。
- `design-system.md` = デザインシステムの**理論**（コンポーネント粒度・含める/含めない・運用）。ツール非依存。

「どの役割名を使うか（語彙）」は `design-tokens.md`、「どう書くか（書式）」は本リファレンスが担う。semantic キー（`primary` / `on-primary` / `surface` …）は design-tokens.md の役割語彙をフラットキーとして用いたもの。

## 二層構造

DESIGN.md は次の2層で構成する:

- **YAML front matter** — 機械可読のデザイントークン（`colors` / `typography` / `rounded` / `spacing` / `components`）。コントラスト検証や自動処理の対象。
- **Markdown 本文** — デザインの根拠・適用指針。8セクション固定。

厚くしすぎない。10分以内に読める分量が目安。

## 設計思想（design.md の哲学）

**重要 — design.md は具体値の網羅よりも散文の質を最優先する。**

- **散文 > 精密な値**: UI の品質は値の精度より「意図がどれだけ明確に記述されているか」で決まる。
- **具体的な参照 > 汎用形容詞**: 「モダンでクリーン、信頼感」より「1970年代の大学講義ハンドアウト」のような**具体的な参照**が情報量を持つ。固有名詞・質感・比喩を盛り込む。
- **強い参照は禁止事項を内包する**: 講義ハンドアウトには gradient も glow もヒーローもない。禁止リストを並べるより鮮明な参照を1つ置くほうが設計空間を自然に制約する。
- **トークンは指示でなく文脈**: front matter の値は人間理解のための参照点であり硬直した命令ではない。`why`（本文）が `what`（トークン）を導く。

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

**トークン参照構文**: 値の重複を避けるため波括弧 + ドット記法で他トークンを参照する。例: `{colors.primary}` / `{typography.label-md}` / `{rounded.lg}` / `{spacing.gutter}`。`colors` を含むどのグループの値でも使える（color の semantic → primitive 参照も可）。参照先は **leaf のトークン**であること（`colors` のようなグループは指せない）。

**2層トークンの背景理論と正規の役割語彙**は `design-tokens.md`（primitive / semantic / component の使い分け、Material 3 系の役割名）と `color-system.md`（セマンティックカラー設計）が所有する。本リファレンスは**書式**だけを定義する。`semantic 命名は Material 3 系を推奨` 等の YAML コメントは語彙をフラットキーで使う際の例示。

**処理規則（CLI 検証の挙動）**:
- 未知セクション・未知トークン名は、値が妥当なら**保持**される（拡張可能）
- **重複見出しはエラー**になる
- 未知のコンポーネントプロパティは**警告付きで許容**
- 色は全て sRGB に変換され WCAG コントラスト検証の対象になる

### 本文8セクション（順序固定）

`##` 見出しで以下の順に記述する。各セクションは**散文中心**で、哲学に従い具体的な参照・質感を込める。

| # | セクション | 内容 |
|---|-----------|------|
| 1 | **Overview** | ブランドの人格・ターゲット・UI が喚起すべき感情。look & feel の全体像を**具体的な参照**で1〜2段落。 |
| 2 | **Colors** | カラーパレットと各ロールの意味。最低限 primary を定義。どの primitive をどの semantic ロールに割り当てたかの意図と、60-30-10・明暗の物語を散文で。 |
| 3 | **Typography** | タイポグラフィレベル（9〜15）。フォント選定理由、ウェイト・字間の使い分け方針。 |
| 4 | **Layout** | グリッドモデルと余白戦略。ベースユニット・コンテナ幅・グルーピング原則。**レスポンシブのブレイクポイント・モバイル/デスクトップの変形もここに統合**する。 |
| 5 | **Elevation & Depth** | 視覚階層の表現方法。ドロップシャドウか、ボーダー／色コントラスト／glassmorphism 等の代替か。 |
| 6 | **Shapes** | 角丸の方針。どのコンポーネントにどのスケールを当てるか、形の言語。 |
| 7 | **Components** | アトム（buttons, inputs, chips, lists, tooltips, checkboxes, radios 等）のスタイル指針。front matter の `components` を散文で補足。 |
| 8 | **Do's and Don'ts** | 実践的な指針と典型的な落とし穴、設計時のガードレール。**タッチターゲット最小44px 等のレスポンシブ・アクセシビリティ要件もここに**。 |

> 見出しは canonical 名（上記）を推奨。プロダクト固有の追加セクション（`## Motion` 等）は順序の最後に置けば保持される。

## 出力例（抜粋）

```markdown
---
version: alpha
name: Aurora Notes Design System
description: 夜の書斎のような、静かで集中できるノートアプリ
colors:
  # Tier 1: primitive — literal
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
  # Tier 2: semantic — primitive を {} で参照
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
照らされているような、静かで集中を妨げないインターフェース。装飾は最小限で読み書きが主役。
toC の個人向けツールで、ターゲットは長文を書く人。

## Colors

藍・淡青・オフホワイトの3系統（primitive）を役割（semantic）に割り当てる。
藍に沈んだ背景 `surface`（`neutral-0`）を基調に、文字 `on-surface`（`neutral-900`）は青みのあるオフホワイト。
アクセント `primary`（`blue-400`）はオーロラの淡い青で保存やリンクなど能動的操作にだけ使う。
最も明るい `blue-300` は hover の浮き上がり専用。彩度の高い色は画面に1点まで。

## Typography

見出しはセリフの Newsreader で「本」の質感、本文は Inter で可読性を担保する二書体構成。
body-md は line-height 26px と広めにとり、長文でも疲れない読書リズムをつくる。

## Layout

760px の単一カラムを中央に置く。8px ベースの余白スケールで段落間は gutter (24px) を基準に呼吸させる。
768px 未満ではコンテナ左右マージンを 16px に詰め、ナビは下部固定タブへ。

## Elevation & Depth

影は使わない。深度は背景色の段差（surface → surface-container）で表現する。
フォーカス時のみ primary の 1px ボーダーで浮き上がりを示す。

## Shapes

角は控えめ。入力欄は rounded-md (8px)、ボタンは rounded-lg (12px)、バッジのみ full。
ノートの罫線のような落ち着いた形。

## Components

button-primary は primary 背景に on-primary の文字、hover でわずかに明るい青へ。
input-field は surface-container に沈め、入力中の文字が背景から浮くコントラストを保つ。

## Do's and Don'ts

- **Do**: アクセントの青は1画面1箇所に絞る — 静けさがブランドの核
- **Do**: タッチターゲットは最小 44×44px を確保する
- **Don't**: 純黒 `#000000` を背景に使わない — 藍を含んだ `#0e1018` が世界観
- **Don't**: ドロップシャドウで階層を作らない — 深度は背景の段差で表現する
```

## このreferenceの使い方

DESIGN.md を読み書きするスキルから参照される:
- `init-design` — DESIGN.md の新規作成・更新（主たる利用者）
- `extract-ui` — 抽出したトークン・コンポーネントを front matter 形式へ反映する際の書式
- `audit-ui` — 準拠率・トークン整合を確認する際の正規定義
- `polish-ui` — リリース前チェックで DESIGN.md と実装の整合を確認する際の参照

**関連リファレンス:**
- `design-tokens.md` — トークンの階層構造・命名・段階導入の理論（2層トークンの「なぜ」）
- `design-system.md` — デザインシステム全体の構成と DESIGN.md の位置づけ
- `color-system.md` — セマンティックカラー設計・コントラスト・ダークモード対応
