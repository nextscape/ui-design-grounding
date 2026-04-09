---
name: implement-ui
description: UI/UXデザインの意図を実装視点に翻訳・整理する。コンポーネント分解・責務整理・UI状態の洗い出し・実装構造の設計を行う。デザインからの実装・コンポーネント設計・実装プラン策定を依頼されたときに使用する。
user-invocable: true
argument-hint: "[デザイン、Figma、画面仕様...]"
---

# implement-ui

## 準備（MANDATORY PREPARATION）

ui-design-grounding スキルを呼び出し、以下のリファレンスを読み込む:

- `ui-design-grounding/reference/design-system.md`
- `ui-design-grounding/reference/design-tokens.md`
- `ui-design-grounding/reference/color-system.md`
- `ui-design-grounding/reference/implementation.md`
- `ui-design-grounding/reference/responsive-design.md`
- `ui-design-grounding/reference/motion-design.md`
- `ui-design-grounding/reference/typography.md`

## 入力

- デザイン（Figma 等）
- 実装対象（Web / Framework 等）
- 制約条件
- 既存のデザインシステム / トークン資産（有無）

## 手順

1. **デザイン意図の整理**: デザインが伝えたい情報階層・インタラクション・状態遷移を読み取る
2. **既存資産の確認**: 既存コンポーネント・CSS・トークン資産の再利用可否を確認する
3. **構成要素への分解**: UIをコンポーネント単位に分解する（Atomic Design: Atoms → Molecules → Organisms）
4. **コンポーネント責務の整理**: 各コンポーネントの責務（表示/ロジック/データ）を明確にする
5. **UI状態の洗い出し**: 各コンポーネントの状態を列挙する
   - Initial（初期表示）
   - Loading（読み込み中）
   - Success（成功）
   - Error（エラー）
   - Empty（データなし）
6. **実装上の注意点**: レスポンシブ、アニメーション、アクセシビリティ、パフォーマンスの観点で注意事項を整理する

## 出力フォーマット

```markdown
## デザイン意図の整理
- ...

## コンポーネント構成案
| コンポーネント | 粒度 | 責務 | 再利用元 |
|-------------|------|------|---------|
| ... | Atom/Molecule/Organism | ... | 既存/新規 |

## UI状態一覧
| コンポーネント | Initial | Loading | Success | Error | Empty |
|-------------|---------|---------|---------|-------|-------|
| ... | ... | ... | ... | ... | ... |

## 実装上の注意点
- ...

## 推奨される次のステップ
- `/audit-ui`（実装後の技術品質監査 — a11y・パフォーマンス・トークン準拠を確認）
- `/optimize-ui`（パフォーマンス最適化 — CWV・レンダリング・バンドルサイズ）
```

## 注意

- コードの最終決定は行わない（構造の整理に専念）
- 実装方針を勝手に変更しない
- 改善提案は行わず、デザイン意図の忠実な翻訳を優先する
