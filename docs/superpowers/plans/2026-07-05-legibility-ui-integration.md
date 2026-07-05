# legibility-ui 統合 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 別プロジェクトで作成済みの `legibility-ui` スキル（初見の分かりやすさ監査）を、プロジェクト固有記述を汎用化したうえで `ui-design-grounding` プラグインの第3の評価系コマンドスキルとして正式統合する。

**Architecture:** `skills/legibility-ui/reference/perspectives.md` を `skills/ui-design-grounding/reference/legibility.md` へ移設してナレッジベースに統合し、`skills/legibility-ui/SKILL.md` を既存20スキルと同じ MANDATORY PREPARATION 構造・DESIGN.md ゲート構造に合わせて全面書き換えする。あわせて `plugin.json` / `ui-design-grounding/SKILL.md` / `ui-help/SKILL.md` / `CLAUDE.md` の一覧・件数表記を更新する。

**Tech Stack:** Markdown（プレーンテキスト）のみ。ビルド・テスト・リント不要。検証は Read/Grep によるファイル内容の直接確認で行う。

## Global Constraints

- 全スキルコンテンツは日本語で記述する（プロジェクト既存の慣例）。
- 各コマンドスキルの SKILL.md は YAML front matter（`name` / `description` / `user-invocable: true` / `argument-hint`）+ Markdown 本文の単一ファイル構成。
- コマンドスキルは `ui-design-grounding` を MANDATORY PREPARATION として呼び出し、リファレンスを参照する構造に統一する。
- legibility-ui は評価専用スキルのため DESIGN.md ゲートは前段のみ（後段ゲートは持たない）。
- プロジェクト固有記述（`packages/web/src/router.tsx` の `NAV_ITEMS`、`start-app` スキル名固定、`.screenshots/legibility-ui/` 固定パス）は残さない。
- Playwright 操作は視覚判定（レンズ①④⑤）で `browser_take_screenshot`、機能判定（レンズ②③）で `browser_snapshot` を使い分ける。

---

### Task 1: リファレンス移設 — `reference/legibility.md` の新規作成

**Files:**
- Create: `skills/ui-design-grounding/reference/legibility.md`
- Delete: `skills/legibility-ui/reference/perspectives.md`

**Interfaces:**
- Produces: `ui-design-grounding/reference/legibility.md` — Task 3 の SKILL.md 本文中の MANDATORY PREPARATION および Phase 1/2/4 の参照先として使われるファイルパス。

- [ ] **Step 1: 新規ファイルを作成する**

`skills/legibility-ui/reference/perspectives.md`（141行）の内容をそのまま `skills/ui-design-grounding/reference/legibility.md` へ書き写す。プロジェクト固有記述は含まれていないため内容の変更は不要。以下の内容で作成する。

```markdown
# legibility-ui 判断基準

## 5つのレンズ

### ① 見た目と機能の不一致（Affordance Mismatch）

**問い**: このコントロールの見た目（形・サイズ・配置・視覚的な重み）から、実際の機能・重要度・関連性を正しく推測できるか？

**チェック方法**: 要素を見て「これは何だと思うか」「他の要素とどういう関係だと思うか」を先に書き、実際にクリック/操作/ホバーした結果と突き合わせる。ズレがあれば指摘にする。

**典型例**:
- フィルタ用のドロップダウン/チップが、通常のラベルやテキストと見分けがつかない
- 検索ボックスの中に、実質はフィルタ条件（期日・優先度等）の入力欄が混在し、「検索」なのか「絞り込み」なのか不明瞭
- トグル/チェックボックスがボタンに見える、またはその逆
- 分割線やペインの端など、操作可能な領域の一部だけにホバー/フォーカスの視覚反応があり、見た目上の境界（全体）と実際の反応領域が一致しない（例: ペインリサイズの仕切りの上端だけ反応がない）
- 意味的に無関係な要素同士が、境界線や配色の差がないために視覚的に連結・同グループに見える（例: サイドバーの主要アイコン群とその下のサブメニューが色差なく繋がって見える）
- 階層・重要度が異なる要素（例: パンくずの中の各セグメント、ページ見出しとその子要素、現在地とその親カテゴリ等、どの組み合わせでもよい）が同じ文字サイズ・太さ・色で表示され、どちらがより上位／現在地かの手がかりが弱い
- 操作した結果の反映先（同一画面内への挿入／モーダル／別画面遷移）が、ボタンの見た目やラベルから予測しにくい（例: 「テンプレート化」ボタンを押すとフォームが画面内にインライン挿入されるが、独立した設定操作に見えるためモーダルの方が期待に合う）
- 同じ画面内に同時に見えている同種コンポーネント（入力欄・ドロップダウン・ボタン等）どうしでサイズ・高さ・余白が不揃いで、意図的な差（主従関係等）なのか単なる作り分けの揺れなのか判別できない（例: タイトル入力欄と優先度ドロップダウンの高さが異なる、同じ画面内に見えている2つのドロップダウンの高さが異なる）。**この観点は対象画面が1画面のときも Phase 1 で必ずチェックする**（⑤画面間一貫性とは異なり、比較のために複数画面を開く必要はない）。

**判断が割れやすいケース**: 対象ユーザーがドメインに習熟している場合（社内ツールの常連ユーザー等）は、多少の見た目の曖昧さは許容されうる。ただし本スキルは「初見」を基準にするため、習熟後に慣れで解決する問題であっても指摘は残す（重篤度で「初見以降は解消するか」を加味する）。

### ② 目的の不透明さ（Purpose Opacity）

**問い**: この機能が「なぜここにあるのか」「使う場面」を、説明なしにイメージできるか？

**チェック方法**: 機能名・アイコン・配置だけを見て、使用シーンを1文で説明できるか自問する。説明できない、または複数の解釈が浮かぶ場合は指摘にする。存在自体の必要性が疑わしい場合（使用頻度・価値が想像できない）も、ここで指摘する。

**典型例**:
- 「テンプレート化」「フィールド」等、名詞だけで動作をイメージできない機能
- 「条件を保存」のように、保存しないと何が困るのかが分からない機能
- ヘルプやツールチップがなく、初見で試すこと自体をためらわせる機能
- 前提条件が満たされる前から、その前提を説明するガイダンス文が表示されている（例: まだ何も保存していないフォームに「保存するとサブタスクを追加できます」という説明が最初から出ている。今は関係のない情報が常時表示され、何のための文か分からない）

**判断が割れやすいケース**: 高度な機能（パワーユーザー向けのショートカット等）は初見で分からなくても許容されることがある。ただし主要ナビゲーションや目立つ位置に置かれている場合は、初見での不透明さがそのまま離脱リスクになるため指摘を残す。

### ③ スコープ不整合（Scope Mismatch）

**問い**: このコントロールが実際に影響する範囲（画面ローカル／機能全体／アプリ全体）と、設置されている場所は一致しているか？

**チェック方法**: コントロールを実際に操作し、変化がどこまで及ぶか確認する。「この画面を離れても効果が残るか」「他の画面からも同じ実体が見えるか」を基準に、画面ローカルな見た目なのにグローバルな実体を操作していないか判定する。

**典型例**:
- 特定の一覧画面にだけ置かれた「編集」ボタンが、実は他画面からも参照される共有エンティティ（プロジェクト名等）を書き換える
- 一時的なフィルタに見える操作が、実は永続設定を変更する

**判断が割れやすいケース**: 「その画面でよく使うから、あえてローカルに置いた」という意図的な複製は正当な場合がある（ショートカット）。その場合は、変更が他画面にも反映されることが分かる視覚的手がかり（例: グローバル設定であることの明示）があれば指摘にしない。手がかりがなければ指摘にする。

### ④ 重複・冗長表示（Redundancy）

**問い**: この情報／要素は、同じ画面（または直前の階層）の別の場所で、すでに表現されていないか？表現されているなら、両方を維持する理由は初見から分かるか？

**チェック方法**: 画面内の見出し・ラベル・識別子（プロジェクト名・エンティティ名等）を列挙し、同じ情報が複数箇所に出ていないか確認する。また、似た名前・似た役割を持つ複数の項目（フィールド、設定、ボタン）がある場合、その使い分けを初見で説明できるか自問する。説明できなければ指摘にする。

**典型例**:
- 画面内の複数の領域（パンくず、ページ見出し、サイドパネルの見出し等）に、同じプロジェクト名・画面名・エンティティ名が繰り返し表示されている（表示箇所の数は問わない。2箇所でも3箇所でも同じ指摘）
- 一覧画面の見出しに専用の編集ボタンがあるが、実質パンくずのメニューと同じ操作を提供している
- 似た名前・似た配置の2つの分類フィールド（例:「カテゴリ」と「セクション」）があり、何を基準に使い分けるのかが画面から読み取れない

**判断が割れやすいケース**: 見た目が重複していても、役割が異なる場合（例: 片方は「現在地のナビゲーション」、もう片方は「今操作している対象の識別」）は許容されうる。ただし役割の違いが視覚的に区別されていない（太さ・配置・文言などの手がかりがない）場合は指摘を残す。

### ⑤ 画面間一貫性（Cross-Screen Consistency）

**問い**: 同じ種類の操作・同じ役割のコンポーネントが、画面や機能をまたいで同じパターン・同じ見た目で実装されているか？

**このレンズが対象とする範囲**: 1つのスクリーンショット／1画面を見ただけでは判定できず、**異なる画面・異なる機能領域を実際に開いて比較しないと分からない**差異のみを扱う。同一画面内に同時に見えているコンポーネント同士の不揃いは①（見た目と機能の不一致）で扱うため、ここには含めない。

**チェック方法**: 対象画面群から「概念的に同じことをしている箇所」を洗い出し（例: Task の新規作成 と Note の新規作成は同じ「エンティティ作成」という概念、両画面の「追加」ボタンは同じ「主要CTA」という役割）、実装パターンとビジュアル（別画面か・インラインか・モーダルか、サイズ・余白・高さ）を並べて比較する。

**典型例**:
- あるエンティティは作成/編集が専用画面、別のエンティティは一覧に埋め込み
- 共通ヘッダーが画面によって意味の異なる情報（アプリ名 / 現在地 / パンくず）を出し分けていて、何を表すヘッダーか一定しない
- 同じ役割の基本コンポーネント（主要CTAボタン等）が、別々の画面間でサイズや高さが揃っていない（例:「ノートを追加」ボタン（Notes画面）と「タスクを追加」ボタン（Tasks画面）のサイズが異なる）

**判断が割れやすいケース**: エンティティの性質上、意図的にパターンを変えるべき場合がある（例: ごく短い属性のインライン編集 vs 長文本体を持つエンティティの専用画面）。その場合はその理由が画面から読み取れるか、または既存の設計判断（`DESIGN.md` 等）に根拠があるかを確認し、根拠があれば指摘にしない。

## 対象外（他スキルに委ねる観点）

初見の「理解」を歪める問題ではなく、理解した上での使いやすさ・堅牢性に関わる以下の観点は本スキルの対象外とする。指摘として気づいた場合は、legibility-ui のレポートには含めず、該当スキルの利用を案内する。

| 観点 | 例 | 委譲先 |
|---|---|---|
| 操作のしやすさ（エルゴノミクス） | ボタンは意味が分かるが、配置上押しにくい（例: 音声入力ボタンが左側で押しにくい） | `/arrange-ui`, `/polish-ui` |
| ピクセル単位の視覚整合 | 意味の誤読を招かない程度の微妙な余白・高さのズレ | `/polish-ui`, `/audit-ui` |
| データ量増加への堅牢性 | 現在は少数だが、件数が数十〜数百に増えたときの表示上限・並び順・スクロール・非表示設定の要否 | `/guard-ui`, `/design-ui` |

## 重篤度

`score-ui` の P0-P3 の語彙を流用しつつ、legibility-ui では「初見での誤解・離脱に直結するか」を基準に再定義する。

| 重篤度 | 定義 |
|---|---|
| P0 | 主要導線のコントロールを誤解し、目的のタスクを完了できない、または意図しない操作（他機能への誤爆）を招く |
| P1 | 主要機能の目的・スコープを誤解するが、試行錯誤すれば復帰できる |
| P2 | 補助的な機能・画面で誤解が起きるが、主要タスクへの影響は小さい |
| P3 | 気づけば些細だが、初見の第一印象・洗練度に影響する |

## 出力フォーマット

```markdown
## legibility-ui レビュー結果

対象: <画面名一覧> / 実施レンズ: ①②③④(⑤)

### ① 見た目と機能の不一致
| 画面 | 指摘 | 重篤度 | 該当コード |
|---|---|---|---|

### ② 目的の不透明さ
| 画面 | 指摘 | 重篤度 | 該当コード |
|---|---|---|---|

### ③ スコープ不整合
| 画面 | 指摘 | 重篤度 | 該当コード |
|---|---|---|---|

### ④ 重複・冗長表示
| 画面 | 指摘 | 重篤度 | 該当コード |
|---|---|---|---|

### ⑤ 画面間一貫性（対象2画面以上のみ）
| 概念 | 画面A | 画面B | 指摘 | 重篤度 |
|---|---|---|---|---|

## 対象外として見送った観点（参考）
- <観点> — <一言> — 委譲先 `/xxx-ui`

## 推奨アクション（優先度順）
1. `/xxx-ui` — 対応件数（代表例）
```

## 推奨アクションのマッピング

| レンズ | 主なマッピング先 |
|---|---|
| ①見た目と機能の不一致 | `/clarify-ui`（用語・視覚的手がかりの改善）、`/arrange-ui`（視覚的グルーピング・階層の修正） |
| ②目的の不透明さ | `/clarify-ui`（説明追加・表示タイミング調整）、`/design-ui`（機能自体の要否再設計） |
| ③スコープ不整合 | `/design-ui`（情報設計の見直し）、`/arrange-ui`（配置変更） |
| ④重複・冗長表示 | `/slim-ui`（重複要素の削除）、`/clarify-ui`（使い分けの明確化） |
| ⑤画面間一貫性 | `/extract-ui`（パターン統一・コンポーネント化）、`/design-ui` |
```

- [ ] **Step 2: 元ファイルを削除する**

```bash
rm "skills/legibility-ui/reference/perspectives.md"
```

- [ ] **Step 3: 移設を検証する**

```bash
test -f "skills/ui-design-grounding/reference/legibility.md" && echo "NEW_EXISTS"
test -f "skills/legibility-ui/reference/perspectives.md" || echo "OLD_REMOVED"
grep -c "^### ①\|^### ②\|^### ③\|^### ④\|^### ⑤" "skills/ui-design-grounding/reference/legibility.md"
```

Expected: `NEW_EXISTS` と `OLD_REMOVED` が両方出力され、レンズ見出しのカウントが `5` であること。

- [ ] **Step 4: Commit**

```bash
git add skills/ui-design-grounding/reference/legibility.md skills/legibility-ui/reference/perspectives.md
git commit -m "$(cat <<'EOF'
refactor(legibility-ui): 判断基準を ui-design-grounding リファレンスへ移設

既存20スキルと同じく、legibility-ui の判断基準を skills/legibility-ui
配下の自己完結ファイルではなく skills/ui-design-grounding/reference/
配下へ置き、ナレッジベース構造に統一する。

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>
EOF
)"
```

---

### Task 2: DESIGN.md ゲート適用区分表の更新

**Files:**
- Modify: `skills/ui-design-grounding/reference/design-md-gate.md:57`

**Interfaces:**
- Consumes: なし
- Produces: なし（ドキュメントのみの変更）

- [ ] **Step 1: 適用区分表に legibility-ui を追加する**

`skills/ui-design-grounding/reference/design-md-gate.md` の57行目を次のように変更する。

変更前:
```
| 第1層・評価 | `audit-ui` `score-ui` | ✓ | —（実装を変更しない） |
```

変更後:
```
| 第1層・評価 | `audit-ui` `score-ui` `legibility-ui` | ✓ | —（実装を変更しない） |
```

- [ ] **Step 2: 検証する**

```bash
grep -n "legibility-ui" "skills/ui-design-grounding/reference/design-md-gate.md"
```

Expected: 57行目に `legibility-ui` を含む行が1件出力される。

- [ ] **Step 3: Commit**

```bash
git add skills/ui-design-grounding/reference/design-md-gate.md
git commit -m "$(cat <<'EOF'
docs(design-md-gate): legibility-ui を前段ゲート適用スキルに追加

legibility-ui は評価専用スキルとして前段ゲートのみを持つため、
audit-ui / score-ui と同じ区分に追加する。

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>
EOF
)"
```

---

### Task 3: `legibility-ui/SKILL.md` の全面書き換え

**Files:**
- Modify: `skills/legibility-ui/SKILL.md`（全文置換）

**Interfaces:**
- Consumes: `ui-design-grounding/reference/legibility.md`（Task 1 で作成）、`ui-design-grounding/reference/design-md-gate.md`（Task 2 で更新）
- Produces: `/legibility-ui` コマンドスキル本体。Task 5/6/7 の一覧表からこのコマンド名が参照される。

- [ ] **Step 1: SKILL.md を以下の内容で全文置換する**

```markdown
---
name: legibility-ui
description: 実装を読む前に、画面を初見で見たときの分かりやすさを5つの観点（見た目と機能の一致・機能の目的の透明性・コントロールのスコープ整合性・重複と冗長表示・画面間の設計一貫性）で監査する。UIが「使えば分かる」を前提にしていないか、初見のユーザー視点で確認したいときに使う。トリガ例「初見で分かりづらい箇所をレビューして」「UIUXレビュー」「この画面何をするか分からない」「画面同士の一貫性をチェック」「同じ情報が重複していないか確認して」。
user-invocable: true
argument-hint: "[対象画面 (画面名／機能横断／省略で全画面)]"
---

# legibility-ui — 初見の分かりやすさ監査

score-ui が「実装を読んでからニールセンの10ヒューリスティクスで評価する」のに対し、本スキルは**実装を一切読まずに、画面だけを見た初見の人間としてどう見えるか**を出発点にする。先にコードを読むと実装意図に引きずられ、初見での誤解を検出できなくなるため、**観察の順序を厳守する**（画面を見る→判断する→最後にコードを裏付けとして見る）。

## 準備（MANDATORY PREPARATION）

ui-design-grounding スキルを呼び出し、以下のリファレンスを読み込む:

- `ui-design-grounding/reference/legibility.md` — 5レンズの判断基準・重篤度・出力フォーマット
- `ui-design-grounding/reference/design-md-gate.md` — DESIGN.md ゲート（前段）の手順

## Phase 0 — 対象確定・基準確認・準備

### 対象確定

呼び出し引数を解釈する。

- 画面名の指定（例:「Home/Tasks/Notesをチェック」）→ 指定画面のみを対象にする。
- 「機能横断」「一連の流れ」等の指定 → 関連する画面をまとめて対象化する（例: タスク登録・編集・一覧）。
- 指定なし／「全画面」 → 一般的なルーター定義パターン（React Router の `routes.tsx`/`router.tsx`、Next.js の `app/` ディレクトリ構造、Vue Router の `router/index.ts` 等）を Glob/Grep で探索し、画面一覧を推定する（この探索は対象範囲の把握のみが目的で、Phase 1 の判断材料にしてはならない）。**見つからなければ、ユーザーに主要画面を確認する。**
- 対象が2画面以上ならレンズ⑤（画面間一貫性）も実施する。1画面のみならレンズ①〜④のみ実施する。

### 基準の確認（DESIGN.md 前段ゲート）

`design-md-gate.md` の前段ゲートを実施する。DESIGN.md があれば front matter のトークンと本文の指針を基準として読み込み、レンズ⑤（画面間一貫性）で「意図的な差異の正当性」を判定する材料にする。無ければ未整備として扱い `/init-design` を提案する。評価専用のため後段ゲートは不要。

### 準備

1. Playwright（`mcp__plugin_playwright_playwright__*`）が使えることを確認する。使えない場合はその旨を出力して終了する（代替手段は判定精度を大きく損なうため用意しない）。
2. アプリが起動していなければ、プロジェクトにアプリ起動用のスキルがあればそれを使う。なければユーザーに起動方法を確認する。
3. **この時点でも対象コンポーネントのソースは読まない。**
4. スクリーンショットは、プロジェクトの一時ファイル置き場の慣例（`.gitignore` が定める一時ディレクトリ等）に保存する。無ければ妥当な一時ディレクトリを使う。ファイル名だけの指定は避け、明示的なパスを指定する。

## Phase 1 — 素朴初見監査（画面ごと・コードを読まずに実施）

対象画面ごとに以下を行う。**この Phase の間はコンポーネントのソースを一切開かない。**

Playwright 操作は判定の性質で使い分ける:
- **視覚判定**（レンズ①④⑤の一部 — 見た目の重み・境界線の連結・不揃い・重複表示）→ `browser_take_screenshot` を使う。レンダリング結果そのものが必要なため。
- **機能判定**（レンズ②③ — 目的理解・スコープ確認）→ `browser_snapshot`（アクセシビリティツリーのテキスト表現）で要素の存在・状態遷移を確認すれば足りる。画像取得は不要。

1. Playwright で画面を開く。視覚判定が必要なら `browser_take_screenshot`、機能判定のみなら `browser_snapshot` を取得する。
2. 画面内の主要な操作可能要素（ボタン・リンク・フィルタ・入力欄・見出し・バッジ等）を一つずつ挙げ、**先に**「初見でこれは何をするものだと思うか」を言語化する。`reference/legibility.md` のレンズ①②の問いを当てる。
3. 実際にクリック・操作して本当の挙動を確認し、手順2の予想とのギャップを記録する。ギャップがあれば①または②の指摘候補にする。操作結果の確認は `browser_snapshot` で行う。
4. 影響が画面外に及びそうなコントロール（設定変更・名称変更・全体に効くフィルタ等）は、実際に操作して影響範囲を確認し、設置場所の妥当性を判定する（レンズ③、`reference/legibility.md` 参照）。
5. 画面内の見出し・識別子（プロジェクト名・画面名等）と、似た名前・似た役割を持つ複数の項目（フィールド・設定・ボタン）を洗い出し、同じ情報や概念が重複していないか、使い分けが初見で説明できるかを確認する（レンズ④、`reference/legibility.md` 参照）。
6. 画面内に同時に見えている同種コンポーネント（入力欄・ドロップダウン・ボタン等）どうしのサイズ・高さ・余白を比べ、不揃いがないか確認する。これは1画面のレビューでも毎回行う（レンズ①、`reference/legibility.md` 参照）。スクリーンショットで確認する。

## Phase 2 — 横断一貫性監査（対象2画面以上のときのみ）

Phase 1 で集めたスクリーンショット・操作結果をもとに、画面をまたいで同種の概念（一覧→作成/編集、フィルタUI、ヘッダー/共通chrome など）および同種の基本コンポーネント（主要CTAボタン・ドロップダウン・入力欄のサイズ等）を比較する。実装パターンやビジュアルが画面ごとに異なる箇所を挙げる（レンズ⑤、`reference/legibility.md` 参照）。DESIGN.md が読み込まれていれば、差異が意図的なもの（既存の設計判断に根拠がある）かをそこに照らして判定する。

## Phase 3 — コード特定

Phase 1/2 で見つかった指摘についてのみ、該当コンポーネントを Grep して `file:line` を特定する。**ここで初めてコードを読む。** 判定そのものは Phase 1/2 の「初見の目」を正とし、コードを読んだことを理由に指摘を取り下げない（コードの意図が分かっても、初見で伝わらないなら指摘は有効）。

## Phase 4 — 出力

`reference/legibility.md` の出力フォーマットに従い、レンズ別の指摘一覧（画面・指摘・重篤度・該当コード）と推奨アクションをまとめて報告する。`reference/legibility.md`「対象外（他スキルに委ねる観点）」に該当する気づき（エルゴノミクス・ピクセル整合・将来のデータ量への堅牢性等）があれば、指摘表には含めず「対象外として見送った観点」として委譲先スキルとともに一言添える。
```

- [ ] **Step 2: 汎用化漏れがないか検証する**

```bash
grep -n "router.tsx\|NAV_ITEMS\|start-app\|\.screenshots/legibility-ui" "skills/legibility-ui/SKILL.md"
```

Expected: 出力なし（該当箇所ゼロ件）。何か出力されればプロジェクト固有記述が残っているのでStep 1を修正する。

- [ ] **Step 3: front matter とMANDATORY PREPARATIONの存在を検証する**

```bash
head -6 "skills/legibility-ui/SKILL.md"
grep -n "MANDATORY PREPARATION\|reference/legibility.md\|reference/design-md-gate.md" "skills/legibility-ui/SKILL.md"
```

Expected: front matter に `name: legibility-ui` `user-invocable: true` `argument-hint` が含まれ、`MANDATORY PREPARATION` の節に2つのリファレンスパスが列挙されていること。

- [ ] **Step 4: Commit**

```bash
git add skills/legibility-ui/SKILL.md
git commit -m "$(cat <<'EOF'
feat(legibility-ui): プラグイン汎用構造に合わせてSKILL.mdを書き換え

移植元プロジェクト固有のロジック（router.tsx の NAV_ITEMS 決め打ち、
start-app スキル固定依存、.screenshots/legibility-ui/ 固定パス）を
一般化し、MANDATORY PREPARATION と DESIGN.md 前段ゲートを既存20スキル
と同じ構造で追加する。Playwright 操作は視覚判定(screenshot)と機能判定
(snapshot)を使い分けてトークン効率化する。

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>
EOF
)"
```

---

### Task 4: `plugin.json` のバージョン・件数更新

**Files:**
- Modify: `.claude-plugin/plugin.json`

**Interfaces:**
- Consumes: なし
- Produces: なし

- [ ] **Step 1: version と description を更新する**

`.claude-plugin/plugin.json` の内容を次のように変更する。

変更前:
```json
{
  "name": "ui-design-grounding",
  "description": "UI/UXの設計・実装における判断軸と知識ベースを提供するプラグイン。ナレッジベース1件と22件のコマンドスキルで構成。",
  "version": "1.2.1",
  "author": {
    "name": "Nextscape Inc."
  },
  "repository": "https://github.com/nextscape/ui-design-grounding",
  "license": "MIT"
}
```

変更後:
```json
{
  "name": "ui-design-grounding",
  "description": "UI/UXの設計・実装における判断軸と知識ベースを提供するプラグイン。ナレッジベース1件と23件のコマンドスキルで構成。",
  "version": "1.3.0",
  "author": {
    "name": "Nextscape Inc."
  },
  "repository": "https://github.com/nextscape/ui-design-grounding",
  "license": "MIT"
}
```

- [ ] **Step 2: 検証する**

```bash
grep -n "\"version\"\|23件のコマンドスキル" ".claude-plugin/plugin.json"
```

Expected: `"version": "1.3.0"` と `23件のコマンドスキル` を含む行がそれぞれ出力される。

- [ ] **Step 3: Commit**

```bash
git add .claude-plugin/plugin.json
git commit -m "$(cat <<'EOF'
chore: legibility-ui 追加に伴いバージョンを1.3.0へ

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>
EOF
)"
```

---

### Task 5: `ui-design-grounding/SKILL.md` の一覧更新

**Files:**
- Modify: `skills/ui-design-grounding/SKILL.md:30`
- Modify: `skills/ui-design-grounding/SKILL.md:44-45`（コマンドスキル一覧表）

**Interfaces:**
- Consumes: `/legibility-ui`（Task 3 で作成したコマンド名）
- Produces: なし

- [ ] **Step 1: 独立コマンドスキル件数を更新する**

30行目を変更する。

変更前:
```
- 20個の独立コマンドスキル（`ui-help` を除く）が MANDATORY PREPARATION として本スキルのリファレンスを参照する
```

変更後:
```
- 21個の独立コマンドスキル（`ui-help` を除く）が MANDATORY PREPARATION として本スキルのリファレンスを参照する
```

- [ ] **Step 2: コマンドスキル一覧表に1行追加する**

44-45行目付近（技術品質の監査・UXヒューリスティクス採点の行の直後）に追加する。

変更前:
```
| 技術品質の監査 | `/audit-ui` |
| UXヒューリスティクス採点 | `/score-ui` |
| リリース前の最終仕上げ | `/polish-ui` |
```

変更後:
```
| 技術品質の監査 | `/audit-ui` |
| UXヒューリスティクス採点 | `/score-ui` |
| 初見の分かりやすさ監査 | `/legibility-ui` |
| リリース前の最終仕上げ | `/polish-ui` |
```

- [ ] **Step 3: 検証する**

```bash
grep -n "21個の独立コマンドスキル\|/legibility-ui" "skills/ui-design-grounding/SKILL.md"
```

Expected: 2行が出力される（件数更新行と一覧表への追記行）。

- [ ] **Step 4: Commit**

```bash
git add skills/ui-design-grounding/SKILL.md
git commit -m "$(cat <<'EOF'
docs(ui-design-grounding): コマンドスキル一覧に legibility-ui を追加

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>
EOF
)"
```

---

### Task 6: `ui-help/SKILL.md` の一覧更新

**Files:**
- Modify: `skills/ui-help/SKILL.md`（第1層テーブル・「迷ったら」節）

**Interfaces:**
- Consumes: `/legibility-ui`
- Produces: なし

- [ ] **Step 1: 第1層テーブルに1行追加する**

`/score-ui` の行の直後に追加する。

変更前:
```
| `/score-ui` | **評価（UX）** — ニールセン10原則で採点（/40点）＋ 5種ペルソナでレッドフラグテスト＋認知負荷アセスメント。課題を対応スキルへ自動マッピング | UX品質を構造的に評価したい |
| `/init-design` | **基準化** — DESIGN.md を生成・更新。design.md 仕様（YAML トークン + Summary + 8セクション散文）準拠。既存CSS/トークンから自動抽出対応 | デザインシステムの基盤を定義したい |
```

変更後:
```
| `/score-ui` | **評価（UX）** — ニールセン10原則で採点（/40点）＋ 5種ペルソナでレッドフラグテスト＋認知負荷アセスメント。課題を対応スキルへ自動マッピング | UX品質を構造的に評価したい |
| `/legibility-ui` | **評価（初見）** — 実装を読まず画面だけを見て、見た目と機能の一致・目的の透明性・スコープ整合・重複表示・画面間一貫性の5レンズで監査。コードは最後に裏付けとしてのみ参照 | 初見のユーザー視点で分かりにくさを洗い出したい |
| `/init-design` | **基準化** — DESIGN.md を生成・更新。design.md 仕様（YAML トークン + Summary + 8セクション散文）準拠。既存CSS/トークンから自動抽出対応 | デザインシステムの基盤を定義したい |
```

- [ ] **Step 2: 「迷ったら」節に1行追加する**

変更前:
```
- **何から始める？** → `/init-design`（デザイン基盤）→ `/audit-ui`（現状把握）
- **全体的に良くしたい** → `/polish-ui`（チェックリスト駆動で網羅的に）
- **どこを直すか分からない** → `/refine-ui`（観点ベースで診断して直す）
```

変更後:
```
- **何から始める？** → `/init-design`（デザイン基盤）→ `/audit-ui`（現状把握）
- **全体的に良くしたい** → `/polish-ui`（チェックリスト駆動で網羅的に）
- **どこを直すか分からない** → `/refine-ui`（観点ベースで診断して直す）
- **初見での分かりやすさを確認したい** → `/legibility-ui`（実装を読まず画面だけで判断）
```

- [ ] **Step 3: 検証する**

```bash
grep -n "/legibility-ui" "skills/ui-help/SKILL.md"
```

Expected: 2行出力される（第1層テーブルと「迷ったら」節）。

- [ ] **Step 4: Commit**

```bash
git add skills/ui-help/SKILL.md
git commit -m "$(cat <<'EOF'
docs(ui-help): コマンド一覧に legibility-ui を追加

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>
EOF
)"
```

---

### Task 7: `CLAUDE.md` の早見表更新

**Files:**
- Modify: `CLAUDE.md`（アーキテクチャ節の件数・コマンドスキル早見表）

**Interfaces:**
- Consumes: `/legibility-ui`
- Produces: なし

- [ ] **Step 1: アーキテクチャ節の件数を更新する**

変更前:
```
2. **コマンドスキル**（`skills/<name>/`） — 22 件のユーザー呼び出し可能なスラッシュコマンド（`/design-ui`、`/audit-ui`、`/score-ui`、`/ui-help` 等）。各スキルが構造化されたワークフローを定義し、必要に応じてナレッジベースのリファレンスを参照しながら分析・評価・実装をガイドする。
```

変更後:
```
2. **コマンドスキル**（`skills/<name>/`） — 23 件のユーザー呼び出し可能なスラッシュコマンド（`/design-ui`、`/audit-ui`、`/score-ui`、`/ui-help` 等）。各スキルが構造化されたワークフローを定義し、必要に応じてナレッジベースのリファレンスを参照しながら分析・評価・実装をガイドする。
```

- [ ] **Step 2: コマンドスキル早見表に1行追加する**

変更前:
```
| 評価 | `/audit-ui` | 技術品質5軸監査 |
| 評価 | `/score-ui` | UXヒューリスティクス採点 |
| 調整 | `/refine-ui` | 曖昧な訴えを観点ベースで診断→委譲して直す（第1層・直す） |
```

変更後:
```
| 評価 | `/audit-ui` | 技術品質5軸監査 |
| 評価 | `/score-ui` | UXヒューリスティクス採点 |
| 評価 | `/legibility-ui` | 初見の分かりやすさ監査 |
| 調整 | `/refine-ui` | 曖昧な訴えを観点ベースで診断→委譲して直す（第1層・直す） |
```

- [ ] **Step 3: 検証する**

```bash
grep -n "23 件のユーザー呼び出し可能\|/legibility-ui" "CLAUDE.md"
```

Expected: 2行出力される（件数更新行と早見表への追記行）。

- [ ] **Step 4: Commit**

```bash
git add CLAUDE.md
git commit -m "$(cat <<'EOF'
docs: CLAUDE.md のコマンドスキル早見表に legibility-ui を追加

Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>
EOF
)"
```

---

### Task 8: 全体整合性の最終検証

**Files:**
- なし（検証のみ、変更は加えない）

**Interfaces:**
- Consumes: Task 1〜7 の全成果物
- Produces: なし

- [ ] **Step 1: プロジェクト固有記述が一切残っていないことを確認する**

```bash
grep -rn "router.tsx\|NAV_ITEMS\|start-app\|\.screenshots/legibility-ui\|perspectives.md" skills/ CLAUDE.md .claude-plugin/ 2>/dev/null
```

Expected: 出力なし。何か出力された場合は該当ファイルを Task 1〜3 に立ち返って修正する。

- [ ] **Step 2: `/legibility-ui` への言及が全ての一覧ファイルに存在することを確認する**

```bash
grep -l "legibility-ui" skills/ui-design-grounding/SKILL.md skills/ui-help/SKILL.md CLAUDE.md .claude-plugin/plugin.json skills/legibility-ui/SKILL.md skills/ui-design-grounding/reference/design-md-gate.md skills/ui-design-grounding/reference/legibility.md
```

Expected: 渡した7ファイル全てのパスが出力される（`grep -l` は該当なしのファイルを出力しないため、7行に満たなければ漏れがある）。

- [ ] **Step 3: `git status` で untracked ファイルが残っていないことを確認する**

```bash
git status --short
```

Expected: `skills/legibility-ui/` 配下・変更した5ファイルがすべて commit 済みで、作業ツリーがクリーンであること（`??` や変更行が出力されない）。

- [ ] **Step 4: 最終ログを確認する**

```bash
git log --oneline -8
```

Expected: Task 1〜7 の7つのコミットが積み上がっていること。
