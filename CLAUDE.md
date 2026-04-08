# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## リポジトリ概要

**Claude Code プラグイン**（`ui-design-grounding`）。UI/UX 設計の判断軸と知識ベースをスキル群として提供する。アプリケーションではなく、ビルド・テスト・ランタイムコードは存在しない。全コンテンツは Markdown で構成されている。

Nextscape Inc. が公開し、`.claude-plugin/plugin.json` でプラグインとして登録されている。

## アーキテクチャ

```
.claude-plugin/plugin.json    ← プラグインマニフェスト（名前・バージョン・著者）
skills/
  ui-design-grounding/         ← コアナレッジベース（ユーザー直接呼び出し不可）
    SKILL.md                   ← ルートスキル: スタンス・出力ポリシー・参照ナビ
    reference/                 ← 15件のUI/UX原則リファレンス
  <command-skill>/             ← 17件のコマンドスキル（ユーザー向けスラッシュコマンド）
    SKILL.md                   ← ナレッジベースを参照するワークフロー定義
```

### 二層構造

1. **ナレッジベース**（`skills/ui-design-grounding/`） — `SKILL.md`（スタンス・ポリシー・ナビゲーション）と 15 件のリファレンス文書で構成。ユーザビリティ、認知科学、色彩、タイポグラフィ、空間レイアウト、インタラクション状態、モーション、アクセシビリティ、レスポンシブ、情報設計、ワーディング、デザインシステム、デザイントークン、実装パターン、アンチパターンをカバー。コマンドスキルが「MANDATORY PREPARATION」ステップとしてこれらを読み込む。

2. **コマンドスキル**（`skills/<name>/`） — 17 件のユーザー呼び出し可能なスラッシュコマンド（`/design-ui`、`/audit-ui`、`/score-ui` 等）。各スキルが構造化されたワークフローを定義し、必要に応じてナレッジベースのリファレンスを参照しながら分析・評価・実装をガイドする。

### 主要な依存関係

- 全コマンドスキルはナレッジベースのリファレンスに依存しており、単独では完結しない。
- `init-design` はユーザーのプロジェクトに `DESIGN.md` を生成し、設計判断とトークンを記録する。

## 編集時の注意事項

- **言語**: 全スキルコンテンツは日本語で記述する。編集時もこの慣例を維持すること。
- **ファイル形式**: 各スキルは YAML フロントマター（`name`、`description`）+ Markdown 本文の `SKILL.md` 単一ファイルで構成。
- **ビルド・テスト・リントなし**: 純粋な Markdown プラグインのため、ファイルを直接読んで検証する。
- **フロントマターの重要性**: `name` と `description` は Claude Code のスキルレジストリでの表示・マッチングに使われる。`description` はユーザーの意図とスキルの紐付けに使用されるため、具体的かつ網羅的に記述すること。
