# 計画: 同意管理BCの本番実態 深掘り → HTML整理

**策定日**: 2026-05-27 / **実行**: 次セッション（フェーズ1=`session:spawn`+`research:controller-search`／フェーズ2=`session:spawn --worktree`+`frontend-design`）

## 背景・動機（重要：前回の論点ずれの是正）
これまでの検証・報告は「患者カルテをBCに格納する本番システムは存在しない」を強調したが、これは**論点ずれ（藁人形論法）**だった。原典 **C1**（site/index.html）もユーザーの構想も、最初から「**同意・アクセス権・監査をBC（スマートコントラクト）で管理し、カルテ本体は暗号化オフチェーン**」（図2: アクセス要求→同意確認(SC)→鍵付与(PRE/閾値)→復号→監査ログを台帳へ／図3: 患者が同意・アクセス権付与、台帳に権限と同意を記録）。**カルテ格納は誰も提案していない**。
→ 正しく検証すべきは「**同意・アクセス権管理BC（カルテ本体オフチェーン型）の本番実態**」。MedRec(MIT/BIDMC 40名パイロット)・ConsentChain 等、まさにこの構想の実装が研究され**実患者パイロットまで到達**しているが本番商用稼働は未確認、という状態。エストニアKSIは監査ログのハッシュ完全性証明であり**同意取得BCとは別物**。

## フェーズ1: 同意管理BCの本番実態を research で深掘り

### サブ質問（重視期間 2024-2026・歴史的事例は適宜遡及）
1. **同意・アクセス権管理BC（カルテ本体オフチェーン型）の具体プロジェクトと到達点**: MedRec(MIT)・ConsentChain・MediChainAI・Dynamic Consent on BC 他。各々の実装方式・規模・本番/パイロット/PoC/撤退の別・実患者データ使用の有無・2024-2026の最新現状。
2. **実患者データで本番商用稼働した「同意・アクセス権管理BC」は存在するか（厳密検証）**: 国家/病院ネットワーク/地域医療での継続本番稼働の有無。ドバイNABIDH・中国・韓国・エストニア等を含め網羅。「パイロット」と「本番継続稼働」を厳密に区別。
3. **本番定着の障壁**: 規制(GDPR削除権 vs BC不変性・Crypto-Shredding)・既存EMR(Epic/Cerner)統合・患者インセンティブ/鍵管理UX・運用コスト・スケーラビリティ。なぜPoC/パイロット止まりか。
4. **中央集権の同意管理との比較**: SMART on FHIR / FHIR Consent リソース / Dynamic Consent プラットフォーム(非BC例: Kanta・Dynamic Consent研究実装等)の普及状況。BCの相対的位置と優劣・代替可能性。
5. **患者主権アクセス制御の最新動向(2024-2026)**: SSI/DID + Verifiable Credentials ベースの同意、patient-mediated exchange、EU EHDS の患者制御権、Gaia-X/Health-Xのデータウォレット。BCを使うもの/使わないものの別。

### 実行方法
- `session:spawn` で別セッションに `research:controller-search` を起動（worktree不要・出力 /tmp/research-search-*/02-final-report.md）。
- 完了監視は「最終レポート新規出現 + 処理停止」の二条件（session-state/capture行数のidle判定は処理中=Cooking/thinkingでも誤発火する教訓）。
- 出力を `exploration/consent-bc-production-reality.md` に永続化（cp→コミット→push）。

## フェーズ2: レポートを使って HTML を深掘り・整理

### 整理の方針
- 既存「現実チェック」章（site/llm-ehr.html・s-reality-bc/s-reality-integration）の **③-2（BC vs 中央集権）・BC撤退テーブル**を、「カルテ格納BC不在」という**不正確な軸**から「**同意・アクセス権管理BCの本番実態（PoC/パイロット豊富・本番商用未確立・障壁）**」という**正確な軸**へ書き直し・整理。
- 原典 **C1（BC=同意・権限・監査の記録層）との整合を明示**。「ユーザー構想＝同意管理BC＝原典C1」が研究フロンティアであることを正確に可視化。
- **MedRec(BIDMC 40名パイロット)・ConsentChain 等を「構想に最も近い実在例」として図/表で提示**。中央集権同意管理(SMART on FHIR/FHIR Consent)との比較表。エストニアKSI=監査ハッシュ証明（同意取得とは別）の注記。
- フェーズ1の新レポートの結論を反映。必要なら新カテゴリ用語(dynamic consent・SSI・DID・VC・FHIR Consent・patient-mediated exchange等)をtermsに追加。

### 実行方法
- `session:spawn --worktree` で `frontend-design` を起動。worktree隔離→**diffレビュー**→ffマージ→push→デプロイ確認。
- 厳守: index.html不変・既存セクション/用語ツールチップ機構(terms+termize+autolinkStaticTerms)維持・新用語はterms追加・変更を正直に列挙。

## 制約・留意点
- **「カルテ格納BC」と「同意・アクセス権管理BC」を明確に区別**（前回の混同を繰り返さない）。
- 信頼度ラベル(verified/deduced/inferred/uncertain)を区別。「本番商用稼働」の有無は厳密に（パイロット/PoC/simulated と区別）。
- マージ前diffレビュー必須（frontend-designは「既存不変」と記しても変更しうる教訓・ただし直近3回は厳守を遵守）。
- 関連: site/index.html(C1), exploration/onprem-bc-necessity.md, exploration/integration-feasibility-cases.md, site/llm-ehr.html「現実チェック」章。
- doobidoo関連: 23e15985(深堀②)・4dbc3386(深堀③)・50533af3(現実チェック章反映)。
