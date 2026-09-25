# 変更承認ゲートと `.design/` 構造再編 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 書き込みを伴う全コマンドスキルに「改善提案 → 承認 →（大: 設計 → 承認）→ 実装計画 → 承認 → 実装」の変更承認ゲートを入れ、`.design/` を `specs/` `plans/` `reports/` の日付付き平置き構造に再編する。

**Architecture:** ゲートの手順は新設リファレンス `change-gate.md` に一元化し、各スキルは MANDATORY PREPARATION で参照したうえで、手順中の1か所に短いゲート手順を置く（`design-md-gate.md` と同じ共通参照方式）。`.design/` の構造・命名・テンプレートは `feature-design.md` を改名した `design-artifacts.md` に集約する。ゲートは入口のスキルが持ち、第1層から第2層への委譲では承認済み計画を渡して二重承認を避ける。

**Tech Stack:** Markdown のみ（ビルド・テスト・リントなし）。検証は `grep` による文字列確認と、代表シナリオの机上トレース。

**Spec:** `docs/superpowers/specs/2026-09-26-change-approval-gate-design.md`

## Global Constraints

- 全スキルコンテンツは日本語で書く。
- MANDATORY PREPARATION の参照行は素のパス（`- \`ui-design-grounding/reference/xxx.md\``）で書き、注記を付けない。
- 参照の追加位置: `change-gate.md` は `design-md-gate.md` の**直後**に置く（`design-md-gate.md` が無いスキルは参照リストの末尾）。
- 新パスの表記は次で統一する（ユーザー指定の形式）:
  - 設計: `.design/specs/yyyy-mm-dd-{name}.md`
  - 実装計画: `.design/plans/yyyy-mm-dd-{name}.md`
  - 評価レポート: `.design/reports/yyyy-mm-dd-{name}.md`（`{name}` = `<skill>-<対象>`）
  - スクリーンショット: `.design/reports/yyyy-mm-dd-{name}/NN.png`
- 規模判定: 次のいずれかで **大**（1. 共通コンポーネント変更 or 複数画面に波及 / 2. 2観点以上 / 3. 構造変更）。**DESIGN.md のトークン変更のみは小**。迷えば大。
- 承認回数: 小 = 提案・計画の2回（計画は会話内のみ、ファイルに残さない）/ 大 = 提案・設計・計画の3回（設計は specs、計画は plans に保存）。
- 概念や価値は「AI エージェント」一般の言葉で書き、特定製品固有の書き方を避ける。
- バージョンは **1.6.0**（現行マニフェストは 1.5.1。spec の「1.7.0」は Task 11 で 1.6.0 に訂正する）。
- 第2層の標準ゲート手順の文言（以下 **G1**。各タスクで `{N}` を見出し番号に置き換えて使う）:

  ```markdown
  ### {N} 変更承認ゲート

  `change-gate.md` に従い、ここまでの診断を改善提案（変更点・対象・規模判定）としてユーザーに提示し、承認を得てから次の手順に進む。規模が小なら実装計画を会話内で示して承認を得る。大なら設計（`.design/specs/`）と実装計画（`.design/plans/`）をそれぞれ保存して承認を得る。以降の手順は承認された項目だけを実施する。

  第1層（`refine-ui` / `implement-ui`）から承認済みの実装計画を受け取っている場合は、このゲートを省略し、計画のうち自分に割り当てられたタスクの範囲だけを実施する。範囲外の変更が必要になったら手を止め、差分を提案する。
  ```

## Review Focus

1. **第1層からの委譲時の二重承認**: `refine-ui` が承認済み計画を渡したのに、第2層が再びゲートで止まると、ユーザーは同じ内容を2回承認させられる。第2層の G1 に「受け取っている場合は省略」が必ず入っていることを Task 8・9 の grep で確認する。
2. **ゲート省略の過剰適用**: 「急いで」「お任せ」を「確認不要」と読み替えると、ゲートが実質的に消える。`change-gate.md` で省略条件を「確認不要・承認不要・そのまま直して等の明示」に限定し、`interview.md` の「お任せ」エスケープとは別物だと明記する（Task 1）。
3. **旧パスの取り残し**: `reports/YYYY-MM-DD/` や `HHmmss`、`<feature-slug>/FEATURE_DESIGN.md` が1か所でも残ると、スキル間で保存先が食い違う。Task 12 の grep で互換記述以外の残存が 0 件であることを確認する。
4. **既存プロジェクトの旧 FEATURE_DESIGN.md**: 旧構造のプロジェクトで `implement-ui` が設計を見落とし、設計なしと誤判定して `design-ui` へ回してしまう。Task 6 でフォールバック読み込みを入れ、Task 12 のシナリオ 5 で確認する。
5. **polish-ui / guard-ui の境界の薄さ**: チェック即修正型のため、ゲートを見出しで挟むだけでは「チェック項目ごとに直す」読み方が残る。手順冒頭の文言そのものを「一覧化 → 承認 → 修正」に書き換え（Task 9）、`即座に` の残存を grep で確認する。

---

### Task 0: 作業ブランチと spec のコミット

**Files:**
- 既存: `docs/superpowers/specs/2026-09-26-change-approval-gate-design.md`（未コミット）
- 既存: `docs/superpowers/plans/2026-09-26-change-approval-gate.md`（本ファイル、未コミット）

- [ ] **Step 1: ブランチを切る**

```bash
git switch -c feat/change-approval-gate
```

- [ ] **Step 2: spec と plan をコミット**

```bash
git add docs/superpowers/specs/2026-09-26-change-approval-gate-design.md docs/superpowers/plans/2026-09-26-change-approval-gate.md
git commit -m "docs: 変更承認ゲートと .design 構造再編の設計・実装計画を追加"
```

---

### Task 1: `change-gate.md` を新設

**Files:**
- Create: `skills/ui-design-grounding/reference/change-gate.md`

**Interfaces:**
- Produces: リファレンス名 `change-gate.md`。節名「フロー」「改善提案の形式」「規模判定」「承認の扱い」「ゲートの省略」「層ごとの責務と受け渡し」「スキル別の適用区分」。後続タスクはこの名前と節名で参照する。

- [ ] **Step 1: 事前確認（まだ存在しないこと）**

Run: `ls skills/ui-design-grounding/reference/change-gate.md`
Expected: `No such file or directory`

- [ ] **Step 2: ファイルを作成**

````markdown
# 変更承認ゲート

プロジェクトのコード・UI ファイル・DESIGN.md に書き込むコマンドスキルが、**改善点を提案してユーザーの承認を得てから変更に入る**ための共通プロトコル。

対象のスキル（下記の**適用区分表**）はこのファイルを MANDATORY PREPARATION で参照し、手順の中の決められた位置でゲートを通す。スキルは実装・修正まで完遂するが、その前に必ず確認を挟む。承認待ちは作業を助言で終わらせることではない。

`design-md-gate.md` との位置関係:

```text
前段ゲート（DESIGN.md の読み込み）→ 診断 → 変更承認ゲート → 実装 → 検証 → 後段ゲート（DESIGN.md との乖離検出）
```

---

## フロー

```text
① 改善提案（変更点・対象・規模判定）→ 承認
   ├ 小: ② 実装計画（会話内）→ 承認 → 実装 → 検証
   └ 大: ② 設計（.design/specs/）→ 承認 → ③ 実装計画（.design/plans/）→ 承認 → 実装 → 検証
```

- 小規模の実装計画は会話内で提示し、ファイルには残さない。
- 大規模の設計と実装計画は `design-artifacts.md` のテンプレートと命名に従って保存し、会話では要約と保存先パスを示して承認を求める。同じ作業の設計と実装計画は同じ `{name}` にする。
- 新規実装（`implement-ui`）では、① の「改善提案」を「実装方針の提案」と読み替える。診断の対象が既存 UI ではなく要件になるためである。

## 改善提案の形式

```markdown
## 改善提案

| # | 変更点 | 対象 | 観点 / 委譲先 |
|---|---|---|---|
| 1 | ... | `path/to/file` / 画面名 | レイアウト / 自前 or `/xxx-ui` |

- 診断の根拠: [DESIGN.md・リファレンス・観察結果との差]
- 規模判定: 小 / 大（該当した条件: ...）
- 承認後の次の工程: 小 → 実装計画 / 大 → 設計
```

- 1行 = 1つの改善にする。ユーザーは一部だけを承認できる（例: 「1 と 3 だけ」）。
- 規模判定はユーザーが上書きできる。

## 規模判定

次のいずれかに当てはまれば **大**、当てはまらなければ **小**。

1. 共通コンポーネントに手を入れる、または複数画面に波及する
2. 2つ以上の観点にまたがる（第2層への委譲が2件以上になる）
3. レイアウトの再構成・コンポーネント分割など、構造を変える

- **DESIGN.md のトークン変更のみ**の場合は、画面への波及があっても **小** とする。トークン変更と他の変更が混在する場合は、トークン以外の変更だけで判定する。
- 判定に迷う場合は大として扱い、提案の中でその旨を示す。

## 実装計画（小規模・会話内）の形式

```markdown
## 実装計画

| # | 対象ファイル | 変更内容 | 観点 / 委譲先 |
|---|---|---|---|
| 1 | ... | ... | ... |

- 実行順: ...
- 検証方法: [観察手順・確認観点]
```

大規模の実装計画は同じ項目を `design-artifacts.md` の実装計画テンプレートでファイルに書く。

## 承認の扱い

- **承認とみなすのは、ユーザーの明示的な肯定だけ**。無関係な返答や沈黙を承認とみなさない。
- 修正の要望があれば、その工程の成果物（提案・設計・計画）を直して再提示する。
- 実装中に承認済みの範囲を超える変更が必要になったら、手を止めて差分を提案し、承認を得てから進める。

## ゲートの省略

ユーザーが依頼の中で「確認不要」「承認不要」「そのまま直して」などと**承認待ちが不要であることを明示した場合に限り**、承認待ちを省略できる。

- 「急いで」「お任せ」はゲートの省略ではない（`interview.md` の「お任せ」は質問を推奨回答で埋めるためのもので、承認とは別）。
- 省略は、その依頼の範囲だけに有効とする。
- 省略しても、改善提案と実装計画は短く提示してから実装する。
- 大規模の場合は、省略しても設計と実装計画のファイルは保存する。

## 層ごとの責務と受け渡し

**ゲートは入口のスキルが持つ。**

| 呼ばれ方 | ゲートを実行するスキル | 委譲先の動作 |
|---|---|---|
| 第1層（`refine-ui` / `implement-ui`）から開始 | 第1層 | 承認済みの計画を受け取り、ゲートを省略して実行する |
| 第2層を直接呼ぶ | 第2層 | — |
| 評価系の推奨アクションから第2層へ進む | 第2層（直接呼びと同じ） | — |

- 第1層は委譲時に「承認済み実装計画のうち、委譲先に割り当てるタスク」を明示して渡す。
- 委譲先は、承認済み計画を受け取ったかどうかでゲートの実行と省略を切り替える。
- 委譲先が計画の範囲外の変更を必要と判断したら、手を止めて第1層経由でユーザーに提案する。
- 設計がすでに承認されている場合（`design-ui` が保存し承認を得た `.design/specs/` の設計がある場合）は、② の設計を飛ばして実装計画から始める。

## スキル別の適用区分

| 区分 | スキル | ゲート |
|---|---|---|
| 第1層・作る | `implement-ui` | ✓（設計があれば実装計画から） |
| 第1層・直す | `refine-ui` | ✓ |
| 第1層・考える | `design-ui` | 設計の承認のみ（インタビューの合意に加え、保存した設計の承認を得る） |
| 第1層・評価 | `audit-ui` `score-ui` `legibility-ui` | —（実装を変更しない） |
| 第1層・基準化 | `init-design` `scan-ui` | —（既存のインタビュー・合意を維持） |
| 補助 | `preview-ui` `ui-help` | —（見本帳の生成・一覧表示のみ） |
| 第2層・実働 | `arrange-ui` `typeset-ui` `recolor-ui` `animate-ui` `clarify-ui` `adapt-ui` `guard-ui` `optimize-ui` `boost-ui` `calm-ui` `slim-ui` `extract-ui` `polish-ui` | ✓（第1層から承認済み計画を受け取った場合は省略） |
````

- [ ] **Step 3: 検証**

Run: `grep -n "^## " skills/ui-design-grounding/reference/change-gate.md`
Expected: 10 行。節見出し 8 つ（フロー / 改善提案の形式 / 規模判定 / 実装計画（小規模・会話内）の形式 / 承認の扱い / ゲートの省略 / 層ごとの責務と受け渡し / スキル別の適用区分）と、コードブロック内の `## 改善提案` `## 実装計画` の 2 つ。

- [ ] **Step 4: コミット**

```bash
git add skills/ui-design-grounding/reference/change-gate.md
git commit -m "feat: 変更承認ゲートの共通プロトコル change-gate.md を追加"
```

---

### Task 2: `feature-design.md` → `design-artifacts.md`（`.design/` 構造とテンプレート）

**Files:**
- Rename: `skills/ui-design-grounding/reference/feature-design.md` → `skills/ui-design-grounding/reference/design-artifacts.md`
- Modify: `skills/ui-design-grounding/reference/interview.md`（`FEATURE_DESIGN` への言及 :10, :21, :30, :39, :41, :57）

**Interfaces:**
- Consumes: Task 1 の `change-gate.md`（フローの参照先）
- Produces: リファレンス名 `design-artifacts.md`。節名「`.design/` の構造」「命名」「機能設計テンプレート」「改善設計テンプレート」「実装計画テンプレート」「旧構造との互換」「昇格導線（設計 → DESIGN.md）」。

- [ ] **Step 1: 改名**

```bash
git mv skills/ui-design-grounding/reference/feature-design.md skills/ui-design-grounding/reference/design-artifacts.md
```

- [ ] **Step 2: 冒頭〜「## テンプレート」直前までを置き換え**

`design-artifacts.md` の1行目から `## テンプレート` の直前までを、次の内容に置き換える。

````markdown
# UI 作業の成果物（`.design/`）

UI 作業で生成するファイル（設計・実装計画・評価レポート・見本帳）の置き場所、命名、テンプレートを定義する。承認の手順は `change-gate.md`、評価レポートの本文規約は `ui-report.md` を参照する。

## DESIGN.md と設計（specs）の関係

| | DESIGN.md | 設計（specs） |
|---|---|---|
| 単位 | プロジェクト全体 | 機能・画面・改善1件 |
| 寿命 | 恒久（使われながら育つ） | 設計〜実装 |
| 内容 | 視覚基準（トークン・散文の指針） | UX 判断・変更方針 |
| 置き場所 | プロジェクトルート | `.design/specs/` |

**視覚基準（色・タイポ等のトークン水準）は設計に直接書かず、DESIGN.md を参照する**（二重管理の回避）。DESIGN.md が無い・最小構成の場合のみ設計に直接書き、恒久化する際に `/init-design` へ移す。

## `.design/` の構造

UI 作業の生成物は、DESIGN.md（ルート常駐）を除きすべて対象プロジェクトの `.design/` に集約する。

```text
project-root/
├── DESIGN.md                                ← 視覚的憲法はルート常駐（エージェントの自動参照が価値の核）
└── .design/
    ├── specs/yyyy-mm-dd-{name}.md           ← 設計（機能設計 / 改善設計）
    ├── plans/yyyy-mm-dd-{name}.md           ← 大規模の実装計画
    ├── reports/yyyy-mm-dd-{name}.md         ← 評価レポート（ui-report.md 参照）
    ├── reports/yyyy-mm-dd-{name}/NN.png     ← そのレポートのスクリーンショット
    └── preview.html                         ← DESIGN.md の見本帳（preview-ui が生成）
```

## 命名

- `yyyy-mm-dd` は実行環境のローカル日付。
- `{name}` は小文字ハイフン区切りの短い名前。
  - specs / plans: 機能・画面・改善内容から導く（例: `2026-09-26-settings-page.md`）。同じ作業の設計と実装計画は同じ `{name}` にする。
  - reports: `<skill>-<対象>`（例: `2026-09-26-audit-ui-settings-page.md`）。
- 同名のファイルが既にあれば末尾に `-2`, `-3` を付け、上書きしない。
- 小規模の実装計画はファイルに残さない（`change-gate.md`）。

## Git 管理

`specs/` と `plans/` はコミットを推奨する（`implement-ui` や第2層が読み込む受け渡しファイル）。`reports/` と `preview.html` をコミットするかは各プロジェクトの判断でよい。

## 旧構造との互換

旧版は機能設計を `.design/<feature-slug>/FEATURE_DESIGN.md`、評価レポートを `.design/reports/YYYY-MM-DD/HHmmss-<skill>.md` に保存していた。

- 設計を読むスキルは、`.design/specs/` に該当する設計が無ければ旧パスの `FEATURE_DESIGN.md` も読む。
- 既存ファイルを自動で移動・改名しない。旧構造を見つけたら、新構造への移行を提案するにとどめる。
````

- [ ] **Step 3: 機能設計テンプレートの見出しと種別行**

- `## テンプレート` を `## 機能設計テンプレート（design-ui）` に置き換える。
- テンプレート内の `# 機能設計: [機能・画面名]` の直後に空行と次の1行を追加する:

```markdown
- 種別: 機能設計
```

- [ ] **Step 4: 改善設計テンプレートと実装計画テンプレートを追加**

機能設計テンプレートのコードブロック終端（````` ```` `````）の直後、`## 昇格導線` の直前に次を挿入する。

`````markdown
## 改善設計テンプレート（refine-ui・大規模判定時の第2層）

````markdown
# 改善設計: [対象と改善内容]

- 種別: 改善設計
- 改善提案: [承認された提案の要約と項目番号]

## 背景と診断結果

何が問題か。根拠（DESIGN.md・リファレンス・観察結果との差）を添える。

## 変更方針

観点ごとにどう直すか。採る手法と、DESIGN.md のどのトークン・規約に従うか。

## 影響範囲

変更する画面・コンポーネント・共通部品。波及する観点。

## 検討した代替案

採らなかった案と理由。

## スコープ外

この改善で扱わないこと（実装中のスコープクリープ防止）。

## 検証方法

観察手順・確認観点（Playwright で見る状態・幅など）。

## 要確認事項

確定していない判断（無ければ「なし」）。
````

## 実装計画テンプレート（大規模）

````markdown
# 実装計画: [対象]

- 設計: `.design/specs/yyyy-mm-dd-{name}.md`

## タスク

| # | 対象ファイル | 変更内容 | 観点 | 実施（自前 / 委譲先） |
|---|---|---|---|---|
| 1 | ... | ... | ... | 自前 or `/xxx-ui` |

## 実行順と依存関係

- [タスク番号の順序と、前提となるタスク]

## 検証方法

- [観察手順・確認観点]
````
`````

- [ ] **Step 5: 昇格導線の文言を更新**

`## 昇格導線（機能設計 → DESIGN.md）` を `## 昇格導線（設計 → DESIGN.md）` に、その直下の「機能設計の完成時に」を「設計の完成時に」に、「**機能設計側から DESIGN.md を自動では書き換えない**」を「**設計側から DESIGN.md を自動では書き換えない**」に置き換える。

- [ ] **Step 6: interview.md の言及を更新**

`skills/ui-design-grounding/reference/interview.md` の `FEATURE_DESIGN.md` をすべて「機能設計」に置き換える。パスを示している箇所は `.design/specs/yyyy-mm-dd-{name}.md` にする。`feature-design.md` への参照があれば `design-artifacts.md` にする。

Run: `grep -n "FEATURE_DESIGN\|feature-design\|feature-slug" skills/ui-design-grounding/reference/interview.md skills/ui-design-grounding/reference/design-artifacts.md`
Expected: `design-artifacts.md` の「旧構造との互換」節の2行（`<feature-slug>/FEATURE_DESIGN.md` と「旧パスの `FEATURE_DESIGN.md`」）だけが出る。

- [ ] **Step 7: コミット**

```bash
git add skills/ui-design-grounding/reference/
git commit -m "feat: feature-design.md を design-artifacts.md に改名し specs/plans/reports 構造とテンプレートを定義"
```

---

### Task 3: `ui-report.md` と `playwright.md` の保存先を新構造へ

**Files:**
- Modify: `skills/ui-design-grounding/reference/ui-report.md`（:5-16 保存先、:34 メタ情報、:40-49 スクリーンショット一覧、フォローアップ手順1）
- Modify: `skills/ui-design-grounding/reference/playwright.md:20`

**Interfaces:**
- Consumes: Task 2 の命名規則（reports の `{name}` = `<skill>-<対象>`）

- [ ] **Step 1: 事前確認**

Run: `grep -c "HHmmss\|YYYY-MM-DD/" skills/ui-design-grounding/reference/ui-report.md skills/ui-design-grounding/reference/playwright.md`
Expected: どちらも 1 以上

- [ ] **Step 2: 「## 保存先」節を置き換え**

`## 保存先` から `## 共通メタ情報` の直前までを次に置き換える。

```markdown
## 保存先

評価レポートは、評価対象プロジェクトの `.design/reports/` 配下に保存する（`.design/` の全体構造と命名は `design-artifacts.md` を参照）。

| 種別 | パス |
|---|---|
| レポート | `.design/reports/yyyy-mm-dd-<skill>-<対象>.md` |
| スクリーンショット | `.design/reports/yyyy-mm-dd-<skill>-<対象>/NN.png` |

- `yyyy-mm-dd` は実行環境のローカル日付を使う。
- `<skill>` は `audit-ui` / `score-ui` / `legibility-ui` のいずれか。`<対象>` は画面・機能から導いた小文字ハイフン区切りの短い名前。
- 同名のレポートが既にあれば末尾に `-2`, `-3` を付けて上書きを避ける。スクリーンショットのフォルダ名もレポートと同じにする。
- スクリーンショットの `NN` は `01` から始め、取得順に連番を付ける。
- このプラグインリポジトリにはサンプルレポートを追加しない。`.design/reports/` は評価対象プロジェクト側の成果物である。

```

- [ ] **Step 3: 共通メタ情報の保存先行**

`| レポート保存先 | \`.design/reports/YYYY-MM-DD/HHmmss-<skill>.md\` |` を次に置き換える。

```markdown
| レポート保存先 | `.design/reports/yyyy-mm-dd-<skill>-<対象>.md` |
```

- [ ] **Step 4: スクリーンショット一覧の例**

`| 1 | <画面・状態・幅など> | [screenshots/HHmmss-<skill>-01.png](screenshots/HHmmss-<skill>-01.png) |` を次に置き換える。

```markdown
| 1 | <画面・状態・幅など> | [yyyy-mm-dd-<skill>-<対象>/01.png](yyyy-mm-dd-<skill>-<対象>/01.png) |
```

- [ ] **Step 5: フォローアップ手順1の文言**

`1. P0/P1 の指摘のうち、この実行内（または直後の修正フェーズ）で修正しないものを列挙する。` を次に置き換える。

```markdown
1. P0/P1 の指摘のうち、ユーザーがこの会話で修正に進むことを承認していないものを列挙する（評価系スキルは実装を変更しない。修正は推奨アクションの各スキルが `change-gate.md` に従って提案・承認を経て行う）。
```

- [ ] **Step 6: playwright.md:20**

`playwright.md` の20行目にある保存先の記述（`.design/reports/YYYY-MM-DD/screenshots/` と `HHmmss-<skill>-NN.png`）を、`.design/reports/yyyy-mm-dd-<skill>-<対象>/` と `NN.png` に置き換える。文の他の部分は変えない。

- [ ] **Step 7: 検証**

Run: `grep -n "HHmmss\|YYYY-MM-DD\|screenshots/" skills/ui-design-grounding/reference/ui-report.md skills/ui-design-grounding/reference/playwright.md`
Expected: 出力なし

- [ ] **Step 8: コミット**

```bash
git add skills/ui-design-grounding/reference/ui-report.md skills/ui-design-grounding/reference/playwright.md
git commit -m "feat: 評価レポートの保存先を reports/yyyy-mm-dd-{name}.md に変更"
```

---

### Task 4: 評価系3スキルの保存先

**Files:**
- Modify: `skills/audit-ui/SKILL.md`（:45, :60, :66, :97）
- Modify: `skills/score-ui/SKILL.md`（:59, :74, :80, :110, :125）
- Modify: `skills/legibility-ui/SKILL.md`（:70, :86, :92, :123）

**Interfaces:**
- Consumes: Task 3 の保存先規約

- [ ] **Step 1: 事前確認**

Run: `grep -n "HHmmss\|YYYY-MM-DD\|screenshots/" skills/audit-ui/SKILL.md skills/score-ui/SKILL.md skills/legibility-ui/SKILL.md`
Expected: 上記の行番号が出る

- [ ] **Step 2: 置換**

各ファイルで次の対応で置き換える（`<skill>` はそのファイルのスキル名）。

| 旧 | 新 |
|---|---|
| `.design/reports/YYYY-MM-DD/HHmmss-<skill>.md` | `.design/reports/yyyy-mm-dd-<skill>-<対象>.md` |
| `.design/reports/YYYY-MM-DD/screenshots/` | `.design/reports/yyyy-mm-dd-<skill>-<対象>/` |
| `screenshots/HHmmss-<skill>-NN.png`（リンク・表の例） | `yyyy-mm-dd-<skill>-<対象>/NN.png` |
| `screenshots/HHmmss-<skill>-01.png` 等の具体例 | `yyyy-mm-dd-<skill>-<対象>/01.png` 等 |

周辺の文（「`ui-report.md` に従い、評価対象プロジェクトの … に詳細レポートを保存する」など）はそのまま残す。

- [ ] **Step 3: 検証**

Run: `grep -n "HHmmss\|YYYY-MM-DD\|screenshots/" skills/audit-ui/SKILL.md skills/score-ui/SKILL.md skills/legibility-ui/SKILL.md`
Expected: 出力なし

Run: `grep -n "yyyy-mm-dd-" skills/audit-ui/SKILL.md skills/score-ui/SKILL.md skills/legibility-ui/SKILL.md`
Expected: Step 1 で出た各行が、新パスの行として同じ位置に出る（audit-ui :45, :60, :66, :97 / score-ui :59, :74, :80, :110, :125 / legibility-ui :70, :86, :92, :123）。

- [ ] **Step 4: コミット**

```bash
git add skills/audit-ui/SKILL.md skills/score-ui/SKILL.md skills/legibility-ui/SKILL.md
git commit -m "feat: 評価系スキルのレポート保存先を新構造へ"
```

---

### Task 5: `design-ui` の出力先と設計承認

**Files:**
- Modify: `skills/design-ui/SKILL.md`（:3, :15, :53, :64-67, :73, :78, :82, :101）

**Interfaces:**
- Consumes: `design-artifacts.md`（機能設計テンプレート）、`change-gate.md`
- Produces: `.design/specs/yyyy-mm-dd-{name}.md`（種別: 機能設計）。`implement-ui` がこれを「設計承認済み」の入力として読む。

- [ ] **Step 1: description（:3）**

`（.design/<feature-slug>/FEATURE_DESIGN.md）として保存する。` を `（.design/specs/yyyy-mm-dd-{name}.md）として保存し、ユーザーの承認を得る。` に置き換える。

- [ ] **Step 2: MANDATORY PREPARATION（:15, :25）**

- `- \`ui-design-grounding/reference/feature-design.md\`` → `- \`ui-design-grounding/reference/design-artifacts.md\``
- 最終行 `- \`ui-design-grounding/reference/design-md-gate.md\`` の直後に `- \`ui-design-grounding/reference/change-gate.md\`` を追加する。

- [ ] **Step 3: :53 の「FEATURE_DESIGN.md の生成へ進む」**

`ユーザーが共通理解を確認してから FEATURE_DESIGN.md の生成へ進む。` → `ユーザーが共通理解を確認してから機能設計の生成へ進む。`

- [ ] **Step 4: 手順8（:64-67）を置き換え**

```markdown
### 8. 機能設計の保存・承認と昇格検出

- 設計結果を `design-artifacts.md` の機能設計テンプレートに従い `.design/specs/yyyy-mm-dd-{name}.md` に保存する（`{name}` は機能・画面から導いた小文字ハイフン区切り）。
- 保存した設計の要約と保存先を示し、`change-gate.md` に従って**設計の承認**を得る。修正の要望があれば設計を直して再提示する。承認されるまで手順9へ進まない。
- DESIGN.md 級の恒久的決定（トーンの明確化・新トークン候補・画面横断の新規約）が生まれていれば「DESIGN.md へ昇格すべき決定」として列挙し、`/init-design` を提案する（本スキルからは書き換えない）。
```

- [ ] **Step 5: :73, :78, :82, :101**

- :73 `FEATURE_DESIGN.md と DESIGN.md を設計入力として渡す。` → `承認済みの機能設計（\`.design/specs/\`）と DESIGN.md を設計入力として渡す。`
- :74 `→ \`/implement-ui\` へ。` → `→ \`/implement-ui\` へ（設計承認済みとして実装計画から始まる）。`
- :78 `FEATURE_DESIGN.md（\`feature-design.md\` のテンプレート準拠）を保存したうえで` → `機能設計（\`design-artifacts.md\` のテンプレート準拠）を保存したうえで`
- :82 `- 機能設計の保存先: \`.design/<feature-slug>/FEATURE_DESIGN.md\`` → `- 機能設計の保存先: \`.design/specs/yyyy-mm-dd-{name}.md\`（承認: 済 / 修正待ち）`
- :101 `実装に受け渡せる形（FEATURE_DESIGN.md）で残す` → `実装に受け渡せる形（\`.design/specs/\` の機能設計）で残す`

- [ ] **Step 6: 検証**

Run: `grep -n "FEATURE_DESIGN\|feature-design\|feature-slug" skills/design-ui/SKILL.md`
Expected: 出力なし

Run: `grep -n "change-gate\|設計の承認" skills/design-ui/SKILL.md`
Expected: MANDATORY PREPARATION の1行と手順8の行が出る

- [ ] **Step 7: コミット**

```bash
git add skills/design-ui/SKILL.md
git commit -m "feat: design-ui の出力を .design/specs へ移し設計承認を追加"
```

---

### Task 6: `implement-ui` に変更承認ゲート

**Files:**
- Modify: `skills/implement-ui/SKILL.md`（:3, :10, :17, :34, 手順1〜2の間, :40, :86, :97）

**Interfaces:**
- Consumes: `change-gate.md`、`design-artifacts.md`、Task 5 の `.design/specs/`（種別: 機能設計）
- Produces: 第2層への委譲時に「承認済み実装計画の該当タスク」を渡す（第2層の G1 が受け取る）

- [ ] **Step 1: description（:3）**

`機能設計（.design/<feature-slug>/FEATURE_DESIGN.md、あれば）を受け、` → `機能設計（.design/specs/、あれば）を受け、実装方針と実装計画をユーザーに提示して承認を得たうえで、`

`必要に応じて第2層スキルへ委譲して実装まで担う。` はそのまま残す。

- [ ] **Step 2: 冒頭説明（:10）**

`判断軸（観点）を効かせながら**実装まで担いきる**。` → `判断軸（観点）を効かせ、**承認を得てから実装まで担いきる**。`

- [ ] **Step 3: MANDATORY PREPARATION（:16-17）**

```markdown
- `ui-design-grounding/reference/design-md-gate.md`
- `ui-design-grounding/reference/change-gate.md`
- `ui-design-grounding/reference/design-artifacts.md`
```

（`feature-design.md` の行を `design-artifacts.md` に置き換え、`change-gate.md` を `design-md-gate.md` の直後に挿入する）

- [ ] **Step 4: :34 の設計読み込み**

```markdown
- `.design/specs/` に該当する設計（種別: 機能設計）があるかを確認し、あれば読み込んで**機能単位の判断基準**にする。無ければ旧パス `.design/<feature-slug>/FEATURE_DESIGN.md` も確認する（`design-artifacts.md`「旧構造との互換」）。DESIGN.md = 恒久の視覚基準、機能設計 = この機能の UX 判断。矛盾する場合は DESIGN.md を優先し、乖離として報告する。
```

- [ ] **Step 5: 手順1と手順2の間に手順1.5を挿入（:37 の直後、`### 2.` の直前）**

```markdown
### 1.5 変更承認ゲート

`change-gate.md` に従い、実装に入る前にユーザーの承認を得る。

- **設計がある場合**（`design-ui` が保存し承認済みの機能設計）: 設計の承認は済んでいるため、実装計画から始める。規模が大なら実装計画を `.design/plans/`（設計と同じ `{name}`）に保存し、小なら会話内で示して承認を得る。
- **設計が無い場合**: 実装方針（作るもの・対象ファイル・観点と委譲先・規模判定）を提案して承認を得る。
  - 規模が **大** → `/design-ui` へ委譲して機能設計を作り、その承認を得てから本スキルへ復帰し、実装計画に進む。
  - 規模が **小** → 実装計画を会話内で示して承認を得る。
- 手順2以降は承認された計画の範囲だけを実装する。範囲外の変更が必要になったら手を止め、差分を提案する。
```

- [ ] **Step 6: 手順2の委譲文（:40）**

`（委譲先は実働ユニットなので、委譲すれば実装が完遂する）` → `（委譲するときは承認済み実装計画のうち委譲先に割り当てるタスクを明示して渡す。委譲先はゲートを省略して実装を完遂する）`

- [ ] **Step 7: 出力フォーマット（:86）**

`## 機能設計との対応（FEATURE_DESIGN.md がある場合）` → `## 機能設計との対応（.design/specs/ の機能設計がある場合）`

- [ ] **Step 8: 注意（:97）**

`- **計画で止めず、実装まで担う**（担うのは…）` の太字部分を `**承認を得たら計画で止めず、実装まで担う**` に置き換える（括弧内はそのまま）。直後に次の行を追加する。

```markdown
- 承認を得る前にファイルを変更しない（`change-gate.md`）。
```

- [ ] **Step 9: 検証**

Run: `grep -n "FEATURE_DESIGN\|feature-slug\|feature-design" skills/implement-ui/SKILL.md`
Expected: 手順1の旧パスのフォールバック1行（`.design/<feature-slug>/FEATURE_DESIGN.md`）だけが出る

Run: `grep -n "change-gate" skills/implement-ui/SKILL.md`
Expected: MANDATORY PREPARATION・手順1.5・注意の3行以上

- [ ] **Step 10: コミット**

```bash
git add skills/implement-ui/SKILL.md
git commit -m "feat: implement-ui に変更承認ゲートを導入し設計入力を .design/specs へ"
```

---

### Task 7: `refine-ui` に変更承認ゲート

**Files:**
- Modify: `skills/refine-ui/SKILL.md`（:3, :10, :18-20, :46-48, :54-57, 出力フォーマット, :79）

**Interfaces:**
- Consumes: `change-gate.md`、`design-artifacts.md`（改善設計テンプレート）
- Produces: 第2層への委譲時に「承認済み実装計画の該当タスク」を渡す

- [ ] **Step 1: description（:3）**

`該当する第2層スキルへ委譲して修正まで担う。` → `改善提案をユーザーに提示して承認を得たうえで、該当する第2層スキルへ委譲して修正まで担う。`

- [ ] **Step 2: 冒頭説明（:10）**

`該当する第2層スキルへ委譲して**修正まで担いきる**。` → `改善提案の承認を得てから、該当する第2層スキルへ委譲して**修正まで担いきる**。`

- [ ] **Step 3: MANDATORY PREPARATION（:18 の直後に2行追加）**

```markdown
- `ui-design-grounding/reference/design-md-gate.md`
- `ui-design-grounding/reference/change-gate.md`
- `ui-design-grounding/reference/design-artifacts.md`
- `ui-design-grounding/reference/anti-patterns.md`
- `ui-design-grounding/reference/usability.md`
```

- [ ] **Step 4: 手順1と手順2の間に手順1.5を挿入（:45 の直後、`### 2. 直す` の直前）**

```markdown
### 1.5 変更承認ゲート

`change-gate.md` に従い、診断結果を改善提案（逸脱観点ごとの変更点・対象・自前 or 委譲先・規模判定）としてユーザーに提示し、承認を得る。

- 規模が **小** → 実装計画を会話内で示して承認を得る。
- 規模が **大** → 改善設計（`design-artifacts.md` の改善設計テンプレート）を `.design/specs/yyyy-mm-dd-{name}.md` に保存して承認を得る。続けて実装計画を `.design/plans/yyyy-mm-dd-{name}.md` に保存して承認を得る。
- 承認された項目だけを手順2で直す。
```

- [ ] **Step 5: 手順2の本文（:48）**

`（委譲先は実働ユニットなので、委譲すれば修正が完遂する）` → `（委譲するときは承認済み実装計画のうち委譲先に割り当てるタスクを明示して渡す。委譲先はゲートを省略して修正を完遂する）`

- [ ] **Step 6: 手順3の横断波及（:52）の末尾に追記**

`波及があれば該当する第2層を追加で呼び、収束させる。` の後に次を追記する。

```markdown
追加の修正が承認済み計画の範囲外なら、手を止めて差分を提案し、承認を得てから呼ぶ。
```

- [ ] **Step 7: 出力フォーマットに改善提案を追加**

出力フォーマットのコードブロック内、`## 診断（観点ベース）` の表の後、`## 修正内容` の前に次を挿入する。

```markdown
## 改善提案と承認
- 規模判定: 小 / 大（該当条件）
- 承認された項目: #1, #3 …
- 設計・計画: 会話内 / `.design/specs/…`・`.design/plans/…`
```

- [ ] **Step 8: 注意（:79）**

`- **助言で止めず、自前修正または委譲で修正まで担う。**` → `- **承認を得たら助言で止めず、自前修正または委譲で修正まで担う。** 承認を得る前にファイルを変更しない（\`change-gate.md\`）。`

- [ ] **Step 9: 検証**

Run: `grep -n "change-gate" skills/refine-ui/SKILL.md`
Expected: MANDATORY PREPARATION・手順1.5・注意の3行以上

Run: `grep -n "止めず" skills/refine-ui/SKILL.md`
Expected: 「承認を得たら助言で止めず」の1行だけ

- [ ] **Step 10: コミット**

```bash
git add skills/refine-ui/SKILL.md
git commit -m "feat: refine-ui に変更承認ゲートを導入"
```

---

### Task 8: 第2層（標準型8件）に G1 を挿入

**Files:**
- Modify: `skills/arrange-ui/SKILL.md`（参照 :18 の後、G1 を `### 3. 改善の実施`（:46）の直前に `### 2.5`）
- Modify: `skills/typeset-ui/SKILL.md`（参照 :17 の後、G1 を `### 3. 改善の実施`（:43）の直前に `### 2.5`）
- Modify: `skills/clarify-ui/SKILL.md`（参照 :17 の後、G1 を `### 3. 改善の実施`（:60）の直前に `### 2.5`）
- Modify: `skills/boost-ui/SKILL.md`（参照 :19 の後、G1 を `### 3. 具体的な変更を実施`（:42）の直前に `### 2.5`）
- Modify: `skills/calm-ui/SKILL.md`（参照 :19 の後、G1 を `### 3. 具体的な変更を実施`（:43）の直前に `### 2.5`）
- Modify: `skills/slim-ui/SKILL.md`（参照 :18 の後、G1 を `### 3. 削減の実施`（:59）の直前に `### 2.5`）
- Modify: `skills/animate-ui/SKILL.md`（参照 :17 の後、G1 を `### 3. カテゴリ別に実装`（:43）の直前に `### 2.5`。手順2「アニメーション戦略の策定」の内容を提案に含める）
- Modify: `skills/adapt-ui/SKILL.md`（参照 :19 の後、G1 を `### 3. レイアウト適応`（:46）の直前に `### 2.5`。手順2「ブレイクポイント設計」の内容を提案に含める）
- Modify: `skills/optimize-ui/SKILL.md`（参照 :18 の後、G1 を `### 2. レンダリング性能`（:30）の直前に `### 1.5`。手順1「現状の確認」の結果から改善項目を挙げて提案する）

注: 上の行番号は編集前のもの。参照行を先に追加すると1行ずれるので、**G1 の挿入を先に行い、参照の追加を後に行う**。

**Interfaces:**
- Consumes: `change-gate.md`、Global Constraints の G1

- [ ] **Step 1: 事前確認**

Run: `grep -L "change-gate" skills/{arrange,typeset,clarify,boost,calm,slim,animate,adapt,optimize}-ui/SKILL.md`
Expected: 9ファイルすべてが出る

- [ ] **Step 2: 各ファイルに G1 を挿入**

上表の位置に、Global Constraints の G1 を `{N}` を置き換えて挿入する。見出しの前後に空行を1行ずつ置く。animate-ui と adapt-ui は G1 の1文目の「ここまでの診断を」を「ここまでの診断と手順2の設計を」にする。optimize-ui は「ここまでの診断を」を「手順1の現状確認から挙げた改善項目を」にする。

- [ ] **Step 3: 各ファイルの MANDATORY PREPARATION に参照を追加**

各ファイルの `- \`ui-design-grounding/reference/design-md-gate.md\`` の直後に次を追加する。

```markdown
- `ui-design-grounding/reference/change-gate.md`
```

- [ ] **Step 4: adapt-ui の :29 の文言**

`（検出 → 修正 → 各幅で再観察）` → `（検出 → 変更承認ゲート → 修正 → 各幅で再観察）`

- [ ] **Step 5: description の調整**

9ファイルの description（:3）の1文目「〜を修正・改善する。」「〜を追加する。」などの動詞の直前に「提案と承認を経て」を入れる（例: arrange-ui `UIのレイアウト・余白・視覚階層を提案と承認を経て修正・改善する。`）。トリガ語（「〜ときに使用する」の部分）は変えない。

- [ ] **Step 6: 検証**

Run: `grep -c "change-gate" skills/{arrange,typeset,clarify,boost,calm,slim,animate,adapt,optimize}-ui/SKILL.md`
Expected: 各ファイル 2（参照行と G1 本文）

Run: `grep -c "このゲートを省略し" skills/{arrange,typeset,clarify,boost,calm,slim,animate,adapt,optimize}-ui/SKILL.md`
Expected: 各ファイル 1

Run: `grep -n "^### " skills/arrange-ui/SKILL.md`
Expected: `### 2. 問題の特定` → `### 2.5 変更承認ゲート` → `### 3. 改善の実施` の順

- [ ] **Step 7: コミット**

```bash
git add skills/{arrange,typeset,clarify,boost,calm,slim,animate,adapt,optimize}-ui/SKILL.md
git commit -m "feat: 第2層の実働スキルに変更承認ゲートを挿入"
```

---

### Task 9: 第2層（例外型4件: guard / polish / recolor / extract）

**Files:**
- Modify: `skills/guard-ui/SKILL.md`（:3, :20, :30, `### 0.5` と `### 1.` の間）
- Modify: `skills/polish-ui/SKILL.md`（:3, :25, :30, :31, :35, 出力フォーマット）
- Modify: `skills/recolor-ui/SKILL.md`（:3, :23, `### 5.` と `### 6. 反映` の間）
- Modify: `skills/extract-ui/SKILL.md`（:3, :19, `### 5.` の後）

**Interfaces:**
- Consumes: `change-gate.md`、G1

- [ ] **Step 1: guard-ui**

1. :20 の後に `- \`ui-design-grounding/reference/change-gate.md\`` を追加。
2. :30 の `**崩れを再現して検出 → 修正 → 再観察で確認**する。` → `**崩れを再現して検出 → 変更承認ゲート → 修正 → 再観察で確認**する。`
3. `### 1. テキストオーバーフロー処理` の直前に次を挿入。

```markdown
### 0.8 検出と変更承認ゲート

手順1〜5の各カテゴリで、まず**崩れの検出だけ**を行い、問題を一覧化する（ここでは修正しない）。一覧を改善提案として `change-gate.md` に従い提示し、承認を得る。規模が小なら実装計画を会話内で示して承認を得る。大なら設計（`.design/specs/`）と実装計画（`.design/plans/`）を保存して承認を得る。手順1〜5の修正は承認された項目だけに行う。

第1層（`refine-ui` / `implement-ui`）から承認済みの実装計画を受け取っている場合は、このゲートを省略し、計画のうち自分に割り当てられたタスクの範囲だけを実施する。範囲外の変更が必要になったら手を止め、差分を提案する。
```

4. description（:3）の `体系的にチェックし修正する。` → `体系的にチェックし、提案と承認を経て修正する。`

- [ ] **Step 2: polish-ui**

1. :25 の後に `- \`ui-design-grounding/reference/change-gate.md\`` を追加。
2. :30 `- review-ui（評価のみ）との違い: **polish-ui は問題を発見次第、実際に修正する**` → `- 評価系スキルとの違い: **polish-ui は承認を得たうえで、問題を実際に修正する**`
3. :31 の `**検出 → 修正 → 再観察で反映を確認**する` → `**検出 → 変更承認ゲート → 修正 → 再観察で反映を確認**する`
4. :35 `以下のチェックリストを順に確認し、問題があれば即座に修正する。` を次に置き換える。

```markdown
以下のチェックリストを順に確認して問題を一覧化する（ここでは修正しない）。すべての項目を確認し終えたら `change-gate.md` に従い、一覧を改善提案として提示して承認を得る。規模が小なら実装計画を会話内で示して承認を得る。大なら設計（`.design/specs/`）と実装計画（`.design/plans/`）を保存して承認を得る。承認された項目だけを修正する。

第1層（`refine-ui` / `implement-ui`）から承認済みの実装計画を受け取っている場合は、このゲートを省略し、計画のうち自分に割り当てられたタスクの範囲だけを実施する。範囲外の変更が必要になったら手を止め、差分を提案する。
```

5. 出力フォーマットの `### 修正済み` の直前に次を追加する。

```markdown
### 承認された項目
- [#番号と内容]
```

6. description（:3）の `問題を実際に修正する。` → `問題を一覧化して提案し、承認を得て実際に修正する。`

- [ ] **Step 3: recolor-ui**

1. :23 の後に `- \`ui-design-grounding/reference/change-gate.md\`` を追加。
2. `### 6. 反映` の直前に G1 を `{N}` = `5.5` で挿入し、1文目の「ここまでの診断を」を「手順1〜5の再配色結果（変更前後の colors とコントラスト検証）を」に置き換える。
3. description（:3）の `OKLCH で破綻なく再配色（リカラー）する。` → `OKLCH で破綻なく再配色（リカラー）し、変更前後を提示して承認を得てから反映する。`

- [ ] **Step 4: extract-ui**

1. :19 の後に `- \`ui-design-grounding/reference/change-gate.md\`` を追加。
2. `### 5. 段階的な移行計画` 節の末尾（`## 出力フォーマット` の直前）に次を挿入する。

```markdown
### 5.5 変更承認ゲート

抽出結果と移行計画は提案として提示する。コードのコンポーネント化・トークン置換・DESIGN.md への反映に着手する場合は、`change-gate.md` に従って改善提案として承認を得てから行う（トークン抽出で DESIGN.md のトークンだけを変える場合は小規模）。

第1層（`refine-ui` / `implement-ui`）から承認済みの実装計画を受け取っている場合は、このゲートを省略し、計画のうち自分に割り当てられたタスクの範囲だけを実施する。
```

3. description（:3）は変えない（抽出・提案が主目的で即時修正を促していないため）。

- [ ] **Step 5: 検証**

Run: `grep -n "即座\|発見次第" skills/*/SKILL.md`
Expected: 出力なし

Run: `grep -c "change-gate" skills/{guard,polish,recolor,extract}-ui/SKILL.md`
Expected: 各ファイル 2 以上

- [ ] **Step 6: コミット**

```bash
git add skills/{guard,polish,recolor,extract}-ui/SKILL.md
git commit -m "feat: guard/polish/recolor/extract に変更承認ゲートを導入"
```

---

### Task 10: ナレッジベースと `design-md-gate.md`

**Files:**
- Modify: `skills/ui-design-grounding/SKILL.md`（出力ポリシー :85-93、参照ナビゲーション :97-147 の `feature-design.md` 行 :136 と手順系リファレンスの並び）
- Modify: `skills/ui-design-grounding/reference/design-md-gate.md`（:7 の後）

- [ ] **Step 1: 出力ポリシーに1行追加**

`- 判断や最終選択は、必ず人間に委ねる` の直後に次を追加する。

```markdown
- コード・UI ファイル・DESIGN.md を変更するときは、改善提案とユーザーの承認を経てから行う（`change-gate.md`）
```

- [ ] **Step 2: 参照ナビゲーション**

- :136 の `feature-design.md` の行を次に置き換える。

```markdown
- `design-artifacts.md` — `.design/` の構造（specs / plans / reports / preview.html）・命名・機能設計と改善設計と実装計画のテンプレート・旧構造との互換・DESIGN.md への昇格導線
```

- `design-md-gate.md` の行の直後に次を追加する。

```markdown
- `change-gate.md` — 変更承認ゲート: 改善提案 → 承認 →（大: 設計 → 承認）→ 実装計画 → 承認 → 実装、規模判定、ゲートの省略条件、第1層→第2層の受け渡し
```

- :146 の `ui-report.md` の行にある保存先の記述を `（\`.design/reports/yyyy-mm-dd-<skill>-<対象>.md\`）` にする。

- [ ] **Step 3: design-md-gate.md に位置関係を追記**

:7（「ゲートの持ち方は層で変わる。…」の段落）の直後に次の段落を追加する。

```markdown
コードや UI ファイルを変更するスキルでは、前段ゲートと後段ゲートの間に **変更承認ゲート**（`change-gate.md`）が入る: 前段ゲート → 診断 → 変更承認ゲート → 実装 → 検証 → 後段ゲート。
```

- [ ] **Step 4: 検証**

Run: `grep -n "feature-design\|FEATURE_DESIGN\|HHmmss" skills/ui-design-grounding/SKILL.md skills/ui-design-grounding/reference/design-md-gate.md`
Expected: 出力なし

Run: `grep -n "change-gate" skills/ui-design-grounding/SKILL.md skills/ui-design-grounding/reference/design-md-gate.md`
Expected: SKILL.md に2行、design-md-gate.md に1行

- [ ] **Step 5: コミット**

```bash
git add skills/ui-design-grounding/SKILL.md skills/ui-design-grounding/reference/design-md-gate.md
git commit -m "feat: ナレッジベースに変更承認ゲートと design-artifacts を登録"
```

---

### Task 11: README / AGENTS / ui-help / バージョン / CHANGELOG / spec の訂正

**Files:**
- Modify: `README.md`（`.design/` 節 :171-191、リファレンス一覧 :320-333）
- Modify: `AGENTS.md`（:13, :36-40 付近のリファレンス件数, :68, :111, :134, :138, :140, :162, リファレンス一覧表）
- Modify: `skills/ui-help/SKILL.md:27`
- Modify: `.claude-plugin/plugin.json:4`
- Modify: `CHANGELOG.md`（:5 の前に新節）
- Modify: `docs/superpowers/specs/2026-09-26-change-approval-gate-design.md`（§5.1 のバージョン）

- [ ] **Step 1: README の `.design/` 節**

`:175-187` の構造ブロックを `design-artifacts.md` の「`.design/` の構造」と同じツリーに置き換える。説明文中の `FEATURE_DESIGN.md` は「機能設計（`specs/`）」に、`reports/YYYY-MM-DD/` は `reports/yyyy-mm-dd-{name}.md` に置き換える。節の末尾に次の段落を追加する。

```markdown
コードや UI を変更するスキルは、いきなり修正せず、まず改善提案を示して承認を得ます。規模が小さければ実装計画 → 実装、大きければ設計（`specs/`）→ 実装計画（`plans/`）→ 実装の順に、工程ごとに承認を得て進みます（`change-gate.md`）。
```

- [ ] **Step 2: README のリファレンス一覧（:320-333）**

`feature-design.md` の行を `| \`design-artifacts.md\` | \`.design/\` の構造・命名、設計（機能設計 / 改善設計）と実装計画のテンプレート |` に置き換え、`design-md-gate.md` の行の直後に `| \`change-gate.md\` | 変更承認ゲート（提案 → 承認 → 設計 → 計画 → 実装）、規模判定 |` を追加する。

- [ ] **Step 3: AGENTS.md**

- :13 `- 現行バージョン: \`1.5.1\`` → `- 現行バージョン: \`1.6.0\``
- リファレンス件数の「22件」「22 件」を「23件」「23 件」に（`grep -n "22" AGENTS.md` で該当箇所を確認）。
- :68 の `.design/<feature-slug>/FEATURE_DESIGN.md` → `.design/specs/yyyy-mm-dd-{name}.md`。
- :111 `.design/reports/YYYY-MM-DD/` → `.design/reports/yyyy-mm-dd-{name}.md`。
- :134 のリファレンス表 `feature-design.md` 行 → `| \`design-artifacts.md\` | \`.design/\` 構造・命名、機能設計 / 改善設計 / 実装計画テンプレート、昇格導線 |`。`design-md-gate.md` 行の直後に `| \`change-gate.md\` | 変更承認ゲート、規模判定、省略条件、層ごとの受け渡し |` を追加。
- :138 `ui-report.md` 行の保存先表記を `.design/reports/` のままにし、説明に「日付付きファイル名」を加える。
- :140 `## 1.6.0 で追加された最新運用` → `## 1.5.1 で追加された運用`（CHANGELOG と合わせる）。その直前に次の節を追加する。

```markdown
## 1.6.0 で追加された最新運用

- 書き込みを伴うスキルは、いきなり修正せず **変更承認ゲート**（`change-gate.md`）を通します: 改善提案 → 承認 → 小: 実装計画（会話内）→ 承認 → 実装 / 大: 設計（`.design/specs/`）→ 承認 → 実装計画（`.design/plans/`）→ 承認 → 実装。
- 規模は「共通コンポーネント・複数画面への波及／2観点以上／構造変更」のいずれかで大。DESIGN.md のトークン変更のみは小。
- ゲートは入口のスキルが持ちます。第1層（`refine-ui` / `implement-ui`）は承認済み計画を第2層へ渡し、第2層はゲートを省略します。
- `.design/` を `specs/` `plans/` `reports/` の日付付き平置き（`yyyy-mm-dd-{name}.md`）に再編しました。`feature-design.md` は `design-artifacts.md` に改名しています。
```

- 「編集時の注意事項」の `**インタビュー・機能設計**` 項目（:162）の `feature-design.md` を `design-artifacts.md` に置き換え、その次に次の項目を追加する。

```markdown
- **変更承認ゲート**: コード・UI ファイル・DESIGN.md を変更するスキルを編集するときは、`change-gate.md` の適用区分と整合させ、承認前にファイルを変更する手順や「即座に修正」のような文言を入れない。
```

- [ ] **Step 4: ui-help（:27）**

`.design/<feature` から始まるパスを `.design/specs/` に置き換える。

- [ ] **Step 5: plugin.json（:4）**

`"version": "1.5.1",` → `"version": "1.6.0",`

- [ ] **Step 6: CHANGELOG（:5 の前）**

```markdown
## [1.6.0] - 2026-09-26

- 書き込みを伴う全スキルに **変更承認ゲート**（新リファレンス `change-gate.md`）を導入。いきなり修正せず、改善提案 → 承認 →（大規模: 設計 → 承認）→ 実装計画 → 承認 → 実装の順に進める。規模は「共通コンポーネント・複数画面への波及／2観点以上／構造変更」で大、DESIGN.md のトークン変更のみは小
- 第1層（`refine-ui` / `implement-ui`）から第2層への委譲では承認済み計画を渡し、第2層はゲートを省略（二重承認の回避）。`polish-ui` の「即座に修正」、`refine-ui` / `implement-ui` の「止めず」を承認前提の文言に変更
- **破壊的変更**: `.design/` の構造を再編。機能設計は `.design/<feature-slug>/FEATURE_DESIGN.md` → `.design/specs/yyyy-mm-dd-{name}.md`、評価レポートは `.design/reports/YYYY-MM-DD/HHmmss-<skill>.md` → `.design/reports/yyyy-mm-dd-<skill>-<対象>.md`（スクリーンショットはレポートと同名フォルダ）。大規模の実装計画は `.design/plans/` に保存。`implement-ui` は旧パスの FEATURE_DESIGN.md も読む（自動移行はしない）
- `feature-design.md` を `design-artifacts.md` に改名し、改善設計・実装計画のテンプレートを追加

```

- [ ] **Step 7: spec の訂正**

spec §5.1 の `バージョンを 1.7.0 に上げる` → `バージョンを 1.6.0 に上げる（現行マニフェストは 1.5.1）`。

- [ ] **Step 8: 検証**

Run: `grep -rn "feature-design\|FEATURE_DESIGN\|feature-slug\|HHmmss\|reports/YYYY-MM-DD" README.md AGENTS.md skills/ui-help/SKILL.md`
Expected: 出力なし

Run: `grep -n "version" .claude-plugin/plugin.json`
Expected: `"version": "1.6.0",`

- [ ] **Step 9: コミット**

```bash
git add README.md AGENTS.md skills/ui-help/SKILL.md .claude-plugin/plugin.json CHANGELOG.md docs/superpowers/specs/2026-09-26-change-approval-gate-design.md
git commit -m "docs: 変更承認ゲートと .design 再編を README/AGENTS/CHANGELOG に反映し 1.6.0 へ"
```

---

### Task 12: 全体検証（grep と机上トレース）

**Files:** なし（確認のみ。問題があれば該当タスクのファイルを直してコミット）

- [ ] **Step 1: 旧パスの残存**

Run: `grep -rn "feature-design\.md\|FEATURE_DESIGN\|<feature-slug>\|HHmmss\|reports/YYYY-MM-DD" --include=*.md skills README.md AGENTS.md`
Expected: `design-artifacts.md`「旧構造との互換」節の行と、`implement-ui` 手順1のフォールバック行だけ

- [ ] **Step 2: 即時修正の文言の残存**

Run: `grep -rn "即座に修正\|発見次第\|計画で止めず、実装\|助言で止めず、自前" skills`
Expected: `implement-ui` の「承認を得たら計画で止めず」と `refine-ui` の「承認を得たら助言で止めず」だけ

- [ ] **Step 3: ゲートの網羅**

Run: `for s in arrange typeset recolor animate clarify adapt guard optimize boost calm slim extract polish refine implement design; do printf "%s-ui: " $s; grep -c "change-gate" skills/$s-ui/SKILL.md; done`
Expected: すべて 2 以上（`design-ui` は 2）

Run: `grep -L "このゲートを省略し" skills/{arrange,typeset,recolor,animate,clarify,adapt,guard,optimize,boost,calm,slim,extract,polish}-ui/SKILL.md`
Expected: 出力なし

- [ ] **Step 4: 参照リストの順序**

Run: `grep -n -A1 "reference/design-md-gate.md\`" skills/*/SKILL.md | grep -v "^--"`
Expected: `change-gate.md` を参照するスキルでは、`design-md-gate.md` の次の行が `change-gate.md`

- [ ] **Step 5: 机上トレース**

次の5シナリオで、該当スキルの手順を上から読み、承認の回数と成果物の置き場所を確認する。結果を会話で報告する。

| # | シナリオ | 期待 |
|---|---|---|
| 1 | `/typeset-ui` を直接呼び、見出しのサイズだけ直す（小） | 提案 → 承認 → 会話内の実装計画 → 承認 → 修正。ファイルは作らない |
| 2 | `/refine-ui`「ごちゃつく」→ レイアウトと簡素化の2観点（大） | 提案 → 承認 → `specs/` 改善設計 → 承認 → `plans/` 実装計画 → 承認 → `arrange-ui` と `slim-ui` に委譲（委譲先はゲートを省略） |
| 3 | `/design-ui` → `/implement-ui` | design-ui: インタビュー合意 → `specs/` 機能設計 → 承認。implement-ui: 設計承認済みで実装計画から（大なら `plans/`）→ 承認 → 実装 |
| 4 | `/polish-ui` を「確認不要でそのまま直して」付きで呼ぶ | 提案と計画を短く示し、承認を待たずに修正 |
| 5 | 旧構造（`.design/settings-page/FEATURE_DESIGN.md`）のプロジェクトで `/implement-ui` | 旧パスの機能設計を読み、設計承認済みとして実装計画から。移行を提案 |

- [ ] **Step 6: 完了報告**

grep の結果とシナリオの確認結果をまとめて報告する。問題を直した場合はそのコミットも示す。
