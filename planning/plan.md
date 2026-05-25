# 実行計画: EHR-blockchain 文書の検証 + インタラクティブ HTML 化

最終更新: 2026-05-25 / ステータス: **ユーザー承認待ち**

## ゴール

`source/emr_blockchain_conclusion_verified.md` を、(A) 情報検証し、(B) 検証ステータス付きの単一インタラクティブ HTML に変換する。

## 確定方針（ユーザー合意済み）

- 検証範囲 = 外部事実3点 + 結論1〜11の妥当性・最新動向（網羅）
- 成果物 = 検証バッジ入り単一自己完結 HTML
- 実行 = `research:controller-search` を `session:spawn` で別セッション並行

---

## フェーズ構成

### フェーズ0: 環境整理 ✅ 完了
- ディレクトリ構成（source/verification/site/planning）
- git init（main）
- プロジェクト CLAUDE.md・.gitignore・本計画書

### フェーズ1: 検証（別セッション並行）
**使用スキル: `session:spawn` → 別セッション内で `research:controller-search`**

検証対象を「検証カード」として洗い出す:

| ID | 主張 | 種別 | 検証観点 |
|---|---|---|---|
| F1 | 厚労省「医療情報システムの安全管理ガイドライン第6.0版」が存在し公的基準 | 外部事実 | 版数・最新版（6.0で最新か／改定有無）の確認 |
| F2 | FHIR は HL7 策定の相互運用標準（Resources/API/JSON/XML/HTTP/OAuth） | 外部事実 | 定義・技術整合性 |
| F3 | HSM は NIST 定義の鍵保護物理デバイス（耐タンパ） | 外部事実 | NIST 定義の確認 |
| C1 | BC は本体保存でなく同意/権限/監査の記録層 | 設計結論 | 業界・学術での妥当性、反例 |
| C2 | 実データ暗号化オフチェーン + 参照/権限のみ分離 | 設計結論 | 実装事例の有無 |
| C3 | 患者単独鍵より患者・病院ハイブリッド | 設計結論 | 日本文脈の妥当性 |
| C4 | 単純鍵共有でなく MPC/閾値署名/プロキシ再暗号化 | 設計結論 | 技術定義・採用状況 |
| C5 | 病院側鍵管理は HSM が有力 | 設計結論 | F3 と接続、妥当性 |
| C6 | 全面置換でなく FHIR ブリッジ型導入 | 設計結論 | 現実性 |
| C7 | 台帳分散と実データ冗長化を分離 | 設計結論 | 概念整合 |
| C8 | 他院共有は常時クラスター参加不要・GW経由 | 設計結論 | IPFS 運用妥当性 |
| C9 | 真の価値は標準化高品質データ活用 | 設計結論 | 論点妥当性 |
| C10 | LLM は医療機関管理下の説明支援に寄せる | 設計結論 | 規制・安全性、最新動向 |
| C11 | 研究利用は同意前提で仮名化段階収集 | 設計結論 | 法・倫理整合 |

**手順**:
1. 上記カードを自己完結プロンプト化（原典の主張＋検証観点を含める）
2. `session:spawn` で別 tmux ウィンドウに検証セッションを起動（`WITH_WATCH` 監視）
3. 起動セッションが `research:controller-search` を実行（plan→parallel→integrate→critique→report）
4. 完了後、`/tmp/research-search-*/02-final-report.md` を `verification/research-report.md` にコピー
5. レポートを基に `verification/verification-matrix.md`（各カードの判定: 裏付け/反証/要更新/不明 + 出典 URL）を作成

### フェーズ2: HTML 設計・実装
**使用スキル: `feature-dev:feature-dev`（フェーズ1と並行で土台着手、検証結果は後で注入）**

- Phase 1 Discovery: 要件確定（原典の構造可視化 + 検証バッジ + インタラクティブ要素）
- Phase 2 Exploration: 原典 md の構造分析、可視化対象の整理
- Phase 3 Clarifying Questions: デザイン/インタラクション詳細をユーザーに確認
- Phase 4 Architecture: 実装方式の比較（単一 HTML 埋め込み vs ライブラリ利用 等）と推奨提示
- Phase 5 Implementation: **ユーザー承認後**に実装
- Phase 6 Quality Review: code-reviewer 並列レビュー
- Phase 7 Summary

**HTML 候補要素（Phase 3 で確定）**: 情報源区分の凡例、結論1〜11のカード/アコーディオン、検証ステータスのフィルタ、アーキテクチャ図（オフチェーン/オンチェーン層・鍵管理フロー）、用語ツールチップ、検証マトリクス表。

### フェーズ3: 検証結果の HTML 統合
- フェーズ1の `verification-matrix.md` を各主張のバッジ/出典リンクとして HTML に反映
- 「原典の主張」「検証で確認」「最新動向との差分」を対比表示

### フェーズ4: 最終確認・コミット
- ブラウザ実機確認（必要なら playwright）
- `git` スキルでフェーズごとにコミット

---

## スキル invoke タイミング早見表

| フェーズ | スキル | いつ |
|---|---|---|
| 1 | `session:spawn` | 検証セッション起動時 |
| 1 | `research:controller-search` | 別セッション内で検証実行 |
| 2 | `feature-dev:feature-dev` | HTML 開発開始時 |
| 0–4 | `git`（skill） | 各フェーズのファイル編集後 |

## 依存関係

- フェーズ2のPhase1-4（設計）はフェーズ1と**並行可能**
- フェーズ3（統合）はフェーズ1完了 **かつ** フェーズ2実装完了が前提
- HTML 実装（Phase5）着手前にユーザー承認が必要
