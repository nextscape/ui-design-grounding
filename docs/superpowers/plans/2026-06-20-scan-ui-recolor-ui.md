# scan-ui / recolor-ui 新規スキル実装プラン

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 外部サイト(URL)を Playwright で分析し DESIGN.md を逆算する `scan-ui` と、既存 DESIGN.md を OKLCH で再配色する `recolor-ui` の2コマンドスキルを追加し、プラグインの索引・相互参照を更新する。

**Architecture:** 各スキルは YAML フロントマター + Markdown 本文の `SKILL.md` 単一ファイル。既存コマンドスキルの構成（準備 → 手順 → 出力フォーマット → 推奨される次のステップ）に倣う。DESIGN.md フォーマットは単一情報源 `reference/design-md-spec.md` を参照し再定義しない。

**Tech Stack:** Markdown のみ（ビルド・テスト・ランタイムなし）。`scan-ui` は実行時に Playwright MCP（`mcp__plugin_playwright_playwright__*`）または Playwright CLI（`npx playwright`）を利用する。

## Global Constraints

- 全スキルコンテンツは**日本語**で記述する。
- 各スキルは YAML フロントマター（`name`, `description`, `user-invocable: true`, `argument-hint`）+ Markdown 本文の `SKILL.md` 単一ファイル。
- `description` は具体的・網羅的に（スキルレジストリのマッチングに使われる）。
- DESIGN.md の書式は `reference/design-md-spec.md` を参照し、フォーマット仕様を再定義しない。
- リファレンス参照名は実在ファイル名を使う: `motion-design.md` / `responsive-design.md` / `design-md-spec.md`（`motion.md` や `responsive.md` は存在しない）。
- 命名規約: コマンドスキルは `<原形動詞>-ui`。確定名は `scan-ui` / `recolor-ui`。
- ビルド・テスト・リントは存在しない。検証はファイル読み戻し・フロントマター目視・cross-reference grep で行う。
- ユーザーのプロジェクトファイル（このリポジトリ内では CLAUDE.md 等）への破壊的変更はしない。追記・更新のみ。
- 既存スキル数の事実: コマンドスキル 18 → 20、リファレンス 16 件（`design-md-spec.md` 追加済み）。索引の件数表記はこの事実に合わせる。

---

### Task 1: scan-ui スキルを作成

**Files:**
- Create: `skills/scan-ui/SKILL.md`

**Interfaces:**
- Consumes: `reference/design-md-spec.md`（DESIGN.md 出力契約）, `reference/design-tokens.md`, `reference/color-system.md`, `reference/typography.md`, `reference/spatial-layout.md`, `reference/interaction.md`, `reference/motion-design.md`, `reference/anti-patterns.md`
- Produces: `/scan-ui` コマンド。出力は `reference/design-md-spec.md` 準拠の DESIGN.md。Task 3 の索引・相互参照が `scan-ui` の name と1行説明を参照する。

- [ ] **Step 1: `skills/scan-ui/SKILL.md` を作成**

以下の内容で新規作成する:

````markdown
---
name: scan-ui
description: 外部の既存サイト(URL)を分析し、デザインシステムを逆算して DESIGN.md として出力する。Playwright で生の computed style・スクリーンショットを読み取り、色・タイポグラフィ・余白・角丸・深度・モーション・コンポーネントを抽出する。単一ページ／サイト全体（複数ページをカバレッジ集計で統合）の2モード。競合サイトや参考サイトのデザイン言語を取り込みたいときに使用する。自分のコードの整理は extract-ui、自社デザイン憲法の新規定義は init-design を使う。
user-invocable: true
argument-hint: "[URL] [--site（サイト全体クロール）]"
---

# scan-ui

## 概要

外部の既存サイト（URL）を Playwright で分析し、デザインシステムを逆算して **DESIGN.md** として出力する。designlang の `extract` / `site` / `brand` の発想を、このプラグインのハブである DESIGN.md 出力に統合したもの。

**使い分け**:
- 外部 URL の分析 → 本スキル `scan-ui`
- 自分のコードの部品化・トークン化（移行計画） → `extract-ui`
- 自社デザイン憲法の新規定義・更新 → `init-design`

出力は**たたき台**であり、最終判断は人間に委ねる。他サイトの取り込みは参照・学習目的とし、商標・独自表現の流用は避ける。

## 準備（MANDATORY PREPARATION）

ui-design-grounding スキルを呼び出し、以下のリファレンスを読み込む:

- `ui-design-grounding/reference/design-md-spec.md` — **出力契約（必須）**: front matter 構造・`{}` 参照構文・CLI 処理規則・本文8セクション
- `ui-design-grounding/reference/design-tokens.md` — 2層トークン理論・役割語彙（Material 3 系）
- `ui-design-grounding/reference/color-system.md`
- `ui-design-grounding/reference/typography.md`
- `ui-design-grounding/reference/spatial-layout.md`
- `ui-design-grounding/reference/interaction.md`
- `ui-design-grounding/reference/motion-design.md`
- `ui-design-grounding/reference/anti-patterns.md`

## モード

引数で分岐する:

- **単一ページ**（デフォルト）: 指定 URL 1枚を分析する。
- **サイト全体**（`--site`）: 代表ページ（home / pricing / docs / blog 等）を巡回し、**カバレッジ**（各トークンが何%のページで使われるか）でトークンを選別。サイト共通値と page-local な外れ値を分離して統合する。

## 分析手段（優先順位・フォールバック）

抽出ロジック（下記の DOM walk スニペット）は**どの手段でも同一のもの**を流す（手段差で結果がぶれないように）。

1. **Playwright MCP（第一候補）** — `browser_navigate` でページを開き、`browser_evaluate` に下記スニペットを渡して単一パス抽出。`browser_take_screenshot` で色・レイアウトを目視検証する。追加インストール不要。
2. **Playwright CLI（フォールバック）** — MCP が無い環境。`npx playwright` で自己完結スクリプトを実行（初回は `npx playwright install chromium`）。特に `--site` の複数ページ巡回は for ループが堅牢で、ユーザーが後で再実行できる成果物にもなる。
3. **WebFetch / 手動貼り付け（最終手段）** — ブラウザが一切使えない環境。静的 HTML/CSS を取得、またはユーザーが貼り付けた computed style から推定する。**精度が落ちることを出力に明示**する。

### DOM walk 抽出スニペット（共通）

```js
() => {
  const bump = (m, k) => { if (k && k !== 'none') m.set(k, (m.get(k) || 0) + 1); };
  const colors = new Map(), bgs = new Map(), fonts = new Map(),
        sizes = new Map(), weights = new Map(), lineHeights = new Map(),
        letterSpacings = new Map(), radii = new Map(), shadows = new Map(),
        spaces = new Map(), transitions = new Map();
  for (const el of document.querySelectorAll('body *')) {
    const r = el.getBoundingClientRect();
    if (r.width === 0 || r.height === 0) continue;       // 非表示要素は除外
    const s = getComputedStyle(el);
    bump(colors, s.color);
    bump(bgs, s.backgroundColor);
    bump(fonts, s.fontFamily);
    bump(sizes, s.fontSize);
    bump(weights, s.fontWeight);
    bump(lineHeights, s.lineHeight);
    bump(letterSpacings, s.letterSpacing);
    bump(radii, s.borderRadius);
    bump(shadows, s.boxShadow);
    bump(spaces, s.padding);
    bump(spaces, s.gap);
    if (s.transitionDuration && s.transitionDuration !== '0s') {
      bump(transitions, s.transitionDuration + ' ' + s.transitionTimingFunction);
    }
  }
  const top = (m, n = 12) => [...m.entries()].sort((a, b) => b[1] - a[1]).slice(0, n);
  return {
    url: location.href, title: document.title,
    colors: top(colors), backgrounds: top(bgs), fontFamilies: top(fonts),
    fontSizes: top(sizes), fontWeights: top(weights), lineHeights: top(lineHeights),
    letterSpacings: top(letterSpacings), radii: top(radii), shadows: top(shadows),
    spacing: top(spaces, 16), transitions: top(transitions),
  };
}
```

### CLI スクリプト雛形（フォールバック時）

```js
// scan-extract.js — node scan-extract.js <url> [url2 ...]
const { chromium } = require('playwright');
const EXTRACT = /* 上記スニペットの関数本体をここに貼る */ null;
(async () => {
  const urls = process.argv.slice(2);
  const browser = await chromium.launch();
  const page = await browser.newPage();
  const out = [];
  for (const url of urls) {
    await page.goto(url, { waitUntil: 'networkidle' });
    out.push(await page.evaluate(EXTRACT));
  }
  await browser.close();
  console.log(JSON.stringify(out, null, 2));
})();
```

## 手順

### 1. 取得

選択した手段で対象 URL（`--site` 時は代表ページ群）を開き、DOM walk スニペットで raw データを回収する。可能ならスクリーンショットも取得し目視検証に使う。

### 2. 正規化・クラスタリング

- 色: `rgb()/rgba()` を集計し、近い色をクラスタリングして primitive パレット候補に。最頻の前景/背景から `on-*` ペアを推定。
- タイポグラフィ: fontFamily × size × weight の組をレベル化（headline / body / label）。
- 余白: padding/gap の最頻値からベースユニット（4/8px 等）を推定。
- 角丸・シャドウ・モーション: 最頻値をスケール化。

### 3. 2層トークンへ整理

`design-tokens.md` の役割語彙に従い、primitive（literal）→ semantic（`{}` 参照）の2層へ。`design-md-spec.md` の書式に厳密に従う。

### 4. `--site` のカバレッジ選別

各トークンの「出現ページ率」を算出し、共通（例: ≥60%）を採用、外れ値は本文に注記。カバレッジ表を出力に併記する。

### 5. DESIGN.md 生成

`design-md-spec.md` の出力契約（front matter + 本文8セクション）に厳密準拠して DESIGN.md を生成。全色を sRGB 変換し WCAG コントラストを検証する。

## 出力フォーマット

```markdown
## scan-ui 分析結果

- 対象: [URL]（モード: 単一ページ / サイト全体 N ページ）
- 分析手段: [Playwright MCP / CLI / WebFetch]
- 色トークン: primitive N色 / semantic N色
- タイポグラフィ: N レベル（主フォント: [名]）
- spacing: ベース [unit]px
- rounded / shadows / motion: [要約]

### カバレッジ（--site 時のみ）
| トークン | 値 | 使用ページ率 |
|---------|----|------------|
| ... | ... | NN% |

### 生成物
- DESIGN.md を [パス] に出力（design.md alpha 準拠）

### 要確認事項
- [人間の判断が必要な項目／低精度だった項目]

### 注意
- 取り込みは参照・学習目的。商標・独自表現の流用は避ける。
```

DESIGN.md 本体は `reference/design-md-spec.md` のフォーマットに従って別途生成し、プロジェクトルート（または指定パス）に書き出す。

## 推奨される次のステップ

- `/audit-ui` — 取り込んだ DESIGN.md に対する技術品質監査
- `/recolor-ui` — 取り込んだパレットを自社ブランド primary に置き換え
- `/init-design` — 生成された DESIGN.md の差分更新・調整
````

- [ ] **Step 2: フロントマターと参照名を検証**

Run: `node -e "const fs=require('fs');const t=fs.readFileSync('skills/scan-ui/SKILL.md','utf8');const m=t.match(/^---\n([\s\S]*?)\n---/);console.log(m?'frontmatter OK':'NO FRONTMATTER');console.log(/name: scan-ui/.test(t)&&/user-invocable: true/.test(t)?'fields OK':'FIELDS MISSING')"`
Expected: `frontmatter OK` と `fields OK` が出力される。

Run: `git grep -n "motion.md\|responsive.md" skills/scan-ui/SKILL.md || echo "no bad refs"`
Expected: `no bad refs`（存在しないリファレンス名を参照していない）。

- [ ] **Step 3: コミット**

```bash
git add skills/scan-ui/SKILL.md
git commit -m "feat: scan-ui スキルを追加（外部URL分析→DESIGN.md）"
```

---

### Task 2: recolor-ui スキルを作成

**Files:**
- Create: `skills/recolor-ui/SKILL.md`

**Interfaces:**
- Consumes: `reference/color-system.md`, `reference/design-tokens.md`, `reference/design-md-spec.md`
- Produces: `/recolor-ui` コマンド。入力は既存 DESIGN.md / CSS トークン、出力は再配色後の `colors` front matter。Task 3 の索引が `recolor-ui` を参照する。

- [ ] **Step 1: `skills/recolor-ui/SKILL.md` を作成**

以下の内容で新規作成する:

````markdown
---
name: recolor-ui
description: 既存の DESIGN.md（または CSS トークン）のカラーパレットを、新しいブランド primary を中心に OKLCH で破綻なく再配色する。タイポグラフィ・余白・角丸・モーションは保持し、明度・彩度の関係と WCAG コントラスト、on-color ペアを維持・再計算する。ブランドカラー変更・配色テーマ切り替え・パレット差し替えを依頼されたときに使用する。外部URLのパレットを取り込みたい場合は scan-ui で分析してから本スキルを使う。
user-invocable: true
argument-hint: "[--primary <hex/oklch>] [対象 DESIGN.md / トークン]"
---

# recolor-ui

## 概要

既存の **DESIGN.md（または CSS トークン）**のカラーパレットを、新しい primary を中心に **OKLCH で破綻なく再配色**する。タイポグラフィ・余白・角丸・モーションは保持する。

入力は自プロジェクトの DESIGN.md / CSS トークン。外部 URL を再配色したい場合は `scan-ui`（分析）→ `recolor-ui`（再配色）のチェーンで行う（本スキルは URL を直接扱わない）。

## 準備（MANDATORY PREPARATION）

ui-design-grounding スキルを呼び出し、以下のリファレンスを読み込む:

- `ui-design-grounding/reference/color-system.md` — OKLCH、コントラスト、ダークモード
- `ui-design-grounding/reference/design-tokens.md` — 2層トークン構造・役割語彙
- `ui-design-grounding/reference/design-md-spec.md` — `colors` の書式・`{}` 参照構文

## 手順

### 1. 入力の読み込み

DESIGN.md の front matter `colors`（または CSS のカラートークン）を読み、primitive ランプと semantic マッピングを把握する。新 primary（`--primary`）を取得する（未指定なら確認）。

### 2. OKLCH へ変換

既存の各 primitive 色と新 primary を OKLCH（L=明度 / C=彩度 / H=色相）へ変換する。

### 3. ランプ再生成

- 新 primary の H（色相）を基準に、**既存ランプの L・C の相対関係を保ったまま**色相を回してパレット全体を再生成する。
- ニュートラル系は「色相を僅かに primary 側へ寄せる（ティンテッドニュートラル）」か「中立を保つ」かを**選択肢として提示**する。
- secondary / tertiary がある場合は primary との色相差（補色・類似等）の関係を維持する。

### 4. コントラスト再検証

- `on-primary` / `on-surface` 等の on-color ペアを WCAG 基準（本文 4.5:1、大文字 3:1）で再検証し、満たさなければ L を調整して再計算する。
- semantic の `{}` 参照構造は壊さない（参照先 primitive の値だけ変える）。

### 5. 反映

front matter `colors` を更新し、必要なら本文 `## Colors` の散文も新パレットに合わせて更新する。

## 出力フォーマット

```markdown
## recolor-ui 結果

- 新 primary: [入力値] → OKLCH(L C H)
- ニュートラル方針: ティンテッド / 中立

### 変更前後（colors）
| トークン | Before | After | コントラスト |
|---------|--------|-------|------------|
| primary | #xxx | #yyy | — |
| on-primary | #xxx | #yyy | N.N:1（対 primary）|
| ... | ... | ... | ... |

### 保持したもの
- タイポグラフィ / 余白 / 角丸 / モーション（変更なし）

### 要確認事項
- [コントラスト調整で意図とズレた可能性のある箇所]
```

## 推奨される次のステップ

- `/audit-ui` — 再配色後のコントラスト・トークン準拠を監査
- `/polish-ui` — 最終仕上げ
````

- [ ] **Step 2: フロントマターを検証**

Run: `node -e "const fs=require('fs');const t=fs.readFileSync('skills/recolor-ui/SKILL.md','utf8');console.log(/^---/.test(t)&&/name: recolor-ui/.test(t)&&/user-invocable: true/.test(t)?'OK':'FAIL')"`
Expected: `OK`

- [ ] **Step 3: コミット**

```bash
git add skills/recolor-ui/SKILL.md
git commit -m "feat: recolor-ui スキルを追加（OKLCH 再配色）"
```

---

### Task 3: 索引・相互参照を更新（プラグインへの組み込み）

**Files:**
- Modify: `CLAUDE.md`（コマンドスキル早見表・件数）
- Modify: `skills/ui-help/SKILL.md`（コマンド一覧）
- Modify: `skills/ui-design-grounding/SKILL.md`（コマンドスキル一覧・件数）
- Modify: `skills/ui-design-grounding/reference/design-md-spec.md`（利用者リスト）
- Modify: `skills/extract-ui/SKILL.md`（description 相互参照）
- Modify: `skills/init-design/SKILL.md`（description 相互参照）
- Modify: `README.md`（コマンドスキル一覧・件数）
- Modify: `.claude-plugin/plugin.json`（バージョン）

**Interfaces:**
- Consumes: Task 1/2 で確定した `scan-ui` / `recolor-ui` の name と1行説明。

- [ ] **Step 1: CLAUDE.md の早見表に2行追加**

[CLAUDE.md](CLAUDE.md) の「コマンドスキル早見表」テーブルに行を追加する。`/extract-ui` の直後（整理カテゴリ）と調整カテゴリにそれぞれ挿入:

整理カテゴリに追加:
```markdown
| 整理 | `/scan-ui` | 外部サイト(URL)分析→DESIGN.md |
```
調整カテゴリ（`/arrange-ui` の並び）に追加:
```markdown
| 調整 | `/recolor-ui` | パレットをOKLCHで再配色 |
```

同ファイルの「アーキテクチャ」節と早見表前後にある「18 件のコマンドスキル」「コマンドスキルの数を18件」等の記述を **20 件**に更新する。

Run: `git grep -n "scan-ui\|recolor-ui" CLAUDE.md`
Expected: 追加した2行がヒットする。

- [ ] **Step 2: ui-help/SKILL.md にコマンド追加**

[skills/ui-help/SKILL.md](skills/ui-help/SKILL.md) の「整理・抽出」テーブルに scan-ui を、「ビジュアル調整」テーブルに recolor-ui を追加:

整理・抽出に:
```markdown
| `/scan-ui` | 外部サイト(URL)を Playwright で分析し、デザインシステムを逆算して DESIGN.md を出力。単一ページ／サイト全体の2モード | 他サイトのデザインを取り込みたい |
```
ビジュアル調整に:
```markdown
| `/recolor-ui` | 既存 DESIGN.md のパレットを新 primary 中心に OKLCH で再配色。タイポ・余白・モーションは保持、コントラスト再検証 | ブランドカラーを変えたい・配色を差し替えたい |
```

「デザインシステムを整えたい」の迷ったらガイドに `/scan-ui` を補足:
```markdown
- **他サイトを参考にしたい** → `/scan-ui`（URL分析→DESIGN.md）→ `/recolor-ui`（自社カラーへ）
```

- [ ] **Step 3: ui-design-grounding/SKILL.md の一覧・件数を更新**

[skills/ui-design-grounding/SKILL.md](skills/ui-design-grounding/SKILL.md) の「コマンドスキル一覧」テーブルに2行追加:
```markdown
| 外部サイト(URL)を分析しDESIGN.md化 | `/scan-ui` |
| パレットをOKLCHで再配色 | `/recolor-ui` |
```
同ファイルの「17個の独立コマンドスキル」を **19個**（ui-help を含めるなら実数に合わせる）に更新する。※現行記述の実数を確認し、+2 した値にすること。

- [ ] **Step 4: design-md-spec.md の利用者リストに追記**

[skills/ui-design-grounding/reference/design-md-spec.md](skills/ui-design-grounding/reference/design-md-spec.md) の「このreferenceの使い方」リストに追加:
```markdown
- `scan-ui` — 外部 URL を分析して DESIGN.md を生成する際の出力契約
- `recolor-ui` — `colors` front matter を再配色する際の書式・`{}` 参照構文
```

- [ ] **Step 5: extract-ui / init-design の description に相互参照を追記**

[skills/extract-ui/SKILL.md](skills/extract-ui/SKILL.md) の description 末尾に一文追加:
```
（外部サイトの分析・取り込みは scan-ui を使う）
```
[skills/init-design/SKILL.md](skills/init-design/SKILL.md) の description 末尾に一文追加:
```
（外部 URL から逆算する場合は scan-ui を使う）
```

- [ ] **Step 6: README.md の一覧・件数を更新**

[README.md](README.md) の「整理・抽出」表に scan-ui、「ビジュアル調整」表に recolor-ui を追加（ui-help と同等の説明）。「18 件のコマンドスキル」「18件のスラッシュコマンド」を **20 件** に、リファレンス件数表記（`15 件`）を実数 **16 件** に更新する。

Run: `git grep -nc "18 件\|18件" README.md CLAUDE.md || echo "none remaining"`
Expected: 旧件数表記が残っていない（`none remaining` か 0）。

- [ ] **Step 7: plugin.json のバージョンを更新**

[.claude-plugin/plugin.json](.claude-plugin/plugin.json) の `version` を現行（1.0.4）から **1.1.0** へ（機能追加のため minor up）。

Run: `node -e "console.log(require('./.claude-plugin/plugin.json').version)"`
Expected: `1.1.0`

- [ ] **Step 8: 全体整合チェックとコミット**

Run: `git grep -nl "scan-ui" -- CLAUDE.md README.md skills/ui-help/SKILL.md skills/ui-design-grounding/SKILL.md skills/ui-design-grounding/reference/design-md-spec.md`
Expected: 5ファイルすべてに scan-ui への参照がある。

Run: `git grep -nl "recolor-ui" -- CLAUDE.md README.md skills/ui-help/SKILL.md skills/ui-design-grounding/SKILL.md`
Expected: 4ファイルすべてに recolor-ui への参照がある。

```bash
git add CLAUDE.md README.md .claude-plugin/plugin.json skills/ui-help/SKILL.md skills/ui-design-grounding/SKILL.md skills/ui-design-grounding/reference/design-md-spec.md skills/extract-ui/SKILL.md skills/init-design/SKILL.md
git commit -m "docs: scan-ui/recolor-ui をプラグイン索引・相互参照へ組み込み（v1.1.0）"
```

---

## Self-Review

**Spec coverage:**
- scan-ui（2モード・分析手段3段・抽出対象・DESIGN.md出力） → Task 1 ✓
- recolor-ui（OKLCH再配色・コントラスト再検証） → Task 2 ✓
- 境界の相互参照（extract-ui/init-design description） → Task 3 Step 5 ✓
- design-md-spec.md 利用者リスト追記 → Task 3 Step 4 ✓
- CLAUDE.md / ui-help / SKILL.md / README 索引・件数 → Task 3 ✓
- plugin.json バージョン → Task 3 Step 7 ✓
- 非目標（grade/remix/brand/PDF） → 着手しない（プランに含めない）✓

**Placeholder scan:** 各スキル本文・スニペットは完全な内容を記載済み。件数更新は「実数に合わせる」と明示（現行記述に揺れがあるため、実装者が実数を確認して +2 する指示）。

**Type consistency:** スキル名は全タスクで `scan-ui` / `recolor-ui` に統一。リファレンス名は実在の `motion-design.md` / `responsive-design.md` / `design-md-spec.md` を使用。
