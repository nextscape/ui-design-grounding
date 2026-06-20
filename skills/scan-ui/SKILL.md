---
name: scan-ui
description: 外部の既存サイト(URL)を分析し、デザインシステムを逆算して DESIGN.md として出力する。Playwright で生の computed style・スクリーンショットを読み取り、色・グラデーション・タイポグラフィ（フォントソース判定含む）・余白・角丸・深度（シャドウ＋z-index）・モーション（ランタイム getAnimations 含む）・インタラクション状態（hover/focus 差分）・セマンティック領域・コンポーネントを抽出する。単一ページ／サイト全体（カバレッジ集計で統合）に加え、ダークモード・レスポンシブ（複数ビューポート）も任意でキャプチャ。競合・参考サイトのデザインを分析・取り込みたい、サイトの配色やフォントを調べたい、デザインを真似たい・リバースエンジニアリングしたいときに使用する。自分のコードの整理は extract-ui、自社デザイン憲法の新規定義は init-design を使う。
user-invocable: true
argument-hint: "[URL] [--site] [--dark] [--responsive] [--interactions] [--motion]"
---

# scan-ui

## 概要

外部の既存サイト（URL）を Playwright で分析し、デザインシステムを逆算して **DESIGN.md** として出力する。色・タイポグラフィ・余白といった「塗料」だけでなく、グラデーション・インタラクション状態の差分・ランタイムモーション・セマンティック領域といった「構造」まで読み取り、このプラグインのハブである DESIGN.md 出力に統合する。

**使い分け**:
- 外部 URL の分析 → 本スキル `scan-ui`
- 自分のコードの部品化・トークン化（移行計画） → `extract-ui`
- 自社デザイン憲法の新規定義・更新 → `init-design`

出力は**たたき台**であり、最終判断は人間に委ねる。他サイトの取り込みは参照・学習目的とし、商標・独自表現の流用は避ける。

## 準備（MANDATORY PREPARATION）

ui-design-grounding スキルを呼び出し、以下のリファレンスを読み込む:

- `ui-design-grounding/reference/design-md-spec.md`
- `ui-design-grounding/reference/design-tokens.md`
- `ui-design-grounding/reference/color-system.md`
- `ui-design-grounding/reference/typography.md`
- `ui-design-grounding/reference/spatial-layout.md`
- `ui-design-grounding/reference/interaction.md`
- `ui-design-grounding/reference/motion-design.md`
- `ui-design-grounding/reference/responsive-design.md`
- `ui-design-grounding/reference/anti-patterns.md`

## モードとキャプチャオプション

引数で分析範囲とキャプチャ内容を制御する。

**範囲モード**:
- **単一ページ**（デフォルト）: 指定 URL 1枚を分析する。
- **サイト全体**（`--site`）: 代表ページ（home / pricing / docs / blog 等）を巡回し、**カバレッジ**（各トークンが何%のページで使われるか）でトークンを選別。サイト共通値と page-local な外れ値を分離して統合する。

**追加キャプチャ**（必要に応じて重ねる。単体パスでは取れない構造を別パスで取得する）:

| フラグ | 取得内容 | 取得方法 |
|--------|---------|---------|
| `--motion` | ランタイムアニメーションの choreography（duration / easing / iterations） | `document.getAnimations()` を別パスで集計 |
| `--interactions` | hover / focus / active の状態差分（色・transform・shadow の delta） | 代表要素を `browser_hover`/`focus` してから再評価し静止状態と差分 |
| `--dark` | ダークモード変種パレット | `prefers-color-scheme: dark` をエミュレート（or サイトのダークトグル）して再抽出し light と対にする |
| `--responsive` | ブレイクポイントごとのレイアウト変形 | ビューポートを 320 / 768 / 1024 / 1280px にリサイズして再抽出し差分を記録 |

> `--motion` 無指定でも、単一パスの `transitionDuration` は常に拾う（静的トランジションの基礎値）。`--motion` は keyframes/spring を含む実際の動きの集計。

## 分析手段（優先順位・フォールバック）

抽出ロジック（下記の DOM walk スニペット）は**どの手段でも同一のもの**を流す（手段差で結果がぶれないように）。

1. **Playwright MCP（第一候補）** — `browser_navigate` でページを開き、`browser_evaluate` に下記スニペットを渡して単一パス抽出。追加パス（motion / interactions / dark / responsive）も `browser_evaluate`・`browser_hover`・`browser_resize` で実行。`browser_take_screenshot` で色・レイアウトを目視検証する。追加インストール不要。
2. **Playwright CLI（フォールバック）** — MCP が無い環境。`npx playwright` で自己完結スクリプトを実行（初回は `npx playwright install chromium`）。`--site` の複数ページ巡回や `--responsive`/`--dark` の `page.setViewportSize()`/`page.emulateMedia()` は for ループが堅牢で、ユーザーが後で再実行できる成果物にもなる。
3. **WebFetch / 手動貼り付け（最終手段）** — ブラウザが一切使えない環境。静的 HTML/CSS を取得、またはユーザーが貼り付けた computed style から推定する。グラデーション・状態差分・ランタイムモーションは取得不可。**精度が落ちること・取れない次元を出力に明示**する。

### DOM walk 抽出スニペット（共通・単一パス）

色・グラデーション・タイポグラフィ・フォントソース・余白・角丸・シャドウ・z-index・トランジション・セマンティック領域を1パスで回収する。CORS で読めないクロスオリジン stylesheet は try/catch でスキップする（フォントソース判定が誤検出しないように）。

```js
() => {
  const bump = (m, k) => { if (k && k !== 'none' && k !== 'normal') m.set(k, (m.get(k) || 0) + 1); };
  const top = (m, n = 12) => [...m.entries()].sort((a, b) => b[1] - a[1]).slice(0, n);

  const colors = new Map(), bgs = new Map(), gradients = new Map(),
        fonts = new Map(), sizes = new Map(), weights = new Map(),
        lineHeights = new Map(), letterSpacings = new Map(),
        radii = new Map(), shadows = new Map(), spaces = new Map(),
        transitions = new Map(), zIndexes = new Map();

  for (const el of document.querySelectorAll('body *')) {
    const r = el.getBoundingClientRect();
    if (r.width === 0 || r.height === 0) continue;       // 非表示要素は除外
    const s = getComputedStyle(el);
    bump(colors, s.color);
    bump(bgs, s.backgroundColor);
    if (s.backgroundImage && /gradient/.test(s.backgroundImage)) bump(gradients, s.backgroundImage);
    bump(fonts, s.fontFamily);
    bump(sizes, s.fontSize);
    bump(weights, s.fontWeight);
    bump(lineHeights, s.lineHeight);
    bump(letterSpacings, s.letterSpacing);
    bump(radii, s.borderRadius);
    bump(shadows, s.boxShadow);
    bump(spaces, s.padding);
    bump(spaces, s.gap);
    if (s.transitionDuration && s.transitionDuration !== '0s')
      bump(transitions, s.transitionDuration + ' ' + s.transitionTimingFunction);
    if (s.zIndex && s.zIndex !== 'auto') bump(zIndexes, s.zIndex);
  }

  // フォントソース判定: <link> と @font-face から（Google Fonts / 自己ホスト / システム）
  const fontSources = [];
  try {
    const links = [...document.querySelectorAll('link[href]')].map(l => l.href)
      .filter(h => /fonts\.(googleapis|gstatic)\.com/.test(h));
    if (links.length) fontSources.push('google-fonts');
    let faceCount = 0;
    for (const sheet of document.styleSheets) {
      let rules; try { rules = sheet.cssRules; } catch { continue; }   // CORS はスキップ
      if (!rules) continue;
      for (const rule of rules) {
        if (rule.type === 5 /* CSSFontFaceRule */) {
          faceCount++;
          const fam = rule.style && rule.style.getPropertyValue('font-family');
          if (fam) fontSources.push('@font-face:' + fam.trim());
        }
      }
    }
    if (!links.length && !faceCount) fontSources.push('system/self-hosted-unknown');
  } catch (e) { fontSources.push('error:' + e.message); }

  // セマンティック領域: ランドマーク + テキストヒューリスティック
  const regions = [];
  const tag = (sel, label) => { if (document.querySelector(sel)) regions.push(label); };
  tag('header,[role=banner]', 'header'); tag('nav,[role=navigation]', 'nav');
  tag('main,[role=main]', 'main'); tag('footer,[role=contentinfo]', 'footer');
  tag('aside,[role=complementary]', 'aside');
  const bt = (document.body.innerText || '').toLowerCase();
  for (const kw of ['pricing', '料金', 'testimonial', 'feature', 'faq', 'hero'])
    if (bt.includes(kw)) regions.push('hint:' + kw);

  return {
    url: location.href, title: document.title,
    viewport: { w: innerWidth, h: innerHeight },
    colorScheme: matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light',
    colors: top(colors), backgrounds: top(bgs), gradients: top(gradients, 8),
    fontFamilies: top(fonts), fontSources: [...new Set(fontSources)],
    fontSizes: top(sizes), fontWeights: top(weights), lineHeights: top(lineHeights),
    letterSpacings: top(letterSpacings), radii: top(radii), shadows: top(shadows),
    spacing: top(spaces, 16), transitions: top(transitions), zIndexes: top(zIndexes),
    regions: [...new Set(regions)],
  };
}
```

### 追加パス: ランタイムモーション（`--motion`）

実際に走っているアニメーションを集計し、`transition` 静的値では見えない choreography（stagger・反復）を取得する。

```js
() => {
  const m = new Map();
  for (const a of document.getAnimations()) {
    const t = a.effect && a.effect.getTiming ? a.effect.getTiming() : {};
    const k = `${t.duration}|${t.easing}|${t.iterations}`;
    m.set(k, (m.get(k) || 0) + 1);
  }
  return {
    animationCount: document.getAnimations().length,
    clusters: [...m.entries()].sort((a, b) => b[1] - a[1]).slice(0, 12),
  };
}
```

### 追加パス: インタラクション状態差分（`--interactions`）

代表的なインタラクティブ要素（CTA ボタン・カード・リンク）について、静止状態を記録 → `browser_hover`/`focus` で状態遷移 → 再評価して **delta**（背景色・文字色・transform・shadow の変化）を抽出する。:hover は JS で合成できないため、必ず実際のポインタ操作（MCP: `browser_hover` / CLI: `locator.hover()`）を挟む。

```js
// 状態スナップショット（静止/hover/focus それぞれで呼び、差分をとる）
(selector) => {
  const el = document.querySelector(selector);
  if (!el) return null;
  const s = getComputedStyle(el);
  return { bg: s.backgroundColor, color: s.color, transform: s.transform,
           boxShadow: s.boxShadow, opacity: s.opacity,
           transition: s.transitionProperty + ' ' + s.transitionDuration };
}
```

### 追加パス: ダークモード（`--dark`）/ レスポンシブ（`--responsive`）

- **ダークモード**: `prefers-color-scheme: dark` をエミュレート（CLI: `page.emulateMedia({ colorScheme: 'dark' })` / サイト独自トグルがあればそれをクリック）してから単一パススニペットを再実行し、light の結果と**対**にして light/dark の semantic ペアを作る。
- **レスポンシブ**: ビューポートを 320 / 768 / 1024 / 1280px に順次リサイズ（MCP: `browser_resize` / CLI: `page.setViewportSize()`）し、各幅で単一パススニペットを再実行。レイアウト・余白・表示要素の差分から**ブレイクポイント**と変形ルールを推定する。

### CLI スクリプト雛形（フォールバック時）

```js
// scan-extract.js — node scan-extract.js <url> [url2 ...]
const { chromium } = require('playwright');
const EXTRACT = /* 上記「単一パススニペット」の関数本体をここに貼る */ null;
const VIEWPORTS = [320, 768, 1024, 1280];
(async () => {
  const urls = process.argv.slice(2);
  const browser = await chromium.launch();
  const out = [];
  for (const url of urls) {
    const page = await browser.newPage();
    await page.goto(url, { waitUntil: 'networkidle' });
    const rec = { light: await page.evaluate(EXTRACT) };
    // --dark
    await page.emulateMedia({ colorScheme: 'dark' });
    rec.dark = await page.evaluate(EXTRACT);
    await page.emulateMedia({ colorScheme: 'light' });
    // --responsive
    rec.responsive = {};
    for (const w of VIEWPORTS) {
      await page.setViewportSize({ width: w, height: 900 });
      rec.responsive[w] = await page.evaluate(EXTRACT);
    }
    // --motion
    rec.motion = await page.evaluate(() => document.getAnimations().map(a => {
      const t = a.effect && a.effect.getTiming ? a.effect.getTiming() : {};
      return { duration: t.duration, easing: t.easing, iterations: t.iterations };
    }));
    out.push(rec);
    await page.close();
  }
  await browser.close();
  console.log(JSON.stringify(out, null, 2));
})();
```

## 手順

### 1. 取得

選択した手段で対象 URL（`--site` 時は代表ページ群）を開き、単一パススニペットで raw データを回収する。指定フラグに応じて追加パス（motion / interactions / dark / responsive）を実行。可能ならスクリーンショットも取得し目視検証に使う。

### 2. 正規化・クラスタリング

- **色**: `rgb()/rgba()` を集計し、近い色をクラスタリングして primitive パレット候補に。最頻の前景/背景から `on-*` ペアを推定。
- **グラデーション**: `linear/radial-gradient` を type・角度/位置・stops に分解。ブランドの象徴的な装飾なら本文 `## Colors` に明記する。
- **タイポグラフィ**: fontFamily × size × weight の組をレベル化（headline / body / label）。フォントソース判定（Google Fonts / 自己ホスト `@font-face` / システム）を本文に記録。
- **余白**: padding/gap の最頻値からベースユニット（4/8px 等）を推定。
- **角丸・シャドウ・モーション**: 最頻値をスケール化。
- **深度**: シャドウのスケールと z-index の階層をまとめ、`## Elevation & Depth` の素材にする。
- **状態差分（--interactions）**: hover/focus の delta を `components` の `<name>-hover` 等として記録。
- **ダーク（--dark）**: light/dark を対にして semantic ペアを作る。
- **レスポンシブ（--responsive）**: ビューポート差分からブレイクポイントを推定し `## Layout` の素材にする。

### 3. 2層トークンへ整理

`design-tokens.md` の役割語彙に従い、primitive（literal）→ semantic（参照）の2層へ。`design-md-spec.md` の書式に厳密に従う。

### 4. `--site` のカバレッジ選別

各トークンの「出現ページ率」を算出し、共通（例: ≥60%）を採用、外れ値は本文に注記。カバレッジ表を出力に併記する。

### 5. DESIGN.md 生成

`design-md-spec.md` の出力契約（front matter + 本文8セクション）に厳密準拠して DESIGN.md を生成。マッピング:

- グラデーション・color scheme → `## Colors`
- フォントソース → `## Typography`
- ブレイクポイント・変形 → `## Layout`
- シャドウ＋z-index → `## Elevation & Depth`
- 角丸 → `## Shapes`
- 状態差分 → `## Components`（front matter `components` の `-hover` 等）
- ランタイムモーション → 末尾の追加セクション `## Motion`（spec の「順序の最後に置けば保持される」に従う）

全色を sRGB 変換し WCAG コントラストを検証する。

## 出力フォーマット

```markdown
## scan-ui 分析結果

- 対象: [URL]（範囲: 単一ページ / サイト全体 N ページ）
- キャプチャ: [単一パス + motion / interactions / dark / responsive のうち実行したもの]
- 分析手段: [Playwright MCP / CLI / WebFetch]
- 色トークン: primitive N色 / semantic N色（グラデーション M件）
- タイポグラフィ: N レベル（主フォント: [名] / ソース: [Google Fonts / 自己ホスト / システム]）
- spacing: ベース [unit]px
- 深度: シャドウ N段 / z-index 階層 [要約]
- rounded / motion: [要約]

### カバレッジ（--site 時のみ）
| トークン | 値 | 使用ページ率 |
|---------|----|------------|
| ... | ... | NN% |

### ダークモード（--dark 時のみ）
| ロール | Light | Dark |
|-------|-------|------|
| surface | #xxx | #yyy |

### レスポンシブ（--responsive 時のみ）
| ブレイクポイント | 変形の要点 |
|---------------|-----------|
| <768px | [例: ナビが下部タブ化 / 余白 16px] |

### インタラクション状態（--interactions 時のみ）
| 要素 | 静止 → hover/focus の delta |
|------|---------------------------|
| button-primary | bg #xxx→#yyy / transform translateY(-2px) |

### 生成物
- DESIGN.md を [パス] に出力（design.md alpha 準拠）

### 要確認事項
- [人間の判断が必要な項目／低精度だった項目／CORS で取れなかったフォントソース等]

### 注意
- 取り込みは参照・学習目的。商標・独自表現の流用は避ける。
```

DESIGN.md 本体は `reference/design-md-spec.md` のフォーマットに従って別途生成し、プロジェクトルート（または指定パス）に書き出す。

## よくある落とし穴

- **フォントソースの誤判定**: クロスオリジンの `@font-face` は CORS で `cssRules` が読めない。自己ホストでも `system/self-hosted-unknown` になりうるため、ネットワークタブ／`<link>`／スクショの書体から人間が最終確認する（実例: Stripe の `sohne-var` は自己ホストだが CORS で検出されない）。
- **:hover を JS で合成しない**: `el.matches(':hover')` や疑似的なクラス付与では実際の hover スタイルは取れない。必ず実ポインタ操作（`browser_hover`/`locator.hover()`）を挟んでから再評価する。
- **ランタイムモーションのタイミング**: エントランスアニメは初回ロード時に終わっていることが多い。`getAnimations()` は遷移直後・スクロール後に取ると拾える。
- **gap と padding の混在**: 単一パスでは両方を `spacing` に集約している。ベースユニット推定時はレイアウト用 gap と内部 padding を分けて見る。

## 推奨される次のステップ

- `/audit-ui` — 取り込んだ DESIGN.md に対する技術品質監査（A–F 的なスコアリングはこちらが担当）
- `/recolor-ui` — 取り込んだパレットを自社ブランド primary に置き換え
- `/init-design` — 生成された DESIGN.md の差分更新・調整
