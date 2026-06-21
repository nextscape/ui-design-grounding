# 変更履歴

本プロジェクトの主な変更点を記録する。バージョンは [プラグインマニフェスト](.claude-plugin/plugin.json) に準拠する。

## [1.2.1] - 2026-06-21

- `preview-ui` を追加（DESIGN.md をほぼ機械的に preview.html へ反映し、ブラウザで視覚確認）
- `scan-ui` / `init-design` / `recolor-ui` の「推奨される次のステップ」に `/preview-ui` を追加
- コマンドスキルを 22 件へ拡充

## [1.1.0] - 2026-06-21

- `scan-ui` を追加（外部サイト URL を分析し DESIGN.md を逆算生成）
- `recolor-ui` を追加（OKLCH による破綻のない再配色）
- `refine-ui` を新設し、スキル群を「5動詞・2層構造」へ再編
- 共通の DESIGN.md ゲート（前段／後段）を導入し、各観点スキルへ反映
- DESIGN.md にサマリ層・トークン外出し・スリム化運用を追加
- リファレンスを design.md 仕様準拠へリファクタ（語彙の SSOT 統一）
- コマンドスキルを 20 件へ拡充

## [1.0.4] - 2026-04-11

- `arrange-ui`・`spatial-layout`・`init-design` にレイアウト基準を追記

## [1.0.3] - 2026-04-09

- 各スキルの推奨アクションを優先度順に提示

## [1.0.1] - 2026-04-09

- `init-design` の DESIGN.md テンプレートを design.md 標準へ準拠
- レビュー指摘の修正・最適化、README 追加

## [1.0.0] - 2026-04-08

- 初版リリース（UI Design Grounding プラグイン）
