# ui-design-grounding

UI/UX の判断軸を AI コーディングエージェントに持たせるためのスキル集です。

このリポジトリはアプリケーション本体ではありません。Claude Code / Codex からはプラグインとして、Gemini CLI からは Agent Skills として、Cursor / GitHub Copilot などではエージェント向け指示として使える **UI設計・実装・レビュー用の知識ベース** です。

## 何を解決するか

AI は UI のコードを書けますが、その UI が「なぜ使いやすいのか」「なぜ読みにくいのか」「どこを直すべきか」を安定して判断するには基準が必要です。

`ui-design-grounding` は、次のような判断を AI エージェントに持たせます。

- 初見で画面の目的が分かるか
- 操作に迷わない情報設計になっているか
- アクセシビリティやレスポンシブが破綻していないか
- 色、余白、タイポグラフィ、モーションが一貫しているか
- DESIGN.md に残した設計方針から外れていないか
- 評価結果を後から追える Markdown レポートとして残せるか

## まず何を使えばよいか

出発点は `/init-design` です。プロジェクトの視覚基準 **DESIGN.md** を作ると、以降のすべてのスキルがそれを判断基準として参照します。`/design-ui` や `/implement-ui` から始めても、DESIGN.md が無ければ `/init-design` に委譲して基準づくりから始まります。

迷ったら `/ui-help` から始めます。

```text
/ui-help
```

よくある入口は次の通りです。

| やりたいこと | 使うスキル |
|---|---|
| デザイン基準（DESIGN.md）を作りたい — 最初の一歩 | `/init-design` |
| 新しい UI を考えたい | `/design-ui` |
| UI を実装したい | `/implement-ui` |
| 既存 UI をよくしたいが、何が悪いか曖昧 | `/refine-ui` |
| 技術品質を監査したい | `/audit-ui` |
| UX と使いやすさを採点したい | `/score-ui` |
| 初見で分かりづらい箇所を見つけたい | `/legibility-ui` |
| 外部サイトのデザインを分析したい | `/scan-ui` |

## 導入方法

### Claude Code

Claude Code ではプラグインとして導入できます。

```text
/plugin marketplace add nextscape/ns-skills
/plugin install ui-design-grounding@nextscape
```

インストール後は、任意の対象プロジェクトで `/ui-help` を実行してください。

### Codex

Codex では Codex プラグインとして導入できます。

```text
codex plugin marketplace add nextscape/ui-design-grounding
```

その後、Codex CLI で `/plugins` を開き、`Nextscape` marketplace から `ui-design-grounding` をインストールします。

### Cursor

Cursor では Rules として使います。このリポジトリには `.cursor/rules/ui-design-grounding.mdc` が含まれています。

Remote Rule は `.mdc` ルールだけを同期します。詳細な `SKILL.md` と `reference/` を使うには、このリポジトリを clone してローカルパスを参照させるか、必要なファイルを Cursor のコンテキストに添付してください。

### Gemini CLI

clone したリポジトリの `skills/` をリンクして使えます。

```text
/skills link /path/to/ui-design-grounding/skills --scope user
/skills reload
```

## 基本の考え方

このプラグインは、スキルを大きく2層に分けています。

### 第1層: 入口になるスキル

最初に呼ぶことが多いスキルです。要件整理、実装、評価、改善の入口になります。

| スキル | 役割 |
|---|---|
| `/init-design` | DESIGN.md（全スキルが参照する視覚基準）を作成・更新する |
| `/design-ui` | UI 構造、画面遷移、状態設計を考え、機能設計として残す |
| `/implement-ui` | DESIGN.md と既存実装に沿って UI を実装する |
| `/refine-ui` | 「読みにくい」「うるさい」など曖昧な問題を診断して直す |
| `/audit-ui` | 技術品質を5軸で監査する |
| `/score-ui` | UX ヒューリスティクスで採点する |
| `/legibility-ui` | 初見で画面だけを見たときの分かりやすさを監査する |
| `/scan-ui` | 外部サイトからデザインシステムを逆算する |
| `/preview-ui` | DESIGN.md を preview.html に反映して見た目を確認する |

### 第2層: 観点ごとに直すスキル

問題の軸が分かっているときに直接使えます。第1層のスキルから委譲されることもあります。

| 観点 | スキル |
|---|---|
| レイアウト・余白 | `/arrange-ui` |
| タイポグラフィ | `/typeset-ui` |
| 色 | `/recolor-ui` |
| モーション | `/animate-ui` |
| 文言 | `/clarify-ui` |
| レスポンシブ | `/adapt-ui` |
| 堅牢性・エッジケース | `/guard-ui` |
| パフォーマンス | `/optimize-ui` |
| 印象を強める | `/boost-ui` |
| 印象を抑える | `/calm-ui` |
| 簡素化 | `/slim-ui` |
| コンポーネント・トークン抽出 | `/extract-ui` |
| リリース前仕上げ | `/polish-ui` |

## 典型的な使い方

### 新規 UI を作る

```text
/init-design
/design-ui
/implement-ui
/audit-ui
```

最初に DESIGN.md を作り、設計方針を明文化してから実装します。最後に `/audit-ui` で技術品質を確認します。

### 既存 UI を改善する

```text
/refine-ui
/audit-ui
/score-ui
```

`/refine-ui` が問題の種類を診断し、必要に応じて `/arrange-ui` や `/clarify-ui` などの観点スキルへつなぎます。

### 初見で分かりづらい画面を見直す

```text
/legibility-ui
```

`legibility-ui` は、実装を読む前に画面だけを観察します。見た目から予想した意味と、実際の挙動のズレを記録します。

### リリース前に確認する

```text
/polish-ui
/score-ui
```

細部の状態、空状態、エラー、hover/focus/loading などを見直し、最後に UX 評価で確認します。

## DESIGN.md とは

DESIGN.md は、対象プロジェクトの UI 方針を残すファイルです。

色、タイポグラフィ、余白、角丸、深度、モーション、コンポーネントの考え方を、プロジェクトの「視覚的な憲法」として固定します。

このプラグインの多くのスキルは、作業前に DESIGN.md を読みます。これにより、毎回プロンプトで「このプロジェクトはこういう見た目で」と説明しなくても、同じ基準を使って判断できます。

DESIGN.md がない場合は、まず `/init-design` を使うのがおすすめです。

## UI 作業の出力先（.design/）

このプラグインのスキルが生成するファイルは、DESIGN.md（プロジェクトルート常駐）を除き、対象プロジェクトの `.design/` に集約されます。

```text
.design/
  <feature-slug>/
    FEATURE_DESIGN.md        ← /design-ui が保存する機能単位の設計判断
  reports/
    YYYY-MM-DD/
      HHmmss-audit-ui.md
      HHmmss-score-ui.md
      HHmmss-legibility-ui.md
      screenshots/
        HHmmss-audit-ui-01.png
  preview.html               ← /preview-ui が生成する DESIGN.md の見本帳
```

FEATURE_DESIGN.md は `/implement-ui` が読み込む受け渡しファイルのため、コミットしておくのがおすすめです。

評価系スキルのスクリーンショットは、レポート内から相対リンクされます。レビュー後に、何を見て、何を根拠に判断したかを追いやすくするためです。

### レポートのイメージ

評価レポートは、最初に結論と次の一手が分かり、その後に根拠と詳細を追える構成です。

#### 技術品質監査: `/audit-ui`

```markdown
# audit-ui レポート: ダッシュボード

| 項目 | 内容 |
|---|---|
| 総合判定 | Needs Improvement |
| 合計スコア | 13/20 |
| P0/P1件数 | P0: 0件 / P1: 3件 |
| 次にやること | モバイル幅の横スクロールと低コントラストの主要CTAを先に直す |

## 5次元スコア

| 次元 | スコア(/4) | 主要な問題 |
|---|---:|---|
| アクセシビリティ | 2 | CTAのコントラスト不足、フォーカス可視性が弱い |
| パフォーマンス | 3 | レイアウトを揺らすアニメーションは未検出 |
| テーミング | 3 | 一部にハードコード色あり |
| レスポンシブ | 1 | 320pxでカード列が横にはみ出す |
| アンチパターン | 4 | 重大なAI slopパターンなし |

## 問題一覧

### P1（Major）
- **320px幅で主要カードが横スクロールする**
  - 根拠: `browser_resize(320)` とスクリーンショットで確認
  - 影響: モバイルユーザーが指標を読み切れない
  - 推奨対応: `/adapt-ui`
  - 関連スクリーンショット: [screenshots/103012-audit-ui-01.png](screenshots/103012-audit-ui-01.png)
```

#### UXヒューリスティクス評価: `/score-ui`

```markdown
# score-ui レポート: 新規作成フロー

| 項目 | 内容 |
|---|---|
| 評価帯 | Acceptable |
| 合計スコア | 26/40 |
| 主要UXリスク | 保存失敗時の回復方法が分かりにくい |
| P0/P1件数 | P0: 0件 / P1: 2件 |
| 次にやること | エラー文言と再試行導線を明確にする |

## ヒューリスティクス評価

| # | ヒューリスティクス | スコア(/4) | 所見 |
|---|---|---:|---|
| H1 | システム状態の視認性 | 2 | 保存中・保存完了の表示が短く見落としやすい |
| H5 | エラー防止 | 3 | 必須項目の事前表示はある |
| H9 | エラー回復の支援 | 1 | 失敗理由と次の操作が曖昧 |

## ペルソナテスト結果

### 初回利用者
- 観察した導線: フォーム入力 → 保存 → 通信失敗
- レッドフラグ: 失敗後に何を直すべきか判断できない
- 推奨改善: `/clarify-ui` でエラー文言を「原因」と「次の操作」に分ける
```

#### 初見理解監査: `/legibility-ui`

```markdown
# legibility-ui レポート: 設定画面

| 項目 | 内容 |
|---|---|
| 初見理解の総合判定 | Needs Improvement |
| P0/P1件数 | P0: 0件 / P1: 2件 |
| 最も誤解されやすい画面・操作 | 「適用」ボタンが画面内だけに効くのか全体設定なのか不明 |
| 次にやること | スコープが広い設定をグローバル設定として見える位置へ移す |

## 6レンズ結果

| レンズ | 実施 | 問題数 | 最大重篤度 | 代表的な誤解 |
|---|---|---:|---|---|
| ① 見た目と機能の不一致 | 実施 | 1 | P2 | 補助ラベルが操作チップに見える |
| ④ スコープ不整合 | 実施 | 2 | P1 | 画面ローカルに見える操作が全体設定へ反映される |
| ⑥ 画面間一貫性 | 未実施 | 0 | - | 対象が1画面のみのため |

## レンズ別詳細

### ④ スコープ不整合

| 画面 | 対象 | 初見の予想 | 実際の挙動 | ギャップ | 影響 | 重篤度 | 推奨対応 |
|---|---|---|---|---|---|---|---|
| 設定 | 適用ボタン | この画面だけに反映 | アプリ全体の表示密度が変わる | 影響範囲が見た目より広い | 意図しない全体変更につながる | P1 | `/design-ui`, `/arrange-ui` |
```

## リポジトリ構成

```text
.claude-plugin/
  plugin.json                 Claude Code プラグインのマニフェスト

.codex-plugin/
  plugin.json                 Codex プラグインのマニフェスト

.agents/
  plugins/
    marketplace.json          Codex repo marketplace

.cursor/
  rules/
    ui-design-grounding.mdc   Cursor Rules 用の導入ルール

skills/
  ui-design-grounding/         共通ナレッジベース
    SKILL.md                   スタンス、参照ナビゲーション
    reference/                 UI/UX の判断基準と共通手順

  audit-ui/
  score-ui/
  legibility-ui/
  ...                          ユーザーが呼び出すコマンドスキル

docs/
  superpowers/                 設計メモ、実装計画
```

このリポジトリ自体にはビルド対象のアプリはありません。主な成果物は Markdown のスキル定義です。

## 代表的なリファレンス

| ファイル | 内容 |
|---|---|
| `usability.md` | ニールセン10ヒューリスティクス、ペルソナテスト |
| `legibility.md` | 初見の分かりやすさを評価する6レンズ |
| `accessibility.md` | WCAG、focus-visible、reduced-motion |
| `responsive-design.md` | モバイルファースト、ブレイクポイント、入力方式 |
| `design-md-spec.md` | DESIGN.md のフォーマット仕様 |
| `design-md-gate.md` | DESIGN.md を作業前後でどう扱うか |
| `playwright.md` | Playwright MCP による実地観察 |
| `ui-report.md` | 評価レポートの保存先、メタ情報、スクリーンショットリンク |
| `interview.md` | インタビュー6原則、実施プロトコル、発動判定、質問の帰属 |
| `feature-design.md` | 機能設計（FEATURE_DESIGN.md）のテンプレートと `.design/` 構造 |

## どのスキルを選べばよいか

判断に迷う場合は、次の順で選ぶと扱いやすいです。

1. DESIGN.md がまだ無いなら `/init-design`（すべてのスキルの判断基準を先に作る）
2. 何を作るか考える段階なら `/design-ui`
3. 実装したいなら `/implement-ui`
4. 問題が曖昧なら `/refine-ui`
5. 品質を測りたいなら `/audit-ui` または `/score-ui`
6. 初見で分かるかを見たいなら `/legibility-ui`
7. 直す軸が明確なら第2層スキルを直接呼ぶ

それでも迷う場合は `/ui-help` を使ってください。

## このプラグインの方針

- 主観的な「良い/悪い」ではなく、根拠と前提を示す
- 人間が判断できるように、選択肢とトレードオフを出す
- 既存コンポーネント、CSS、デザイントークンを尊重する
- いきなり大改修せず、優先度に基づいて段階的に改善する
- 観察が必要な評価では、実際の画面と操作結果を根拠にする

## ライセンス

MIT
