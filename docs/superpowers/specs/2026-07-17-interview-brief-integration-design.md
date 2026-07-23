# インタビュー駆動の要件明確化と DESIGN_BRIEF 統合 設計仕様

- **日付**: 2026-07-17
- **対象**: `design-ui` / `init-design` / `implement-ui` へのインタビュー機構と機能単位ブリーフ（DESIGN_BRIEF.md）の統合、および UI 作業生成物の `.design/` への出力統合
- **ステータス**: 設計合意済み（実装計画はこの後 writing-plans で作成）
- **改訂（2026-07-17 実装後）**: 成果物の名称を DESIGN_BRIEF.md（デザインブリーフ）→ **FEATURE_DESIGN.md（機能設計）** へ、リファレンス design-brief.md → **feature-design.md** へ改名した。また、配布物（README・CHANGELOG・スキル本文）への着想元クレジットの記載は行わない方針に変更した。本文中の旧名称・クレジット記述は合意当時の記録としてそのまま残す。
- **改訂2（2026-07-17）**: 実装ワークフローとの接続を追加。`design-ui` に手順9「実装への受け渡し」（固有の実装ワークフローがあれば FEATURE_DESIGN.md を設計入力として渡す。判定材料には、明記が無くても導入済みの実装プロセス系スキルへの暗黙の期待を含め、迷えばユーザーに確認）、`implement-ui` に実装範囲の判定（UI に閉じない実装では UI 部分のみを担う。接続コード = API 呼び出し・状態管理・モックまで、サーバー側実装・スキーマ変更は範囲外）を追加した。観点（本プラグイン）とプロセス（プロジェクトのワークフロー）は直交し、本プラグインは実装プロセス全体を所有しない。

## 1. 背景・目的

**design-brief**（[julianoczkowski/designer-skills](https://github.com/julianoczkowski/designer-skills)、Apache-2.0）の成果物化の考え方と、依存関係を順に解くインタビュー手法を本プラグインに統合する。外部資料の丸ごと移植ではなく、既存の2層アーキテクチャ・DESIGN.md 常駐の設計思想に合わせて日本語で再設計する。

現状の課題:

1. `design-ui` は要件を「受け取る前提」で、ユーザーから聞き出す工程が明文化されていない
2. `design-ui` の成果物はチャット応答のみで永続化されず、`implement-ui` への受け渡しがファイルとして存在しない
3. `init-design` の Step 1 が「不明な点は仮置き + 明示」で、聞く手順がないまま機械的な DESIGN.md が生成されうる。値は抽出できても**意図（散文の設計指針）が薄い DESIGN.md は憲法として機能しない**
4. UI 作業の生成物の出力先が分散している（評価レポート = `ui-reports/`、`preview.html` = プロジェクトルート）

## 2. 取り込む核心: インタビュー手法（`reference/interview.md` 新設）

インタビューの5原則を新リファレンス `interview.md` に一元化し、`init-design` / `design-ui` の両方が MANDATORY PREPARATION で参照する:

1. **決定木を枝ごとに** — 設計の決定木を枝ごとに下り、決定間の依存関係を1つずつ解決する
2. **1問ずつ** — 質問は1つずつ提示する（複数同時に聞くと混乱させる）
3. **調べれば分かることは聞かない** — 調べる先: コードベース / DESIGN.md / 既存ブリーフ / 既存コンポーネント・トークン
4. **推奨回答を添える** — 各質問に自分の推奨回答と理由を添える
5. **共通理解まで確定しない** — 共通理解に達したことを確認してから成果物の生成に進む

エスケープ: ユーザーが「お任せ」「急ぐ」と表明した場合は、残る質問を推奨回答で埋めて進み、確定していない判断を成果物の「要確認事項」に明示する。

## 3. インタビュー発動判定（対称形）

| スキル | ヒアリング単位 | 成果物 | 発動判定 |
|---|---|---|---|
| `init-design` | システム全体 | DESIGN.md | 既存コードからの**抽出確度** |
| `design-ui` | 機能単位 | DESIGN_BRIEF.md | 要件の**明確化状況** |

- `init-design`: 既存資産が豊富 → 値は抽出し、**コードから読めない意図項目**（現状追認でよいか・感情トーン・参照/アンチ参照・ブランド制約）だけ質問する。薄い/無い → 本格ヒアリング
- `design-ui`: 要件が明確（画面・ユーザー・成功条件が特定できる）→ インタビュー省略可。曖昧・複数解釈がある → インタビュー実施
- **仮置きは「質問しても確定しなかった場合の最後の手段」に降格する**（現行 `init-design` の「不明な点は仮置き + 明示」を置き換える）

## 4. 質問の帰属振り分け

| 帰属 | 質問 | 聞くスキル |
|---|---|---|
| **DESIGN.md**（恒久・システム全体） | 感情トーン / 参照・アンチ参照プロダクト / ブランド制約 / 対象デバイス / アクセシビリティ基準 | `init-design` |
| **DESIGN_BRIEF.md**(機能単位) | 主要ユーザーと JTBD / 成功の定義 / コンテンツ（実データ/プレースホルダ）/ 機能固有の制約 / スコープ外 | `design-ui` |

`design-ui` のインタビューでは、**DESIGN.md が既に答えを持つ質問はスキップする**（原則3の「調べる先」に DESIGN.md 自体が含まれる）。これにより DESIGN.md 常駐の設計思想とインタビュー原則が接続する。

## 5. フロー全体像

```text
design-ui 起動
→ 前段ゲート: DESIGN.md 不在 → init-design へ委譲
    → 抽出確度を判定 → システム全体ヒアリング → 意味のある DESIGN.md
→ design-ui に復帰: 要件の明確化状況を判定 → 不明瞭なら機能単位インタビュー
    （DESIGN.md・コードが答えを持つ質問はスキップ）
→ .design/<feature-slug>/DESIGN_BRIEF.md 生成（＋恒久的決定の昇格検出）
→ implement-ui: DESIGN.md（恒久基準）＋ ブリーフ（機能判断）を読んで実装
```

前段ゲートの拡張:

- **設計・実装の入口**（`design-ui` / `implement-ui`）は、DESIGN.md 不在時に「`/init-design` の提案」に留めず **`init-design` へ委譲する**
- 委譲時は「これから基準づくりの質問を数問する」と宣言してから始める
- ユーザーが急ぐ場合は**最小構成 DESIGN.md**（Summary + 主要トークン + 聞けた範囲の意図）で切り上げられる逃げ道を用意する
- 評価系（`audit-ui` / `score-ui` / `legibility-ui`）は従来通り提案止まり（変更しない）
- `design-md-gate.md` にはこの区分の存在を1行注記し、動作の詳細は各スキル本文に書く（参照は載せるだけ・ゲートの使い方は本文に書く、の規約に従う）

## 6. DESIGN_BRIEF.md の仕様（`reference/design-brief.md` 新設）

- **保存先**: `.design/<feature-slug>/DESIGN_BRIEF.md`。`<feature-slug>` は機能・画面から導いた小文字ハイフン区切りの短い名前（例: `onboarding-flow`、`settings-page`）。機能ごとにサブフォルダを分け、複数回実行しても過去の成果物を上書きしない
- **DESIGN.md との関係**: DESIGN.md = プロジェクト恒久の視覚的憲法 / DESIGN_BRIEF.md = 機能単位の設計判断の記録。**視覚基準（トークン水準）はブリーフに直接書かず DESIGN.md を参照する**（二重管理の回避）。DESIGN.md が無い・最小構成の場合のみブリーフに直接書き、恒久化する際に `/init-design` へ移す
- **テンプレート構成**（日本語で再設計。原型 design-brief の10節に、従来 `design-ui` がチャット応答として出していた「画面構成・遷移」「情報設計」「ワーディング方向性」「判断とトレードオフ」の受け皿と、インタビューのエスケープと接続する「要確認事項」を加えた15節）:
  1. **課題** — ユーザー視点の摩擦。技術・ビジネス指標ではなく人間の困りごと
  2. **解決** — 機能リストではなく体験として記述
  3. **体験原則** — 最大3つ。各原則は緊張関係を解決する形で書く（例: 「事前の網羅より段階的開示」）
  4. **画面構成・遷移** — 画面一覧と役割・遷移フロー・各画面の UI 状態（従来の design-ui 出力を移設）
  5. **情報設計** — 情報の階層とグルーピング・ナビゲーション構造（同上）
  6. **美的方向性** — DESIGN.md を参照し、この機能での逸脱・強調があれば差分だけ書く
  7. **既存パターン** — 再利用する既存コンポーネント・トークン（DESIGN.md・コードベースへの参照）
  8. **コンポーネント一覧** — 既存 / 修正 / 新規の表
  9. **主要インタラクション** — 状態変化・遷移・フィードバック
  10. **ワーディング方向性** — ラベル・メッセージ・ガイダンスの方向性（従来の design-ui 出力を移設）
  11. **レスポンシブ挙動** — ブレイクポイントでの変形（サイズだけでなく挙動が変わるもの）
  12. **アクセシビリティ要件** — コントラスト・キーボード・スクリーンリーダー・フォーカス管理
  13. **判断とトレードオフ** — 採った選択肢と検討した代替案（本プラグインの出力ポリシーに由来）
  14. **スコープ外** — このブリーフが扱わないことを具体的に（スコープクリープ防止）
  15. **要確認事項** — インタビューで確定しなかった判断・推奨で埋めた判断（`interview.md` のエスケープと接続）
- **昇格導線**: ブリーフ完成時、DESIGN.md 級の恒久的決定（トーンの明確化・新トークン・新規約）が生まれていないか検出し、あれば `/init-design` へ誘導する（design-md-gate の後段ゲートと同じ思想。自動では書き換えない）

## 7. implement-ui の受け口

- 手順1で `.design/<feature-slug>/DESIGN_BRIEF.md` の有無を確認し、あれば読み込んで**機能単位の判断基準**にする（DESIGN.md = 恒久基準、ブリーフ = 機能判断。矛盾する場合は DESIGN.md を優先し乖離を報告する）
- 実装結果の提示に「ブリーフとの対応」（体験原則の反映・コンポーネント一覧との突合・スコープ外の遵守）を含める

## 8. `.design/` への出力統合

```text
project-root/
├── DESIGN.md                              ← ルート常駐を維持
└── .design/
    ├── <feature-slug>/
    │   └── DESIGN_BRIEF.md                ← 新設
    ├── reports/YYYY-MM-DD/
    │   ├── HHmmss-<skill>.md              ← ui-reports/ から移設
    │   └── screenshots/HHmmss-<skill>-NN.png
    └── preview.html                       ← プロジェクトルートから移設
```

- **DESIGN.md は `.design/` に入れない**。「プロジェクトルートに置くだけで AI エージェントが自動参照する」性質が価値の核であり、隠しディレクトリに入れるとその性質を失う（google-labs-code/design.md 仕様もルート前提）。「憲法だけはルート、UI 作業の生成物はすべて `.design/`」という線引き
- **Git 管理の推奨**: `DESIGN_BRIEF.md` はコミット推奨（`implement-ui` が読む前提の受け渡しファイル）。`reports/` と `preview.html` は各プロジェクトの判断（規約にはその旨を記載）

## 9. ファイルへの反映内容

### 新規作成

| ファイル | 内容 |
|---|---|
| `skills/ui-design-grounding/reference/interview.md` | インタビュー5原則・発動判定の枠組み・質問の帰属振り分け・エスケープ |
| `skills/ui-design-grounding/reference/design-brief.md` | DESIGN_BRIEF.md の保存先・テンプレート・DESIGN.md との関係・昇格導線・`.design/` 全体構造と Git 管理推奨 |

### 変更

| ファイル | 変更内容 |
|---|---|
| `skills/design-ui/SKILL.md` | frontmatter description 更新 / 参照に `interview.md`・`design-brief.md` 追加 / 手順0で DESIGN.md 不在時に `init-design` へ委譲 / 手順に「要件の明確化判定 → インタビュー」を追加 / 出力を DESIGN_BRIEF.md 保存に変更 / 昇格導線を追加 |
| `skills/init-design/SKILL.md` | 参照に `interview.md` 追加 / Step 1 を「抽出確度の判定 → 意図のヒアリング」に作り替え / 仮置きを最後の手段に降格 / 委譲された場合の宣言と最小構成の逃げ道を追加 |
| `skills/implement-ui/SKILL.md` | 参照に `design-brief.md` 追加 / 手順0で DESIGN.md 不在時に `init-design` へ委譲 / 手順1にブリーフ読み込みを追加 / 出力に「ブリーフとの対応」を追加 |
| `skills/ui-design-grounding/reference/design-md-gate.md` | 前段ゲートに「設計・実装の入口は委譲まで踏み込む（詳細は各スキル本文）」の注記を追加 |
| `skills/ui-design-grounding/reference/ui-report.md` | 保存先を `ui-reports/` → `.design/reports/` に変更 |
| `skills/audit-ui/SKILL.md` `skills/score-ui/SKILL.md` `skills/legibility-ui/SKILL.md` | レポート保存先パスの記述を `.design/reports/` に更新 |
| `skills/preview-ui/SKILL.md` | `preview.html` の書き出し先を `.design/preview.html` に変更 |
| `skills/ui-design-grounding/SKILL.md` | 参照ナビゲーションに `interview.md`・`design-brief.md` の一行要約を追加 / リファレンス件数を更新 |
| `AGENTS.md` | リファレンス一覧・件数 / DESIGN.md の扱い（ブリーフとの関係）/ 代表的なワークフロー / 保存先記述 / バージョン記述 / 最新運用の節 |
| `README.md` | 出力先・design-ui の説明更新 / 関連文書の説明 |
| `skills/ui-help/SKILL.md` | design-ui の一言説明が変わる場合に追随 |
| `CHANGELOG.md` | 1.4.0 の変更記録 |
| `.claude-plugin/plugin.json` | バージョン 1.3.0 → 1.4.0 |

## 10. スコープ外（YAGNI）

- designer-skills の他スキル（information-architecture / design-tokens / brief-to-tasks / design-review）は取り込まない（既存スキルが同等の役割を持つ）
- インタビュー単独の入口スキルは新設しない（`design-ui` / `init-design` 内の工程で足りる）
- `refine-ui` へのブリーフ機構追加は今回見送る（既存 UI 修正は DESIGN.md が基準。必要になったら別途検討）
- ブリーフの自動アーカイブ・ライフサイクル管理（実装完了後の移動・削除等）は定義しない
- `ui-reports/` から `.design/reports/` への既存レポートの移行手順・後方互換は提供しない（評価対象プロジェクト側の成果物のため、新規実行分から新パスを使う）

## 11. 受け入れ基準

- `reference/interview.md` が5原則・発動判定・質問の帰属・エスケープを定義し、`init-design` と `design-ui` の MANDATORY PREPARATION から参照されている
- `reference/design-brief.md` が保存先 `.design/<feature-slug>/DESIGN_BRIEF.md`・テンプレート・DESIGN.md との参照関係・昇格導線・Git 管理推奨を定義している
- `design-ui` が「要件の明確化状況の判定 → 必要ならインタビュー → DESIGN_BRIEF.md 保存 → 昇格検出」の流れを持ち、DESIGN.md 不在時に `init-design` へ委譲する
- `init-design` が「抽出確度の判定 → 意図項目のヒアリング（豊富時）/ 本格ヒアリング（希薄時）」を持ち、仮置きが最後の手段に降格され、最小構成の逃げ道が定義されている
- `implement-ui` がブリーフを読み込み、出力に「ブリーフとの対応」を含み、DESIGN.md 不在時に `init-design` へ委譲する
- 評価系3スキルと `ui-report.md` の保存先が `.design/reports/YYYY-MM-DD/` に統一されている
- `preview-ui` の書き出し先が `.design/preview.html` になっている
- DESIGN.md の保存先はプロジェクトルートのまま変わっていない
- AGENTS.md・README.md・ui-design-grounding/SKILL.md の参照ナビ・CHANGELOG・plugin.json が新構成と整合している
