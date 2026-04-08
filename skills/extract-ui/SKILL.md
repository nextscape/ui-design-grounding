---
name: extract-ui
description: UIからコンポーネント・デザイントークンを抽出する。繰り返し使われるパターンを再利用可能な単位に分解し、トークン化する。コンポーネント化・トークン抽出・デザインシステム整備を依頼されたときに使用する。
user-invocable: true
argument-hint: "[対象 (画面、コンポーネント、機能...)]"
---

# extract-ui

## 準備（MANDATORY PREPARATION）

ui-design-grounding スキルを呼び出し、以下のリファレンスを読み込む:

- `ui-design-grounding/reference/design-system.md`
- `ui-design-grounding/reference/design-tokens.md`
- `ui-design-grounding/reference/implementation.md`
- `ui-design-grounding/reference/color-system.md`
- `ui-design-grounding/reference/spatial-layout.md`

## 手順

### 1. パターンの特定

- **繰り返し出現する視覚パターン**: 3回以上使われている要素を候補とする
- **一貫性が必要な要素**: 同じ意味を持つが異なる実装になっている箇所
- **ハードコードされた値**: 色コード、余白、フォントサイズの直接指定

### 2. コンポーネント抽出

- **Atomic Design**: 適切な粒度を判断
  - Atoms: ボタン、入力、ラベル、アイコン
  - Molecules: フォームフィールド、カード、メニュー項目
  - Organisms: ヘッダー、フォームグループ、リスト
- **単一責務**: 1つのコンポーネントが1つの責務を持つ
- **Props設計**: 必要なバリエーションを Props で表現（意味のある区別のみ）
- **状態管理**: Local / Shared / Global の適切なスコープ

### 3. デザイントークン抽出

- **Primitive トークン**: 値そのもの（`--blue-500: oklch(0.6 0.15 250)`）
- **Semantic トークン**: 意味（`--color-primary: var(--blue-500)`）
- **Component トークン**: 用途特化（`--button-bg: var(--color-primary)`）

**抽出対象**:
- 色: text, surface, border, action, state（success/warning/error/info）, focus
- 余白: 4ptグリッドに基づくスケール
- 角丸: radius.sm, radius.md, radius.lg
- タイポグラフィ: font-size, font-weight, line-height, font-family
- シャドウ: shadow.sm, shadow.md, shadow.lg

### 4. 命名規則の統一

- `color.*`, `space.*`, `radius.*`, `font.*`, `shadow.*` の構造
- セマンティックトークンを優先使用
- 直接値（#xxx、16px等）の使用を例外に限定

### 5. 段階的な移行計画

- **見た目を変えない**: 抽出はリファクタリングであり、視覚的変更を伴わない
- **影響度順**: spacing → border → background → text → state の順で移行
- **検証**: 抽出前後で見た目が同一であることを確認

## 出力フォーマット

```markdown
## 抽出結果

### コンポーネント候補
| コンポーネント | 粒度 | 出現箇所 | Props |
|-------------|------|---------|-------|
| ... | Atom/Molecule/Organism | N箇所 | ... |

### トークン候補
| カテゴリ | トークン名 | 現在の値 | 使用箇所数 |
|---------|----------|---------|-----------|
| color | --color-primary | #xxx | N箇所 |
| space | --space-md | 16px | N箇所 |

### 移行計画
1. [Phase 1]: ...
2. [Phase 2]: ...
```

## 推奨される次のステップ

トークン・コンポーネント抽出後、以下のコマンドスキルの実行を検討する:
- `/init-design` — 抽出結果を DESIGN.md に反映
- `/audit-ui` — 抽出後の技術品質（トークン使用率等）を監査
