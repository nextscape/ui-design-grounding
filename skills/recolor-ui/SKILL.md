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
