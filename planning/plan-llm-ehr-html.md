# 計画: LLM×EHR 考察の HTML化（新ページ追加）

**策定日**: 2026-05-26 / **実行**: 次セッション（`session:spawn` + `frontend-design:frontend-design`）

## 目的
`exploration/` の2考察レポートを、人間に分かりやすいインタラクティブHTMLとして可視化する。検証ビューア（`site/index.html`）とは別の「考察ページ」を新規追加する。

## 確定方針（2026-05-26 ユーザー合意）
- **ページ構成 = 新ページ追加**: 検証ビューア `site/index.html` はそのまま維持。考察専用の新ページ（例 `site/llm-ehr.html`）を追加し、index と相互リンク。各ページはオフライン完結（単一HTML・依存ゼロ）。
- **方法**: `session:spawn` で別セッションに `frontend-design:frontend-design` を起動（worktree 隔離推奨＝公開中の index に影響させず試行 → レビュー後マージ）。
- **デザイン**: 既存 `index.html` のデザインを踏襲（常時ライト・フルードタイポ clamp・clinical paper 配色・カードレイアウト・インラインSVG図・アクセシビリティ）。サイト全体の一貫性を保つ。

## 入力（考察レポート）
- `exploration/llm-chatbot-ehr.md`（325行）: コンテキスト注入技術、セキュリティ懸念6項目、BC統合、規制、推奨アーキ。
- `exploration/e2e-consent-decrypt-context.md`（364行）: E2E統合の具体設計、★RAG vs 構造化ナビ+grep（前回結論の修正）、9ステップ統合シーケンス、サイドチャネル、未解決課題。

## 可視化すべき内容（考察ページのセクション案）
1. 概要・最重要発見（「同意→復号→コンテキスト取得」E2E統合は研究フロンティア／構成要素は揃う＝本プロジェクトの貢献余地）
2. **コンテキスト注入方式の比較**（★RAG vs 構造化ナビ+grep。CLEAR比較表 F1 0.90>0.86・トークン1.1k<3.8k。ユーザーが重視した論点なので強調）
3. E2E統合シーケンス（9ステップのインタラクティブ図／ステップ図＋信頼境界）
4. セキュリティ懸念6項目＋緩和策（カード or マトリクス。間接プロンプトインジェクションが最重要）
5. 4層防御アーキテクチャ（Layer0-3 のSVG図）
6. ブロックチェーン統合（オン/オフチェーン分担・コンポーネント図）
7. 規制マトリクス（HIPAA/GDPR/日本C11）
8. 未解決課題・研究ギャップ（信頼度ラベル付き）
9. 既存検証（C1/C2/C3/C10/C11）との接続（index へのリンク）

## frontend-design への指示骨子
- 上記2レポートを精読し、考察ページ（新HTML）を設計・実装。
- 既存 `index.html` のデザイン・CSS変数・コンポーネントを踏襲（一貫性）。テーマは常時ライト。
- E2Eシーケンス・4層防御・BC統合をインラインSVG図で可視化。図要素にホバー補足（既存踏襲）。
- RAG vs grep の比較は表＋根拠で強調。信頼度ラベル（verified/deduced/inferred/uncertain）をバッジ化。
- index と新ページの相互リンク（ヘッダーナビ等）。
- 単一HTML・依存ゼロ・オフライン完結・アクセシビリティ（focus-visible/aria/role）維持。
- 確認: `python3 -m http.server 8765 --directory site` で 375/768/1280px。サーバー停止は port 指定 kill（pkill -f は自爆）。

## 実行手順（次セッション）
1. `session:spawn --worktree` で `frontend-design` を起動（自己完結プロンプトに本計画の要点を内包）。worktree 隔離で公開中の index に影響させない。
2. frontend-design が考察ページを実装。
3. ブラウザ確認（レスポンシブ・コンソールエラー0）。
4. メインセッションで **diff レビュー**（前回教訓: spawn 先は「既存不変」と言っても実際に変更しうる）→ 問題なければ main マージ → push → GitHub Pages 自動デプロイ。
5. `index.html` に考察ページへのリンクを追加（相互リンク完成）。

## 制約・留意点
- 検証データ（既存 index の `app-data` JSON）は変更しない（考察は別ページ）。
- 考察ページもデータ駆動（JSON）化すると保守的だが、図が多いので frontend-design の判断に委ねる。
- spawn 先の成果は **マージ前に必ず diff レビュー**（前回 frontend-design がフィルタ/ダーク削除を「既存不変」と記しつつ実施した教訓）。
- 関連: 検証本体は [verification/research-report.md]、考察は [exploration/llm-chatbot-ehr.md] [exploration/e2e-consent-decrypt-context.md]。
