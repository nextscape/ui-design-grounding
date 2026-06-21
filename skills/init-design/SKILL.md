---
name: init-design
description: プロジェクトルートに DESIGN.md（リポジトリの視覚的アイデンティティ定義）を作成・更新する。google-labs-code/design.md 仕様に準拠し、YAML front matter の機械可読デザイントークン（色・タイポグラフィ・余白・コンポーネント）と散文のデザイン指針を、既存コード・CSS・トークンの分析から生成する。DESIGN.md作成・デザインシステム定義・ガイドライン策定・デザイントークン整理・ブランド定義を依頼されたとき、またデザインガイドラインが不明確なときに使用する。（外部 URL から逆算する場合は scan-ui を使う）
user-invocable: true
argument-hint: "[プロダクト情報、既存デザイン、ブランド方針...]"
---

# init-design

## 概要

プロジェクトルートに **DESIGN.md** を作成・メンテナンスする。

DESIGN.md は「リポジトリの視覚的アイデンティティの憲法」であり、AI コーディングエージェント（Claude Code, Cursor, Copilot 等）がスキル呼び出しなしで自動参照できるデザインシステム文書である。**google-labs-code/design.md** フォーマット（`version: alpha`）に準拠する。

**DESIGN.md の価値**:
- **ツール横断**: 特定の AI ツールに依存しない（Markdown 1枚でどのエージェントも参照可能）
- **自動参照**: プロジェクトルートに置くだけでエージェントが読み込む
- **プロンプト削減**: デザイン指示を毎回書く必要がなくなる
- **一元管理**: トークンの正規値を1ファイルに集約、Git で履歴追跡可能

> **フォーマットの正規定義は `ui-design-grounding/reference/design-md-spec.md` にある。** 本スキルは DESIGN.md を生成・更新する**ワークフロー**を担い、二層構造（YAML front matter / 本文8セクション）・トークン書式・`{}` 参照構文・CLI 処理規則・design.md の設計思想はそのリファレンスに従う。DESIGN.md は `/extract-ui` 等の他スキルも出力しうるため、書式の単一情報源をリファレンス側に置く。

厚くしすぎないこと。10分以内に全体を読める分量が目安。

## 準備（MANDATORY PREPARATION）

ui-design-grounding スキルを呼び出し、以下のリファレンスを読み込む:

- `ui-design-grounding/reference/design-md-spec.md` — **DESIGN.md のフォーマット仕様・設計思想（必読）**
- `ui-design-grounding/reference/design-system.md` — デザインシステムの構成・コンポーネント設計の判断軸
- `ui-design-grounding/reference/design-tokens.md`
- `ui-design-grounding/reference/color-system.md`
- `ui-design-grounding/reference/typography.md`
- `ui-design-grounding/reference/spatial-layout.md`
- `ui-design-grounding/reference/interaction.md`
- `ui-design-grounding/reference/responsive-design.md`
- `ui-design-grounding/reference/anti-patterns.md`

## 動作モード

DESIGN.md の有無で分岐する:

- **A. 新規作成** — DESIGN.md が存在しない。入力収集 → 既存コード分析 → 生成。
- **B. 更新** — 既存 DESIGN.md を読み込み、差分更新。front matter のトークンと本文の整合を保つ。
- **C. 抽出（リバース生成）** — 既存 CSS / トークン / コンポーネントを分析し、値を front matter に、根拠を本文に起こす。
- **D. スリム化（重複統合）** — 更新の積み重ねで重複・冗長が溜まった DESIGN.md を、既存値を統合して整理する。**情報量は減らさず冗長だけ削る**。大規模化していれば `tokens.css` / `tokens.json` への外出しを提案する。

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

### Step 4: 本文（Summary ＋ 8セクション）の記述

哲学に従い**散文中心**で記述する。汎用形容詞を避け、具体的な参照・質感・比喩を込める。レスポンシブ観点は Layout / Do's and Don'ts に統合する。

冒頭に **`## Summary`（サマリ層）** を置き、全観点の基準を 10〜20 行で一行要約する（忘却防止）。続けて8セクション（詳細層）を順に書く。書式は `design-md-spec.md` のサマリ層・出力例に従う。

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

完全な出力例（Aurora Notes Design System）は `ui-design-grounding/reference/design-md-spec.md` の「出力例（抜粋）」を参照する。front matter（primitive→semantic の2層トークン）と本文8セクションが揃った具体例をそこに集約している。

## 出力品質チェックリスト

- [ ] **front matter** が有効な YAML で、`name` を持ち `colors.primary` が定義されている
- [ ] `colors` が **primitive（literal パレット）→ semantic（`{}` で primitive を参照）の2層**で、semantic は Material 3 系命名・on-color ペアが揃い、参照先が leaf の primitive になっている
- [ ] `typography` が 9〜15 レベル（小規模なら最低限でも可、その旨を明示）
- [ ] `components` の値が可能な範囲で `{参照}` を使い、値の重複がない
- [ ] 冒頭に **`## Summary`（サマリ層）** があり、全観点の基準を 10〜20 行で一覧している
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
- `/preview-ui`（生成・更新した DESIGN.md を preview.html に起こしてブラウザで視覚確認）
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
