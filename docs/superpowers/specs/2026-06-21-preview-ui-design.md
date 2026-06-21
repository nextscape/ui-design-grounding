# /preview-ui 設計仕様

- **日付**: 2026-06-21
- **対象**: `ui-design-grounding` プラグインへの新規コマンドスキル `preview-ui` の追加
- **ステータス**: 設計合意済み（実装計画はこの後 writing-plans で作成）

## 1. 目的

DESIGN.md（YAML front matter のデザイントークン＋散文本文）を **ほぼ機械的に** `preview.html` へ変換し、ブラウザで開いて視覚的に把握できるようにする。

主たるユースケースは、`scan-ui` / `init-design` / `recolor-ui` が提案・更新した DESIGN.md を、テキストのトークン羅列ではなく**実際の見た目**として確認すること。

### 性格（最重要原則）

AI は「デザインしない」。DESIGN.md のトークンを固定テンプレの定位置へ**転記**するだけ。創造的判断を持ち込まない。

## 2. 機械的転記契約

`preview.html` 生成が満たすべき不変条件:

1. **全トークン露出**: front matter の全トークン（colors / typography / rounded / spacing / components）は specimen のどこかに必ず現れる。取りこぼしはバグとして扱う。
2. **既存トークンのみで例画面を組む**: 末尾の「例画面」は既存トークンの参照（CSS カスタムプロパティ経由）だけで構成する。新しい色・サイズ・角丸・字間などの値を一切発明しない。
3. **欠落は省略（捏造しない）**: DESIGN.md に存在しないトークン群に対応する節は出力から省略する。デフォルト値で埋めない。
4. **決定論**: 同一の DESIGN.md からはほぼ同一の HTML が生成される。スキル本文に固定の HTML 骨格＋CSS 構造を全文掲載し、AI は値の差し込みのみを行う。

## 3. 出力物

- 生成先: プロジェクトルートの `preview.html`（DESIGN.md の隣）。既存があれば上書き。
- **自己完結 HTML**: CSS はインライン（`<style>`）。外部スクリプト・外部 CSS への依存なし。
- **フォントのみ例外**: `fontFamily` が既知の Web フォント（Google Fonts 等）の場合は `<link>` を `<head>` に入れる。必ずシステムフォント fallback を CSS で併記し、オフラインでもレイアウトが壊れないようにする。
- **トークンの CSS 変数化**: front matter のトークンを `:root { --primary: …; --body-md-size: …; … }` へ展開し、specimen も例画面もこの変数を参照する。これにより HTML が DESIGN.md と完全連動する。
- `{}` 参照（例 `{colors.primary}`）はトークン解決して実値を CSS 変数に流し込む。解決規則は `design-md-spec.md` に従う（参照先は leaf primitive）。

## 4. 画面構成（上から順）

| # | 節 | 内容 |
|---|----|------|
| 0 | ヘッダ | `name` / `description` / `version`、生成元 DESIGN.md のパス |
| 1 | Colors | primitive ランプ（ヒュー×階調で整列）→ semantic ロール。各 on-color ペアにコントラスト比＋WCAG **AA/AAA バッジ**。未解決 `{}` 参照は警告チップ |
| 2 | Typography | 各レベルを実サイズでサンプル表示。脇に family / size / weight / line-height / letter-spacing を明記 |
| 3 | Spacing / Rounded | ベースユニット倍数バー、角丸スケールの矩形見本 |
| 4 | Elevation & Depth | 影／ボーダー／背景段差の見本（本文 §5 Elevation & Depth に準拠） |
| 5 | Components | front matter `components` の atom を実レンダー。状態違い（`-hover` 等）も並置 |
| 6 | 例画面 | 上記トークンだけで組んだ小さな代表 UI（カード＋フォーム＋ナビ程度）を1つ |

存在しない節（例: `components` が空なら §5）は省略する（契約3）。

## 5. 軽量検証表示

- **コントラスト**: on-color ペア（`on-primary`×`primary`、`on-surface`×`surface` 等）について、sRGB 変換後のコントラスト比を計算し、本文 4.5:1（AA）/ 7:1（AAA）、大文字 3:1 の基準で合否バッジを付す。計算規則は `color-system.md` に準拠。
- **未解決参照**: `{}` 参照が解決できない（参照先が無い／group を指している）場合は該当箇所に警告チップを出す。
- 検証は specimen に軽く添えるのみ。本格的な準拠監査は `audit-ui` の役割（重複させない）。

## 6. スキル構造

リポジトリ慣習に準拠:

- ファイル: `skills/preview-ui/SKILL.md` 単一ファイル。
- front matter: `name` / `description` / `user-invocable: true` / `argument-hint: "[対象 DESIGN.md]"`。
- 本文: 日本語。
- **MANDATORY PREPARATION**: `ui-design-grounding` を呼び、以下を読む。
  - `reference/design-md-spec.md` — front matter 書式・`{}` 解決規則・本文8セクション
  - `reference/color-system.md` — sRGB 変換・コントラスト計算
- 本文に**固定 HTML テンプレと差し込みルール**を全文掲載（決定論の源）。
- 手順:
  1. DESIGN.md 読込 → front matter パース → `{}` 解決
  2. on-color ペアのコントラスト計算
  3. 固定テンプレへトークンを転記（存在する節のみ）
  4. `preview.html` 書出し
  5. 取りこぼし自己チェック（全トークンが露出したか、新値を発明していないか）

## 7. 連携

- `scan-ui` / `init-design` / `recolor-ui` の「推奨される次のステップ」に `/preview-ui`（DESIGN.md を視覚確認）を1行追加。既存スキルの改変はこの1行に留める。
- `CLAUDE.md` のコマンドスキル早見表に1行追加（カテゴリ案: 評価 もしくは新カテゴリ「可視化」）。
- `ui-help` スキルの一覧に反映。

## 8. スコープ外（YAGNI）

- ダークモード／レスポンシブの複数ビューポート切替。MVP は front matter 定義をそのまま単一テーマで描画する。
- 自動リロード・ライブサーバ・ファイル監視。
- 例画面の複数バリエーション。代表 UI は1つに絞る。

## 9. 受け入れ基準

- 仕様例（`design-md-spec.md` の Aurora Notes 抜粋）を入力すると、全トークンが現れた `preview.html` が生成され、ブラウザで開くとレイアウトが破綻しない。
- on-color ペアにコントラストバッジが表示される。
- 例画面に DESIGN.md に無い色・サイズが現れない（CSS 変数参照のみ）。
- `components` が空の DESIGN.md では §5 が省略される。
