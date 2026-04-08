---
name: init-design
description: プロジェクトルートにDESIGN.mdを作成・更新する。既存コード・CSS・デザイントークンを分析し、9セクション構成のデザインシステムドキュメントを生成する。DESIGN.md作成・デザインシステム定義・ガイドライン策定・デザイントークン整理・ブランド定義を依頼されたときに使用する。デザインガイドラインが不明確な場合にも使用する。
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

以下の **9セクション構成** で生成する:

```markdown
# DESIGN.md

## 1. Visual Theme & Atmosphere（視覚テーマと雰囲気）

設計哲学、ムード、密度、全体の方向性を記述する。
- デザインの基調（ミニマル、クリーン、大胆、遊び心等）
- 視覚的密度（情報量の多寡）
- 設計判断の優先順位（例: 1.ユーザーが迷わないか → 2.誤操作を防げるか → 3.状態が理解できるか → 4.変更に耐えられるか）
- トレードオフ時の考え方（見た目 vs 分かりやすさ、再利用性 vs 単純さ等）

## 2. Color Palette & Roles（カラーパレットと役割）

色の具体値とセマンティックな役割を定義する。
- Primary / Secondary / Accent の色値（hex, oklch 等）
- Neutral スケール（背景、テキスト、ボーダー）
- Semantic カラー（success, warning, error, info）
- ダークモード対応値（該当する場合）
- 60-30-10 の配色比率

## 3. Typography Rules（タイポグラフィ規則）

フォントファミリー、サイズスケール、ウェイトを定義する。
- フォントファミリー（primary, mono, 日本語等）
- サイズスケール表（display, h1-h6, body, small, xs）
- ウェイト使い分け（body, UI, heading）
- 行間（line-height）の基準
- letter-spacing の調整値

## 4. Component Stylings（コンポーネントスタイル）

主要UIパーツの仕様を定義する。
- ボタン（primary, secondary, ghost, destructive）
- カード（padding, radius, shadow）
- 入力フィールド（border, focus, error state）
- ナビゲーション（header, sidebar, mobile）
- インタラクティブ状態（hover, focus, active, disabled）

## 5. Layout Principles（レイアウト原則）

配置・間隔・グリッドのルールを定義する。
- ベースユニット（4px / 8px）
- スペーシングスケール（トークン名と値）
- グリッドシステム（カラム数、ガター）
- コンテナ最大幅
- セクション間余白の考え方

## 6. Depth & Elevation（深度と段階）

シャドウ、レイヤー、奥行きの表現を定義する。
- シャドウスケール（sm, md, lg, xl）
- z-index 階層（base, dropdown, sticky, modal, toast）
- サーフェスの使い分け（base, elevated, overlay）

## 7. Do's and Don'ts（推奨と禁止）

設計判断のガードレールを明示する。
- 推奨パターン（Do's）
- 禁止パターン（Don'ts）
- トレードオフ時の判断基準
- ブランドとして避けるべきスタイル

## 8. Responsive Behavior（レスポンシブ動作）

画面サイズ対応のルールを定義する。
- ブレイクポイント（具体値）
- モバイルファースト or デスクトップファースト
- ナビゲーションの変形パターン
- タッチターゲットサイズ（44px 最小）
- 折り返し・スタック化のルール

## 9. Agent Prompt Guide（エージェント向けガイド）

AI エージェントが参照しやすい形式でサマリーを提供する。
- クイックカラーリファレンス（コピペ用）
- よく使うコンポーネントの生成プロンプト例
- 「このファイルの値のみ使用すること」の明示
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

## 出力フォーマット

### 新規作成時

1. `DESIGN.md` ファイルをプロジェクトルートに作成
2. CLAUDE.md への参照追記（存在する場合）
3. 生成サマリーを提示:

```markdown
## DESIGN.md 生成結果

- セクション数: 9/9
- 色定義: N色
- フォント: [ファミリー名]
- スペーシング: [ベースユニット]pt ベース
- コンポーネント: N種定義
- ダークモード: 対応/未対応
- 抽出元: [CSS / トークンファイル / 手動入力]

### 要確認事項
- [人間の判断が必要な項目]
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

- DESIGN.md は**たたき台**として生成する。最終判断は人間に委ねる
- DESIGN.md は「完成させるもの」ではなく、**使われながら育つもの**
- 既存のデザイン資産がある場合は、**既存値を尊重**し勝手に変更しない
- ブランドガイドラインが別途存在する場合は、それを正として DESIGN.md に反映する
- テキスト中心設計: DESIGN.md は Markdown のみ。ロゴ等バイナリは `assets/` に分離
- 具体的な色値やフォント名が不明な場合は、仮置き値を `/* TODO: 確定待ち */` で明示する
- DESIGN.md の更新は Git で履歴管理し、変更理由をコミットメッセージに残すことを推奨する
