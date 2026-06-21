# スキル群 5動詞再編 実装プラン

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** ADR（[docs/skill-architecture.md](../../skill-architecture.md)）で決着した「第1層5動詞＋第2層実働ユニット＋DESIGN.md ゲート/常駐」を、実際のスキル群へ反映する。

**Architecture:** 第1層を「考える(design)／作る(implement)／直す(refine)／評価(audit・score)／基準化(init)」の5動詞に整える。`implement-ui` は薄い実働オーケストレーターへ全面改訂、`refine-ui` を新設。両者は前段ゲートで DESIGN.md を読み、観点を横断して第2層へ委譲し、横断波及判定で収束させる。第2層（12観点スキル）は既に実働＋ゲート参照済みのため個別改修せず、「最小自己波及チェック」を共通ゲートへ1箇所追加して継承させる。DESIGN.md 仕様にサマリ層・トークン外出し・スリム化を足す。最後にナビ3面（ルート SKILL／ui-help／CLAUDE.md 早見表）へ refine を反映する。

**Tech Stack:** 純粋な Markdown プラグイン。ビルド・テスト・リントは無い（[CLAUDE.md](../../../CLAUDE.md) 参照）。各タスクの「検証」は対象ファイルを Read / Grep で読み、判定基準を満たすか確認することで行う。

## Global Constraints

- 全スキルコンテンツは**日本語**で記述する。
- 各スキルは YAML フロントマター（`name` / `description`、コマンドスキルは `user-invocable: true` と `argument-hint`）＋ Markdown 本文の `SKILL.md` 単一ファイル。
- フロントマターの `description` はスキルマッチングに使われるため、具体的かつ網羅的に書く。
- 第1層オーケストレーター（design / implement / refine）は**前段ゲートのみ**を持ち、後段（乖離検出）は実働層（第2層・recolor・polish）に委ねる。
- 本スキル群の価値は「判断軸を効かせて**実装・修正まで完遂する**」こと。教育的可視化を主目的にしない（教材的言辞を持ち込まない）。
- DESIGN.md は**自動で書き換えない**。憲法の更新は `/init-design`（色は `/recolor-ui`）へ誘導する。
- コミットはタスク単位。コミットメッセージ末尾に `Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>` を付す。

## File Structure

| ファイル | 役割 | 本プランでの扱い |
|---------|------|----------------|
| `skills/ui-design-grounding/reference/design-md-gate.md` | 共通ゲート | 後段に「最小自己波及チェック」を追加（第2層が継承） |
| `skills/implement-ui/SKILL.md` | 第1層・作る | 全面改訂（薄い実働オーケストレーター） |
| `skills/refine-ui/SKILL.md` | 第1層・直す | 新規作成 |
| `skills/design-ui/SKILL.md` | 第1層・考える | 前段ゲート追加・「DESIGN.md は書かない」明記 |
| `skills/ui-design-grounding/reference/design-md-spec.md` | DESIGN.md 仕様 | サマリ層・トークン外出し・スリム化を追加 |
| `skills/init-design/SKILL.md` | 基準化 | サマリ層・スリム化（重複統合）に対応 |
| `skills/ui-design-grounding/SKILL.md` | ルート（ナビ） | 件数 19→20・refine 追加・2層反映 |
| `skills/ui-help/SKILL.md` | コマンド一覧 | refine 追加・第1層/第2層に再構成 |
| `CLAUDE.md`（リポジトリroot） | 早見表 | refine 行追加・implement 行更新 |

**触らないと決めたもの（理由）**:
- 第2層12スキル（`boost` `calm` `typeset` `animate` `arrange` `slim` `clarify` `guard` `adapt` `optimize` `recolor` `polish`）の個別本文 — 既に実働フレーミング＆ゲート参照済み。波及チェックは共通ゲート継承で足りる（DRY）。
- `audit-ui` `score-ui` `extract-ui` `scan-ui` — 役割・ゲート区分に変更なし（`scan-ui` の出力契約は design-md-spec.md 継承でサマリ層に追従）。

---

## Task 1: 共通ゲートに「最小自己波及チェック」を追加

**Files:**
- Modify: `skills/ui-design-grounding/reference/design-md-gate.md`（後段ゲート節）

**Interfaces:**
- Produces: 第2層スキルが「`design-md-gate.md` の後段ゲートを実施する」と書くだけで波及チェックを継承できる状態。第1層の横断波及判定（Task 2/3）と役割分担する。

- [ ] **Step 1: 後段ゲート節に自己波及チェックを挿入**

`## 後段ゲート（乖離の検出と誘導）— 修正完了後に必ず実施` の直後（ブロック引用 `> 第1層（...）は ... 前段のみ）。...` の次）に、以下を挿入する。`old_string` は当該ブロック引用、`new_string` はブロック引用＋追加文とする。

挿入する内容:

```markdown

**S0. 最小自己波及チェック（実働層）**: 乖離検出の前に、今回の修正が**自分の観点の外側**を壊していないかを最小限で確認する。典型例: タイポ変更 → 行長・垂直リズム／色変更 → コントラスト・on-color ペア／余白変更 → 視覚階層。壊している場合は、その観点も直すか、第1層オーケストレーター（`refine` / `implement`）経由ならその**横断波及判定**に委ねる。直接呼びの場合はこの自己チェックが安全網になる。

以下は DESIGN.md との乖離検出と誘導:
```

- [ ] **Step 2: 検証（挿入を確認）**

Run: `Grep pattern "最小自己波及チェック" path "skills/ui-design-grounding/reference/design-md-gate.md" output_mode "content"`
Expected: 1件ヒットし、後段ゲート節（「乖離の検出と誘導」見出しの後、番号付きリスト「1. 今回の修正が...」の前）に位置している。

- [ ] **Step 3: コミット**

```bash
git add skills/ui-design-grounding/reference/design-md-gate.md
git commit -m "$(cat <<'EOF'
feat(gate): 後段ゲートに最小自己波及チェックを追加（第2層が継承）

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>
EOF
)"
```

---

## Task 2: implement-ui を第1層オーケストレーターへ全面改訂

**Files:**
- Modify（全置換）: `skills/implement-ui/SKILL.md`

**Interfaces:**
- Consumes: `design-md-gate.md`（前段）、第2層スキル名（`recolor-ui` `typeset-ui` `arrange-ui` `animate-ui` `clarify-ui` `guard-ui` `adapt-ui` `optimize-ui`）。
- Produces: 「作る」第1層入口。`design-ui`（Task 4 後）の成果や要件を受け、実装まで担う。ナビ（Task 7-9）が参照する name/description。

- [ ] **Step 1: implement-ui/SKILL.md を以下で全置換**

Write で以下の内容に置き換える（既存の「実装計画・構造翻訳／コード最終決定しない」版を破棄）:

````markdown
---
name: implement-ui
description: 新規UIを実装する第1層オーケストレーター。要件・デザインを受け、DESIGN.md を基準に観点（色・タイポ・レイアウト・モーション・文言・堅牢性・性能）を横断的に通しながら、必要に応じて第2層スキルへ委譲して実装まで担う。デザインからの実装・新規画面実装・コンポーネント実装を依頼されたときに使用する。
user-invocable: true
argument-hint: "[デザイン、要件、画面仕様...]"
---

# implement-ui

新規UIを「作る」第1層オーケストレーター。判断軸（観点）を効かせながら**実装まで担いきる**。深掘りが要る観点は第2層スキルへ委譲する。

## 準備（MANDATORY PREPARATION）

ui-design-grounding スキルを呼び出し、以下のリファレンスを読み込む:

- `ui-design-grounding/reference/design-md-gate.md` — DESIGN.md ゲート（前段）の手順
- `ui-design-grounding/reference/design-system.md`
- `ui-design-grounding/reference/design-tokens.md`
- `ui-design-grounding/reference/implementation.md`

## 手順

### 0. 基準の確認（DESIGN.md 前段ゲート）

`design-md-gate.md` の **前段ゲート** を実施する。DESIGN.md があれば front matter のトークンと本文の指針を**基準（source of truth）として読み込み**（リファレンスより優先）、既存トークンで表現できるものは新しい値を持ち込まない。無ければ未整備として扱い `/init-design`（外部 URL からは `/scan-ui`）を提案する。基準なしに実装を始めない。

### 1. 実装対象と UI 状態の把握

- 何を作るか（画面・コンポーネント・フロー）と入力（デザイン / 要件 / 制約）を確認する。
- 各コンポーネントの UI 状態を洗い出す: Initial / Loading / Success / Error / Empty。
- 既存コンポーネント・CSS・トークン資産の再利用可否を確認する。

### 2. 観点を横断的に通す（実装）

DESIGN.md（無ければ reference）の基準に沿って実装する。各観点を漏れなく通す。軽微な観点は本タスク内で実装し、専門的な深掘りが要る観点だけ該当の第2層スキルへ委譲する（委譲先は実働ユニットなので、委譲すれば実装が完遂する）:

| 観点 | 確認すること | 深掘り時に委譲する第2層 |
|------|-------------|----------------------|
| 構造・責務 | コンポーネント分解（Atomic Design）・責務（表示 / ロジック / データ） | — |
| 色 | トークン準拠・コントラスト・on-color | `recolor-ui` |
| タイポグラフィ | スケール・階層・可読性 | `typeset-ui` |
| レイアウト・余白 | グリッド・スペーシングリズム・視覚階層 | `arrange-ui` |
| モーション | 状態遷移・マイクロインタラクション | `animate-ui` |
| 文言 | ラベル・エラー・空状態 | `clarify-ui` |
| 堅牢性 | オーバーフロー・i18n・エッジケース | `guard-ui` |
| レスポンシブ | ブレイクポイント・入力方式 | `adapt-ui` |
| 性能 | CWV・画像・フォント | `optimize-ui` |

### 3. 横断波及判定（第1層オーケストレーターの責務）

実装・委譲が一巡したら、ある観点の実装が**他観点に波及していないか**を判定する（例: タイポを変えた → 垂直リズム / 行長、色を変えた → コントラスト）。波及があれば該当する第2層を追加で呼び、収束させる。

### 4. 実装結果の提示

- 作成 / 変更したファイルと、各コンポーネントの構成・UI 状態を要約する。
- DESIGN.md の基準に対し新しい値を持ち込んだ箇所があれば明示する（値の確定は `/init-design`、色は `/recolor-ui` へ誘導）。

## 出力フォーマット

```markdown
## 実装サマリ
- 作成/変更ファイル: ...

## コンポーネント構成
| コンポーネント | 粒度 | 責務 | 再利用元 |
|-------------|------|------|---------|
| ... | Atom/Molecule/Organism | ... | 既存/新規 |

## UI状態
| コンポーネント | Initial | Loading | Success | Error | Empty |
|-------------|---------|---------|---------|-------|-------|
| ... | ... | ... | ... | ... | ... |

## 通した観点と委譲
- [観点]: 本体実装 / `/xxx-ui` へ委譲

## 横断波及対応
- [波及]: 追加で `/xxx-ui` を実施

## 基準との差分（あれば）
- [新しい値]: → `/init-design` or `/recolor-ui`
```

## 注意

- **計画で止めず、実装まで担う。**
- DESIGN.md がある場合は既存トークンを最優先で使い、新しい値を勝手に作らない。
- トークン水準の最終確定（DESIGN.md への反映）は自動で行わず `/init-design`・`/recolor-ui` へ誘導する（本スキルは前段のみ・後段は委譲先が担う）。

## 推奨される次のステップ

- `/audit-ui`（実装後の技術品質監査 — a11y・パフォーマンス・トークン準拠）
- `/polish-ui`（リリース前の仕上げ）
````

- [ ] **Step 2: 検証（実態と整合）**

Run: `Grep pattern "コードの最終決定は行わない|実装計画・構造翻訳" path "skills/implement-ui/SKILL.md" output_mode "content"`
Expected: ヒット 0件（旧フレーミングが残っていない）。

Run: `Grep pattern "横断波及判定|前段ゲート|実装まで担" path "skills/implement-ui/SKILL.md" output_mode "content"`
Expected: 3パターンすべてヒット。

- [ ] **Step 3: コミット**

```bash
git add skills/implement-ui/SKILL.md
git commit -m "$(cat <<'EOF'
feat(implement-ui): 第1層・薄い実働オーケストレーターへ全面改訂

前段ゲート・観点横断委譲・横断波及判定を備え、計画で止めず実装まで担う。

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>
EOF
)"
```

---

## Task 3: refine-ui を新設

**Files:**
- Create: `skills/refine-ui/SKILL.md`

**Interfaces:**
- Consumes: `design-md-gate.md`（前段）、第2層スキル名（全12観点）。
- Produces: 「直す」第1層入口。ナビ（Task 7-9）が参照する name/description。診断軸は観点ベース（うるささではない）。

- [ ] **Step 1: refine-ui/SKILL.md を以下で新規作成**

````markdown
---
name: refine-ui
description: 既存UIを直す第1層オーケストレーター。「なんか変・うるさい・読みにくい・ごちゃつく」等の曖昧な訴えを、DESIGN.md を基準に観点ベースで診断し（逸脱した観点を特定、複数可）、該当する第2層スキルへ委譲して修正まで担う。既存UIの改善・違和感の解消を依頼されたが、どの観点の問題か定まっていないときに使用する。観点が明確なら第2層スキルを直接使う。
user-invocable: true
argument-hint: "[対象と気になる点 (画面、違和感...)]"
---

# refine-ui

既存UIを「直す」第1層オーケストレーター。曖昧な訴えを観点へ翻訳し、該当する第2層スキルへ委譲して**修正まで担いきる**。

> **診断の基準は「うるささ」ではない。** 基準＝DESIGN.md（無ければ reference）からの**逸脱した観点**を特定する。うるさい / 地味（`calm` / `boost`）はその一分岐にすぎない。

## 準備（MANDATORY PREPARATION）

ui-design-grounding スキルを呼び出し、以下のリファレンスを読み込む:

- `ui-design-grounding/reference/design-md-gate.md` — DESIGN.md ゲート（前段）の手順
- `ui-design-grounding/reference/anti-patterns.md`
- `ui-design-grounding/reference/usability.md`

## 手順

### 0. 基準の確認（DESIGN.md 前段ゲート）

`design-md-gate.md` の **前段ゲート** を実施する。DESIGN.md があれば front matter のトークンと本文の指針を**基準として読み込む**（リファレンスより優先）。無ければ未整備として扱い `/init-design`（外部 URL からは `/scan-ui`）を提案する。基準なしに「問題なし」と断定しない。

### 1. 観点ベースで診断

訴え（「なんか変」「うるさい」「読みにくい」等）と対象 UI を基準に照らして観点ごとに点検し、**逸脱している観点を特定する（複数可）**:

| 逸脱の兆候 | 観点 | 委譲する第2層 |
|-----------|------|--------------|
| 階層が不明確・余白がバラバラ | レイアウト | `arrange-ui` |
| 文字が読みにくい・フォント階層が弱い | タイポグラフィ | `typeset-ui` |
| 色がちぐはぐ・コントラスト不足 | 色 | `recolor-ui` |
| 動きが無い／過剰 | モーション | `animate-ui` |
| 文言が曖昧・用語がバラバラ | 文言 | `clarify-ui` |
| 壊れやすい・i18n で崩れる | 堅牢性 | `guard-ui` |
| モバイルで崩れる | レスポンシブ | `adapt-ui` |
| 表示が遅い | 性能 | `optimize-ui` |
| 情報過多・ごちゃつく | 簡素化 | `slim-ui` |
| 派手すぎる・うるさい | 印象（抑える） | `calm-ui` |
| 地味・印象が弱い | 印象（強める） | `boost-ui` |

### 2. 委譲して直しきる

特定した観点それぞれについて、該当する第2層スキルへ委譲する。第2層は実働ユニットなので、委譲すれば修正が完遂する。複数観点に逸脱があれば順に委譲する。

### 3. 横断波及判定（第1層オーケストレーターの責務）

修正が一巡したら、ある観点の修正が**他観点に波及していないか**を判定する（例: タイポを変えた → 垂直リズム / 行長、色を変えた → コントラスト）。波及があれば該当する第2層を追加で呼び、収束させる。

### 4. 結果の提示

- 診断した逸脱観点と、委譲・修正の内容を要約する。
- 各第2層スキルの後段ゲートが検出した DESIGN.md 乖離があれば、その誘導（色 → `/recolor-ui`、他 → `/init-design`）をまとめて提示する。

## 出力フォーマット

```markdown
## 診断（観点ベース）
| 逸脱観点 | 根拠（基準との差） | 委譲先 |
|---------|------------------|-------|
| ... | ... | `/xxx-ui` |

## 修正内容
- [観点]: ... （`/xxx-ui` で実施）

## 横断波及対応
- [波及]: 追加で `/xxx-ui` を実施

## 基準との乖離（委譲先が検出したもの）
- 色 → `/recolor-ui` / その他 → `/init-design`
```

## 注意

- **助言で止めず、委譲を通じて修正まで担う。**
- 後段ゲート（DESIGN.md 乖離検出）は委譲先の第2層が実施する。本スキルは前段のみを持つ。
- 観点が最初から明確な場合は、本スキルを介さず第2層スキルを直接使ってよい。

## 推奨される次のステップ

- `/audit-ui` / `/score-ui`（修正後の品質評価）
- `/polish-ui`（リリース前の仕上げ）
````

- [ ] **Step 2: 検証（新設とフロントマター妥当性）**

Run: `Read file "skills/refine-ui/SKILL.md"`
Expected: フロントマターに `name: refine-ui` / `user-invocable: true` / `argument-hint` が揃い、診断表に11観点（arrange/typeset/recolor/animate/clarify/guard/adapt/optimize/slim/calm/boost）が並ぶ。「診断の基準は『うるささ』ではない」の注記がある。

- [ ] **Step 3: コミット**

```bash
git add skills/refine-ui/SKILL.md
git commit -m "$(cat <<'EOF'
feat(refine-ui): 第1層・直す（観点ベース診断→第2層委譲）を新設

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>
EOF
)"
```

---

## Task 4: design-ui に前段ゲートを追加

**Files:**
- Modify: `skills/design-ui/SKILL.md`

**Interfaces:**
- Consumes: `design-md-gate.md`（前段）。
- Produces: 第1層「考える」。DESIGN.md を**制約として読むが書かない**ことが明示された状態。

- [ ] **Step 1: MANDATORY PREPARATION に gate を追加**

`old_string`:
```markdown
- `ui-design-grounding/reference/responsive-design.md`
```
`new_string`:
```markdown
- `ui-design-grounding/reference/responsive-design.md`
- `ui-design-grounding/reference/design-md-gate.md` — DESIGN.md ゲート（前段）の手順
```

- [ ] **Step 2: 手順の先頭に Step 0（前段ゲート）を挿入**

`old_string`:
```markdown
## 手順

1. **UX目的の言語化**: この UI が達成すべきユーザー体験を明文化する
```
`new_string`:
```markdown
## 手順

### 0. 基準の確認（DESIGN.md 前段ゲート）

`design-md-gate.md` の **前段ゲート** を実施する。DESIGN.md があれば front matter のトークンと本文の指針を**制約として読み込み**（リファレンスより優先）、構造設計はその視覚基準の中で行う。無ければ未整備として扱い `/init-design`（外部 URL からは `/scan-ui`）を提案する。**本スキルは構造（画面・遷移・情報設計）を設計するもので、DESIGN.md（視覚的憲法）は書き換えない。**

1. **UX目的の言語化**: この UI が達成すべきユーザー体験を明文化する
```

- [ ] **Step 3: 注意に「DESIGN.md は書かない」を明示**

`old_string`:
```markdown
- 画面デザインの完成ではなく、「考え方」と「構造」を明確にすることが目的
```
`new_string`:
```markdown
- 画面デザインの完成ではなく、「考え方」と「構造」を明確にすることが目的
- DESIGN.md は制約として読むが、本スキルは DESIGN.md を書き換えない（視覚的憲法の定義・更新は `/init-design`）
```

- [ ] **Step 4: 検証**

Run: `Grep pattern "前段ゲート|DESIGN.md を書き換えない" path "skills/design-ui/SKILL.md" output_mode "content"`
Expected: 両パターンがヒット（前段ゲート追加・非書込明記）。

- [ ] **Step 5: コミット**

```bash
git add skills/design-ui/SKILL.md
git commit -m "$(cat <<'EOF'
feat(design-ui): 第1層「考える」に前段ゲートを追加（DESIGN.md は読むが書かない）

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>
EOF
)"
```

---

## Task 5: DESIGN.md 仕様にサマリ層・トークン外出し・スリム化を追加

**Files:**
- Modify: `skills/ui-design-grounding/reference/design-md-spec.md`

**Interfaces:**
- Produces: DESIGN.md の出力契約に「Summary（サマリ層）」と肥大化対策（トークン外出し・重複統合）を追加。`init-design`（Task 6）・`scan-ui` がこれを継承する。

> 注: サマリ層の具体形（下記 Summary ブロック）はこのプランで新規提案する書式。プランレビュー時に確認すること。

- [ ] **Step 1: 「本文8セクション」節の冒頭にサマリ層の説明を追加**

`old_string`:
```markdown
### 本文8セクション（順序固定）

`##` 見出しで以下の順に記述する。各セクションは**散文中心**で、哲学に従い具体的な参照・質感を込める。
```
`new_string`:
```markdown
### 本文 — サマリ層（任意・推奨）＋ 8セクション

本文の冒頭に、任意だが推奨の **`## Summary`（サマリ層）** を置ける。全観点の基準を 10〜20 行で一覧する忘却防止ブロックで、これ一読で色・タイポ・余白・角丸・深度・モーション・文言トーンの要点が掴める。詳細は続く8セクション（詳細層）に委ねる（front matter の機械可読トークンと役割分担し、Summary は人間が読む観点別の一行要約）。

その後、`##` 見出しで以下の8セクションを順に記述する。各セクションは**散文中心**で、哲学に従い具体的な参照・質感を込める。
```

- [ ] **Step 2: 出力例にサマリ層を追加**

`old_string`:
```markdown
## Overview

Aurora Notes は深夜の書斎の机上を思わせる。明かりを落とした部屋で1冊のノートだけが
```
`new_string`:
```markdown
## Summary

- **Brand**: 深夜の書斎 — 静かで集中を妨げないノートアプリ（toC・長文を書く人）
- **Color**: primary=blue-400（淡青・1画面1箇所）/ surface=neutral-0（藍）/ on-surface=neutral-900。純黒禁止
- **Type**: 見出し Newsreader(serif) / 本文 Inter。body 16px・行間 26px
- **Layout**: 8px グリッド / container 760px 単一カラム / <768px は下部タブ
- **Depth**: 影なし。背景段差（surface→surface-container）で階層
- **Shape**: 入力 md(8px) / ボタン lg(12px) / バッジ full
- **Motion**: 控えめ。フォーカスは primary 1px ボーダー
- **Voice**: 静かで簡潔。ツール然としすぎない

## Overview

Aurora Notes は深夜の書斎の机上を思わせる。明かりを落とした部屋で1冊のノートだけが
```

- [ ] **Step 3: 肥大化を防ぐ運用節を追加**

`old_string`:
```markdown
## このreferenceの使い方
```
`new_string`:
```markdown
## 肥大化を防ぐ運用

DESIGN.md は「結論だけ・更新型」で小さく保つ:

- **結論のみ**: 理由・汎用論は書かない（理由は `reference/` に委ねる）。「primary=oklch(...)」は書くが「なぜこの色か」は書かない。standard なら front matter 50〜100 行＋散文 50〜100 行＝**150〜250 行程度**が目安。
- **更新型（追記でなく上書き）**: スキルが書き戻す際は新セクションを足さず、既存値を上書き・統合する。
- **トークン外出し**: 大規模化したら `tokens.css` / `tokens.json` に分離し、DESIGN.md は「Summary ＋散文＋参照」に保つ。
- **スリム化は init-design が担う**: 重複・冗長が溜まったら `/init-design` が重複統合で整理する（情報量は減らさず冗長だけ削る）。

## このreferenceの使い方
```

- [ ] **Step 4: 検証**

Run: `Grep pattern "## Summary|サマリ層|トークン外出し|スリム化は init-design" path "skills/ui-design-grounding/reference/design-md-spec.md" output_mode "content"`
Expected: 4パターンすべてヒット（説明・例・運用節）。

- [ ] **Step 5: コミット**

```bash
git add skills/ui-design-grounding/reference/design-md-spec.md
git commit -m "$(cat <<'EOF'
feat(design-md-spec): サマリ層・トークン外出し・スリム化運用を仕様に追加

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>
EOF
)"
```

---

## Task 6: init-design をサマリ層・スリム化に対応

**Files:**
- Modify: `skills/init-design/SKILL.md`

**Interfaces:**
- Consumes: `design-md-spec.md`（Task 5 で追加した Summary・運用）。
- Produces: 生成・更新時に Summary を出力し、重複統合（スリム化）を担うワークフロー。

- [ ] **Step 1: 手順 Step 4 に Summary 記述を追加**

`old_string`:
```markdown
### Step 4: 本文8セクションの記述

哲学に従い**散文中心**で記述する。汎用形容詞を避け、具体的な参照・質感・比喩を込める。レスポンシブ観点は Layout / Do's and Don'ts に統合する。
```
`new_string`:
```markdown
### Step 4: 本文（Summary ＋ 8セクション）の記述

哲学に従い**散文中心**で記述する。汎用形容詞を避け、具体的な参照・質感・比喩を込める。レスポンシブ観点は Layout / Do's and Don'ts に統合する。

冒頭に **`## Summary`（サマリ層）** を置き、全観点の基準を 10〜20 行で一行要約する（忘却防止）。続けて8セクション（詳細層）を順に書く。書式は `design-md-spec.md` のサマリ層・出力例に従う。
```

- [ ] **Step 2: 動作モードにスリム化を追加**

`old_string`:
```markdown
- **C. 抽出（リバース生成）** — 既存 CSS / トークン / コンポーネントを分析し、値を front matter に、根拠を本文に起こす。
```
`new_string`:
```markdown
- **C. 抽出（リバース生成）** — 既存 CSS / トークン / コンポーネントを分析し、値を front matter に、根拠を本文に起こす。
- **D. スリム化（重複統合）** — 更新の積み重ねで重複・冗長が溜まった DESIGN.md を、既存値を統合して整理する。**情報量は減らさず冗長だけ削る**。大規模化していれば `tokens.css` / `tokens.json` への外出しを提案する。
```

- [ ] **Step 3: 出力品質チェックリストに Summary を追加**

`old_string`:
```markdown
- [ ] 本文が **8セクションを順序通り**に持つ（Overview → … → Do's and Don'ts）
```
`new_string`:
```markdown
- [ ] 冒頭に **`## Summary`（サマリ層）** があり、全観点の基準を 10〜20 行で一覧している
- [ ] 本文が **8セクションを順序通り**に持つ（Overview → … → Do's and Don'ts）
```

- [ ] **Step 4: 検証**

Run: `Grep pattern "## Summary|サマリ層|スリム化（重複統合）" path "skills/init-design/SKILL.md" output_mode "content"`
Expected: 3パターンすべてヒット（手順・モード・チェックリスト）。

- [ ] **Step 5: コミット**

```bash
git add skills/init-design/SKILL.md
git commit -m "$(cat <<'EOF'
feat(init-design): サマリ層の生成とスリム化（重複統合）モードに対応

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>
EOF
)"
```

---

## Task 7: ルート SKILL.md のコマンド一覧・件数を更新

**Files:**
- Modify: `skills/ui-design-grounding/SKILL.md`

**Interfaces:**
- Consumes: `refine-ui`（Task 3）の存在。
- Produces: ナビの単一情報源に refine を反映し、第1層/第2層の関係を示す。

- [ ] **Step 1: コマンドスキル件数を 19→20 に更新**

`old_string`:
```markdown
- 19個の独立コマンドスキル（`ui-help` を除く）が MANDATORY PREPARATION として本スキルのリファレンスを参照する
```
`new_string`:
```markdown
- 20個の独立コマンドスキル（`ui-help` を除く）が MANDATORY PREPARATION として本スキルのリファレンスを参照する
- 入口は2層: **第1層（動詞）** = 考える `design-ui` / 作る `implement-ui` / 直す `refine-ui` / 評価 `audit-ui`・`score-ui` / 基準化 `init-design`。**第2層（観点・実働ユニット）** = 色・タイポ・レイアウト等を実際に直すスキル。第1層から委譲され、軸が明確なら直接も呼べる
```

- [ ] **Step 2: コマンドスキル一覧表に refine-ui 行を追加**

`old_string`:
```markdown
| デザインを実装構造に翻訳 | `/implement-ui` |
```
`new_string`:
```markdown
| デザイン・要件から実装（第1層・作る） | `/implement-ui` |
| 既存UIを観点ベースで診断し直す（第1層・直す） | `/refine-ui` |
```

- [ ] **Step 3: 検証**

Run: `Grep pattern "20個|/refine-ui|第1層（動詞）" path "skills/ui-design-grounding/SKILL.md" output_mode "content"`
Expected: 3パターンすべてヒット。

- [ ] **Step 4: コミット**

```bash
git add skills/ui-design-grounding/SKILL.md
git commit -m "$(cat <<'EOF'
docs(ui-design-grounding): refine-ui 追加・2層構造を反映（19→20）

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>
EOF
)"
```

---

## Task 8: ui-help に refine-ui を追加し第1層/第2層に再構成

**Files:**
- Modify: `skills/ui-help/SKILL.md`

**Interfaces:**
- Consumes: `refine-ui`（Task 3）。
- Produces: ユーザー向けコマンド一覧。第1層5動詞を先頭に置き、refine を「直す」入口として案内。

- [ ] **Step 1: 「設計」「評価」見出しブロックを第1層に再構成**

`old_string`:
```markdown
### 設計

| コマンド | 何をするか | こんなとき |
|---------|-----------|-----------|
| `/design-ui` | 要件からUI構造・画面設計を整理。画面遷移・情報階層・UI状態（初期/ローディング/成功/エラー/空）を設計 | 新しい画面を設計したい |
| `/implement-ui` | デザインをコンポーネント分解（Atomic Design）・責務整理・状態マトリックスに翻訳 | デザインから実装プランへ落としたい |
| `/init-design` | DESIGN.md を生成・更新。design.md 仕様（YAML トークン + 8セクション散文）準拠。既存CSS/トークンから自動抽出対応 | デザインシステムの基盤を定義したい |

### 評価

| コマンド | 何をするか | こんなとき |
|---------|-----------|-----------|
| `/audit-ui` | a11y・パフォーマンス・トークン準拠・レスポンシブ・アンチパターンの5軸で監査（各0-4点、/20点）。問題を対応スキルへ自動マッピング | 技術品質を一括チェックしたい |
| `/score-ui` | ニールセン10原則で採点（/40点）＋ 5種ペルソナ（初回/熟練/a11y/モバイル/ストレス）でレッドフラグテスト＋認知負荷アセスメント。課題を対応スキルへ自動マッピング | UX品質を構造的に評価したい |
```
`new_string`:
```markdown
### 第1層（まずここ — 動詞で選ぶ）

| コマンド | 何をするか | こんなとき |
|---------|-----------|-----------|
| `/design-ui` | **考える** — 要件からUI構造・画面設計を整理。画面遷移・情報階層・UI状態（初期/ローディング/成功/エラー/空）を設計（DESIGN.md は制約として読むが書かない） | 新しい画面を設計したい |
| `/implement-ui` | **作る** — DESIGN.md を基準に観点を横断し、第2層へ委譲しながら新規UIを実装まで担う第1層オーケストレーター | デザイン・要件から実装したい |
| `/refine-ui` | **直す** — 「なんか変・うるさい・読みにくい」等の曖昧な訴えを観点ベースで診断し、該当する第2層へ委譲して修正まで担う | 既存UIを直したいが、どの観点の問題か定まっていない |
| `/audit-ui` | **評価（技術）** — a11y・パフォーマンス・トークン準拠・レスポンシブ・アンチパターンの5軸で監査（各0-4点、/20点）。問題を対応スキルへ自動マッピング | 技術品質を一括チェックしたい |
| `/score-ui` | **評価（UX）** — ニールセン10原則で採点（/40点）＋ 5種ペルソナでレッドフラグテスト＋認知負荷アセスメント。課題を対応スキルへ自動マッピング | UX品質を構造的に評価したい |
| `/init-design` | **基準化** — DESIGN.md を生成・更新。design.md 仕様（YAML トークン + Summary + 8セクション散文）準拠。既存CSS/トークンから自動抽出対応 | デザインシステムの基盤を定義したい |

> 観点が最初から明確なら、第1層を介さず下の第2層スキルを直接使ってよい。
```

- [ ] **Step 2: 「ビジュアル調整」見出しを第2層と明示**

`old_string`:
```markdown
### ビジュアル調整

| コマンド | 何をするか | こんなとき |
|---------|-----------|-----------|
| `/boost-ui` |
```
`new_string`:
```markdown
### 第2層・ビジュアル調整（観点を直す実働ユニット）

| コマンド | 何をするか | こんなとき |
|---------|-----------|-----------|
| `/boost-ui` |
```

- [ ] **Step 3: ワークフロー・迷ったら節を refine 対応に更新**

`old_string`:
```markdown
**新規設計**: `/design-ui` → `/init-design` → `/implement-ui` → `/audit-ui`
**既存UIの改善**: `/init-design` → `/audit-ui` or `/score-ui` → 推奨アクションのN番を実行
**リリース前**: `/polish-ui` → `/score-ui`
```
`new_string`:
```markdown
**新規設計**: `/init-design`（基準）→ `/design-ui`（構造）→ `/implement-ui`（実装）→ `/audit-ui`
**既存UIの改善（観点が曖昧）**: `/refine-ui`（診断→委譲で修正）→ `/audit-ui` or `/score-ui`
**既存UIの改善（観点が明確）**: 該当する第2層スキルを直接実行
**リリース前**: `/polish-ui` → `/score-ui`
```

`old_string`:
```markdown
- **印象を変えたい** → `/boost-ui`（強める）or `/calm-ui`（抑える）
```
`new_string`:
```markdown
- **どこを直すか分からない** → `/refine-ui`（観点ベースで診断して直す）
- **印象を変えたい** → `/boost-ui`（強める）or `/calm-ui`（抑える）
```

- [ ] **Step 4: 検証**

Run: `Grep pattern "/refine-ui" path "skills/ui-help/SKILL.md" output_mode "content"`
Expected: 3件以上ヒット（第1層表・既存UI改善ワークフロー・迷ったら）。

Run: `Grep pattern "第1層（まずここ|第2層・ビジュアル調整" path "skills/ui-help/SKILL.md" output_mode "content"`
Expected: 両方ヒット。

- [ ] **Step 5: コミット**

```bash
git add skills/ui-help/SKILL.md
git commit -m "$(cat <<'EOF'
docs(ui-help): refine-ui を追加し第1層5動詞/第2層に再構成

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>
EOF
)"
```

---

## Task 9: CLAUDE.md 早見表に refine-ui を追加し implement-ui を更新

**Files:**
- Modify: `CLAUDE.md`（リポジトリ root）

**Interfaces:**
- Consumes: `refine-ui`（Task 3）。
- Produces: リポジトリ早見表が新5動詞構造と整合。

- [ ] **Step 1: 設計カテゴリの implement 行を更新**

`old_string`:
```markdown
| 設計 | `/implement-ui` | デザイン→実装構造 |
```
`new_string`:
```markdown
| 設計 | `/implement-ui` | 要件・デザイン→実装（第1層・作る／実装まで担う） |
```

- [ ] **Step 2: 調整カテゴリの先頭に refine 行を追加**

`old_string`:
```markdown
| 調整 | `/boost-ui` | 印象を強める（地味→大胆） |
```
`new_string`:
```markdown
| 調整 | `/refine-ui` | 曖昧な訴えを観点ベースで診断→委譲して直す（第1層・直す） |
| 調整 | `/boost-ui` | 印象を強める（地味→大胆） |
```

- [ ] **Step 3: 検証**

Run: `Grep pattern "/refine-ui|第1層・作る" path "CLAUDE.md" output_mode "content"`
Expected: 両パターンがヒット。

- [ ] **Step 4: コミット**

```bash
git add CLAUDE.md
git commit -m "$(cat <<'EOF'
docs(CLAUDE): 早見表に refine-ui を追加し implement-ui を第1層へ更新

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>
EOF
)"
```

---

## Task 10: 全体整合の検証

**Files:**
- 読み取りのみ（修正が出たら該当タスクへ戻す）

**Interfaces:**
- Consumes: Task 1-9 の成果。
- Produces: refine 参照・件数・ゲート区分の一貫性確認。

- [ ] **Step 1: refine-ui 参照の一貫性**

Run: `Grep pattern "refine-ui" glob "**/*.md" output_mode "files_with_matches"`
Expected: 少なくとも `skills/refine-ui/SKILL.md` / `skills/ui-help/SKILL.md` / `skills/ui-design-grounding/SKILL.md` / `CLAUDE.md` / `docs/skill-architecture.md` / `skills/ui-design-grounding/reference/design-md-gate.md` が含まれる。

- [ ] **Step 2: ゲート区分表との突合**

Run: `Read file "skills/ui-design-grounding/reference/design-md-gate.md"`
確認: 適用区分表の第1層（design/implement/refine）が前段✓後段—、第2層・recolor・polish が後段✓。実ファイル（Task 2/3/4）の手順と矛盾しない（implement/refine/design は後段節を持たず前段のみ）。

- [ ] **Step 3: 件数整合**

Run: `Glob pattern "skills/*/SKILL.md"`
Expected: 22件（コマンドスキル20＋`ui-design-grounding`＋`ui-help`… 実際の内訳を数え、ルート SKILL.md の「20個の独立コマンドスキル（ui-help を除く）」と一致するか確認）。不一致なら Task 7 の件数を修正。

- [ ] **Step 4: 旧フレーミングの残存チェック**

Run: `Grep pattern "コードの最終決定は行わない|意図ベースのナレッジ・ルーター兼・教材" glob "**/*.md" output_mode "content"`
Expected: ヒット 0件。

- [ ] **Step 5: 最終コミット（必要な微修正があれば）**

```bash
git add -A
git commit -m "$(cat <<'EOF'
chore: 5動詞再編の整合性検証と微修正

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>
EOF
)"
```

---

## 自己レビュー結果（プラン作成者による）

**1. Spec coverage（ADR の各決定 → タスク対応）**
- 決定1（観点スキルを残す・根拠=実働）→ 第2層を個別改修しない方針＋ナビで第2層を実働ユニットと明示（Task 7/8）。
- 決定2（5動詞・薄い実働オーケストレーター）→ Task 2/3（implement/refine）。
- 決定5（design-ui 格上げ・init と直交）→ Task 4＋ナビ（Task 7/8/9）。
- 決定3（DESIGN.md 常駐・波及二層）→ Task 1（第2層の自己波及）＋ Task 2/3（第1層の横断波及）。
- 決定4（DESIGN.md 結論のみ・2階建て・更新型・外出し・スリム化）→ Task 5/6。
- ゲート配置（第1層前段のみ・後段は実働層）→ Task 2/3/4 が後段節を持たない＋ Task 10 で突合。

**2. Placeholder scan**: 新規/改訂ファイルは全文を記載。Summary の具体形のみ「プラン提案・レビューで確認」と明示（Task 5 冒頭注記）。それ以外に TBD/TODO 無し。

**3. Type/名称整合**: 第2層スキル名は全タスクで `xxx-ui` 表記に統一。診断/委譲表の観点⇔スキル対応は implement（8観点）と refine（11観点）で重複せず一貫。委譲先名は実在スキルのみ。

**未確認の前提**: Task 10 Step 3 の SKILL.md 総数は実環境で数えて確定する（現状 Glob で 21 ファイル＝コマンド20＋ルート1、`ui-help` は「コマンドスキル」に含むため「ui-help を除く」表現と件数の数え方を Task 7/10 で最終確認）。
