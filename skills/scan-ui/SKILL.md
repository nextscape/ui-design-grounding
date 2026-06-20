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

- `ui-design-grounding/reference/design-md-spec.md`
- `ui-design-grounding/reference/design-tokens.md`
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
