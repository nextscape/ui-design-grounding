---
name: init-design
description: プロジェクトルートにDESIGN.mdを作成・更新する。既存コード・CSS・デザイントークンを分析し、Google Stitch / getdesign.md 互換の9セクション構成（Visual Theme, Color Palette, Typography, Components, Layout, Elevation, Do's/Don'ts, Responsive, Agent Prompt Guide）でデザインシステムドキュメントを生成する。各セクションはAIエージェントが直接参照して実装に使えるレベルの具体値（hex, px, CSS shadow値等）をインラインで含む。DESIGN.md作成・デザインシステム定義・ガイドライン策定・デザイントークン整理・ブランド定義を依頼されたときに使用する。デザインガイドラインが不明確な場合にも使用する。
user-invocable: true
argument-hint: "[プロダクト情報、既存デザイン、ブランド方針...]"
---

# init-design

## 概要

プロジェクトルートに **DESIGN.md** を作成・メンテナンスする。

DESIGN.md は「リポジトリの見た目の憲法」であり、AI エージェント（Claude Code, Cursor, Copilot 等）がスキル呼び出しなしで自動参照できるデザインシステムドキュメントである。

**DESIGN.md の価値**:
- **ツール横断**: 特定のAIツールに依存しない（Markdown 1枚でどのエージェントも参照可能）
- **自動参照**: プロジェクトルートに置くだけでエージェントが読み込む
- **プロンプト削減**: デザイン指示をプロンプトに毎回書く必要がなくなる（最大70%削減）
- **一元管理**: デザインの正規値を1ファイルに集約、Git で履歴追跡可能

**DESIGN.md の位置づけ**:
- DESIGN.md は「判断に迷ったときに立ち戻るための軸」と「具体的なデザイン値」の両方を1ファイルに統合したもの
- 見た目の具体値（色コード、フォントサイズ等）はここに集約する
- コンポーネント一覧やレイアウトの細則も DESIGN.md に含める
- 厚くしすぎないこと: 10分以内に全体を読める分量が目安

## 準備（MANDATORY PREPARATION）

ui-design-grounding スキルを呼び出し、以下のリファレンスを読み込む:

- `ui-design-grounding/reference/color-system.md`
- `ui-design-grounding/reference/typography.md`
- `ui-design-grounding/reference/spatial-layout.md`
- `ui-design-grounding/reference/interaction.md`
- `ui-design-grounding/reference/responsive-design.md`
- `ui-design-grounding/reference/anti-patterns.md`
- `ui-design-grounding/reference/design-tokens.md`

## 動作モード

DESIGN.md の有無で動作が分岐する:

### A. 新規作成モード（DESIGN.md が存在しない）

1. **入力の収集**
2. **既存コードの分析**
3. **DESIGN.md の生成**（9セクション構成）
4. **assets/ ディレクトリの整備**（必要に応じて）

### B. 更新モード（DESIGN.md が既に存在する）

1. **既存 DESIGN.md の読み込み**
2. **変更要求の把握**
3. **差分更新**（既存セクションの値更新、新セクション追加等）
4. **整合性の検証**

### C. 抽出モード（既存プロジェクトからリバース生成）

1. **既存 CSS / トークン / コンポーネントの分析**
2. **デザイン値の抽出**（色、フォント、余白、シャドウ等）
3. **DESIGN.md の生成**（抽出値をベースに）

## 手順（新規作成）

### Step 1: 入力の収集

以下を確認する（不明な場合は仮置き + 明示）:

- プロダクトの性質（toB / toC / 社内ツール）
- ブランドの視覚的基調（モダン、クリーン、遊び心等）
- 主な利用者の特性
- 既存のデザイン資産（Figma、CSS、コンポーネントライブラリ等）
- 制約（技術スタック、既存UIとの整合性等）

### Step 2: 既存コードの分析

プロジェクト内の以下を走査する:

- CSS / SCSS / Tailwind 設定 — 色定義、フォント、余白、シャドウ
- デザイントークンファイル — 変数定義、テーマ設定
- コンポーネント実装 — ボタン、カード、フォーム等のスタイルパターン
- package.json — フォント依存、UIフレームワーク

### Step 3: DESIGN.md の生成

以下の **9セクション構成** で生成する。各セクションには **記述フォーマット指示** と **出力例** を示す。生成時はこのフォーマットに従うこと。

> **重要**: DESIGN.md は AI エージェントが**直接参照して実装に使える**レベルの具体性を持つ必要がある。「色を統一する」のような抽象記述ではなく、「`#c96442` をプライマリCTAに使用する」のように**常に具体値をインラインで記載**すること。

```markdown
# DESIGN.md

## 1. Visual Theme & Atmosphere（視覚テーマと雰囲気）

<!-- 記述フォーマット:
  - 2〜3段落の**叙述的な散文**でデザインの人格・世界観を描写する
  - 段落内にインラインで具体値（hex等）を `backtick` で埋め込む
  - 末尾に **Key Characteristics:** として箇条書きサマリーを付ける
  - 設計判断の優先順位・トレードオフの考え方もここに含める
-->

[プロダクト名]のインターフェースは……（散文形式で、ムード・密度・設計哲学を描写する。
具体的な色値 `#hex` やフォント名、余白の感覚をインラインで織り込む。
「どんな空間にいるか」を読み手にイメージさせる書き方をする。）

（第2段落で、他との差別化ポイント、特徴的な設計判断を記述する。
設計判断の優先順位やトレードオフの考え方もここで触れる。）

**Key Characteristics:**
- 特徴1 — 具体値をインラインで含める（例: 暖色系キャンバス `#f5f4ed`）
- 特徴2 — フォント・タイポグラフィの方針（例: カスタムセリフ weight 500）
- 特徴3 — ブランドアクセント色の位置づけ（例: テラコッタ `#c96442` — CTA限定）
- 特徴4 — ニュートラル色の方針（例: 全グレーに黄褐色のアンダートーン）
- 特徴5 — シャドウ・深度の方針（例: ring-based shadow `0px 0px 0px 1px`）
- 特徴6 — 全体的な角丸・密度の方針

## 2. Color Palette & Roles（カラーパレットと役割）

<!-- 記述フォーマット:
  - カテゴリ別にサブ見出し（### Primary、### Surface & Background 等）で分割
  - 各色は以下の形式で記述:
    **セマンティック名** (`#hex`): 用途・役割の説明文。CSS変数名がある場合は併記。
  - カテゴリ: Primary / Secondary & Accent / Surface & Background / Neutrals & Text / Semantic / Borders / Gradient System
  - 60-30-10 の配色比率に言及する
  - ダークモード対応がある場合はライト/ダーク両方の値を記載
-->

### Primary
- **[ブランド名]** (`#hex`): 主要ブランドカラー。CTAボタン、ブランドアクセントに使用。
- **[Near Black]** (`#hex`): プライマリテキスト色。純粋な黒ではなく……

### Secondary & Accent
- **[色名]** (`#hex`): 用途の説明。

### Surface & Background
- **[Canvas]** (`#hex`): ページ背景。……の印象を与える。
- **[Card Surface]** (`#hex`): カード・コンテナの背景。
- **[Dark Surface]** (`#hex`): ダークテーマ時のコンテナ背景。

### Neutrals & Text
- **[色名]** (`#hex`): プライマリテキスト。
- **[色名]** (`#hex`): セカンダリテキスト。
- **[色名]** (`#hex`): ターシャリテキスト、メタデータ。

### Semantic
- **Success** (`#hex`): 成功状態。
- **Warning** (`#hex`): 警告状態。
- **Error** (`#hex`): エラー状態。
- **Info** (`#hex`): 情報提示。

### Borders
- **[Border Light]** (`#hex`): ライトテーマ標準ボーダー。
- **[Border Dark]** (`#hex`): ダークテーマ標準ボーダー。

### Gradient System
グラデーションの方針（使わない場合は「グラデーション不使用」と明記）。

## 3. Typography Rules（タイポグラフィ規則）

<!-- 記述フォーマット:
  - ### Font Family でフォールバック付きで定義
  - ### Hierarchy で Markdown テーブルを使用（下記カラム）
  - ### Principles で設計意図を叙述的に説明
-->

### Font Family
- **Headline**: `[フォント名]`, fallback: `[フォールバック]`
- **Body / UI**: `[フォント名]`, fallback: `[フォールバック]`
- **Code**: `[フォント名]`, fallback: `[フォールバック]`
- **日本語**: `[フォント名]`, fallback: `[フォールバック]`（該当する場合）

### Hierarchy

| Role | Font | Size | Weight | Line Height | Letter Spacing | Notes |
|------|------|------|--------|-------------|----------------|-------|
| Display / Hero | [フォント] | ??px (??rem) | ?? | 1.10 | normal | 最大インパクト |
| H1 | [フォント] | ??px (??rem) | ?? | 1.20 | normal | セクション見出し |
| H2 | [フォント] | ??px (??rem) | ?? | 1.25 | normal | サブセクション |
| H3 | [フォント] | ??px (??rem) | ?? | 1.30 | normal | カードタイトル等 |
| Body Large | [フォント] | ??px (??rem) | ?? | 1.60 | normal | リード文 |
| Body | [フォント] | ??px (??rem) | ?? | 1.60 | normal | 本文標準 |
| Body Small | [フォント] | ??px (??rem) | ?? | 1.50 | normal | コンパクト本文 |
| Caption | [フォント] | ??px (??rem) | ?? | 1.40 | normal | メタデータ |
| Label | [フォント] | ??px (??rem) | ?? | 1.25 | ?.?px | バッジ、小ラベル |
| Code | [フォント] | ??px (??rem) | ?? | 1.60 | normal | コードブロック |

### Principles
- **[原則1]**: 設計意図の説明（例: 「セリフは権威、サンセリフは実用」）
- **[原則2]**: 設計意図の説明（例: 「本文 line-height 1.60 で書籍に近い読書体験」）
- **[原則3]**: ウェイト使い分けの方針

## 4. Component Stylings（コンポーネントスタイル）

<!-- 記述フォーマット:
  - ### カテゴリ見出し（Buttons, Cards, Inputs, Navigation 等）
  - 各バリアントは **太字名** で小見出しにし、CSS値をリストで列挙:
    Background / Text / Padding / Radius / Shadow / Border / Font / Hover / Use
  - 状態（hover, focus, active, disabled）を含める
  - プロダクト固有の特徴的コンポーネントは ### Distinctive Components で記述
-->

### Buttons

**Primary**
- Background: `#hex`
- Text: `#hex`
- Padding: ??px ??px
- Radius: ??px
- Shadow: `[CSS shadow値]`
- Hover: `#hex` background
- Use: プライマリCTA

**Secondary**
- Background: `#hex`
- Text: `#hex`
- Padding: ??px ??px
- Radius: ??px
- Border: `1px solid #hex`
- Hover: background shifts to `rgba(...)`
- Use: セカンダリアクション

**Ghost**
- Background: transparent
- Text: `#hex`
- Padding: ??px ??px
- Radius: ??px
- Hover: `rgba(...)` background
- Use: ターシャリアクション

**Destructive**
- Background: `#hex`
- Text: `#hex`
- Use: 削除・破壊的操作

### Cards & Containers
- Background: `#hex`（ライト）/ `#hex`（ダーク）
- Border: `1px solid #hex`
- Radius: ??px（標準）/ ??px（フィーチャー）
- Shadow: `[CSS shadow値]`
- Padding: ??px

### Inputs & Forms
- Border: `1px solid #hex`
- Focus: `[フォーカスリング/ボーダーの値]`
- Error: `[エラー時のボーダー/色]`
- Radius: ??px
- Padding: ??px ??px
- Label: `#hex`, ??px [フォント]

### Navigation
- 構造: [sticky top / sidebar / etc.]
- Background: `#hex`
- Links: `#hex`（default）→ `#hex`（hover）
- CTA: [ボタンスタイル名] を配置
- Border: `[ボーダー値]`
- Mobile: [ハンバーガー / タブバー / etc.]

### Distinctive Components
（プロダクト固有の特徴的コンポーネントがあれば記述）

## 5. Layout Principles（レイアウト原則）

<!-- 記述フォーマット:
  - ### Spacing System: ベースユニットとスケール値のリスト
  - ### Grid & Container: コンテナ幅、カラム数、ガター
  - ### Whitespace Philosophy: 余白に対する設計思想を叙述で記述
  - ### Border Radius Scale: 名前付きスケール（名前 (値): 用途）
-->

### Spacing System
- Base unit: ??px
- Scale: ??px, ??px, ??px, ??px, ??px, ??px, ??px, ??px
- Button padding: ??px ??px
- Card internal padding: ??px
- Section vertical spacing: ??px〜??px

### Grid & Container
- Max container width: ??px
- Grid columns: ??
- Gutter: ??px
- レイアウトパターン: [センタード / サイドバー+メイン / etc.]

### Whitespace Philosophy
[余白に対する設計思想を叙述で記述する。例: 「エディトリアルなペーシング — 各セクションが雑誌の見開きのように呼吸する」]

### Border Radius Scale
- Sharp (??px): [用途]
- Subtle (??px): [用途]
- Comfortable (??px): [用途 — 標準ボタン、カード]
- Generous (??px): [用途 — プライマリボタン、入力欄]
- Large (??px): [用途 — フィーチャーコンテナ]
- Full (9999px): [用途 — ピル型バッジ等]

## 6. Depth & Elevation（深度と段階）

<!-- 記述フォーマット:
  - Markdown テーブル（Level / Treatment / Use）で定義
  - **Shadow Philosophy** サブセクションで設計意図を叙述
  - z-index 階層も含める
-->

| Level | Treatment | Use |
|-------|-----------|-----|
| Flat (Level 0) | shadow なし | ページ背景、インラインテキスト |
| Subtle (Level 1) | `[CSS shadow値]` | 標準カード、セクション |
| Elevated (Level 2) | `[CSS shadow値]` | ドロップダウン、ポップオーバー |
| Prominent (Level 3) | `[CSS shadow値]` | モーダル、フローティングパネル |
| Focus Ring | `[CSS outline/ring値]` | キーボードフォーカス |

### Shadow Philosophy
[シャドウに対する設計思想を叙述で記述する。色調、ブラー、スプレッドの方針。]

### z-index Scale
- Base (0): 通常コンテンツ
- Dropdown (100): ドロップダウンメニュー
- Sticky (200): スティッキーヘッダー
- Modal (300): モーダルダイアログ
- Toast (400): トースト通知
- Tooltip (500): ツールチップ

## 7. Do's and Don'ts（推奨と禁止）

<!-- 記述フォーマット:
  - ### Do / ### Don't で分割
  - 各項目にインライン値と理由を含める
  - 「なぜそうするのか」が分かる記述にする
-->

### Do
- [具体的なルール] — [理由]（例: 「`#f5f4ed` をページ背景に使う — 暖色キャンバスがブランドの人格そのもの」）
- [具体的なルール] — [理由]
- …

### Don't
- [禁止事項と具体値] — [理由]（例: 「純粋な黒 `#000000` をテキストに使わない — ブランドは暖色系ニュートラルのみ」）
- [禁止事項と具体値] — [理由]
- …

## 8. Responsive Behavior（レスポンシブ動作）

<!-- 記述フォーマット:
  - ### Breakpoints: Markdown テーブル（Name / Width / Key Changes）
  - ### Touch Targets: タッチ操作に関するルール
  - ### Collapsing Strategy: 画面縮小時の変形パターン
  - ### Image Behavior: 画像・メディアの応答方針
-->

### Breakpoints

| Name | Width | Key Changes |
|------|-------|-------------|
| Mobile | <??px | 単一カラム、ハンバーガーナビ、縮小タイポグラフィ |
| Tablet | ??–??px | 2カラムグリッド開始、ナビ凝縮 |
| Desktop | ??px+ | フルマルチカラム、展開ナビ、最大タイポグラフィ |

### Touch Targets
- 最小タッチターゲット: 44×44px
- ボタン padding: ??px 以上
- リンク間隔: 十分なスペーシング

### Collapsing Strategy
- **Navigation**: [フル水平ナビ → ハンバーガー / タブバー]
- **Grid**: [マルチカラム → スタック単一カラム]
- **Typography**: [??px → ??px のプログレッシブスケーリング]
- **Cards**: [横並び → 縦スタック]

### Image Behavior
- [レスポンシブ画像の方針: アスペクト比維持、srcset、art direction 等]

## 9. Agent Prompt Guide（エージェント向けガイド）

<!-- 記述フォーマット:
  - ### Quick Color Reference: キー:値形式のコピペ用リスト
  - ### Example Component Prompts: そのまま使えるプロンプト例（3〜5件）
  - ### Iteration Guide: エージェントへの指示ルール（番号付きリスト）
  - 冒頭に「このファイルの値のみ使用すること」を明記
-->

> **このファイルに定義された値のみを使用すること。** 独自の色・フォント・余白を発明しないこと。

### Quick Color Reference
- Brand CTA: "[名前] (#hex)"
- Page Background: "[名前] (#hex)"
- Card Surface: "[名前] (#hex)"
- Primary Text: "[名前] (#hex)"
- Secondary Text: "[名前] (#hex)"
- Border: "[名前] (#hex)"
- Dark Surface: "[名前] (#hex)"

### Example Component Prompts
- "[ページ背景] `#hex` にヒーローセクションを作成。見出しは ??px [フォント] weight ??、line-height ??。テキスト色 `#hex`。サブタイトルは `#hex` で ??px [フォント]。CTAボタンは `#hex` 背景、`#hex` テキスト、??px radius。"
- "[カード背景] `#hex` にフィーチャーカードを作成。ボーダー `1px solid #hex`、radius ??px。タイトルは [フォント] ??px weight ??。説明文は `#hex` で ??px。シャドウ `[CSS値]`。"
- （プロダクト固有の典型的なUI構築プロンプトを3〜5件記載する）

### Iteration Guide
1. 一度に**1つのコンポーネント**に集中する
2. 色は必ずセマンティック名と hex を併記する — 「グレーにして」ではなく「Olive Gray (`#5e5d59`) を使って」
3. フォントの用途を明示する — 「見出しは [Serif]、ラベルは [Sans]」
4. シャドウはレベル名で指示する — 「Elevated シャドウを適用」
5. 背景を明示する — 「[Canvas名] (`#hex`) の上に配置」
6. レスポンシブ時の変形を指定する — 「モバイルではスタック、デスクトップでは3カラム」
```

### Step 4: ファイル配置

```
project-root/
├── DESIGN.md          ← 生成したファイル
├── CLAUDE.md          ← 既存の場合、DESIGN.md参照の一文を追加
└── assets/            ← ロゴ等のバイナリ（必要に応じて）
    └── README.md      ← アセット配置ガイド
```

CLAUDE.md が存在する場合、以下を追記する:

```markdown
UI生成・修正時は DESIGN.md の値のみを使用すること。
```

## 出力品質チェックリスト

生成した DESIGN.md が以下を満たしているか確認する:

- [ ] **Section 1** が散文形式（2段落以上）で、インライン hex 値を含み、Key Characteristics 箇条書きで締めている
- [ ] **Section 2** の各色が `**セマンティック名** (\`#hex\`): 説明` 形式で記述されている
- [ ] **Section 3** に Markdown テーブル（Hierarchy）と Principles サブセクションがある
- [ ] **Section 4** の各コンポーネントバリアントが CSS 値リスト（Background / Text / Padding / Radius 等）を持つ
- [ ] **Section 5** に Border Radius Scale と Whitespace Philosophy がある
- [ ] **Section 6** が Markdown テーブルで、Shadow Philosophy の叙述がある
- [ ] **Section 7** の各 Do/Don't にインライン値と理由がある
- [ ] **Section 8** に Breakpoints テーブルと Collapsing Strategy がある
- [ ] **Section 9** に Quick Color Reference、Example Component Prompts（3件以上）、Iteration Guide がある
- [ ] 全セクション通じて `??` や `[プレースホルダー]` が残っていない（不明値は `/* TODO: 確定待ち */` で明示）
- [ ] 10分以内に全体を読める分量に収まっている

## 出力フォーマット

### 新規作成時

1. `DESIGN.md` ファイルをプロジェクトルートに作成
2. CLAUDE.md への参照追記（存在する場合）
3. 生成サマリーを提示:

```markdown
## DESIGN.md 生成結果

- セクション数: 9/9
- 色定義: N色（Primary N / Neutral N / Semantic N）
- フォント: [ファミリー名]（fallback: [フォールバック]）
- タイポグラフィ階層: N段階
- スペーシング: [ベースユニット]px ベース、N段階
- コンポーネント: N種 × N バリアント定義
- Border Radius: N段階スケール
- Elevation: N段階
- ダークモード: 対応/未対応
- 抽出元: [CSS / トークンファイル / Figma / 手動入力]

### 要確認事項
- [人間の判断が必要な項目]

### 推奨される次のステップ
- `/extract-ui`（既存UIからコンポーネント・トークンを抽出し、DESIGN.md の定義と整合させる）
- `/audit-ui`（既存コードのデザイントークン準拠率を監査する）
```

### 更新時

```markdown
## DESIGN.md 更新結果

### 変更箇所
- [セクション]: [変更内容]

### 追加箇所
- [セクション]: [追加内容]

### 要確認事項
- [人間の判断が必要な項目]
```

## 注意

- **CLAUDE.md への副作用**: CLAUDE.md が存在する場合、「UI生成・修正時は DESIGN.md の値のみを使用すること。」の一文を追記する。これはユーザーのプロジェクトの CLAUDE.md を変更する操作であるため、追記前にユーザーに確認すること
- DESIGN.md は**たたき台**として生成する。最終判断は人間に委ねる
- DESIGN.md は「完成させるもの」ではなく、**使われながら育つもの**
- 既存のデザイン資産がある場合は、**既存値を尊重**し勝手に変更しない
- ブランドガイドラインが別途存在する場合は、それを正として DESIGN.md に反映する
- テキスト中心設計: DESIGN.md は Markdown のみ。ロゴ等バイナリは `assets/` に分離
- 具体的な色値やフォント名が不明な場合は、仮置き値を `/* TODO: 確定待ち */` で明示する
- DESIGN.md の更新は Git で履歴管理し、変更理由をコミットメッセージに残すことを推奨する
- **HTMLコメント（`<!-- -->`）はテンプレート内の生成指示であり、最終出力には含めない**
