# Playwright MCP による実地観察の勘所

UI を「動いている実物」として観察するための共通リファレンス。評価系（`audit-ui` / `score-ui` / `legibility-ui`）と修正系（`polish-ui` / `guard-ui` / `adapt-ui`）が MANDATORY PREPARATION で読み込み、タスク固有の観察ステップだけを各スキル側に持たせる。ここには「どのツールで・何を・どう観るか」の機械的な手順を集約する。

## なぜ MCP か（CLI を使わない理由）

観察系タスクの本質は **「見る → 判断する → 次に何を見るか決める」ループ**にある。次の一手が直前の観察結果に依存するため、事前にスクリプト化できない。Playwright MCP は各ツール呼び出しの戻り値（スクリーンショット・アクセシビリティツリー・評価結果）をその場で受け取り、次の操作を Claude が判断できる。CLI スクリプトに落とすと「何を見るか」を全部決め打ちする羽目になり、想定外の不具合・崩れ（＝観察の価値そのもの）を取りこぼす。

- **本リファレンスを読むスキルでは Playwright MCP を必須とし、CLI フォールバックは用意しない。** 使えない場合は判定精度が大きく損なわれるため、その旨を出力して終了する。
- 例外は `scan-ui` の**一括抽出**（N ページ × 複数ブレイクポイントの決め打ちループ）。あれは適応ループではなくバッチ処理なので CLI が有利であり、本リファレンスの対象外。`scan-ui` は自前の手順に従う。

MCP ツールは `mcp__plugin_playwright_playwright__*`（`browser_navigate` / `browser_snapshot` / `browser_take_screenshot` / `browser_evaluate` / `browser_hover` / `browser_press_key` / `browser_click` / `browser_resize` 等）。

## 準備（共通）

各スキルの観察フェーズに入る前に、必ず以下を満たす。

1. **MCP 利用可否の確認** — Playwright MCP（`mcp__plugin_playwright_playwright__*`）が使えることを確認する。使えなければ**その旨を出力して終了**する（代替手段は判定精度を大きく損なうため用意しない）。
2. **アプリ起動** — 対象アプリが起動していなければ、プロジェクトにアプリ起動用のスキルがあればそれを使う。なければユーザーに起動方法を確認する。
3. **スクリーンショット保存先** — 評価系（`audit-ui` / `score-ui` / `legibility-ui`）では `ui-report.md` に従い、評価対象プロジェクトの `.design/reports/YYYY-MM-DD/screenshots/` に保存する。ファイル名は `HHmmss-<skill>-NN.png` とし、レポート本文から相対リンクする。修正系ではプロジェクトの一時ファイル置き場の慣例（`.gitignore` が定める一時ディレクトリ等）に保存する。いずれもファイル名だけの指定は避け、**明示的なパス**を指定する。

## ツール選択マトリクス

判定の性質でツールを使い分ける。**画像取得は高コストなので、機能・構造の確認だけで済むなら `browser_snapshot`・`browser_evaluate` を優先する。**

| 判定の性質 | 使うツール | 理由 |
|---|---|---|
| **視覚判定** — 余白・色の重み・整列・視覚階層・状態の見た目・レイアウトの崩れ | `browser_take_screenshot` | レンダリング結果そのものが必要。テキスト表現では潰れる |
| **構造・機能判定** — 要素の存在・ラベル・ロール・状態遷移・アクセシビリティツリー | `browser_snapshot` | a11y ツリーのテキスト表現で足りる。画像は不要 |
| **実測** — computed style・寸法・コントラスト比・アニメーション対象プロパティ | `browser_evaluate` | 目視では正確な数値が取れない。カスケード適用後の実効値を測る |
| **状態のトリガ** — hover / focus / click 等を実際に発火 | `browser_hover` / `browser_press_key` / `browser_click` | :hover 等は JS で合成できず、実際のポインタ/キー操作が要る |
| **ビューポート変形** — ブレイクポイントごとの再観察 | `browser_resize` | 単一幅では判定できない |

## 効率化 — 安く広く拾い、高い観察は絞る

観察コストは **往復回数** と **画像トークン** で決まる。全画面・全状態・全幅を一律に screenshot すると急激に膨らむ。**安い手段で候補を広く回収し、高い手段は確認だけに絞る。**

| 手段 | コスト | 使いどころ |
|---|---|---|
| `browser_evaluate` | 安い（テキスト/JSON・1往復で複数計測を一括） | **最優先**。機械的に測れる指摘候補をまとめて回収 |
| `browser_snapshot` | 安い（a11y ツリーのテキスト） | 構造・ラベル・ロール・状態の確認 |
| `browser_take_screenshot` | 高い（画像トークン大） | 視覚判断が要るものの**確認だけ**に使う。全画面より要素スコープを優先 |

### 指摘のトリアージ（3段）

1. **一括スイープ（安い・広い）** — 下記スイープを1回の `browser_evaluate` で流し、機械判定できる指摘候補をまとめて回収する。`browser_snapshot` でラベル/ロール/状態を確認。**この段では画像を撮らない。**
2. **候補の選別** — 返った候補を重篤度で並べる。機械判定できたもの（コントラスト比・寸法・オーバーフロー等）はこの時点でほぼ確定でよい。
3. **確認観察（高い・絞る）** — 視覚判断が要る・曖昧・高重篤度のものだけ screenshot / 状態トリガ / 複数幅で確認する。**全要素・全状態・全幅を一律に撮らない。**

観察範囲も絞る — **差分駆動**（変更した／対象の画面・コンポーネントだけを観る）、**代表サンプリング**（繰り返し要素は代表1つを観て、残りが同一コンポーネントかはコードで確認）、**状態は必要なものだけ**発火（存在しない・関係ない状態は飛ばす）。

> スイープで拾えるのは「測れる」指摘（コントラスト・タッチターゲット・オーバーフロー・未ラベル・レイアウトを揺らすアニメ）。視覚階層・アフォーダンス・初見の分かりやすさといった「判断」の指摘は依然として目視が要るが、トリアージにより**目視は少数の要確認箇所に集約**できる。

### 一括監査スイープ（1往復で候補を回収）

`browser_evaluate` にこの関数を渡すと、機械的に測れる指摘候補を1回でまとめて返す。コントラスト・タッチターゲット・アニメ等の個別計測はこれに統合してある。返り値の各配列が、そのまま（セレクタ付きの）指摘候補になる。

```js
() => {
  const parse = s => (s.match(/[\d.]+/g) || []).slice(0, 3).map(Number);
  const lum = c => { const a = c.map(v => { v /= 255; return v <= 0.03928 ? v / 12.92 : ((v + 0.055) / 1.055) ** 2.4; }); return 0.2126 * a[0] + 0.7152 * a[1] + 0.0722 * a[2]; };
  const effBg = el => { for (let n = el; n; n = n.parentElement) { const b = getComputedStyle(n).backgroundColor; if (b && b !== 'transparent' && !/,\s*0\)$/.test(b)) return parse(b); } return [255, 255, 255]; };
  const cr = (f, b) => { const [x, y] = [lum(f), lum(b)].sort((p, q) => q - p); return (x + 0.05) / (y + 0.05); };
  const path = el => { let s = el.tagName.toLowerCase(); if (el.id) s += '#' + el.id; else if (typeof el.className === 'string' && el.className.trim()) s += '.' + el.className.trim().split(/\s+/)[0]; return s; };
  const vw = innerWidth;
  const out = { contrast: [], smallTargets: [], unlabeled: [], clippedX: [], layoutAnim: [], overflowX: document.documentElement.scrollWidth > vw + 1 };
  for (const el of document.querySelectorAll('body *')) {
    const r = el.getBoundingClientRect(); if (!r.width || !r.height) continue;
    const s = getComputedStyle(el);
    if ([...el.childNodes].some(n => n.nodeType === 3 && n.textContent.trim())) {   // テキストを直接持つ要素だけ
      const size = parseFloat(s.fontSize), need = (size >= 24 || (size >= 18.66 && +s.fontWeight >= 700)) ? 3 : 4.5;
      const ratio = cr(parse(s.color), effBg(el));
      if (ratio < need) out.contrast.push({ el: path(el), ratio: +ratio.toFixed(2), need });
    }
    if (r.right > vw + 1) out.clippedX.push(path(el));   // ビューポート右端から溢れている
  }
  for (const el of document.querySelectorAll('button, a, [role="button"], input, select, textarea')) {
    const r = el.getBoundingClientRect(); if (!r.width || !r.height) continue;
    if (r.width < 44 || r.height < 44) out.smallTargets.push({ el: path(el), w: Math.round(r.width), h: Math.round(r.height) });
    const label = (el.textContent || '').trim() || el.getAttribute('aria-label') || el.getAttribute('title') || el.getAttribute('placeholder');
    if (!label) out.unlabeled.push(path(el));   // 名前のないコントロール
  }
  for (const a of document.getAnimations())                    // レイアウトを揺らす（ジャンク要因の）アニメ
    for (const k of (a.effect?.getKeyframes() || []))
      for (const p of Object.keys(k))
        if (!['transform', 'opacity', 'offset', 'composite', 'easing'].includes(p)) out.layoutAnim.push(p);
  out.layoutAnim = [...new Set(out.layoutAnim)];
  return out;
};
```

### フォーカス可視性（フォーカス後に個別確認）

`:focus-visible` はフォーカスの当たった要素にしか出ないため、スイープには含めない。`browser_press_key`（Tab）で当ててから確認する。何も変化しない要素はフォーカス不可視。

```js
() => { const el = document.activeElement; const s = getComputedStyle(el);
  return { tag: el.tagName, outline: s.outlineStyle !== 'none' && parseFloat(s.outlineWidth) > 0, shadow: s.boxShadow !== 'none' }; }
```

## ビューポート / メディア変種

| 目的 | 方法 |
|---|---|
| レスポンシブ | `browser_resize` で 320 / 768 / 1024 / 1280px に順次リサイズし、各幅で**一括監査スイープを流す（安い）**。`clippedX` / `overflowX` / `smallTargets` が出た幅**だけ** screenshot で崩れを確認する |
| ダークモード | `prefers-color-scheme: dark` をエミュレート（またはサイトのダークトグル）して再観察 |
| モーション低減 | `prefers-reduced-motion: reduce` をエミュレートし、アニメーションが無効化/クロスフェード代替されるか確認 |
| ハイコントラスト | `forced-colors: active` 相当で `forced-colors` メディアクエリ対応を確認 |

## 観察の規律（スキルの型で使い分ける）

### 評価系（`audit-ui` / `score-ui` / `legibility-ui`）— 観察先行

**採点・判定は観察した実物を根拠にする。ソースからの推測で埋めない。** まず一括監査スイープ（→ 効率化）で機械的な指摘候補を回収し、screenshot は要確認箇所だけに絞る。特に `legibility-ui` は「先入観を排すためコードを読む前に観察する」順序を厳守する（`legibility.md` 参照）。`audit-ui` の a11y・レスポンシブ次元はスイープで数値を取り、`score-ui` のペルソナ・レッドフラグテストは実際に操作して再現する（エラーを起こす・初回導線を踏む・Tab で辿る）。

### 修正系（`polish-ui` / `guard-ui` / `adapt-ui`）— 検出 → 修正 → 再観察

1. **検出** — まずスイープ＋`browser_snapshot` で候補を洗い、視覚判断が要るものだけ状態を発火／長文を注入／ビューポートを変えて確認する。「そのはず」で修正対象を決めない。
2. **修正** — コードを直す。
3. **再観察** — **同じ観察（多くはスイープの再実行）で、修正が実際にレンダリング結果へ反映されたことを確認する。** 直しただけで確認しないのは未完了とみなす。

## アンチパターン

- 安い一括スイープを飛ばして、いきなり全画面・全状態・全幅を screenshot で観察する（往復と画像トークンを浪費する）。
- スクリーンショット/操作を撮らずに「実装上そう書いてあるから大丈夫なはず」で状態・見た目を判定する。
- :hover / :focus-visible を JS（`el.matches(':hover')` 生成やクラス付与）で擬似再現し、実操作を省く。
- 単一ビューポートだけを見てレスポンシブの可否を判定する。
- 修正系で、直した後に再観察せず「修正済み」と報告する。
- 適応的観察タスクを CLI スクリプトに落とし、「次に何を見るか」を決め打ちして観察ループを殺す。
