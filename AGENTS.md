# AGENTS.md

このファイルは、このリポジトリで作業する AI エージェント向けの単一情報源です。

## リポジトリ概要

`ui-design-grounding` は、UI/UX 設計の判断軸と知識ベースを提供する Claude Code プラグインです。
アプリケーションではなく、ビルド・テスト・ランタイムコードは存在しません。主な成果物は Markdown のスキル定義とリファレンスです。

Nextscape Inc. が公開し、`.claude-plugin/plugin.json` でプラグインとして登録されています。

- プラグイン名: `ui-design-grounding`
- 現行バージョン: `1.4.0`
- 構成: ナレッジベース 1 件 + コマンドスキル 23 件
- ライセンス: MIT

## アーキテクチャ

```text
.claude-plugin/
  plugin.json                 プラグインマニフェスト（名前・バージョン・著者）

skills/
  ui-design-grounding/         コアナレッジベース（ユーザー直接呼び出し不可）
    SKILL.md                   ルートスキル: スタンス・出力ポリシー・参照ナビ
    reference/                 22件のリファレンス（UI/UX原則＋共通手順）

  <command-skill>/             23件のコマンドスキル（ユーザー向けスラッシュコマンド）
    SKILL.md                   ナレッジベースを参照するワークフロー定義

docs/
  superpowers/                 設計メモ、実装計画
```

## 二層構造

### ファイル構成の二層

1. **ナレッジベース**（`skills/ui-design-grounding/`）
   - `SKILL.md` と `reference/` 配下の 22 件のリファレンス文書で構成します。
   - ユーザビリティ、認知科学、情報設計、色彩、タイポグラフィ、空間レイアウト、インタラクション状態、モーション、アクセシビリティ、レスポンシブ、ワーディング、デザインシステム、デザイントークン、初見の分かりやすさ、DESIGN.md 仕様、DESIGN.md ゲート、インタビュー手法、機能設計規約、実装パターン、アンチパターン、Playwright MCP 観察手順、評価レポート出力ルールを扱います。

2. **コマンドスキル**（`skills/<name>/`）
   - 23 件のユーザー呼び出し可能なスラッシュコマンドです。
   - 各スキルは構造化されたワークフローを定義し、必要に応じてナレッジベースのリファレンスを `MANDATORY PREPARATION` として読み込みます。

### コマンドスキルの二層

README で説明している「第1層 / 第2層」は、ファイル構成ではなくコマンドスキルの役割軸です。

- **第1層: 入口になる動詞**
  - `design-ui` / `implement-ui` / `refine-ui` / `audit-ui` / `score-ui` / `legibility-ui` / `init-design` / `scan-ui` / `preview-ui`
  - 要件整理、実装、評価、改善、基準化の入口です。
- **第2層: 観点ごとの実働ユニット**
  - `arrange-ui` / `typeset-ui` / `recolor-ui` / `animate-ui` / `clarify-ui` / `guard-ui` / `adapt-ui` / `optimize-ui` / `slim-ui` / `extract-ui` / `boost-ui` / `calm-ui` / `polish-ui`
  - 問題の軸が明確な場合に直接使えます。第1層から委譲されることもあります。

## DESIGN.md の扱い

`DESIGN.md` は、対象プロジェクトの UI 方針を固定する「視覚的な憲法」です。
色、タイポグラフィ、余白、角丸、深度、モーション、コンポーネントの判断基準を 1 ファイルに集約します。

- 多くのスキルは作業前に `DESIGN.md` を読む前段ゲートを持ちます。
- `init-design` / `scan-ui` / `extract-ui` は `DESIGN.md` の作成・更新に関わります。
- `preview-ui` は `DESIGN.md` を `preview.html` に機械的に反映し、視覚確認できる形にします。
- 修正系スキルは、値の乖離を検出して誘導します。色は `recolor-ui`、その他のトークン更新は原則 `init-design` へ誘導します。
- `DESIGN.md` を自動で大きく書き換える場合は、人間の承認が前提です。
- `design-ui` は要件をインタビューで明確化し、機能単位の判断を `.design/<feature-slug>/FEATURE_DESIGN.md` に残します。`implement-ui` は機能設計があれば読み込みます。
- `design-ui` / `implement-ui` は DESIGN.md 不在時に `/init-design` へ委譲して基準を先に作ります（評価系は提案止まり）。

## コマンドスキル早見表

ユーザーが UI に関する相談をした際、適切なスキルを提案するための参照です。`/ui-help` で一覧を表示できます。

| カテゴリ | コマンド | 一言 |
|---------|---------|------|
| 設計 | `/design-ui` | 要件から UI 構造・画面遷移・状態設計を整理 |
| 設計 | `/implement-ui` | DESIGN.md と既存実装に沿って UI を実装 |
| 設計 | `/init-design` | DESIGN.md 生成・更新 |
| 可視化 | `/preview-ui` | DESIGN.md から preview.html を生成 |
| 評価 | `/audit-ui` | 技術品質を5軸で監査 |
| 評価 | `/score-ui` | UXヒューリスティクスで採点 |
| 評価 | `/legibility-ui` | 実装を読む前に画面だけを見て、初見の分かりやすさを監査 |
| 調整 | `/refine-ui` | 曖昧な訴えを観点ベースで診断し、必要に応じて委譲して直す |
| 調整 | `/boost-ui` | 印象を強める（地味から大胆へ） |
| 調整 | `/calm-ui` | 印象を抑える（派手から洗練へ） |
| 調整 | `/arrange-ui` | レイアウト・余白・階層修正 |
| 調整 | `/typeset-ui` | タイポグラフィ改善 |
| 調整 | `/animate-ui` | モーション・アニメ追加 |
| 調整 | `/recolor-ui` | パレットを OKLCH で再配色 |
| 文言 | `/clarify-ui` | テキスト・ワーディング改善 |
| 堅牢化 | `/guard-ui` | エッジケース・i18n 強化 |
| 堅牢化 | `/adapt-ui` | レスポンシブ・マルチデバイス対応 |
| 堅牢化 | `/optimize-ui` | パフォーマンス最適化 |
| 整理 | `/slim-ui` | UI を本質へ削ぎ落とす |
| 整理 | `/extract-ui` | コンポーネント・トークン抽出 |
| 整理 | `/scan-ui` | 外部サイト URL 分析から DESIGN.md を生成 |
| 仕上げ | `/polish-ui` | リリース前最終チェックと修正 |
| ヘルプ | `/ui-help` | コマンド一覧表示 |

## 代表的なワークフロー

- **新規 UI**: `/design-ui`（DESIGN.md が無ければ `/init-design` へ委譲 → 要件インタビュー → 機能設計の生成）→ `/implement-ui` → `/audit-ui`
- **フルスタック実装（UI に閉じない）**: `/design-ui` で機能設計を作り、プロジェクトの実装ワークフローへ設計入力として渡す。UI フェーズで `/implement-ui` を部品として使う（実装プロセス全体は本プラグインの所有外）。
- **既存 UI 改善（問題が曖昧）**: `/refine-ui` → `/audit-ui` または `/score-ui`
- **既存 UI 改善（観点が明確）**: `/typeset-ui` など第2層スキルを直接実行
- **初見の分かりやすさ確認**: `/legibility-ui`
- **リリース前**: `/polish-ui` → `/score-ui`

評価系スキル（`audit-ui` / `score-ui` / `legibility-ui`）は、検出した問題を対応スキルへ自動マッピングします。
詳細な評価結果は、対象プロジェクトの `.design/reports/YYYY-MM-DD/` に Markdown レポートとして保存されます。

## リファレンス一覧

| ファイル | カバー領域 |
|---------|-----------|
| `usability.md` | ニールセン10ヒューリスティクス、ペルソナテスト、重篤度分類 |
| `cognitive.md` | 認知負荷、Cowan の限界、違反パターン |
| `information-arch.md` | 情報階層、ナビゲーション設計 |
| `color-system.md` | OKLCH、ダークモード、60-30-10 ルール |
| `typography.md` | フォント選定、モジュラースケール、垂直リズム |
| `spatial-layout.md` | 4pt グリッド、視覚階層、コンテナクエリ |
| `interaction.md` | 8 インタラクティブ状態、フォーカスリング、Popover API |
| `motion-design.md` | デュレーション体系、イージング、reduced-motion |
| `wording.md` | ボタンラベル、エラーメッセージ、ボイス / トーン |
| `accessibility.md` | WCAG、focus-visible、フォントサイズ、reduced-motion |
| `responsive-design.md` | モバイルファースト、ブレイクポイント、入力方式検出 |
| `design-system.md` | コンポーネント設計、Atomic Design、バリアント |
| `design-tokens.md` | Primitive / Semantic / Component トークン、命名 |
| `legibility.md` | 初見の分かりやすさを評価する 6 レンズ |
| `design-md-spec.md` | DESIGN.md フォーマット仕様・設計思想 |
| `design-md-gate.md` | DESIGN.md ゲート（前段 / 後段）の共通プロトコル |
| `interview.md` | インタビュー5原則、発動判定、質問の帰属 |
| `feature-design.md` | FEATURE_DESIGN.md（機能設計）テンプレート、`.design/` 構造、昇格導線 |
| `implementation.md` | コンポーネント粒度、責務分離、UI 状態管理 |
| `anti-patterns.md` | 横断的アンチパターン、AI 生成 UI 品質ゲート |
| `playwright.md` | Playwright MCP による実地観察、状態トリガ、一括監査スイープ |
| `ui-report.md` | 評価レポート保存先（`.design/reports/`）、共通メタ情報、スクリーンショットリンク |

## 1.4.0 で追加された最新運用

- `interview.md` を追加しました。`init-design` / `design-ui` 共通のインタビュープロトコル（5原則・発動判定・質問の帰属）を定義します。
- `feature-design.md` を追加しました。機能単位の設計判断を `.design/<feature-slug>/FEATURE_DESIGN.md` として残します。
- `design-ui` は要件の明確化状況を判定してインタビューし、設計結果を機能設計として保存します。`implement-ui` は機能設計を読み込んで実装します。
- `init-design` は抽出確度を判定し、コードから読めない意図をヒアリングしてから DESIGN.md を生成します（無断の仮置きを廃止）。
- UI 作業の生成物の出力先を `.design/` に統合しました（評価レポート = `.design/reports/`、`preview.html` = `.design/preview.html`。DESIGN.md はルート常駐のまま）。

## 編集時の注意事項

- **言語**: 全スキルコンテンツは日本語で記述します。編集時もこの慣例を維持してください。
- **ファイル形式**: 各スキルは YAML フロントマター（`name`、`description`）+ Markdown 本文の `SKILL.md` 単一ファイルで構成します。
- **ビルド・テスト・リントなし**: 純粋な Markdown プラグインのため、ファイルを直接読んで検証します。
- **フロントマターの重要性**: `name` と `description` は Claude Code のスキルレジストリでの表示・マッチングに使われます。`description` はユーザーの意図とスキルの紐付けに使用されるため、具体的かつ網羅的に記述してください。
- **参照リスト（MANDATORY PREPARATION）の書き方**: コマンドスキル側の参照行は、常に「素のパス」で統一し、注記（`— 説明`）は付けません。
- **参照説明の単一情報源**: 各リファレンスの一行要約は `skills/ui-design-grounding/SKILL.md` の「参照ナビゲーション」に一元化します。
- **参照リストの役割**: 参照リストは「このファイル群をコンテキストに載せる」ことだけを担います。そのスキルでの使い方や実施するゲートは、各スキル本文に書きます。
- **参照順**: 重要度順に並べます。主目的に近いドメイン ref、横断 ref（`anti-patterns.md`）、手順 ref（`playwright.md` → `design-md-gate.md`）の順を基本にします。
- **DESIGN.md 基準のスキル**: `implement-ui` や `refine-ui` など DESIGN.md を作業土台にするスキルでは、`design-md-gate.md` を先頭に置きます。
- **評価レポート**: 評価系スキルを編集するときは、`ui-report.md` の保存先・メタ情報・スクリーンショットリンク規約と整合させます。
- **インタビュー・機能設計**: インタビューを行うスキル（`init-design` / `design-ui`）は `interview.md` の5原則・発動判定と、機能設計を扱うスキル（`design-ui` / `implement-ui`）は `feature-design.md` の保存先・テンプレート規約と整合させます。
- **実地観察**: Playwright を使う評価・修正系スキルは、`playwright.md` の観察手順と一括監査スイープを参照します。

## 設計方針

- 主観的な「良い / 悪い」ではなく、根拠と前提を示す。
- 人間が判断できるように、選択肢とトレードオフを出す。
- 既存コンポーネント、CSS、デザイントークンを尊重する。
- いきなり大改修せず、優先度に基づいて段階的に改善する。
- 観察が必要な評価では、実際の画面と操作結果を根拠にする。
