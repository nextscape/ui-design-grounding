# インタビュー駆動の要件明確化と DESIGN_BRIEF 統合 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** `init-design` / `design-ui` にインタビュー機構を、`design-ui` / `implement-ui` に機能単位ブリーフ（`.design/<feature-slug>/DESIGN_BRIEF.md`）の生成・受け渡しを統合し、UI 作業生成物の出力先を `.design/` に統一する。

**Architecture:** 新リファレンス `interview.md`（インタビュープロトコル）と `design-brief.md`（ブリーフ規約・テンプレート）をナレッジベースに追加し、第1層スキル3件（`init-design` / `design-ui` / `implement-ui`）から参照させる。出力先変更は `ui-report.md`（規約）と評価系3スキル＋`preview-ui`（参照側）を同時に更新する。設計仕様は `docs/superpowers/specs/2026-07-17-interview-brief-integration-design.md`。

**Tech Stack:** Markdown（プレーンテキスト）のみ。ビルド・テスト・リント不要。検証は Read/Grep によるファイル内容の直接確認と、最終タスクのサブエージェント読解テストで行う。

## Global Constraints

- 全スキルコンテンツは日本語で記述する。
- 作業ブランチ: `feat/interview-brief-integration`（`main` から作成）。
- 保存先規約: ブリーフ = `.design/<feature-slug>/DESIGN_BRIEF.md` / 評価レポート = `.design/reports/YYYY-MM-DD/HHmmss-<skill>.md` / スクリーンショット = `.design/reports/YYYY-MM-DD/screenshots/HHmmss-<skill>-NN.png` / プレビュー = `.design/preview.html`。**DESIGN.md はプロジェクトルート常駐のまま変更しない**。
- MANDATORY PREPARATION の参照行は「素のパス」のみ。各リファレンスの一行要約は `skills/ui-design-grounding/SKILL.md` の参照ナビゲーションに一元化する。
- frontmatter の `description` は具体的・網羅的に書くが、手順の逐次的な要約はしない（トリガーと成果物を中心に書く）。
- バージョンは `1.3.0` → `1.4.0`（`plugin.json` / `AGENTS.md` / `CHANGELOG.md` を整合させる）。
- コミットメッセージは既存慣例に従い `feat:` / `docs:` プレフィックスの日本語または英語1行。

---

### Task 1: ブランチ作成と `reference/interview.md` 新設

**Files:**
- Create: `skills/ui-design-grounding/reference/interview.md`

**Interfaces:**
- Produces: インタビュー5原則・発動判定・質問の帰属・エスケープ。Task 5（init-design）と Task 6（design-ui）が MANDATORY PREPARATION で参照する。

- [ ] **Step 1: ブランチを作成する**

Run: `git checkout -b feat/interview-brief-integration`
Expected: `Switched to a new branch 'feat/interview-brief-integration'`

- [ ] **Step 2: `skills/ui-design-grounding/reference/interview.md` を以下の内容で作成する**

````markdown
# インタビュー（要件・意図の明確化）

`init-design`（システム全体）と `design-ui`（機能単位）が、ユーザーとの共通理解に達するために使う共通インタビュープロトコル。DESIGN.md 常駐の本プラグインアーキテクチャに合わせて再設計している。

## 5原則

1. **決定木を枝ごとに** — 設計の決定木を枝ごとに下り、決定間の依存関係を1つずつ解決する。前の回答が次の質問を決める。
2. **1問ずつ** — 質問は必ず1つずつ提示する。複数の質問を同時に並べない（回答者を混乱させ、回答の質が下がる）。
3. **調べれば分かることは聞かない** — 次で答えが得られる質問はユーザーに聞かず自分で調べる: コードベース（既存コンポーネント・スタイル・トークン）/ DESIGN.md / 既存の DESIGN_BRIEF.md。
4. **推奨回答を添える** — 各質問に、自分の推奨回答と理由を添える。ユーザーは「それでよい」と答えるだけで先に進める。
5. **共通理解まで確定しない** — すべての枝が解決し、共通理解に達したことを確認してから成果物の生成に進む。

## 発動判定

インタビューは常に実施するものではない。呼び出し元スキルの判定基準で発動する。

| 呼び出し元 | 単位 | 成果物 | 発動判定 |
|---|---|---|---|
| `init-design` | システム全体 | DESIGN.md | 既存コードからの**抽出確度**。資産が豊富なら意図項目のみ質問、薄ければ本格ヒアリング |
| `design-ui` | 機能単位 | DESIGN_BRIEF.md | 要件の**明確化状況**。対象画面・主要ユーザー・成功条件が入力から特定できるなら省略可 |

## 質問の帰属

質問はどの成果物に属するかで振り分ける。**DESIGN.md が既に答えを持つ質問は聞かない**（原則3の「調べる先」に DESIGN.md 自体が含まれる）。

| 帰属 | 質問領域 |
|---|---|
| DESIGN.md（恒久・システム全体） | 感情トーン（落ち着き・緊急・遊び心・権威・温かさ・クリニカル）/ 参照・アンチ参照プロダクト（何に似せ、何に似せないか）/ ブランド制約 / 対象デバイス / アクセシビリティ基準 |
| DESIGN_BRIEF.md（機能単位） | 主要ユーザーとその JTBD / この画面の成功の定義 / コンテンツ（実データかプレースホルダか）/ 機能固有のハード制約（性能予算・法務等）/ スコープ外にすること |

`design-ui` のインタビュー中に DESIGN.md 級の話題（トーン・参照・新トークン）が出たら、その場で DESIGN.md を書き換えず、ブリーフに記録して完成時に `/init-design` への昇格を提案する（`design-brief.md` の昇格導線）。

## エスケープ（お任せ・急ぎ）

ユーザーが「お任せ」「そこはいい感じに」「急いでいる」等を表明したら:

1. 残る質問は推奨回答で埋めて進む。
2. 推奨で埋めた判断は、成果物の「要確認事項」に列挙して明示する。
3. 仮置き（プレースホルダー値）は、質問しても確定しなかった場合の最後の手段。**無断の仮置きで質問を省略しない**。

## アンチパターン

- 質問を5個まとめて箇条書きで投げる（原則2違反）
- コードや DESIGN.md を見れば分かるフォント名・色値を聞く（原則3違反）
- 推奨なしのオープンクエスチョンだけを投げる（「どんなトーンにしますか？」で終わる。原則4違反）
- 回答を言い換えて確認せずに次へ進む（原則5違反）
- 要件が明確な依頼に対して形式的に質問する（発動判定を通していない）
````

- [ ] **Step 3: 検証**

Run: `Read skills/ui-design-grounding/reference/interview.md`
Expected: 5原則・発動判定・質問の帰属・エスケープ・アンチパターンの5セクションが揃っている。

- [ ] **Step 4: コミット**

```bash
git add skills/ui-design-grounding/reference/interview.md
git commit -m "feat: interview.md を追加（init-design / design-ui 共通のインタビュープロトコル）"
```

---

### Task 2: `reference/design-brief.md` 新設

**Files:**
- Create: `skills/ui-design-grounding/reference/design-brief.md`

**Interfaces:**
- Consumes: Task 1 の `interview.md`（「要確認事項」がエスケープと接続）
- Produces: ブリーフの保存先・`.design/` 全体構造・テンプレート・昇格導線。Task 3（ui-report.md が `.design/` 構造を参照）、Task 6（design-ui が生成）、Task 7（implement-ui が読み込み）が参照する。

- [ ] **Step 1: `skills/ui-design-grounding/reference/design-brief.md` を以下の内容で作成する**

`````markdown
# DESIGN_BRIEF.md（機能単位の設計判断）

`design-ui` が生成し `implement-ui` が読み込む、機能単位の設計判断の記録。DESIGN.md（プロジェクト恒久の視覚的憲法）と対をなす。

## DESIGN.md との関係

| | DESIGN.md | DESIGN_BRIEF.md |
|---|---|---|
| 単位 | プロジェクト全体 | 機能・画面 |
| 寿命 | 恒久（使われながら育つ） | 機能の設計〜実装 |
| 内容 | 視覚基準（トークン・散文の指針） | UX 判断（課題・原則・構成・スコープ） |
| 置き場所 | プロジェクトルート | `.design/<feature-slug>/` |

**視覚基準（色・タイポ等のトークン水準）はブリーフに直接書かず、DESIGN.md を参照する**（二重管理の回避）。DESIGN.md が無い・最小構成の場合のみブリーフに直接書き、恒久化する際に `/init-design` へ移す。

## 保存先と `.design/` の構造

UI 作業の生成物は、DESIGN.md（ルート常駐）を除きすべて対象プロジェクトの `.design/` に集約する。

```text
project-root/
├── DESIGN.md                              ← 視覚的憲法はルート常駐（エージェントの自動参照が価値の核）
└── .design/                               ← UI 作業の生成物はすべてここ
    ├── <feature-slug>/
    │   └── DESIGN_BRIEF.md
    ├── reports/YYYY-MM-DD/                ← 評価レポート（ui-report.md 参照）
    └── preview.html                       ← DESIGN.md の見本帳（preview-ui が生成）
```

- `<feature-slug>` は機能・画面から導いた小文字ハイフン区切りの短い名前（例: `onboarding-flow`、`settings-page`）。機能ごとにサブフォルダを分け、複数回の実行で過去の成果物を上書きしない。
- Git 管理: `DESIGN_BRIEF.md` はコミットを推奨する（`implement-ui` が読み込む受け渡しファイル）。`reports/` と `preview.html` をコミットするかは各プロジェクトの判断でよい。

## テンプレート

````markdown
# デザインブリーフ: [機能・画面名]

## 課題

ユーザーが直面している問題を、ユーザーの視点で書く。技術でもビジネス指標でもなく、人間の摩擦。

## 解決

このインターフェースが課題をどう解決するかを、機能リストではなく体験として書く。

## 体験原則（最大3つ）

すべての設計判断を導く原則。各原則は緊張関係を解決する形で書く（例: 「事前の網羅より段階的開示」「速さより安心」）。

1. [原則] — [実践上の意味]
2. [原則] — [実践上の意味]
3. [原則] — [実践上の意味]

## 画面構成・遷移

- 画面一覧と各画面の役割
- 画面遷移フロー
- 各画面の UI 状態（初期 / ローディング / 成功 / エラー / 空）

## 情報設計

- 情報の階層とグルーピング
- ナビゲーション構造
- 優先順位の考え方

## 美的方向性

DESIGN.md を基準とする。この機能での逸脱・強調があれば差分だけ書く（無ければ「DESIGN.md に従う」と書く）。

- 基準: DESIGN.md
- この機能での強調・逸脱: [あれば]

## 既存パターン

再利用する既存資産。

- コンポーネント: [再利用・拡張するもの]
- トークン: [DESIGN.md の該当トークン群]

## コンポーネント一覧

| コンポーネント | 状態 | 備考 |
|---|---|---|
| [名前] | 既存 / 修正 / 新規 | [詳細] |

## 主要インタラクション

ユーザーの操作と、それに対するインターフェースの応答。状態変化・遷移・フィードバックに焦点。

## ワーディング方向性

ラベル・メッセージ・ガイダンスの方向性。用語の統一方針。

## レスポンシブ挙動

ブレイクポイントをまたいだレイアウトの変形。サイズだけでなく挙動が変わるコンポーネントを特記。

## アクセシビリティ要件

コントラスト比・キーボードナビゲーション・スクリーンリーダー・フォーカス管理の最低要件。

## 判断とトレードオフ

採った選択肢と、検討した代替案・見送った理由。

## スコープ外

このブリーフが扱わないことを具体的に書く（実装中のスコープクリープ防止）。

## 要確認事項

インタビューで確定しなかった判断・推奨で埋めた判断（無ければ「なし」）。
````

## 昇格導線（ブリーフ → DESIGN.md）

ブリーフ完成時に、DESIGN.md 級の恒久的決定が生まれていないか確認する:

- 感情トーン・参照/アンチ参照の明確化（DESIGN.md の Overview / 散文に属する）
- 新しいトークン候補（色・タイポ・余白等の値）
- 画面横断で使う新しい規約・用語

あれば「DESIGN.md へ昇格すべき決定」として列挙し、`/init-design` を提案する。**ブリーフ側から DESIGN.md を自動では書き換えない**（憲法の更新は人間の承認のもとで行う。`design-md-gate.md` の後段ゲートと同じ思想）。
`````

- [ ] **Step 2: 検証**

Run: `Read skills/ui-design-grounding/reference/design-brief.md`
Expected: DESIGN.md との関係表・`.design/` 構造図・15節テンプレート・昇格導線が揃っている。テンプレートの節順は spec 6章（課題→解決→体験原則→画面構成・遷移→情報設計→美的方向性→既存パターン→コンポーネント一覧→主要インタラクション→ワーディング方向性→レスポンシブ挙動→アクセシビリティ要件→判断とトレードオフ→スコープ外→要確認事項）と一致する。

- [ ] **Step 3: コミット**

```bash
git add skills/ui-design-grounding/reference/design-brief.md
git commit -m "feat: design-brief.md を追加（DESIGN_BRIEF.md の規約・テンプレート・.design/ 構造）"
```

---

### Task 3: `ui-report.md` の保存先変更と `design-md-gate.md` への注記

**Files:**
- Modify: `skills/ui-design-grounding/reference/ui-report.md`
- Modify: `skills/ui-design-grounding/reference/design-md-gate.md`

**Interfaces:**
- Consumes: Task 2 の `.design/` 構造定義
- Produces: `.design/reports/` の保存先規約（Task 4 の評価系3スキルが参照）、前段ゲートの委譲注記（Task 6・7 のスキル本文が詳細を定義）

- [ ] **Step 1: `ui-report.md` の保存先セクションを次の Edit で置換する**

old_string:
```markdown
評価レポートは、評価対象プロジェクトのルート直下に `ui-reports/` を作成して保存する。

| 種別 | パス |
|---|---|
| レポート | `ui-reports/YYYY-MM-DD/HHmmss-<skill>.md` |
| スクリーンショット | `ui-reports/YYYY-MM-DD/screenshots/HHmmss-<skill>-NN.png` |
```

new_string:
```markdown
評価レポートは、評価対象プロジェクトの `.design/reports/` 配下に保存する（`.design/` は UI 作業の生成物を集約するディレクトリ。全体構造は `design-brief.md` を参照）。

| 種別 | パス |
|---|---|
| レポート | `.design/reports/YYYY-MM-DD/HHmmss-<skill>.md` |
| スクリーンショット | `.design/reports/YYYY-MM-DD/screenshots/HHmmss-<skill>-NN.png` |
```

- [ ] **Step 2: `ui-report.md` の残り2箇所を Edit で置換する**

old_string: `- このプラグインリポジトリにはサンプルレポートを追加しない。`ui-reports/` は評価対象プロジェクト側の成果物である。`
new_string: `- このプラグインリポジトリにはサンプルレポートを追加しない。`.design/reports/` は評価対象プロジェクト側の成果物である。`

old_string: `| レポート保存先 | `ui-reports/YYYY-MM-DD/HHmmss-<skill>.md` |`
new_string: `| レポート保存先 | `.design/reports/YYYY-MM-DD/HHmmss-<skill>.md` |`

- [ ] **Step 3: `design-md-gate.md` の前段ゲート「存在しない場合」に委譲の注記を追加する**

old_string:
```markdown
3. **存在しない、または対象領域が未定義の場合**:
   - リファレンスを基準として作業を進める。
   - 「基準が未整備である」ことを明示し、`/init-design`（外部サイトから逆算するなら `/scan-ui`）での整備を提案する。
   - 基準なしに「問題なし」と断定しない。
```

new_string:
```markdown
3. **存在しない、または対象領域が未定義の場合**:
   - リファレンスを基準として作業を進める。
   - 「基準が未整備である」ことを明示し、`/init-design`（外部サイトから逆算するなら `/scan-ui`）での整備を提案する。
   - **設計・実装の入口**（`design-ui` / `implement-ui`）は提案に留めず、`/init-design` へ委譲して基準を先に作ってから復帰する（宣言・最小構成の逃げ道を含む動作の詳細は各スキル本文に定義）。
   - 基準なしに「問題なし」と断定しない。
```

- [ ] **Step 4: 検証**

Run: `Grep "ui-reports" skills/ui-design-grounding/reference/` → 0件。`Grep "設計・実装の入口" skills/ui-design-grounding/reference/design-md-gate.md` → 1件。

- [ ] **Step 5: コミット**

```bash
git add skills/ui-design-grounding/reference/ui-report.md skills/ui-design-grounding/reference/design-md-gate.md
git commit -m "feat: 評価レポート保存先を .design/reports/ へ変更し、前段ゲートに設計・実装入口の委譲を注記"
```

---

### Task 4: 評価系3スキルと `preview-ui` の保存先パス更新

**Files:**
- Modify: `skills/audit-ui/SKILL.md:45,59`
- Modify: `skills/score-ui/SKILL.md:59,73`
- Modify: `skills/legibility-ui/SKILL.md:69,83`
- Modify: `skills/preview-ui/SKILL.md:69,258,265`（＋frontmatter/本文に保存場所の記述が他にあれば追随）

**Interfaces:**
- Consumes: Task 3 の `.design/reports/` 規約

- [ ] **Step 1: 評価系3スキルの手順・メタ情報のパスを置換する**

各ファイルで `ui-reports/YYYY-MM-DD` → `.design/reports/YYYY-MM-DD` を全置換（replace_all）。対象箇所:
- `audit-ui/SKILL.md`: 手順7「レポート保存」の2箇所、メタ情報表の「レポート保存先」
- `score-ui/SKILL.md`: 手順8「レポート保存」の2箇所、メタ情報表の「レポート保存先」
- `legibility-ui/SKILL.md`: レポート保存段落の2箇所、メタ情報表の「レポート保存先」

- [ ] **Step 2: `preview-ui/SKILL.md` の書き出し先を変更する**

old_string: `` `preview.html` をプロジェクトルート（DESIGN.md の隣）に書き出す。既存があれば上書き。``
new_string: `` `preview.html` を `.design/preview.html` に書き出す（`.design/` が無ければ作成する）。既存があれば上書き。``

old_string: `- 生成先: preview.html（DESIGN.md の隣）`
new_string: `- 生成先: .design/preview.html`

old_string: `ブラウザで preview.html を開く（例: `start preview.html` / `open preview.html`）。`
new_string: `ブラウザで .design/preview.html を開く（例: `start .design/preview.html` / `open .design/preview.html`）。`

- [ ] **Step 3: 残存確認と追随**

Run: `Grep -n "ui-reports" skills/` → 0件。`Grep -n "preview\.html" skills/preview-ui/SKILL.md` → 残った行（frontmatter の description・機械的転記契約・注意欄等）を読み、「プロジェクトルート」「DESIGN.md の隣」という場所の記述が残っていればすべて `.design/preview.html` 前提に書き換える。単に `preview.html` とだけ呼ぶ行は変更不要。

- [ ] **Step 4: コミット**

```bash
git add skills/audit-ui/SKILL.md skills/score-ui/SKILL.md skills/legibility-ui/SKILL.md skills/preview-ui/SKILL.md
git commit -m "feat: 評価レポートと preview.html の出力先を .design/ 配下へ統一"
```

---

### Task 5: `init-design` の改修（抽出確度判定とヒアリング）

**Files:**
- Modify: `skills/init-design/SKILL.md`

**Interfaces:**
- Consumes: Task 1 の `interview.md`（5原則・質問の帰属「DESIGN.md 恒久」行）
- Produces: 「委譲された場合の宣言・最小構成 DESIGN.md」の動作。Task 6・7 の前段ゲート委譲がこれを前提にする。

- [ ] **Step 1: frontmatter の `description` を更新する**

old_string: `description: プロジェクトルートに DESIGN.md（リポジトリの視覚的アイデンティティ定義）を作成・更新する。google-labs-code/design.md 仕様に準拠し、YAML front matter の機械可読デザイントークン（色・タイポグラフィ・余白・コンポーネント）と散文のデザイン指針を、既存コード・CSS・トークンの分析から生成する。DESIGN.md作成・デザインシステム定義・ガイドライン策定・デザイントークン整理・ブランド定義を依頼されたとき、またデザインガイドラインが不明確なときに使用する。（外部 URL から逆算する場合は scan-ui を使う）`

new_string: `description: プロジェクトルートに DESIGN.md（リポジトリの視覚的アイデンティティ定義）を作成・更新する。google-labs-code/design.md 仕様に準拠し、YAML front matter の機械可読デザイントークン（色・タイポグラフィ・余白・コンポーネント）と散文のデザイン指針を、既存コード・CSS・トークンの分析と、コードから読めない意図（トーン・参照・ブランド制約）のヒアリングから生成する。DESIGN.md作成・デザインシステム定義・ガイドライン策定・デザイントークン整理・ブランド定義を依頼されたとき、またデザインガイドラインが不明確なときに使用する。design-ui / implement-ui から DESIGN.md 不在時に委譲される。（外部 URL から逆算する場合は scan-ui を使う）`

- [ ] **Step 2: MANDATORY PREPARATION の先頭に `interview.md` を追加する**

old_string:
```markdown
- `ui-design-grounding/reference/design-md-spec.md`
- `ui-design-grounding/reference/design-system.md`
```

new_string:
```markdown
- `ui-design-grounding/reference/design-md-spec.md`
- `ui-design-grounding/reference/interview.md`
- `ui-design-grounding/reference/design-system.md`
```

- [ ] **Step 3: Step 1「入力の収集」を「抽出確度の判定とヒアリング」に置換する**

old_string:
```markdown
### Step 1: 入力の収集

不明な点は仮置き + 明示。
- プロダクトの性質（toB / toC / 社内）・ターゲット・喚起したい感情
- ブランドの視覚的基調 — **具体的な参照**を引き出す（「何に似ているか」を1つ）
- 既存デザイン資産（Figma、CSS、コンポーネントライブラリ）・技術制約
```

new_string:
```markdown
### Step 1: 抽出確度の判定とヒアリング

まず既存コード資産（CSS / トークン / コンポーネント / テーマ設定）を軽く走査して**抽出確度**を判定し、`interview.md` の5原則（決定木を1問ずつ・調べれば分かることは聞かない・推奨回答を添える）でヒアリングする。**聞けば分かることを無断で仮置きしない**（仮置きは質問しても確定しなかった場合の最後の手段）。

- **資産が豊富**（確立されたトークン・コンポーネント群がある）→ 値は Step 2 の抽出で得る。**コードから読めない意図項目だけ**質問する:
  - 現状のデザインを追認して基準化してよいか（それとも直したい方向があるか）
  - プロダクトの性質（toB / toC / 社内）・ターゲット・喚起したい感情（トーン）
  - ブランドの視覚的基調 — **具体的な参照**を引き出す（「何に似ているか」を1つ）・アンチ参照（何に似せないか）
  - ブランドガイドライン等のハード制約・対象デバイス・アクセシビリティ基準
- **資産が薄い / 無い**（新規プロジェクト等）→ 上記に加え、既存デザイン資産（Figma、コンポーネントライブラリ）・技術制約を含めて本格的にヒアリングする。

**他スキルから委譲された場合**（`design-ui` / `implement-ui` の前段ゲート経由）:
- 「これから基準づくりの質問を数問する」と宣言してから始める。
- ユーザーが急ぐ場合は**最小構成 DESIGN.md**（Summary + 主要トークン + 聞けた範囲の意図）で切り上げ、未確定項目を「要確認事項」に明示する。
```

- [ ] **Step 4: 注意欄の仮置き記述を置換する**

old_string: `- 不明な値は仮置きし `/* TODO: 確定待ち */` で明示する`
new_string: `- 質問しても確定しなかった値のみ仮置きし `/* TODO: 確定待ち */` で明示する（聞けば分かることを無断で仮置きしない）`

- [ ] **Step 5: 検証**

Run: `Read skills/init-design/SKILL.md` → 「不明な点は仮置き + 明示」が消え、抽出確度の2分岐・委譲時の宣言・最小構成の逃げ道が入っている。`Grep "interview.md" skills/init-design/SKILL.md` → 1件以上。

- [ ] **Step 6: コミット**

```bash
git add skills/init-design/SKILL.md
git commit -m "feat: init-design に抽出確度判定とヒアリングを導入（無断の仮置きを廃止）"
```

---

### Task 6: `design-ui` の全面改訂（インタビュー＋ブリーフ保存）

**Files:**
- Modify: `skills/design-ui/SKILL.md`（全文置換）

**Interfaces:**
- Consumes: Task 1 `interview.md`（発動判定「明確化状況」・機能単位の質問）、Task 2 `design-brief.md`（テンプレート・保存先・昇格導線）、Task 5 の init-design 委譲動作
- Produces: `.design/<feature-slug>/DESIGN_BRIEF.md`。Task 7 の implement-ui が読み込む。

- [ ] **Step 1: ファイル全文を以下の内容で置換する**

````markdown
---
name: design-ui
description: 要件・仕様からUI/UXの構造と設計方針を整理し、機能単位のデザインブリーフ（.design/<feature-slug>/DESIGN_BRIEF.md）として保存する。要件が曖昧なときはインタビューで明確化してから設計する。新規UI設計・画面設計・UI構造の検討・デザインブリーフ作成・実装前の要件整理を依頼されたときに使用する。
user-invocable: true
argument-hint: "[要件、課題、シナリオ...]"
---

# design-ui

## 準備（MANDATORY PREPARATION）

ui-design-grounding スキルを呼び出し、以下のリファレンスを読み込む:

- `ui-design-grounding/reference/interview.md`
- `ui-design-grounding/reference/design-brief.md`
- `ui-design-grounding/reference/information-arch.md`
- `ui-design-grounding/reference/wording.md`
- `ui-design-grounding/reference/design-system.md`
- `ui-design-grounding/reference/design-tokens.md`
- `ui-design-grounding/reference/color-system.md`
- `ui-design-grounding/reference/accessibility.md`
- `ui-design-grounding/reference/typography.md`
- `ui-design-grounding/reference/spatial-layout.md`
- `ui-design-grounding/reference/responsive-design.md`
- `ui-design-grounding/reference/design-md-gate.md`

## 入力

- 要望・課題
- 利用シナリオ
- 対象ユーザー
- 制約条件

## 手順

### 0. 基準の確認（DESIGN.md 前段ゲート）

`design-md-gate.md` の **前段ゲート** を実施する。DESIGN.md があれば front matter のトークンと本文の指針を**制約として読み込み**（リファレンスより優先）、構造設計はその視覚基準の中で行う。

**DESIGN.md が無い場合は提案に留めず、`/init-design` へ委譲して基準を先に作る**:

- 「これから基準づくりの質問を数問する」と宣言してから init-design を開始する。
- ユーザーが急ぐ場合は最小構成 DESIGN.md（Summary + 主要トークン + 聞けた範囲の意図）で切り上げてよい。
- DESIGN.md ができたら本スキルへ復帰する。

**本スキルは構造（画面・遷移・情報設計）を設計するもので、DESIGN.md（視覚的憲法）は書き換えない。**

### 1. 要件の明確化判定とインタビュー

要件の明確化状況を判定する: 対象画面・主要ユーザー・成功条件が入力から特定できるか。

- **特定できる** → インタビューを省略し、把握した前提を一度だけ要約して確認する。
- **曖昧・複数解釈がある** → `interview.md` の5原則でインタビューを実施する。質問は機能単位（主要ユーザーと JTBD / 成功の定義 / コンテンツ / 機能固有の制約 / スコープ外）。**DESIGN.md・コードベースが答えを持つ質問は聞かずに調べる。**

インタビュー中に DESIGN.md 級の話題（トーン・参照・新トークン）が出たら、ブリーフに記録して手順8の昇格検出に回す。

2. **UX目的の言語化**: この UI が達成すべきユーザー体験を明文化する
3. **画面・状態の整理**: 必要な画面と各画面の状態（初期/ローディング/成功/エラー/空）を洗い出す
4. **情報構造と画面遷移**: 情報の階層、グルーピング、ナビゲーション、画面間の遷移を設計する
5. **既存資産の確認**: 既存のコンポーネント・CSS・トークン資産を確認し、再利用可能なものを特定する
6. **ワーディング方向性**: ラベル、メッセージ、ガイダンスの方向性を検討する
7. **実装を見据えた構造提案**: レスポンシブ、アクセシビリティ、パフォーマンスを考慮した構造を提案する

### 8. ブリーフの保存と昇格検出

- 設計結果を `design-brief.md` のテンプレートに従い `.design/<feature-slug>/DESIGN_BRIEF.md` に保存する（`<feature-slug>` は機能・画面から導いた小文字ハイフン区切り）。
- DESIGN.md 級の恒久的決定（トーンの明確化・新トークン候補・画面横断の新規約）が生まれていれば「DESIGN.md へ昇格すべき決定」として列挙し、`/init-design` を提案する（本スキルからは書き換えない）。

## 出力フォーマット

DESIGN_BRIEF.md（`design-brief.md` のテンプレート準拠）を保存したうえで、会話では要約を示す:

```markdown
## 設計サマリ
- ブリーフ保存先: `.design/<feature-slug>/DESIGN_BRIEF.md`
- UX目的: ...
- 画面構成: [画面一覧と遷移の要点]
- 体験原則: [最大3つ]

## DESIGN.md へ昇格すべき決定（あれば）
- [決定]: → `/init-design`

## 要確認事項（あれば）
- [インタビューで確定しなかった判断・推奨で埋めた判断]

## 推奨される次のステップ
- `/implement-ui`（DESIGN.md とブリーフを基準にコンポーネント分解・実装）
```

## 注意

- 見た目の細部を断定しない（色、フォント等の具体値は参考程度。視覚基準は DESIGN.md に委ねる）
- 実装可能性を無視しない
- 画面デザインの完成ではなく、「考え方」と「構造」を明確にし、実装に受け渡せる形（ブリーフ）で残すことが目的
- DESIGN.md は制約として読むが、本スキルは DESIGN.md を書き換えない（視覚的憲法の定義・更新は `/init-design`）
````

- [ ] **Step 2: 検証**

Run: `Read skills/design-ui/SKILL.md` → 手順0に委譲、手順1に明確化判定、手順8にブリーフ保存と昇格検出がある。`Grep "DESIGN_BRIEF" skills/design-ui/SKILL.md` → 2件以上。

- [ ] **Step 3: コミット**

```bash
git add skills/design-ui/SKILL.md
git commit -m "feat: design-ui にインタビューと DESIGN_BRIEF.md 保存・昇格検出を統合"
```

---

### Task 7: `implement-ui` の改修（ブリーフ受け口）

**Files:**
- Modify: `skills/implement-ui/SKILL.md`

**Interfaces:**
- Consumes: Task 2 `design-brief.md`（保存先・テンプレート）、Task 5 の init-design 委譲動作、Task 6 が生成するブリーフ

- [ ] **Step 1: frontmatter の `description` を更新する**

old_string: `description: 新規UIを実装する第1層オーケストレーター。要件・デザインを受け、DESIGN.md を基準に観点（色・タイポ・レイアウト・モーション・文言・堅牢性・性能）を横断的に通しながら、必要に応じて第2層スキルへ委譲して実装まで担う。デザインからの実装・新規画面実装・コンポーネント実装を依頼されたときに使用する。`

new_string: `description: 新規UIを実装する第1層オーケストレーター。要件・デザイン・デザインブリーフ（.design/<feature-slug>/DESIGN_BRIEF.md、あれば）を受け、DESIGN.md を基準に観点（色・タイポ・レイアウト・モーション・文言・堅牢性・性能）を横断的に通しながら、必要に応じて第2層スキルへ委譲して実装まで担う。デザインからの実装・新規画面実装・コンポーネント実装を依頼されたときに使用する。`

- [ ] **Step 2: MANDATORY PREPARATION に `design-brief.md` を追加する**

old_string:
```markdown
- `ui-design-grounding/reference/design-md-gate.md`
- `ui-design-grounding/reference/design-system.md`
```

new_string:
```markdown
- `ui-design-grounding/reference/design-md-gate.md`
- `ui-design-grounding/reference/design-brief.md`
- `ui-design-grounding/reference/design-system.md`
```

- [ ] **Step 3: 手順0に委譲を追加する**

old_string: `` `design-md-gate.md` の **前段ゲート** を実施する。DESIGN.md があれば front matter のトークンと本文の指針を**基準（source of truth）として読み込み**（リファレンスより優先）、既存トークンで表現できるものは新しい値を持ち込まない。無ければ未整備として扱い `/init-design`（外部 URL からは `/scan-ui`）を提案する。基準なしに実装を始めない。``

new_string: `` `design-md-gate.md` の **前段ゲート** を実施する。DESIGN.md があれば front matter のトークンと本文の指針を**基準（source of truth）として読み込み**（リファレンスより優先）、既存トークンで表現できるものは新しい値を持ち込まない。

**DESIGN.md が無い場合は提案に留めず、`/init-design` へ委譲して基準を先に作る**（「これから基準づくりの質問を数問する」と宣言。急ぎなら最小構成 DESIGN.md で切り上げ可）。DESIGN.md ができたら本スキルへ復帰する。基準なしに実装を始めない。``

- [ ] **Step 4: 手順1にブリーフ読み込みを追加する**

old_string:
```markdown
- 何を作るか（画面・コンポーネント・フロー）と入力（デザイン / 要件 / 制約）を確認する。
- 各コンポーネントの UI 状態を洗い出す: Initial / Loading / Success / Error / Empty。
```

new_string:
```markdown
- 何を作るか（画面・コンポーネント・フロー）と入力（デザイン / 要件 / 制約）を確認する。
- `.design/<feature-slug>/DESIGN_BRIEF.md` の有無を確認し、あれば読み込んで**機能単位の判断基準**にする（DESIGN.md = 恒久の視覚基準、ブリーフ = この機能の UX 判断。矛盾する場合は DESIGN.md を優先し、乖離として報告する）。
- 各コンポーネントの UI 状態を洗い出す: Initial / Loading / Success / Error / Empty。
```

- [ ] **Step 5: 出力フォーマットに「ブリーフとの対応」を追加する**

old_string:
```markdown
## 横断波及対応
- [波及]: 追加で `/xxx-ui` を実施
```

new_string:
```markdown
## 横断波及対応
- [波及]: 追加で `/xxx-ui` を実施

## ブリーフとの対応（ブリーフがある場合）
- 体験原則: [各原則を実装でどう反映したか]
- コンポーネント一覧との突合: [計画どおり / 差分]
- スコープ外の遵守: [ブリーフのスコープ外に踏み込んでいないか]
```

- [ ] **Step 6: 検証**

Run: `Grep "DESIGN_BRIEF\|design-brief" skills/implement-ui/SKILL.md` → description・参照リスト・手順1・出力フォーマットの4箇所以上でヒット。

- [ ] **Step 7: コミット**

```bash
git add skills/implement-ui/SKILL.md
git commit -m "feat: implement-ui にデザインブリーフの受け口を追加"
```

---

### Task 8: `ui-design-grounding/SKILL.md` の参照ナビ更新

**Files:**
- Modify: `skills/ui-design-grounding/SKILL.md`

**Interfaces:**
- Consumes: Task 1・2 の新リファレンス

- [ ] **Step 1: リファレンス件数を更新する**

old_string: `- 20個のリファレンス（`reference/`）にUI/UX設計の判断基準・原則・パターン・観察手順を集約`
new_string: `- 22個のリファレンス（`reference/`）にUI/UX設計の判断基準・原則・パターン・観察手順を集約`

- [ ] **Step 2: コマンドスキル一覧の design-ui 行を更新する**

old_string: `| 要件からUI構造を設計 | `/design-ui` |`
new_string: `| 要件からUI構造を設計しブリーフ化 | `/design-ui` |`

- [ ] **Step 3: 参照ナビゲーション「実装・システム」に2行を追加する**

old_string:
```markdown
- `design-md-gate.md` — DESIGN.md ゲート（前段=基準読込／後段=乖離時の誘導）の手順
- `implementation.md` — コンポーネント粒度、責務分離、UI状態管理
```

new_string:
```markdown
- `design-md-gate.md` — DESIGN.md ゲート（前段=基準読込／後段=乖離時の誘導）の手順
- `interview.md` — インタビュー5原則（決定木を1問ずつ・調べれば分かることは聞かない・推奨回答つき）、発動判定、質問の帰属（`init-design` / `design-ui` 共通）
- `design-brief.md` — DESIGN_BRIEF.md のテンプレート・保存先（`.design/<feature-slug>/`）・DESIGN.md との関係・昇格導線・`.design/` 全体構造
- `implementation.md` — コンポーネント粒度、責務分離、UI状態管理
```

- [ ] **Step 4: `ui-report.md` の一行要約を更新する**

old_string: `- `ui-report.md` — 評価系スキルの Markdown レポート保存先、共通メタ情報、スクリーンショットリンク、未検証・制約の共通ルール`
new_string: `- `ui-report.md` — 評価系スキルの Markdown レポート保存先（`.design/reports/`）、共通メタ情報、スクリーンショットリンク、未検証・制約の共通ルール`

- [ ] **Step 5: 検証とコミット**

Run: `Grep "interview.md\|design-brief.md" skills/ui-design-grounding/SKILL.md` → 参照ナビに2行。

```bash
git add skills/ui-design-grounding/SKILL.md
git commit -m "docs: 参照ナビゲーションに interview.md / design-brief.md を追加"
```

---

### Task 9: `AGENTS.md` の更新

**Files:**
- Modify: `AGENTS.md`

- [ ] **Step 1: バージョンとリファレンス件数を更新する**

old_string: `- 現行バージョン: `1.3.0``
new_string: `- 現行バージョン: `1.4.0``

old_string: `    reference/                 20件のリファレンス（UI/UX原則＋共通手順）`
new_string: `    reference/                 22件のリファレンス（UI/UX原則＋共通手順）`

old_string: `   - `SKILL.md` と `reference/` 配下の 20 件のリファレンス文書で構成します。`
new_string: `   - `SKILL.md` と `reference/` 配下の 22 件のリファレンス文書で構成します。`

old_string: `   - ユーザビリティ、認知科学、情報設計、色彩、タイポグラフィ、空間レイアウト、インタラクション状態、モーション、アクセシビリティ、レスポンシブ、ワーディング、デザインシステム、デザイントークン、初見の分かりやすさ、DESIGN.md 仕様、DESIGN.md ゲート、実装パターン、アンチパターン、Playwright MCP 観察手順、評価レポート出力ルールを扱います。`
new_string: `   - ユーザビリティ、認知科学、情報設計、色彩、タイポグラフィ、空間レイアウト、インタラクション状態、モーション、アクセシビリティ、レスポンシブ、ワーディング、デザインシステム、デザイントークン、初見の分かりやすさ、DESIGN.md 仕様、DESIGN.md ゲート、インタビュー手法、デザインブリーフ規約、実装パターン、アンチパターン、Playwright MCP 観察手順、評価レポート出力ルールを扱います。`

- [ ] **Step 2: 「DESIGN.md の扱い」節の末尾に2項目を追加する**

old_string: `- `DESIGN.md` を自動で大きく書き換える場合は、人間の承認が前提です。`
new_string: `- `DESIGN.md` を自動で大きく書き換える場合は、人間の承認が前提です。
- `design-ui` は要件をインタビューで明確化し、機能単位の判断を `.design/<feature-slug>/DESIGN_BRIEF.md` に残します。`implement-ui` はブリーフがあれば読み込みます。
- `design-ui` / `implement-ui` は DESIGN.md 不在時に `/init-design` へ委譲して基準を先に作ります（評価系は提案止まり）。`

- [ ] **Step 3: 代表的なワークフローと保存先を更新する**

old_string: `- **新規 UI**: `/init-design` → `/design-ui` → `/implement-ui` → `/audit-ui``
new_string: `- **新規 UI**: `/design-ui`（DESIGN.md が無ければ `/init-design` へ委譲 → 要件インタビュー → ブリーフ生成）→ `/implement-ui` → `/audit-ui``

old_string: `詳細な評価結果は、対象プロジェクトの `ui-reports/YYYY-MM-DD/` に Markdown レポートとして保存されます。`
new_string: `詳細な評価結果は、対象プロジェクトの `.design/reports/YYYY-MM-DD/` に Markdown レポートとして保存されます。`

- [ ] **Step 4: リファレンス一覧表を更新する**

old_string:
```markdown
| `design-md-gate.md` | DESIGN.md ゲート（前段 / 後段）の共通プロトコル |
| `implementation.md` | コンポーネント粒度、責務分離、UI 状態管理 |
```

new_string:
```markdown
| `design-md-gate.md` | DESIGN.md ゲート（前段 / 後段）の共通プロトコル |
| `interview.md` | インタビュー5原則、発動判定、質問の帰属 |
| `design-brief.md` | DESIGN_BRIEF.md テンプレート、`.design/` 構造、昇格導線 |
| `implementation.md` | コンポーネント粒度、責務分離、UI 状態管理 |
```

old_string: `| `ui-report.md` | 評価レポート保存先、共通メタ情報、スクリーンショットリンク |`
new_string: `| `ui-report.md` | 評価レポート保存先（`.design/reports/`）、共通メタ情報、スクリーンショットリンク |`

- [ ] **Step 5: 「1.3.0 で追加された最新運用」節を 1.4.0 の内容に置き換える**

old_string:
```markdown
## 1.3.0 で追加された最新運用

- `legibility-ui` を追加しました。実装を読む前に画面だけを見て、初見の分かりやすさを 6 レンズで監査します。
- `legibility.md` を追加しました。見た目と機能の一致、現在地、目的、スコープ、重複、画面間一貫性を判断基準として扱います。
- `playwright.md` を追加しました。評価系・修正系スキル共通の実地観察手順と一括監査スイープを定義します。
- `ui-report.md` を追加しました。評価系スキルの Markdown レポート保存先、共通メタ情報、スクリーンショットリンクを統一します。
- 評価レポートは対象プロジェクトの `ui-reports/YYYY-MM-DD/` に保存します。
```

new_string:
```markdown
## 1.4.0 で追加された最新運用

- `interview.md` を追加しました。`init-design` / `design-ui` 共通のインタビュープロトコル（5原則・発動判定・質問の帰属）を定義します。
- `design-brief.md` を追加しました。機能単位の設計判断を `.design/<feature-slug>/DESIGN_BRIEF.md` として残します。
- `design-ui` は要件の明確化状況を判定してインタビューし、設計結果をブリーフとして保存します。`implement-ui` はブリーフを読み込んで実装します。
- `init-design` は抽出確度を判定し、コードから読めない意図をヒアリングしてから DESIGN.md を生成します（無断の仮置きを廃止）。
- UI 作業の生成物の出力先を `.design/` に統合しました（評価レポート = `.design/reports/`、`preview.html` = `.design/preview.html`。DESIGN.md はルート常駐のまま）。
```

- [ ] **Step 6: 「編集時の注意事項」に1行を追加する**

old_string: `- **評価レポート**: 評価系スキルを編集するときは、`ui-report.md` の保存先・メタ情報・スクリーンショットリンク規約と整合させます。`
new_string: `- **評価レポート**: 評価系スキルを編集するときは、`ui-report.md` の保存先・メタ情報・スクリーンショットリンク規約と整合させます。
- **インタビュー・ブリーフ**: インタビューを行うスキル（`init-design` / `design-ui`）は `interview.md` の5原則・発動判定と、ブリーフを扱うスキル（`design-ui` / `implement-ui`）は `design-brief.md` の保存先・テンプレート規約と整合させます。`

- [ ] **Step 7: 検証とコミット**

Run: `Grep "ui-reports" AGENTS.md` → 0件。`Grep "1.3.0" AGENTS.md` → 0件。

```bash
git add AGENTS.md
git commit -m "docs: AGENTS.md を 1.4.0 の構成（インタビュー・ブリーフ・.design/）に更新"
```

---

### Task 10: `README.md` と `ui-help` の更新

**Files:**
- Modify: `README.md`
- Modify: `skills/ui-help/SKILL.md`

- [ ] **Step 1: README の第1層スキル表（89-90行付近）を更新する**

old_string: `| `/design-ui` | UI 構造、画面遷移、状態設計を考える |`
new_string: `| `/design-ui` | UI 構造、画面遷移、状態設計を考え、デザインブリーフとして残す |`

- [ ] **Step 2: README「評価レポート」節を「UI 作業の出力先（.design/）」に改訂する**

old_string:
````markdown
## 評価レポート

評価系スキルは、結果を Markdown レポートとして保存します。

対象プロジェクトに次のようなディレクトリが作られます。

```text
ui-reports/
  YYYY-MM-DD/
    HHmmss-audit-ui.md
    HHmmss-score-ui.md
    HHmmss-legibility-ui.md
    screenshots/
      HHmmss-audit-ui-01.png
```

スクリーンショットを取得した場合は、レポート内から相対リンクされます。レビュー後に、何を見て、何を根拠に判断したかを追いやすくするためです。
````

new_string:
````markdown
## UI 作業の出力先（.design/）

このプラグインのスキルが生成するファイルは、DESIGN.md（プロジェクトルート常駐）を除き、対象プロジェクトの `.design/` に集約されます。

```text
.design/
  <feature-slug>/
    DESIGN_BRIEF.md          ← /design-ui が保存する機能単位の設計判断
  reports/
    YYYY-MM-DD/
      HHmmss-audit-ui.md
      HHmmss-score-ui.md
      HHmmss-legibility-ui.md
      screenshots/
        HHmmss-audit-ui-01.png
  preview.html               ← /preview-ui が生成する DESIGN.md の見本帳
```

DESIGN_BRIEF.md は `/implement-ui` が読み込む受け渡しファイルのため、コミットしておくのがおすすめです。

評価系スキルのスクリーンショットは、レポート内から相対リンクされます。レビュー後に、何を見て、何を根拠に判断したかを追いやすくするためです。
````

- [ ] **Step 3: README のリファレンス表に2行を追加する**

old_string:
```markdown
| `playwright.md` | Playwright MCP による実地観察 |
| `ui-report.md` | 評価レポートの保存先、メタ情報、スクリーンショットリンク |
```

new_string:
```markdown
| `playwright.md` | Playwright MCP による実地観察 |
| `ui-report.md` | 評価レポートの保存先、メタ情報、スクリーンショットリンク |
| `interview.md` | インタビュー5原則、発動判定、質問の帰属 |
| `design-brief.md` | デザインブリーフのテンプレートと `.design/` 構造 |
```

- [ ] **Step 4: README「ライセンス」節の前に「着想元」節を追加する**

old_string:
```markdown
## ライセンス

MIT
```

new_string:
```markdown
## 着想元

インタビューとデザインブリーフの仕組みは、本プラグインのアーキテクチャに合わせて再設計したものです。

## ライセンス

MIT
```

- [ ] **Step 5: `ui-help/SKILL.md` のコマンド表を更新する**

old_string: `| `/design-ui` | **考える** — 要件からUI構造・画面設計を整理。画面遷移・情報階層・UI状態（初期/ローディング/成功/エラー/空）を設計（DESIGN.md は制約として読むが書かない） | 新しい画面を設計したい |`
new_string: `| `/design-ui` | **考える** — 要件が曖昧ならインタビューで明確化し、UI構造・画面遷移・情報階層・UI状態を設計して `.design/<feature-slug>/DESIGN_BRIEF.md` に保存（DESIGN.md は制約として読むが書かない） | 新しい画面を設計したい |`

old_string: `| `/implement-ui` | **作る** — DESIGN.md を基準に観点を横断し、第2層へ委譲しながら新規UIを実装まで担う第1層オーケストレーター | デザイン・要件から実装したい |`
new_string: `| `/implement-ui` | **作る** — DESIGN.md とデザインブリーフ（あれば）を基準に観点を横断し、第2層へ委譲しながら新規UIを実装まで担う第1層オーケストレーター | デザイン・要件から実装したい |`

old_string: `| `/init-design` | **基準化** — DESIGN.md を生成・更新。design.md 仕様（YAML トークン + Summary + 8セクション散文）準拠。既存CSS/トークンから自動抽出対応 | デザインシステムの基盤を定義したい |`
new_string: `| `/init-design` | **基準化** — DESIGN.md を生成・更新。design.md 仕様（YAML トークン + Summary + 8セクション散文）準拠。既存CSS/トークンからの抽出と、コードから読めない意図のヒアリングに対応 | デザインシステムの基盤を定義したい |`

- [ ] **Step 6: 検証とコミット**

Run: `Grep "ui-reports" README.md skills/ui-help/SKILL.md` → 0件。

```bash
git add README.md skills/ui-help/SKILL.md
git commit -m "docs: README / ui-help を .design/ 出力とブリーフ対応に更新（着想元クレジット追加）"
```

---

### Task 11: バージョンと変更履歴

**Files:**
- Modify: `.claude-plugin/plugin.json`
- Modify: `CHANGELOG.md`

- [ ] **Step 1: `plugin.json` のバージョンを更新する**

old_string: `  "version": "1.3.0",`
new_string: `  "version": "1.4.0",`

- [ ] **Step 2: `CHANGELOG.md` に 1.4.0 節を追加する**

old_string: `## [1.3.0] - 2026-07-07`
new_string: `## [1.4.0] - 2026-07-17

- `interview.md` を追加（`init-design` / `design-ui` 共通のインタビュープロトコル。決定木を1問ずつ・調べれば分かることは聞かない・推奨回答つき）
- `design-brief.md` を追加（機能単位の設計判断を `.design/<feature-slug>/DESIGN_BRIEF.md` として保存する規約とテンプレート）
- `design-ui` を拡張（要件の明確化状況を判定してインタビュー → 設計結果をデザインブリーフとして保存 → DESIGN.md へ昇格すべき決定の検出）
- `init-design` を拡張（抽出確度を判定し、コードから読めない意図をヒアリング。無断の仮置きを廃止し、最小構成 DESIGN.md を定義）
- `implement-ui` にブリーフの受け口を追加（DESIGN.md = 恒久基準、ブリーフ = 機能判断として読み込み）
- `design-ui` / `implement-ui` は DESIGN.md 不在時に `/init-design` へ委譲する運用に変更
- UI 作業の生成物の出力先を `.design/` に統合（評価レポート = `.design/reports/`、`preview.html` = `.design/preview.html`。DESIGN.md はルート常駐のまま）
- 関連: design-brief（Julian Oczkowski, Apache-2.0）

## [1.3.0] - 2026-07-07`

- [ ] **Step 3: コミット**

```bash
git add .claude-plugin/plugin.json CHANGELOG.md
git commit -m "chore: バージョン 1.4.0（インタビュー・ブリーフ統合と .design/ 出力統一）"
```

---

### Task 12: 整合検証（残存チェックとサブエージェント読解テスト）

**Files:**
- 変更なし（検証のみ。問題があれば該当タスクのファイルを修正）

- [ ] **Step 1: 旧パスの残存をチェックする**

Run: `Grep -n "ui-reports" <リポジトリルート>` → ヒットは `docs/superpowers/` 配下（歴史文書）と本 plan/spec のみ。skills/ / AGENTS.md / README.md に残存があれば該当タスクに戻って修正。
Run: `Grep -n "プロジェクトルート（DESIGN.md の隣）" skills/` → 0件。

- [ ] **Step 2: バージョン整合をチェックする**

Run: `Grep -n "1\.4\.0" .claude-plugin/plugin.json AGENTS.md CHANGELOG.md` → 3ファイルすべてでヒット。

- [ ] **Step 3: サブエージェント読解テスト（design-ui の発動判定）**

サブエージェント（Read/Grep のみ許可）に `skills/design-ui/SKILL.md` と `skills/ui-design-grounding/reference/interview.md` を読ませ、次の2シナリオでの動作を答えさせる:

1. 「社内管理者向けのユーザー招待画面。対象ユーザー・成功条件・画面数が明記された要件」→ 期待: インタビュー省略、前提の要約確認のみ。ブリーフは `.design/user-invitation/DESIGN_BRIEF.md` 等に保存。
2. 「なんかいい感じのダッシュボードが欲しい、とだけ言われた」→ 期待: インタビュー実施。質問は1つずつ・推奨回答つき。DESIGN.md にトーン定義があればトーンは聞かない。

期待と異なる読解（例: シナリオ1でも全質問を実施する、質問をまとめて投げる）が返ったら、`design-ui/SKILL.md` または `interview.md` の該当記述を明確化し、再テストする。

- [ ] **Step 4: サブエージェント読解テスト（init-design の抽出確度判定）**

サブエージェントに `skills/init-design/SKILL.md` と `interview.md` を読ませ、「確立された tokens.css とコンポーネントライブラリを持つプロジェクトで DESIGN.md を作る」シナリオでの動作を答えさせる。期待: 値は抽出で得て、質問は意図項目（現状追認・トーン・参照/アンチ参照・制約）のみ。トークン値をユーザーに聞き始めたら記述を修正して再テスト。

- [ ] **Step 5: 修正があれば追加コミット、なければ完了報告**

```bash
git add -A
git commit -m "fix: 読解テストで判明した記述の明確化"   # 修正があった場合のみ
```

## Self-Review 済み事項

- spec 9章の反映先ファイル一覧と Task 1〜11 の対応を確認済み（全ファイルにタスクあり）
- パス表記は Global Constraints の規約に統一（`.design/reports/YYYY-MM-DD/HHmmss-<skill>.md` 等）
- `refine-ui` は意図的に変更しない（spec 10章 スコープ外）
- `scan-ui` の DESIGN.md 出力先はルートのまま（変更不要のため対象外）
