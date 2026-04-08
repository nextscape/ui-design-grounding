# ui-design-grounding

UI/UX 設計の判断軸と知識ベースを提供する Claude Code プラグイン。

## 概要

AI エージェントが UI/UX の設計・実装・レビューを行う際に、根拠に基づいた判断ができるようナレッジベースとワークフローを提供します。「なんとなく良さそう」ではなく「なぜその判断なのか」を明示するための設計支援ツールです。

## インストール

```bash
claude install-plugin nextscape/ui-design-grounding
```

## 構成

```
.claude-plugin/plugin.json    ← プラグインマニフェスト
skills/
  ui-design-grounding/         ← ナレッジベース（コマンドスキルが参照）
    SKILL.md                   ← スタンス・出力ポリシー・参照ナビゲーション
    reference/                 ← 15件のUI/UX原則リファレンス
  <command-skill>/             ← 17件のコマンドスキル（スラッシュコマンド）
    SKILL.md                   ← ワークフロー定義
```

### 二層構造

1. **ナレッジベース** (`skills/ui-design-grounding/`) - 15 件のリファレンス文書に UI/UX 設計の判断基準・原則・パターンを集約
2. **コマンドスキル** (`skills/<name>/`) - 17 件のスラッシュコマンドが構造化されたワークフローを提供し、ナレッジベースを参照

## コマンドスキル一覧

| コマンド | 用途 |
|---------|------|
| `/design-ui` | 要件からUI構造を設計 |
| `/implement-ui` | デザインを実装構造に翻訳 |
| `/init-design` | DESIGN.md の作成・更新 |
| `/audit-ui` | 技術品質の監査 |
| `/score-ui` | UXヒューリスティクス採点 |
| `/polish-ui` | リリース前の最終仕上げ |
| `/guard-ui` | エッジケースの堅牢化 |
| `/optimize-ui` | パフォーマンス最適化 |
| `/boost-ui` | デザインの印象を強める |
| `/calm-ui` | デザインの印象を抑える |
| `/animate-ui` | アニメーション追加 |
| `/typeset-ui` | タイポグラフィ修正 |
| `/arrange-ui` | レイアウト・余白修正 |
| `/extract-ui` | コンポーネント・トークン抽出 |
| `/adapt-ui` | マルチデバイス適応 |
| `/clarify-ui` | テキスト・コピー改善 |
| `/slim-ui` | UIの簡素化 |

## リファレンス一覧

| ファイル | カバー領域 |
|---------|-----------|
| `usability.md` | ニールセン10ヒューリスティクス、ペルソナテスト、重篤度分類 |
| `cognitive.md` | 認知負荷、Cowanの限界、違反パターン |
| `information-arch.md` | 情報階層、ナビゲーション設計 |
| `color-system.md` | OKLCH、ダークモード、60-30-10ルール |
| `typography.md` | フォント選定、モジュラースケール、垂直リズム |
| `spatial-layout.md` | 4ptグリッド、視覚階層、コンテナクエリ |
| `interaction.md` | 8インタラクティブ状態、フォーカスリング、Popover API |
| `motion-design.md` | デュレーション体系、イージング、reduced-motion |
| `wording.md` | ボタンラベル、エラーメッセージ、ボイス/トーン |
| `accessibility.md` | WCAG、focus-visible、フォントサイズ |
| `responsive-design.md` | モバイルファースト、ブレイクポイント、入力方式検出 |
| `design-system.md` | コンポーネント設計、Atomic Design、バリアント |
| `design-tokens.md` | Primitive/Semantic/Component トークン、命名 |
| `implementation.md` | コンポーネント粒度、責務分離、UI状態管理 |
| `anti-patterns.md` | 横断的アンチパターン、AI生成UI品質ゲート |

## 設計方針

- **判断の透明性**: 主観的好みではなく、理由と根拠を伴う提案を行う
- **人間の意思決定を尊重**: 選択肢とトレードオフを提示し、最終判断は人間に委ねる
- **既存資産の尊重**: 既存のコンポーネント・CSS・トークンの再利用を優先する
- **段階的な改善**: 完璧を目指すのではなく、優先度に基づいた改善を支援する

## ライセンス

MIT
