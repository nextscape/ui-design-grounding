# 新規スキル設計: scan-ui / recolor-ui

- 日付: 2026-06-20
- 対象リポジトリ: `ui-design-grounding`（Claude Code プラグイン）
- 参考: [Manavarya09/design-extract（designlang）](https://github.com/Manavarya09/design-extract)

## 背景と目的

designlang はヘッドレスブラウザを任意の URL に向け、生きた DOM からデザインシステムを読み取る CLI ツール。`extract` / `site` / `grade` / `theme-swap` / `brand` / `remix` の各コマンドを持つ。

このプラグイン（純 Markdown のスキル群）にはまだ無いが価値が高い発想を、**それを実行する Claude が Playwright（MCP / CLI）を使える**前提で 2 本の新スキルに落とす。

- designlang の `extract` / `site` / `brand` の発想 → **`scan-ui`**（外部 URL を分析し DESIGN.md を逆算）
- designlang の `theme-swap` の発想 → **`recolor-ui`**（既存 DESIGN.md を新 primary で OKLCH 再配色）
- `grade` は既存の `audit-ui` + `score-ui` でカバー済みのため新規化しない
- `remix`（名前付きデザイン語彙への変換）は今回スコープ外（将来検討）

## スキル境界（既存スキルとの使い分け）

「ソース × 出力 × 意図」で境界を定義する。description にも相互参照を明記して混同を防ぐ。

| スキル | ソース | 出力 | 意図 |
|---|---|---|---|
| `init-design`（既存） | 自分のコード/入力 | DESIGN.md | 自社のデザイン憲法を定義 |
| `extract-ui`（既存） | 自分のコード | コンポーネント候補表＋トークン候補表＋移行計画 | 自分の UI を部品化・整理（リファクタ） |
| **`scan-ui`（新規）** | **外部の生きたサイト(URL)** | **DESIGN.md** | 他サイトのデザイン言語を逆算・参照 |
| **`recolor-ui`（新規）** | 自分の DESIGN.md / CSS トークン | 再配色後のトークン（+本文 Colors 更新） | 既存パレットを別 primary で塗り替え |

- `scan-ui` の出力は `init-design` の「モードC（リバース生成）」と同一フォーマットの DESIGN.md。違いは**ソースが外部 URL かつ分析手段が Playwright** という点のみ。DESIGN.md のフォーマット仕様は `init-design` を**参照**し二重定義しない。
- `extract-ui` と統合しない（入力＝ファイル vs ブラウザ、出力＝移行計画 vs DESIGN.md、意図が異なる）。

## スキル1: `scan-ui`

### 役割

外部の既存サイト（URL）を Playwright で分析し、デザインシステムを逆算して **DESIGN.md** として出力する。

### フロントマター

```yaml
---
name: scan-ui
description: 外部の既存サイト(URL)を分析し、デザインシステムを逆算して DESIGN.md として出力する。Playwright で生の computed style・スクリーンショットを読み取り、色・タイポグラフィ・余白・角丸・深度・モーション・コンポーネントを抽出。単一ページ／サイト全体（複数ページをカバレッジ集計で統合）の2モード。競合サイトや参考サイトのデザイン言語を取り込みたいときに使用する。自分のコードの整理は extract-ui、自社デザイン憲法の新規定義は init-design を使う。
user-invocable: true
argument-hint: "[URL] [--site（サイト全体クロール）]"
---
```

### 2モード

- **単一ページ**: 指定 URL 1枚を分析。
- **サイト全体**: 代表ページ（home / pricing / docs 等）を巡回し、**カバレッジ**（そのトークンが何%のページで使われるか）でトークンを選別。サイト共通値と page-local な外れ値を分離して統合する。

### 分析手段（優先順位・フォールバック）

抽出ロジック（DOM walk の JS スニペット）は**スキル内に1回だけ定義**し、どの手段でも同一のものを流す（手段差で結果がぶれないように）。

1. **Playwright MCP（第一候補）** — `browser_navigate` → `browser_evaluate` で単一パス抽出（DOM を1回walkして computed style を回収）。`browser_take_screenshot` で色・レイアウトの目視検証。追加インストール不要。
2. **Playwright CLI（フォールバック）** — `npx playwright` で自己完結スクリプトを実行。特に**サイト全体モードの複数ページ巡回**はスクリプトの for ループが堅牢で、ユーザーが後で再実行できる成果物にもなる（初回は `npx playwright install chromium` が必要）。
3. **WebFetch / 手動貼り付け（最終手段）** — ブラウザが一切使えない環境では静的 HTML/CSS を取得、またはユーザーが貼り付けた computed style から推定。精度低下を明示する。

### 抽出対象

- 色 → primitive / semantic の2層に整理（Material 3 系命名・on-color ペア）
- タイポグラフィ（fontFamily / size / weight / lineHeight / letterSpacing）
- 余白スケール（ベースユニット推定）
- 角丸スケール
- シャドウ／深度表現
- モーション（transition / animation）
- 主要コンポーネントのスタイル（button / input / card 等）
- 可能なら: ブランドボイス（コピーのトーン）、アクセシビリティ所見（コントラスト等）

### 出力

- `init-design` と**完全に同一フォーマット**の DESIGN.md（design.md alpha 準拠: front matter トークン群 + 本文8セクション）。
- 全色は sRGB 変換のうえ WCAG コントラスト検証にかける。
- 出力は**たたき台**として提示し、最終判断は人間に委ねる。サイト全体モードではカバレッジ表（トークン × 使用ページ率）を併記する。
- 法的・倫理的注意: 他サイトのデザインの取り込みは参照・学習目的であり、商標・独自表現の流用は避ける旨を出力に添える。

### 準備（MANDATORY PREPARATION）

`init-design` 同様、ui-design-grounding のリファレンスを読み込む:
- `color-system.md` / `typography.md` / `spatial-layout.md` / `design-tokens.md` / `interaction.md` / `motion.md` / `anti-patterns.md`

### 推奨される次のステップ

- `/audit-ui`（取り込んだ DESIGN.md に対する技術品質監査）
- `/recolor-ui`（取り込んだパレットを自社ブランド primary に置き換え）
- `/init-design`（生成された DESIGN.md の差分更新・調整）

## スキル2: `recolor-ui`

### 役割

既存の **DESIGN.md（または CSS トークン）**を読み、新しい primary を中心に **OKLCH で破綻なく再配色**する。タイポグラフィ・余白・角丸・モーションは保持。

### フロントマター

```yaml
---
name: recolor-ui
description: 既存の DESIGN.md（または CSS トークン）のカラーパレットを、新しいブランド primary を中心に OKLCH で破綻なく再配色する。タイポグラフィ・余白・角丸・モーションは保持し、明度・彩度の関係と WCAG コントラスト、on-color ペアを維持・再計算する。ブランドカラー変更・配色テーマ切り替え・パレット差し替えを依頼されたときに使用する。
user-invocable: true
argument-hint: "[--primary <hex/oklch>] [対象 DESIGN.md / トークン]"
---
```

### 入力

- プロジェクトの DESIGN.md / CSS トークン。
- 外部 URL を再配色したい場合は `scan-ui` → `recolor-ui` のチェーン（recolor-ui 自体は URL を直接扱わない）。

### 手法

- 新 primary を OKLCH に変換。
- 既存 primitive ランプの **明度(L)・彩度(C) の関係**を保ったまま色相(H)を回し、ランプ全体を再生成。
- semantic（primary / on-primary / surface / outline 等）の参照構造は維持。
- **WCAG コントラスト**を再検証し、on-color ペア（on-primary 等）を必要に応じて再計算。
- ニュートラル系は色相を僅かに primary 側へ寄せるか中立を保つかを選択肢として提示。

### 出力

- 再配色後の front matter `colors`（primitive + semantic）。
- 必要なら本文 `## Colors` セクションの記述も更新。
- **変更前後の差分**とコントラスト検証結果を提示。

### 準備（MANDATORY PREPARATION）

- `color-system.md` / `design-tokens.md`

### 推奨される次のステップ

- `/audit-ui`（再配色後のコントラスト・トークン準拠監査）
- `/polish-ui`（最終仕上げ）

## 付随する更新（既存ファイル）

1. **`CLAUDE.md`** — コマンドスキル早見表に2行追加:
   - 分析 or 設計カテゴリ: `/scan-ui` 外部サイト分析→DESIGN.md
   - 調整カテゴリ: `/recolor-ui` パレット再配色
   - 「コマンドスキルの数 18 → 20」への記載更新。
2. **`skills/ui-help/SKILL.md`** — コマンド一覧に2件追加。
3. **`skills/ui-design-grounding/SKILL.md`** — コマンドスキル数の記載があれば更新。`extract-ui` / `init-design` の説明に scan-ui との相互参照を追記。
4. **`extract-ui` / `init-design` の description** — 相互参照の一文を追記（混同防止）。
5. **`.claude-plugin/plugin.json`** — バージョン更新（運用に従う）。
6. **`README.md`** — スキル一覧・件数の更新。

## 非目標（YAGNI）

- designlang の `grade`（既存 audit-ui / score-ui でカバー）。
- designlang の `remix`（名前付き語彙変換）— 将来検討。
- ブランドガイド PDF 生成（`brand`）— このプラグインは Markdown 出力に限定。
- recolor-ui で URL を直接扱うこと（scan-ui とチェーンする）。
