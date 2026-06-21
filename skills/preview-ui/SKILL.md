---
name: preview-ui
description: DESIGN.md（YAML front matter のデザイントークン＋本文）を、ほぼ機械的に preview.html へ変換し、ブラウザで開いて視覚的に把握できるようにする。色（primitive ランプ→semantic ロール、on-color のコントラスト合否バッジ付き）・タイポグラフィ・余白・角丸・深度・コンポーネント atom の見本帳（specimen）に加え、既存トークンだけで組んだ小さな例画面を1つ生成する。scan-ui / init-design / recolor-ui が提案・更新した DESIGN.md の見た目を確認したいとき、デザイントークンをビジュアルでプレビュー・可視化したいときに使用する。
user-invocable: true
argument-hint: "[対象 DESIGN.md（省略時はプロジェクトルート）]"
---

# preview-ui

## 概要

**DESIGN.md** を **ほぼ機械的に** `preview.html` へ変換する。ブラウザで開けば、色・タイポ・余白・角丸・深度・コンポーネントのトークンが**実際の見た目**として一望でき、scan-ui / init-design / recolor-ui が提案・更新した DESIGN.md を視覚的に把握できる。

**このスキルはデザインしない。** DESIGN.md のトークンを固定テンプレの定位置へ**転記**するだけ。創造的判断・新しい値の発明は禁止。

## 準備（MANDATORY PREPARATION）

ui-design-grounding スキルを呼び出し、以下のリファレンスを読み込む:

- `ui-design-grounding/reference/design-md-spec.md` — **front matter 書式・`{}` 参照の解決規則・本文8セクション（必読）**
- `ui-design-grounding/reference/color-system.md` — sRGB 変換・WCAG コントラスト計算

## 機械的転記契約（不変条件）

生成する `preview.html` は次を必ず満たす。違反はバグ。

1. **全トークン露出** — front matter の全トークン（`colors` / `typography` / `rounded` / `spacing` / `components`）は specimen のどこかに必ず現れる。取りこぼさない。
2. **既存トークンのみで例画面を組む** — 末尾の例画面は CSS カスタムプロパティ（`var(--…)`）経由で既存トークンだけを参照する。新しい色・サイズ・角丸・字間などの値を一切発明しない。
3. **欠落は省略（捏造しない）** — DESIGN.md に無いトークン群の節は出力から省く。デフォルト値で埋めない。
4. **本文の散文も転記** — front matter だけでなく **DESIGN.md 本文の文言**（`## Summary`・各セクションの散文・`## Do's and Don'ts`）も、該当する specimen 節へ **verbatim（要約・改変せず）** に載せる。トークンと散文を併置することで「値＋意図」を同時に把握できる。本文に無い文言は足さない。
5. **決定論** — 同じ DESIGN.md からはほぼ同一の HTML が出る。下の固定テンプレを骨格として使い、値の差し込みだけを行う。

## 手順

### 1. DESIGN.md の読込・パース

対象 DESIGN.md（引数指定がなければプロジェクトルートの `DESIGN.md`）を読み、front matter を YAML としてパースする。あわせて**本文の各セクション散文**（Summary・Overview・Colors・Typography・Layout・Elevation・Shapes・Components・Do's and Don'ts）も抽出する（契約4で該当節へ転記する）。本文の `## Overview` はヘッダ説明にも流用してよい。

### 2. `{}` 参照の解決

`{colors.primary}` `{typography.label-md}` `{rounded.lg}` `{spacing.gutter}` 等の参照を、`design-md-spec.md` の規則に従って実値へ解決する。

- semantic → primitive、component → semantic/primitive を辿り、**最終的な literal 値**を得る。
- 解決できない（参照先が無い／`colors` のような group を指している）場合は、その箇所を**警告チップ**として描画対象に残す（握りつぶさない）。

### 3. コントラスト計算

on-color ペア（`on-primary`×`primary`、`on-surface`×`surface`、`on-error`×`error`、その他 `on-*` と対応するベース）について、色を sRGB 相対輝度へ変換しコントラスト比を計算する。

- 各ペアには「比率＋**達成した最上位レベルを1つだけ**示すバッジ」を添える（AA/AAA を両方並べない — 冗長で意味が伝わりにくい）。判定は下表のとおり:

  | コントラスト比 | バッジ | CSS クラス | 意味 |
  |---|---|---|---|
  | ≥ 7:1 | `AAA` | `pass` | 強化基準クリア（最良） |
  | 4.5–7:1 | `AA` | `pass` | 通常テキストの最低基準クリア |
  | 3–4.5:1 | `AA Large` | `warn` | 大文字（18px+/14px+bold）のみ可。本文は不足 |
  | < 3:1 | `Fail` | `fail` | 不足 |

### 4. テンプレへ転記

下の「固定 HTML テンプレ」を骨格に、解決済みの値を流し込む。

- front matter の全トークンを `:root` の CSS カスタムプロパティへ展開する。
- specimen の各節は、存在するトークンぶんだけ要素を繰り返して埋める（契約1・3）。
- 例画面は `var(--…)` 参照だけで組む（契約2）。

### 5. 書き出し

`preview.html` をプロジェクトルート（DESIGN.md の隣）に書き出す。既存があれば上書き。

### 6. 取りこぼし自己チェック

書き出し後に必ず確認する:

- [ ] front matter の全トークンが specimen に現れているか（契約1）
- [ ] 例画面に DESIGN.md に無い色・サイズ・角丸が現れていないか（契約2）
- [ ] 存在しないトークン群の節を捏造していないか（契約3）
- [ ] 本文の散文（Summary・各節・Do's and Don'ts）を該当節へ転記したか（契約4）
- [ ] 未解決 `{}` 参照を警告チップとして可視化したか
- [ ] on-color ペアに**1つだけ**の達成レベルバッジが付いているか

## 固定 HTML テンプレ（決定論の源）

このスケルトンを骨格として使う。`<!-- FILL: … -->` の箇所だけを DESIGN.md の値で差し込み、構造・クラス名・CSS の骨組みは変えない。トークンが無い節は丸ごと削除する（契約3）。`{{name}}` 等の二重波括弧はプレースホルダで、実値へ置換する。

```html
<!doctype html>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>{{name}} — preview</title>
<!-- FILL(任意): fontFamily が既知 Web フォントなら Google Fonts 等の <link> をここに。必ず system fallback を併記 -->
<style>
  :root {
    /* FILL: front matter の全トークンを CSS 変数へ展開
       colors:     --color-<token>: <解決済み literal>;
       typography: --type-<token>-family/-size/-weight/-line/-tracking: …;
       rounded:    --rounded-<level>: …;
       spacing:    --spacing-<key>: …;
       （未定義の群は出力しない） */
    --color-surface: /* FILL or fallback */ #fff;
    --color-on-surface: /* FILL or fallback */ #111;
  }
  * { box-sizing: border-box; }
  body {
    margin: 0; padding: 0;
    background: var(--color-surface, #fff);
    color: var(--color-on-surface, #111);
    font-family: system-ui, -apple-system, "Segoe UI", sans-serif;
    line-height: 1.5;
  }
  .pv-wrap { max-width: 1040px; margin: 0 auto; padding: 32px 24px 96px; }
  .pv-header { margin-bottom: 40px; }
  .pv-header h1 { margin: 0 0 4px; font-size: 28px; }
  .pv-header .pv-desc { opacity: .7; margin: 0 0 8px; }
  .pv-header .pv-meta { font-size: 12px; opacity: .5; font-family: ui-monospace, monospace; }
  section.pv-block { margin: 0 0 56px; }
  section.pv-block > h2 {
    font-size: 13px; letter-spacing: .08em; text-transform: uppercase;
    opacity: .55; border-bottom: 1px solid currentColor; padding-bottom: 6px; margin: 0 0 20px;
  }
  /* 本文の散文（DESIGN.md 文言の verbatim 転記） */
  .pv-prose { max-width: 760px; opacity: .85; font-size: 14px; line-height: 1.75; margin: 0 0 24px; white-space: pre-wrap; }
  .pv-prose.summary { padding: 16px 18px; border: 1px solid rgba(128,128,128,.2); border-radius: 10px; }
  .pv-prose ul { margin: 0; padding-left: 1.2em; }
  /* Colors */
  .pv-hue { margin-bottom: 12px; }
  .pv-hue .hue-label { font-size: 11px; opacity: .5; font-family: ui-monospace, monospace; margin-bottom: 4px; }
  .pv-ramp { display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px; }
  .pv-swatch { width: 96px; }
  .pv-swatch .chip { height: 56px; border-radius: 8px; border: 1px solid rgba(128,128,128,.25); }
  .pv-swatch .name { font-size: 11px; font-family: ui-monospace, monospace; margin-top: 4px; }
  .pv-swatch .val  { font-size: 10px; opacity: .55; font-family: ui-monospace, monospace; }
  .pv-roles { display: grid; grid-template-columns: repeat(auto-fill, minmax(220px,1fr)); gap: 12px; }
  .pv-role { border: 1px solid rgba(128,128,128,.2); border-radius: 8px; overflow: hidden; }
  .pv-role .pair { padding: 14px 12px; font-size: 13px; }       /* background:on-color, color:base text */
  .pv-role .meta { padding: 8px 12px; font-size: 11px; display: flex; justify-content: space-between; align-items: center; gap: 8px; }
  .badge { font-size: 10px; font-weight: 700; padding: 2px 6px; border-radius: 999px; }
  .badge.pass { background: #d8f5dd; color: #0a5a23; }
  .badge.fail { background: #fde2e2; color: #8a1414; }
  .badge.warn { background: #fff4d6; color: #7a5a00; }
  /* Typography */
  .pv-type { margin-bottom: 20px; }
  .pv-type .spec { font-size: 11px; opacity: .55; font-family: ui-monospace, monospace; margin-bottom: 4px; }
  /* Scales */
  .pv-scale { display: flex; flex-wrap: wrap; align-items: flex-end; gap: 16px; }
  .pv-bar { background: rgba(128,128,128,.25); height: 24px; }
  .pv-radius { width: 72px; height: 72px; background: rgba(128,128,128,.18); border: 1px solid rgba(128,128,128,.3); }
  .pv-scale .label { font-size: 11px; opacity: .6; font-family: ui-monospace, monospace; }
  /* Components / Example：トークン変数のみで構成（契約2） */
  .pv-components { display: flex; flex-wrap: wrap; gap: 16px; align-items: center; }
  .pv-example { border: 1px solid rgba(128,128,128,.2); border-radius: 12px; padding: 24px; }
</style>
<div class="pv-wrap">

  <header class="pv-header">
    <h1>{{name}}</h1>
    <p class="pv-desc">{{description}}</p>
    <p class="pv-meta">version: {{version}} · source: {{DESIGN.md のパス}}</p>
  </header>

  <!-- 0. Summary（本文 ## Summary があれば verbatim。無ければこの section を削除） -->
  <section class="pv-block">
    <h2>Summary</h2>
    <div class="pv-prose summary"><!-- FILL: ## Summary の本文を verbatim（箇条書きは <ul><li> で） --></div>
  </section>

  <!-- Overview（本文 ## Overview があれば verbatim。無ければ削除） -->
  <section class="pv-block">
    <h2>Overview</h2>
    <div class="pv-prose"><!-- FILL: ## Overview の散文を verbatim --></div>
  </section>

  <!-- 1. Colors -->
  <section class="pv-block">
    <h2>Colors</h2>
    <!-- FILL: primitive を色相プレフィックス（blue / ink / neutral / red …）ごとに .pv-hue で改行。
         各 .pv-hue 内は同一色相を階調順（明→暗）に並べる -->
    <div class="pv-hue">
      <div class="hue-label">{{hue}}</div>
      <div class="pv-ramp">
        <div class="pv-swatch"><div class="chip" style="background: var(--color-{{primitive}})"></div><div class="name">{{primitive}}</div><div class="val">{{literal}}</div></div>
      </div>
    </div>
    <div class="pv-roles" style="margin-top:20px">
      <!-- FILL: semantic ロールごとに 1 つ。on-color ペアは pair の背景/文字で対比し、比率＋達成レベルバッジを1つ添える -->
      <div class="pv-role">
        <div class="pair" style="background: var(--color-{{base}}); color: var(--color-{{on-base}})">{{base}} / {{on-base}}</div>
        <div class="meta"><span>{{ratio}}:1</span><span class="badge {{pass|warn|fail}}">{{AAA|AA|AA Large|Fail}}</span></div>
      </div>
      <!-- 未解決参照は: <span class="badge warn">未解決 {colors.xxx}</span> -->
    </div>
    <div class="pv-prose" style="margin-top:24px"><!-- FILL: 本文 ## Colors の散文を verbatim（無ければ削除） --></div>
  </section>

  <!-- 2. Typography -->
  <section class="pv-block">
    <h2>Typography</h2>
    <!-- FILL: typography レベルごとに 1 つ。実サイズでサンプル表示。サンプル文は固定文言を使う -->
    <div class="pv-type">
      <div class="spec">{{token}} · {{family}} {{size}}/{{line}} w{{weight}} tracking {{tracking}}</div>
      <div style="font-family: var(--type-{{token}}-family); font-size: var(--type-{{token}}-size); font-weight: var(--type-{{token}}-weight); line-height: var(--type-{{token}}-line); letter-spacing: var(--type-{{token}}-tracking);">これはサンプルテキストです Design System 0123</div>
    </div>
    <div class="pv-prose" style="margin-top:8px"><!-- FILL: 本文 ## Typography の散文を verbatim（無ければ削除） --></div>
  </section>

  <!-- 3. Spacing / Rounded（どちらか欠けたら該当ブロックを削除） -->
  <section class="pv-block">
    <h2>Spacing &amp; Rounded</h2>
    <div class="pv-scale" style="margin-bottom:24px">
      <!-- FILL: spacing の各値（unit の倍数など）を幅に反映したバー -->
      <div><div class="pv-bar" style="width: var(--spacing-{{key}})"></div><div class="label">{{key}} {{value}}</div></div>
    </div>
    <div class="pv-scale">
      <!-- FILL: rounded の各レベルを角丸矩形で -->
      <div><div class="pv-radius" style="border-radius: var(--rounded-{{level}})"></div><div class="label">{{level}} {{value}}</div></div>
    </div>
    <div class="pv-prose" style="margin-top:24px"><!-- FILL: 本文 ## Layout / ## Shapes の散文を verbatim（無ければ削除） --></div>
  </section>

  <!-- 4. Elevation & Depth（本文 §5 に深度の手段がある場合のみ。影/ボーダー/背景段差を見本化） -->
  <section class="pv-block">
    <h2>Elevation &amp; Depth</h2>
    <!-- FILL: DESIGN.md が採る深度手段（shadow / border / surface 段差 等）の見本を数点 -->
    <div class="pv-prose" style="margin-top:16px"><!-- FILL: 本文 ## Elevation & Depth の散文を verbatim（無ければ削除） --></div>
  </section>

  <!-- 5. Components -->
  <section class="pv-block">
    <h2>Components</h2>
    <div class="pv-components">
      <!-- FILL: components の atom を実レンダー。状態違い（-hover 等）も並置。スタイルは var(--…) 参照のみ -->
    </div>
    <div class="pv-prose" style="margin-top:16px"><!-- FILL: 本文 ## Components の散文を verbatim（無ければ削除） --></div>
  </section>

  <!-- 6. 例画面（既存トークンだけで組む。新値禁止＝契約2） -->
  <section class="pv-block">
    <h2>Example</h2>
    <div class="pv-example">
      <!-- FILL: カード＋フォーム＋ナビ程度の小さな代表 UI。色・サイズ・角丸・タイポはすべて var(--…) を参照 -->
    </div>
  </section>

  <!-- 7. Do's and Don'ts（本文にあれば verbatim。無ければ削除） -->
  <section class="pv-block">
    <h2>Do's and Don'ts</h2>
    <div class="pv-prose"><!-- FILL: 本文 ## Do's and Don'ts を verbatim（箇条書きは <ul><li>） --></div>
  </section>

</div>
```

## 出力フォーマット

```markdown
## preview-ui 生成結果

- 生成先: preview.html（DESIGN.md の隣）
- 反映トークン: 色 primitive N / semantic N、タイポ N レベル、rounded N、spacing N、components N
- コントラスト: on-color ペア N 組を検証（AAA N / AA N / AA Large N / Fail N）
- 未解決 `{}` 参照: N 件（あれば一覧）
- 省略した節: [トークン不在で省いたセクション]

### 確認方法
ブラウザで preview.html を開く（例: `start preview.html` / `open preview.html`）。

### 要確認事項
- [コントラスト fail の箇所 / 未解決参照など、人間の判断が要る項目]
```

## 注意

- **デザインしない**: 不足を埋めたくなっても新しい値を発明しない。欠落は欠落として（節を省略して）見せることが、DESIGN.md の不備を可視化する価値になる。
- **自己完結**: CSS はインライン、外部スクリプト・外部 CSS 依存なし。Web フォントの `<link>` だけは例外だが、必ず system fallback を併記しオフラインでもレイアウトが壊れないようにする。
- **検証は軽量に**: コントラスト合否と未解決参照の可視化に留める。本格的な準拠監査は `/audit-ui` の役割（重複させない）。
- **preview.html はユーザーのプロジェクトファイル**: 既存を上書きする前に、それが本スキル生成物でない疑いがあれば確認する。
- ダークモード・複数ビューポートの切替は対象外（front matter 定義を単一テーマで描画する）。

## 推奨される次のステップ

- `/audit-ui` — トークン準拠・コントラストを本格的に監査
- `/recolor-ui` — 見た目を見て配色を調整したくなった場合
- `/init-design` — preview を見て DESIGN.md 自体を差分更新
