# 「患者SC同意 → LLM復号 → コンテキスト取得」E2E統合の具体的システム設計 — 調査レポート

| 項目 | 内容 |
|---|---|
| 調査日 | 2026-05-26 |
| トピック | 「患者がスマートコントラクトで同意 → LLMがオフチェーンの暗号化カルテを復号 → コンテキストとして取得」というエンドツーエンド(E2E)統合の具体設計 |
| 調査方式 | `research:controller-search`（plan→5並列researcher→integrate→critique→report）。重視期間2024-2026 |
| 品質検証 | critic判定 **PASS**（Coverage/Relevance/Citation Accuracy 合格。WARNING 2件＝EDPBラベル過剰・一部二次ブログのラベル過剰、いずれも本レポートで修正反映済） |
| 一次/公式ソース | EIP-712(ethereum.org)・NIST関連・Azure/AWS/NVIDIA/Intel Confidential Computing公式docs・EDPB 02/2025・45 CFR 164.312・HL7 FHIR・Verily/LlamaIndex公式ベンチマーク |
| 学術ソース | 2024–2026のarXiv/PMC/JACC/NPJ Digital Medicine/Springer/Frontiers/EURASIPを中心に40+件 |
| 接続する既存結論 | C1(BC=記録層)・C2(暗号化オフチェーン+CID)・C3(患者/病院二者分散鍵)・C10(LLM患者説明支援+TEE初期本番期)・C11(対話ログは制度的空白) |
| 前回調査との関係 | `exploration/llm-chatbot-ehr.md`（構成要素の同定）を出発点に、**構成要素を統合する具体設計**へ深掘り。前回の「ベクトルRAGが支配的設計」は本調査で実証的に修正 |

> 信頼度ラベル: `verified`＝一次資料/実証で確認 / `deduced`＝公式docs・複数学術から論理導出 / `inferred`＝状況証拠からの推測 / `uncertain`＝未確認。

---

## エグゼクティブサマリ

1. **E2Eループ全体の明示実装は依然「研究フロンティア」（3ワーカーが独立に確認）**。「EIP-712署名で同意ゲート → 対称鍵取得 → オフチェーン暗号EHR復号 → LLMにコンテキスト供給」を**一貫実装した一次文献は2026-05時点で不在**。同意(researcher-1)・鍵フロー(researcher-2)・TEE推論(researcher-3)の3調査が独立して同じ空白を報告しており、前回調査の判定を補強する。一方、**構成要素はすべて出揃い、かつ一次文献・公式docsで内部機構まで具体化できる**段階に達した。[verified]

2. **構成要素は「具体コード/公式機構」レベルまで特定できた**。同意ゲートは患者中心FW(arXiv 2511.17464)が`Permission` struct + `grantPermissionBySig`のEIP-712検証ロジックを実コードで提示（許可付与78,000 Gas）。鍵フローは AWS KMS/Azure SKR の **attestation-gated key release（`CiphertextForRecipient`＝平文鍵をTEE内部にしか出さない機構）** が公式仕様として存在。TEE処理は NVIDIA H100/B200 の8ステップアテステーションシーケンスが公式化。監査は EDPB 02/2025 と HIPAA 45 CFR 164.312(b) に整合する `AuditEvent` スキーマ+Merkleアンカリングが導出できる。[verified]

3. **本調査の最重要の修正点：コンテキスト取得は「ベクトルRAG」より「構造化ナビゲーション＋語彙的検索」が医療/FHIR文脈で優位**（前回レポート「ベクトルRAG支配的」を一次ソースで修正）。CLEAR(NPJ Digital Medicine 2025)はNER+正規表現の構造化検索がembedding RAGを **F1 0.90 vs 0.86・入力トークン1.1k vs 3.8k** で上回ると実証。FHIR-AgentBench/LLMonFHIR/EHR-MCP も function calling/構造化APIナビを採用。「LLMonFHIRのRAG＝function calling」という命名混乱も解明。E2Eの「コンテキスト取得」ステップは **FHIR階層fetch（構造化ナビ）を一次手段、非構造化ノートにCLEAR/BM25、大規模時のみhybrid+reranking** とすべき。[verified]

4. **「同意ポリシー通りに推論される」ことはアテステーション単体では証明できない**。リモートアテステーションはコードとハードウェアの完全性を証明するが、推論の意味論的正しさ（consent scopeに沿った回答のみ）は別途TEE内ガードレール/ポリシーエンジンで担保する**構成的アプローチ**が必要。[deduced]

5. **TEEを使っても防げない残存リスクが実証されている**。KV-Cacheタイミング攻撃で医療患者名×病名対応を95.4%精度推定(arXiv 2409.20002)、CPU Flush+ReloadでAPIキー80-90%回復(arXiv 2505.00817)、BudgetLeakでRAG知識ベースのメンバーシップ推定(arXiv 2511.12043)。さらに**取得したカルテ本文そのものが間接プロンプトインジェクションの媒体**になる（前回調査の核心と接続）。「TEE＝万能」ではない。[verified]

6. **最大の非技術的障壁は日本の制度的空白（C11）**。本調査の設計はすべて国際基準（HIPAA/GDPR/EDPB）ベース。対話ログの法的性質は日本で未定義のまま。[verified]

---

## 焦点1：同意メカニズムの実装

### 1.1 EIP-712 型署名による患者同意の構造化

EIP-712（公式仕様）は「型付き構造化データのハッシュと署名」標準で、`encode = "\x19\x01" ‖ domainSeparator ‖ hashStruct(message)` の形式。`EIP712Domain = {name, version, chainId, verifyingContract, salt}` のドメインセパレータがDApp間・チェーン間の署名リプレイを防ぐ。[verified｜S1]

**最も具体的な実装＝患者中心FW（arXiv 2511.17464）**[verified｜S2]:
```solidity
struct Permission { uint64 expiration; bool revoked; bytes wrappedKey; }
// PERMISSION_TYPEHASH のフィールド: (rid, grantee, expiration, keccak256(wk), nonce)
```
- **`grantPermissionBySig` のオンチェーン検証**: ①nonce未使用(リプレイ防止) ②`expiration > block.timestamp` ③`ecrecover`→`require(signer == patient)` ④nonce使用済マーク ⑤permissions保存。
- **`hasValidPermission`（アクセス前ゲート）**: 非本人は `!revoked && expiration>0 && expiration>block.timestamp`。
- **`revokePermission`（オプトアウト）**: `onlyPatient`で患者のみ即時失効。
- **復号連鎖**: `wrappedKey`が`Permission`に含まれるため、**同意ゲートが開いて初めて対称鍵を`ECIES-Dec(SK_R, W_R)`で解包できる**。これが「同意→復号」を結ぶ核心機構。
- セキュリティ定理: 機密性=AES-GCMでIND-CCA2、認可真正性=secp256k1 ECDSAでEUF-CMA。コスト=許可付与78,000 Gas(L1)、E2Eアクセス遅延0.7〜1.4秒。**ただしZKPは未実装(future work)**。

### 1.2 スマートコントラクトの同意ゲート（HIPAA/GDPR検証）

- **MediChainAI(PMC12650700)**[verified設計/deduced実装｜S3]: 同意SC `C_consent(P, D, A)`（要求者/データカテゴリ/アクセス種別）。**AI推論ゲートのシーケンス**＝「SC同意検証 → KMSが検証済みIDにのみ鍵リリース → **TEE内で復号** → ML/LLMアクセス(ε=1.0差分プライバシー)」。患者が粒度付トークン(データサブセット/期限/目的/TEE要件)を署名し、SCがトークンハッシュ記録+`CONSENT_APPROVED`発行。**ただしZKP回路仕様・公開/非公開入力は非公開**（架構提案レベル）。
- **BXHF(arXiv 2509.14987)**[verified構造/uncertain実装｜S4]: SC同意ゲート `ϕ(u,d) = {1 if authorized; 0 otherwise}`。HIPAA/GDPR準拠をSCで検証してからモデル呼出。**具体的検証ロジックは非公開**。
- **HIPAA最小必要原則のオンチェーン化**[deduced｜S8,S9]: データカテゴリ属性 `[demographics, lab_results, mental_health]` を研究目的にバインド（「心臓研究認証者は心機能関連ラボ結果のみ」）。同意条件に時間/場所/IP/頻度制約（「8am-5pmのみ」等）も埋込可能。

### 1.3 ZKP（データ非開示での同意・ポリシー充足証明）

- **TeleZK-L2（Frontiers 2026）**[verified｜S5]: R1CS+Poseidonハッシュ(~160制約/入力)+Bit-Decompositionガジェット。公開入力=IPFS CID/コミットメント/タイムスタンプ、非公開入力=生理値/暗号鍵/患者ID。オンチェーン`verifyBatch()`=4ペアリング(定数コスト)、Polygon zkEVM。**重要な留保**: これは**バイタル値の範囲証明**であり、「同意条件充足」の証明回路ではない。
- **一般パターン**[inferred]: 証明対象=「患者が有効な同意を保持し、要求アクセスがポリシー条件を充足する事実」。公開入力=コミットメント/ポリシーID/タイムスタンプ、非公開入力=同意内容/条件フィールド、**非証明=実ヘルスデータ**。**この「同意充足ZKP回路」の具体実装は未確認＝研究ギャップ**。

### 1.4 Dynamic Consent・失効・オプトアウト

- **PMC8659597**[verified｜S6]: 同意アグリーメント`CA`と同意リクエスト`CR`（`OP=[COLLECT,STORE,PROCESS,DISCLOSE,COPY,SHARE]`、`CST`に期間/地理制約）。失効=`CAstatus→invalid`（削除でなく状態フラグ）、リアルタイム反映。
- **ConsentChain(PMC8600428)**[verified｜S7]: `setConsent`/`cancelConsent`/`checkConsent`の3関数。データタイプ・目的・役割をコード化しBoolean論理式でポリシーツリー化。`cancelConsent`で即時無効化。
- **GDPR Art.9**[deduced｜S10]: 健康データは明示的同意必須。**SCが「自動でデータ記録をトリガー」する設計は明示的同意要件と衝突しうる** → 各アクセスイベントに患者のEIP-712署名Txが必要。

---

## 焦点2：復号権付与と鍵フロー

### 2.1 ABE / PRE による復号権付与

- **Multi-Authority CP-ABE**[high｜S12,S13,S17]: 複数KGCがPedersen Secret Sharing(k-of-n)で部分鍵発行 → ユーザーが結合してメイン秘密鍵導出。**エスクローフリー**（単一KGCは乱数αのシェアのみ、単一侵害で全鍵再構成不可）。アクセスポリシーをLSSS/アクセスツリーで暗号文に埋込。失効はSCのupdate key発行で実現(MA-RABE)。
- **Umbral閾値PRE**[high｜S15,S16,S25]: 再暗号化鍵 `rk_{A→B}` = Alice秘密鍵+Bob公開鍵のみで生成(非対話的)。k-of-nでkFragsに分割し分散プロキシ(Ursulas)に配布。**プロキシは両者の秘密鍵不要・k-1結託耐性**。SSX-EHRsのD-PREは負荷に応じ動的適応+zkSNARKs匿名認証統合。**限界**: arXiv 2307.01175は単一プロキシ使用中で真の閾値PRE+BC統合は将来作業。

### 2.2 C3「患者鍵+病院鍵の二者分散」の具体化（PMC12979818）

[high｜S14] **患者(データオーナー)**＝アクセスポリシー定義・乱数`r`生成→`R=rG`・ABE暗号化鍵導出。**病院スタッフ(データユーザー)**＝KGCへ資格証明提示後に役割属性ごとの部分秘密鍵受領→複数KGCの部分鍵を結合してメイン秘密鍵導出。**病院クラウドは「半信頼」**（暗号化EHR保管するが復号不可）。患者鍵と医師鍵は非共有・複数ユーザー結託でも不正復号困難。

→ ただし**この論文のLLM(ClinicalBERT)は「属性抽出ツール」**（非構造化ノートからNERで診断名/薬名を抽出してABEポリシーに組込・院内ローカル実行）であり、**ABE/PRE復号後のEHRをLLMが受信して推論する設計ではない**。「LLMを復号データの受信者とする統合」はここでも不在。[verified]

### 2.3 復号鍵がTEEへ安全到達するまで（attestation-gated key release）

E2E統合の鍵の安全性を担保する核心機構。**鍵がTEE外に漏れない保証は公式仕様として存在する**:

- **AWS KMS + Nitro Enclaves**[verified｜S19,S27]: KMSキーポリシーに `kms:RecipientAttestation:ImageSha384`/`PCR0〜8` 条件を記述。`Decrypt`/`GenerateDataKey`/`DeriveSharedSecret` をアテステーションドキュメント付きで呼ぶと、**KMSは平文を返さず、アテステーション内の公開鍵で暗号化した `CiphertextForRecipient` を返す**。ホスト(親)は受け取るが復号不可、エンクレーブ内部秘密鍵でのみ復号可能。`DeriveSharedSecret`はFIPS 140-2 HSM内でECDH完結。
- **Azure Key Vault SKR**[verified｜S18]: TEE→MAAアテステーション要求→測定値入りJWT発行→AKVがSKRポリシーとJWTクレーム照合一致で**TEE内部のみ利用可能な形式で鍵リリース**。
- **Fortanix CCM Composite Attestation**[high｜S20]: AMD KDS/Intel PCSでCPU検証+NVIDIA RASでGPU検証を**Composite Attestation Certificate(CAC)**に統合し、RAGパイプライン全体の信頼境界を一括証明。鍵は「attested context外で再利用不可」なephemeral credential。

**未解決**: 上記は**対称コンテンツ鍵**のリリースが主対象。**ABE秘密鍵（属性ベース複合鍵素材）をTEEへアテステーション連動でリリースする具体実装は未確認**[uncertain]。

---

## 焦点3：TEE内処理フロー

### 3.1 アテステーション連動の処理シーケンス（公式）

- **NVIDIA H100/H200/B200 GPU TEE 8ステップ**[high｜S21]: ①HW Root of Trust検証 ②セキュアブート+署名FW検証 ③CVMローンチ ④CC-Onモード(HWファイアウォール) ⑤SPDM鍵交換 ⑥アテステーションレポート生成(GPU固有ECC-384署名) ⑦NRAS(NVIDIA Remote Attestation Service)がJWT発行 ⑧KMSがJWTクレーム`"x-nvidia-cc-mode-enabled":true`検証後に鍵リリース。VRAMはCCEがAES-256-GCM暗号化(鍵はチップ外不出)。**B200は暗号化PCIe 5.0+NVLink暗号化を追加**（H100のNVLink非暗号化を解決）。Corvex(2026-03)がHGX B200で本番デプロイ実証、70B級でオーバーヘッドほぼゼロ。[S31]
- **Azure Confidential Inferencing（ECDH+OHTTP）**[high｜S28,S32,S33]: モデルをクリーンルームで署名・AES暗号化→TEE起動→MAAトークン取得→AKVがポリシー(MRENCLAVE/MRSIGNER)確認後にモデル鍵リリース→TEE内復号(ウェイトはTEE境界外不出)→TEEがECDH秘密鍵生成→クライアントがOHTTPでプロンプト暗号化送信→TEE内でのみ復号・推論。
- **AWS Nitro Enclaves + MedGemma 4B（2025-12公式）**[high｜S29,S30]: Docker→EIF変換時にPCRハッシュ(SHA-384)計算→アテステーション文書を`Recipient`としてKMSへDecrypt要求→PCR照合一致で鍵リリース→エンクレーブ内でモデル復号→クライアント↔エンクレーブはvsockのみ(外部ネット接続なし)→応答をDynamoDB監査(PHI本文含まない)。**公式実装にRAG未統合（直接推論のみ）**。

### 3.2 TEE内RAG（復号→取得→推論）の設計パターン

[medium｜研究段階・公式完全実装なし]
- **連合RAG(arXiv 2603.25374)**[S35]: 各サイロがローカルFAISS検索→トップk文書+スコアのみTEEへ転送(生EHRはサイロ外不出)→TEE内でReciprocal Rank Fusion集約→TEE内LLM推論。アテステーション確認済サイロのみ参加。
- **Full-TEE RAG(Fortanix/Opaque)**[S36]: ingestion→ベクトル化→取得→推論の全工程をTEE内、ベクトルDBもTEE内(暗号化)。製品として存在するが技術詳細非公開。
- **障壁**: TEE内ベクトルDB検索スループット、SGXメモリ制限(数百MB〜数GB)と大容量EHRインデックスの非整合。→ **焦点5の「構造化ナビ優先」がこの障壁の現実的回避策**にもなる（全量索引化を避けFHIR階層fetchで絞る）。

### 3.3 「同意ポリシー通りの推論」証明の限界（構成的アプローチ）

[deduced] アテステーションはコード・HWの完全性を証明できるが、**「推論の意味論的正しさ（consent scopeに沿った回答のみ）」はアテステーション単体では証明不可**。構成的アプローチ＝①アテステーションでガードレールロジックの非改竄を証明 + ②TEE内実行コードにconsentポリシーエンフォースメント（ポリシーエンジン）を組込む。NVIDIAはNeMo Guardrails共同ホスティングを推奨。

### 3.4 サイドチャネル残存リスク（TEEで防げない・2025-2026実証）

| 攻撃 | 手法 | 漏洩 | TEEで防げるか |
|---|---|---|---|
| KV-Cacheタイミング(arXiv 2409.20002)[S37] | TTFT差(ヒット~0.35μs vs ミス~45μs) | **医療患者名×病名対応を95.4%精度推定**、システムプロンプト89.0%復元 | ✗ ソフト層共有問題 |
| CPU Flush+Reload(arXiv 2505.00817)[S38] | 埋め込み層アクセス観測(CUDA unified memory) | APIキー80-90%回復、平文クエリ40%回復 | ✗ 埋め込み層に非適用 |
| BudgetLeak(arXiv 2511.12043)[S39] | トークン予算下の出力長差 | RAG知識ベースのメンバーシップ推定 | ✗ 出力長が観測可能 |
| 物理/CVE[S41] | コールドブートHBM・PCIe DMA・FW脆弱性 | ウェイト/データ | △ B200で一部軽減・FW更新依存 |

緩和=K≥2トークン単位キャッシュ共有/Intel CATキャッシュ分割/バジェットランダム化/差分プライバシー。

---

## 焦点4：オンチェーン監査

### 4.1 監査イベントスキーマ（「誰がいつ誰のカルテをどのLLMに渡したか」）

[deduced] PHI本文・患者氏名・生データは一切オンチェーンに置かない:
```
AuditEvent {
  event_id: bytes32          // SHA-256(actor||patient||ts)
  event_type: string         // LLM_DATA_ACCESS|CONSENT_APPROVED|MODEL_INFERENCE|DATA_FETCH
  actor_did: string          // 医療者/LLMエージェントのDID
  patient_record_id: bytes32 // H(patient_id||epoch_salt) ─ 患者ID非記録
  model_id / model_version_hash: string / bytes32  // モデル名+版 / 重みSHA-256
  consent_id_hash: bytes32   // 同意トークンのハッシュ
  data_scope_hash: bytes32   // アクセス対象サブセットのハッシュ
  timestamp: uint256; access_status: string  // GRANTED|DENIED|EMERGENCY_OVERRIDE
  tee_enclave_id / zkp_proof_hash / session_key_hash: bytes32 (任意)
  purpose: string            // HIPAA 164.312(b) purpose of use
}
```
`emit LLMAccessEvent(...)`でindexed化。[S43,S3,S10]

### 4.2 Merkle化バッチアンカリング

[deduced｜S46,S47] 時間窓(1h/1日)の監査イベントをオフチェーン収集→SHA-256で葉ノード→Merkleツリー→**Merkle Root(32B)のみオンチェーンアンカリング**(`emit MerkleRootAnchored`)。個別包含証明はオフチェーン発行、Merkle Mountain Range(MMR)で追記最適化。MedBeads(arXiv 2602.01086)は`ID_B=SHA256(JSON(B))`のコンテンツアドレスDAG。

### 4.3 AI出力の暗号証明

- **BXHF型SHAP証明**[deduced/uncertain｜S4]: SHAP説明を「top-k feature indices+importance scores」JSONにシリアライズ→SHA-256→オンチェーン("explanation hash")。**限界**: 浮動小数点SHAP値のハッシュ化は精度依存の再現性問題、Solidity定義は未記載。
- **Attestable Audits（arXiv 2506.23706, ICML 2025）**[verified｜S44]: Remote Attestation Report=`{pcr_values[],model_hash,audit_code_hash,dataset_hash,timestamp,vendor_signature}`。LLM推論は`(prompt_hash,model_hash,response_hash,A_{M,p→x,R})`を透明性ログに記録、BC へはアテステーションハッシュのみアンカリング。**AWS Nitro上でLlama-3.1-8B実証済**。

### 4.4 GDPR忘れられる権利との両立

[deduced｜公式PDF未取得・解説2件一致｜S10] EDPB Guidelines 02/2025（2025-04-14）の要点（※EDPB公式PDFは両researcherとも直接取得失敗、activeMind/privacyworld解説2件の一致から導出）: ①暗号化・ハッシュ化データもGDPR個人データ ②技術的不可能は免責にならない ③本文オフチェーン+ハッシュのみオンチェーン ④消去の代替=オフチェーンデータ削除+復号鍵の安全消去(key erasure)でオンチェーンハッシュを事実上アクセス不能化 ⑤パーミッションドBC推奨。患者IDは`H(patient_id||epoch_salt)`、消去時にソルトをローテーション/削除。

### 4.5 HIPAA 45 CFR 164.312(b) 監査ログ要件

[verified｜S45] ePHIシステムの活動記録・検査メカニズムを実装(Required)。必須=user_id/timestamp(UTC)/event_type/success-failure/resource_id。**最低6年保存**(§164.316(b)(2)(i))。PHI本文は監査ログに含めない。

---

## 焦点5：コンテキスト取得方式の比較（ベクトルRAG vs 階層型インデックス+grep/BM25）★追加重点

> **本焦点は前回レポート(`exploration/llm-chatbot-ehr.md`)の「ベクトルRAGが支配的設計」という結論を、一次ソースで修正する。**

### 5.1 一次ソースによる主張（前面）

- **CLEAR（NPJ Digital Medicine 2025, PMC11743751）= 最も直接的な医療エビデンス**[verified｜S52]: zero-shot NER→オントロジー拡張→正規表現マッチの構造化検索がembedding RAGを上回る:

| 指標 | CLEAR（NER+regex） | Embedding RAG | Full-Note |
|------|-------|---------------|-----------|
| F1 | **0.90** | 0.86 | 0.79 |
| 入力トークン | **1.1k** | 3.8k | 6.1k |
| 推論時間 | **4.95s** | 17.41s | 20.08s |

原因=(a)長コンテキストでLLM性能低下(lost in the middle) (b)embeddingが最重要チャンクを下位ランク付け。

- **arXiv 2604.01733（2026-04）**[verified｜S48]: 財務QA(23,088クエリ)でBM25がtext-embedding-3-largeを全指標で上回る(Recall@5: 0.644 vs 0.587)。専門用語・標準ラベルは語彙的一致で捕捉できるため。
- **arXiv 2505.11582（2025-05）**[verified｜S49]: **医療文書分類でBM25がsemanticモデルより高精度かつ高速**。
- **FHIR-AgentBench（arXiv 2509.19319, Verily/KAIST/MIT）**[verified｜S50,S61]: 2,931実臨床質問で構造化API(`Observation?patient=X&code=Y`)とRetriever Toolを比較。**ベンチマーク自体がベクトルRAGを評価対象外**とした（最高=multi-turn+Retriever+Code Interpreter o4-miniで50.0%）。**留保（過剰解釈回避）**: RAGを評価外とした理由が「不適切と判断」か「単純スコープ外」かは論文から不明。
- **LLMonFHIR（JACC 2025, PMC12144420）**[verified｜S51]: **「RAG abstraction = function calling」を明示** = ベクトル埋め込みでなく構造化取得。Stanford医師5名評価でAccuracy/Understandability/Relevance中央値5/5。→ 前回レポートの「LLMonFHIRはRAG」という記述の**命名混乱を解明**。
- **EHR-MCP（arXiv 2509.15957）**[verified｜S53]: MCP経由SQL決定論的クエリが数値精度で優れる（「embeddingは定量的大きさと単位の区別を保持できない」）。

### 5.2 補足的傍証（二次分析・信頼度を下げて扱う）

- アジェント的grep検索がRAG同等以上との報告（LlamaIndex公式ベンチマーク[verified｜S59]＝小〜中規模でファイルシステムツールがCorrectness 8.4 vs 6.4、**大規模100-1000+件ではRAGがスケール優位**）。
- Claude Code開発者の「glob+grep+readがベクトルRAGを大幅に上回る」証言[inferred｜podcast経由・S58,S60]、Amazon研究ベースの分析[deduced｜二次・S56]、ハイブリッド設計の実務知見[deduced｜S57]。※これらは二次分析であり、結論の主たる根拠は5.1の一次ソース。

### 5.3 データ種別ごとの使い分け（E2E「コンテキスト取得」ステップの設計判断）

| データ種別 | 推奨手法 | 根拠 |
|-----------|---------|------|
| FHIR構造化リソース(Observation数値/Medication/Condition) | **構造化ナビゲーション**(function calling/MCP/FHIR API + リソースタイプ階層) | EHR-MCP, LLMonFHIR, FHIR-AgentBench |
| 非構造化臨床ノート(DocumentReference等) | **CLEAR型NER+regex** または BM25ベースhybrid | CLEAR(NPJ DM 2025), arXiv 2505.11582 |
| 大量の非構造化文書(>1000件) | **BM25+dense hybrid + reranking** | arXiv 2604.01733, LlamaIndex 2026 |

**300万トークン問題の新解法**: 全量をベクトルインデックス化してembedding検索するのでなく、「**FHIR階層+リソースタイプ指定でフィルタ → 必要リソースのみfetch → 非構造化部分にCLEAR/BM25**」というアジェント的ナビゲーション。FHIR階層構造(Patient>Encounter>Observation>...)はリソースタイプ指定で辿る構造化ナビと本質的に相性が良い。FHIR固有の失敗（リソースタイプ誤識別・参照リンク追跡失敗）は**retrievalでなくnavigationの問題**でありベクトルRAG/hybridでは解決不可。

---

## 統合E2Eシーケンスと信頼境界（焦点1〜4の統合設計）

### コンポーネント構成図（テキスト）
```
┌─────────┐  EIP-712署名(同意)   ┌──────────────────────────┐
│ 患者     │ ───────────────────▶ │ ブロックチェーン(記録層=C1) │
│ ウォレット│ ◀─ CONSENT_APPROVED ─ │ ・同意SC(consentゲート)     │
└─────────┘                       │ ・Permission/失効 mapping  │
     │ 患者鍵                      │ ・コミットメント d=H(C‖T‖N) │
     ▼                            │ ・監査イベント/Merkle Root  │
┌──────────────┐                  └──────────────────────────┘
│ 鍵管理(C3二者分散)│                          │ hasValidPermission / ϕ(u,d)
│ 患者鍵+病院HSM鍵 │                          ▼ 検証OK→鍵リリース許可
│ MA-ABE/閾値PRE  │             ┌────────────────────────────────┐
└──────────────┘  ───────────▶ │ KMS/HSM (Azure AKV-SKR/AWS KMS) │
     │                          │ attestation-gated key release   │
     │ 暗号化EHR(C2)             │ → CiphertextForRecipient        │
     ▼                          └────────────────────────────────┘
┌──────────────┐                          │ (TEE公開鍵で暗号化された鍵)
│ オフチェーン保存 │ ─CID(BC)─▶               ▼
│ IPFS/S3 暗号化  │ ─暗号文─▶ ┌────────────────────────────────────┐
│ FHIR + ノート   │           │ TEE / Confidential Computing(C10)    │
└──────────────┘           │ (Intel TDX/AMD SEV-SNP + NVIDIA H100/ │
                            │  B200 GPU TEE / Nitro / Azure CI)     │
                            │ ① 鍵をTEE内部秘密鍵で復号             │
                            │ ② EHR本文をTEE内で復号                │
                            │ ③ コンテキスト取得=構造化ナビ優先     │
                            │    (FHIR階層fetch→CLEAR/BM25→hybrid)  │
                            │ ④ ガードレール(取得本文=信頼境界外)   │
                            │ ⑤ LLM推論                             │
                            │ ⑥ 出力をECDH/ML-KEMで暗号化           │
                            └────────────────────────────────────┘
                                   │ 監査ハッシュ/アテステーション証跡
                                   ▼ をオンチェーンへ(誰が・いつ・誰のを・どのモデルに)
```

### E2Eステップと信頼境界
| # | ステップ | 主体 | 信頼境界・保証 | 残存リスク |
|---|---------|------|--------------|----------|
| 1 | EIP-712同意署名(rid/grantee/expiration/wrappedKey/nonce) | 患者ウォレット | 患者鍵(ECDSA EUF-CMA)・クライアント署名 | wrappedKeyをcalldataに含める設計はL1で可視 |
| 2 | SC `grantPermissionBySig`検証→`CONSENT_APPROVED` | スマートコントラクト | BC不変性 | 失効伝播のブロック確認ラグ／ZKP同意充足は未実装 |
| 3 | 復号権付与: MA-ABE属性検証 or 閾値PRE再暗号化 | KGC群/プロキシ群 | エスクローフリー・k-1結託耐性 | ABE鍵のTEEリリース実装は未確認 |
| 4 | アテステーション→KMS/HSMがPCR/MAA照合→`CiphertextForRecipient` | TEE+KMS | **平文鍵はTEE内部のみ**・ホスト復号不可 | KMSは特定アテステーションのみ信頼 |
| 5 | TEE内で鍵→EHR本文復号 | TEE | HW保証(ホスト/HV/他テナント不可視) | KV-cache/Flush+Reload/BudgetLeak |
| 6 | コンテキスト取得(**構造化ナビ優先**) | TEE内取得エンジン | 最小必要原則と整合 | TEE内RAGは研究段階 |
| 7 | ガードレール(取得本文=信頼境界外)→LLM推論 | TEE内LLM+ポリシーエンジン | コード完全性はアテステーション証明 | 意味論的正しさは別途担保／間接PI |
| 8 | 出力をECDH/ML-KEM/Noiseで暗号化返却 | TEE→クライアント | フォワードセキュリティ | — |
| 9 | AuditEventハッシュ/Merkle Root/アテステーション証跡をオンチェーン | 監査SC | 本文非記録でGDPR第17条両立・HIPAA6年保存 | model版管理断絶/長期検証性 |

---

## 実装上の未解決課題・トレードオフ

| # | 課題 | 内容 | 信頼度 |
|---|---|---|---|
| 1 | **E2Eループ全体の明示実装が不在** | 「EIP-712同意→復号→LLMコンテキスト」を一貫実装した一次文献は未確認。**researcher-1/2/3が独立に同じ空白を報告**＝研究フロンティアの判定は堅固 | verified |
| 2 | ZKP「同意条件充足」回路 | MediChainAIは宣言のみ、TeleZK-L2はバイタル範囲証明で同意充足ではない | verified |
| 3 | ABE秘密鍵のTEEリリース | KMS→TEEは対称鍵が主対象。ABE複合鍵素材のアテステーション連動リリース実装は未確認 | uncertain |
| 4 | TEE内RAGの完全公式実装不在 | AWS/Azure公式は直接推論のみ。連合RAG(arXiv)/Full-TEE RAG(Fortanix/Opaque・非公開) | verified |
| 5 | 性能/プライバシー税 | GPU TEEは4-8%(70B級でほぼ0)だがサイドチャネルが残存。構造化ナビはCLEARで70%トークン削減 | verified |
| 6 | 鍵管理の複雑性 | MA-ABE×閾値PRE×二者分散×TEE鍵リリースの統合は概念的に可能だが完全実装例なし。患者鍵紛失時の回復(C3が示す課題) | deduced |
| 7 | 間接プロンプトインジェクション | 正当に復号・取得したカルテ本文自体が攻撃媒体。取得データを信頼境界外扱いする多層防御が必須(前回調査の核心) | verified |
| 8 | 「同意通り推論」の証明限界 | アテステーションはコード完全性のみ。意味論的正しさはガードレールで構成的に担保 | deduced |
| 9 | GDPR鍵削除消去の法的確定性/LLMプロンプトのGDPR該当性 | EDPBはkey erasureを推奨技術とするが各国DPA解釈依存 | uncertain |
| 10 | **日本の制度的空白(C11)** | 対話ログの法的性質が未定義。本設計は国際基準ベースで日本固有適合は別途要検証。日本語EHRのBM25(形態素解析)動作は調査外 | verified |

---

## 既存結論（C1/C2/C3/C10/C11）との接続

- **C1（BC=同意/権限/監査の記録層）**: 焦点1の同意SC・焦点4の`AuditEvent`スキーマが直接裏付け。本文非記録(EDPB 02/2025)で一貫。**強化**。
- **C2（暗号化オフチェーン+CID参照）**: 焦点1のレコードコミットメント`d=SHA-256(C‖T‖N)`・焦点2のIPFS/S3暗号化保存が裏付け。**強化**。
- **C3（患者鍵+病院鍵の二者分散）**: 焦点2のMA-ABEエスクローフリー鍵生成(PMC12979818)が具体化。**ただしLLMを受信者とする統合は未実装**＝C3とLLM統合の接続が研究ギャップ。
- **C10（LLM患者説明支援＋TEE初期本番期）**: 焦点3のNVIDIA/Azure/AWS公式フロー・Corvex B200本番・BeeKeeperAIが裏付け。臨床LLM患者説明支援のTEE本番直接事例はなお萌芽。**強化**。
- **C11（対話ログの制度的空白）**: 焦点4のGDPR/HIPAA要件は国際基準。日本の制度的空白は未解決のまま。**変化なし**。

---

## 結論

「患者がスマートコントラクトで同意 → LLMがオフチェーンの暗号化カルテを復号 → コンテキストとして取得」というE2E統合は、**構成要素の一つ一つが一次文献・公式docsで具体機構（実コード/API仕様）まで特定できる成熟段階に達している**。同意ゲート（EIP-712 `Permission`+`grantPermissionBySig`）・復号権付与（MA-ABEエスクローフリー＋閾値PRE）・鍵の安全到達（attestation-gated key release の `CiphertextForRecipient`）・TEE推論（NVIDIA 8ステップ/Azure CI/AWS Nitro）・オンチェーン監査（`AuditEvent`+Merkle+Attestable Audits）は、それぞれ実装可能性が明確である。

しかし、**これら全部を一気通貫で結ぶ「同意→復号→LLMコンテキスト」のE2E実装は依然として一次文献に存在せず（researcher-1/2/3が独立に確認）、研究フロンティアであり続けている**。本プロジェクトの貢献余地は、まさにこの統合プロトコル（特に「ZKP同意充足証明 → アテステーション → ABE鍵のTEEリリース → 構造化ナビによる最小コンテキスト取得 → ガードレール付き推論 → オンチェーン監査」の連鎖）の設計にある。

そして本調査の最も実務的な発見は、**「コンテキスト取得」ステップはベクトルRAG一択ではなく、FHIR文脈では構造化ナビゲーション（function calling）を一次手段とし、非構造化ノートにCLEAR/BM25、大規模時のみhybrid+rerankingを使うべき**という、前回結論の実証的修正である。これはTEE内RAGのメモリ制約を回避し、HIPAA最小必要原則とも整合し、間接プロンプトインジェクションの攻撃面を縮小するという点で、E2E設計全体にとって望ましい方向に収束する。

最後に、TEEは万能ではない——KV-Cacheタイミングで患者名×病名対応が95.4%精度で漏れうるという実証があり、取得カルテ本文自体が攻撃媒体になりうる。そして最大の非技術的障壁は、日本における対話ログの制度的空白（C11）である。

---

## Evidence Table（主要ソース）

| # | Title | URL | Type |
|---|-------|-----|------|
| S1 | EIP-712: Typed structured data hashing and signing | https://eips.ethereum.org/EIPS/eip-712 | 公式仕様 |
| S2 | A Patient-Centric Blockchain Framework for Secure EHR Management (arXiv 2511.17464) | https://arxiv.org/html/2511.17464v1 | 学術arXiv 2024 |
| S3 | Ethical AI in Healthcare: ZKPs and Smart Contracts (MediChainAI, PMC12650700) | https://pmc.ncbi.nlm.nih.gov/articles/PMC12650700/ | 学術PMC 2025 |
| S4 | Blockchain-Enabled Explainable AI for Trusted Healthcare (BXHF, arXiv 2509.14987) | https://arxiv.org/html/2509.14987v1 | 学術arXiv 2025 |
| S5 | TeleZK-L2: scalable zk-SNARK for privacy-preserving telehealth (Frontiers) | https://www.frontiersin.org/journals/blockchain/articles/10.3389/fbloc.2026.1762781/full | 学術Frontiers 2026 |
| S6 | Smart Contract-Based Dynamic Consent Management under GDPR (PMC8659597) | https://pmc.ncbi.nlm.nih.gov/articles/PMC8659597/ | 学術PMC |
| S7 | ConsentChain: patient-centric consent for genomic data (PMC8600428) | https://pmc.ncbi.nlm.nih.gov/articles/PMC8600428/ | 学術PMC |
| S8 | Advancing Compliance with HIPAA and GDPR: Blockchain Strategy (PMC12563691) | https://pmc.ncbi.nlm.nih.gov/articles/PMC12563691/ | 学術PMC |
| S9 | Healthcare Policy Compliance: Blockchain Smart Contract Approach (arXiv 2312.10214) | https://arxiv.org/html/2312.10214v1 | 学術arXiv |
| S10 | EDPB Guidelines 02/2025 on processing personal data through blockchain | https://www.edpb.europa.eu/system/files/2025-04/edpb_guidelines_202502_blockchain_en.pdf | 公式規制EDPB 2025（本文は解説2件経由で確認） |
| S11 | Blockchain Framework With ZKP for Genomic Data Sharing (PMC12860433) | https://pmc.ncbi.nlm.nih.gov/articles/PMC12860433/ | 学術PMC |
| S12 | Toward blockchain EHR management with fine-grained ABE and IPFS (PMC12494854) | https://pmc.ncbi.nlm.nih.gov/articles/PMC12494854/ | 学術PMC 2025 |
| S13 | Revocable ABE EHR sharing with multiple authorities in blockchain (PMC9510293) | https://pmc.ncbi.nlm.nih.gov/articles/PMC9510293/ | 学術PMC |
| S14 | Secure EHR via blockchain, dual-ABE, LLM attribute extraction (PMC12979818) | https://pmc.ncbi.nlm.nih.gov/articles/PMC12979818/ | 学術PMC/SciRep 2025 |
| S15 | Patient-centric health data sovereignty using PRE (arXiv 2307.01175) | https://arxiv.org/html/2307.01175 | 学術arXiv |
| S16 | SSX-EHRs: blockchain sharding + dynamic PRE (EURASIP 2025) | https://jis-eurasipjournals.springeropen.com/articles/10.1186/s13635-025-00200-y | 学術Springer 2025 |
| S17 | Revocable blockchain-aided multi-authority ABE escrow-free for EHR (Springer) | https://link.springer.com/article/10.1007/s11227-025-07088-y | 学術Springer 2025 |
| S18 | Secure Key Release with Azure Key Vault and Confidential Computing | https://learn.microsoft.com/en-us/azure/confidential-computing/concept-skr-attestation | 公式docs(Microsoft) |
| S19 | Cryptographic attestation support in AWS KMS | https://docs.aws.amazon.com/kms/latest/developerguide/cryptographic-attestation.html | 公式docs(AWS) |
| S20 | Attestation-Gated Secure Key Release for AI Pipelines (Fortanix) | https://www.fortanix.com/blog/securing-enterprise-applications-and-ai-pipelines-with-attestation-gated-secure-key-release | ベンダーブログ |
| S21 | Confidential Computing on NVIDIA H100 GPUs (NVIDIA Dev Blog) | https://developer.nvidia.com/blog/confidential-computing-on-h100-gpus-for-secure-and-trustworthy-ai/ | 公式ブログ(NVIDIA) |
| S22 | End-to-End Encrypted AI Inference with Post-Quantum Cryptography (Chutes) | https://chutes.ai/news/end-to-end-encrypted-ai-inference-with-post-quantum-cryptography | ベンダー技術仕様 |
| S23 | Enhancing AI Inference Security with Confidential Computing (Red Hat) | https://next.redhat.com/2025/10/23/enhancing-ai-inference-security-with-confidential-computing-a-path-to-private-data-inference-with-proprietary-llms/ | ベンダーブログ |
| S24 | Private Inference (Confer) | https://confer.to/blog/2026/01/private-inference/ | ベンダー技術仕様 |
| S25 | NuCypher KMS: Decentralized key management (arXiv 1707.06140) | https://arxiv.org/pdf/1707.06140 | 学術arXiv |
| S26 | Leveraging Blockchain and PRE for Medical IoT Records (arXiv 2509.08402) | https://arxiv.org/abs/2509.08402 | 学術arXiv 2025 |
| S27 | AWS KMS DeriveSharedSecret (ECDH) API Reference | https://docs.aws.amazon.com/kms/latest/APIReference/API_DeriveSharedSecret.html | 公式docs(AWS) |
| S28 | Azure AI Confidential Inferencing Preview | https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/azure-ai-confidential-inferencing-preview/4248181 | 公式(Microsoft) |
| S29 | Building zero trust GenAI in healthcare with AWS Nitro Enclaves | https://aws.amazon.com/blogs/compute/building-zero-trust-generative-ai-applications-in-healthcare-with-aws-nitro-enclaves/ | 公式(AWS) 2025 |
| S30 | aws-samples: secure medical LLM inference with Nitro Enclaves | https://github.com/aws-samples/sample-for-secure-medical-llm-inference-with-nitro-enclaves | 公式サンプル |
| S31 | Corvex: Confidential Computing Meets NVIDIA HGX B200 | https://www.corvex.ai/blog/confidential-computing-meets-nvidia-hgxtm-b200-secure-ai-without-the-performance-trade-off | ベンダーブログ 2026 |
| S32 | microsoft/confidential-ai inference arch.md | https://github.com/microsoft/confidential-ai/blob/main/inference/docs/arch.md | 公式docs(Microsoft) |
| S33 | Azure AI Confidential Inferencing Technical Deep-Dive | https://techcommunity.microsoft.com/blog/azureconfidentialcomputingblog/azure-ai-confidential-inferencing-technical-deep-dive/4253150 | 公式(Microsoft) |
| S34 | GPU Remote Attestation With Intel Trust Authority | https://docs.trustauthority.intel.com/main/articles/articles/ita/concept-gpu-attestation.html | 公式docs(Intel) |
| S35 | Supercharging Federated Intelligence Retrieval (arXiv 2603.25374) | https://arxiv.org/html/2603.25374 | 学術arXiv 2026 |
| S36 | Confidential Agents for RAG (OPAQUE) | https://www.opaque.co/confidential-agents-for-rag | ベンダーdocs |
| S37 | Timing Side Channels in LLM Serving (arXiv 2409.20002) | https://arxiv.org/html/2409.20002v5 | 学術arXiv |
| S38 | Spill The Beans: CPU Cache Side-Channels on LLMs (arXiv 2505.00817) | https://arxiv.org/html/2505.00817v1 | 学術arXiv 2025 |
| S39 | BudgetLeak: Membership Inference via Generation Budget Side Channel (arXiv 2511.12043) | https://arxiv.org/pdf/2511.12043 | 学術arXiv 2025 |
| S40 | Confidential LLM Inference: Performance and Cost Across CPU/GPU TEEs (arXiv 2509.18886) | https://arxiv.org/abs/2509.18886 | 学術arXiv 2025 |
| S41 | AMD SEV Confidential Computing Vulnerability Bulletin | https://www.amd.com/en/resources/product-security/bulletin/amd-sb-3019.html | 公式セキュリティ(AMD) |
| S42 | BeeKeeperAI: Azure CC + Intel SGX for Healthcare AI (Microsoft) | https://www.microsoft.com/en/customers/story/1503405357498110670-beekeeper-ai-healthcare-microsoft-security-solutions | 事例(Microsoft) |
| S43 | Blockchain-enabled EHR access auditing (PMC11381610) | https://pmc.ncbi.nlm.nih.gov/articles/PMC11381610/ | 学術PMC |
| S44 | Attestable Audits: Verifiable AI Safety Benchmarks Using TEEs (arXiv 2506.23706, ICML 2025) | https://arxiv.org/html/2506.23706v1 | 学術arXiv 2025 |
| S45 | 45 CFR § 164.312 Technical safeguards (Cornell LII) | https://www.law.cornell.edu/cfr/text/45/164.312 | 法令原文 |
| S46 | MedBeads: Agent-Native Immutable Data Substrate for Medical AI (arXiv 2602.01086) | https://arxiv.org/abs/2602.01086 | 学術arXiv 2026 |
| S47 | Efficient Logging/Querying for Blockchain Genomic Access Audit (PMC7372873) | https://pmc.ncbi.nlm.nih.gov/articles/PMC7372873/ | 学術PMC |
| S48 | From BM25 to Corrective RAG: Benchmarking Retrieval Strategies (arXiv 2604.01733) | https://arxiv.org/abs/2604.01733 | 学術arXiv 2026 |
| S49 | Comparing Lexical and Semantic Vector Search for Medical Documents (arXiv 2505.11582) | https://arxiv.org/abs/2505.11582 | 学術arXiv 2025 |
| S50 | FHIR-AgentBench: Benchmarking LLM Agents for Interoperable EHR QA (arXiv 2509.19319) | https://arxiv.org/abs/2509.19319 | 学術arXiv 2025 |
| S51 | LLMonFHIR: Physician-Validated LLM App for Querying EHR (PMC12144420) | https://pmc.ncbi.nlm.nih.gov/articles/PMC12144420/ | 学術PMC/JACC 2025 |
| S52 | Clinical Entity Augmented Retrieval CLEAR (PMC11743751, NPJ Digital Medicine) | https://pmc.ncbi.nlm.nih.gov/articles/PMC11743751/ | 学術PMC/NPJ DM 2025 |
| S53 | EHR-MCP: Clinical Info Retrieval via Model Context Protocol (arXiv 2509.15957) | https://arxiv.org/html/2509.15957v1 | 学術arXiv 2025 |
| S54 | Enhancing Clinical Decision Support via MCP-FHIR Framework (arXiv 2506.13800) | https://arxiv.org/html/2506.13800v1 | 学術arXiv 2025 |
| S55 | EHR-RAG: Bridging Long-Horizon Structured EHR and LLMs (arXiv 2601.21340) | https://arxiv.org/html/2601.21340v1 | 学術arXiv 2026 |
| S56 | Keyword Search is All You Need (aktagon, AWS研究ベース) | https://signals.aktagon.com/articles/2026/02/keyword-search-is-all-you-need-achieving-rag-level-performance-without-vector-databases-using-agentic-tool-use/ | 技術分析(二次) |
| S57 | Hybrid Search in Production: Why BM25 Still Wins (TianPan) | https://tianpan.co/blog/2026-04-12-hybrid-search-production-bm25-dense-embeddings | 技術ブログ(二次) |
| S58 | Why Cursor, Claude Code, Devin Use grep Not Vectors (MindStudio) | https://www.mindstudio.ai/blog/is-rag-dead-what-ai-agents-use-instead | 業界分析(二次) |
| S59 | Vector Search vs Filesystem Tools: 2026 Benchmarks (LlamaIndex公式) | https://www.llamaindex.ai/blog/did-filesystem-tools-kill-vector-search | 公式ベンチマーク |
| S60 | Settling the RAG Debate: Why Claude Code Dropped Vector DB (SmartScope) | https://smartscope.blog/en/ai-development/practices/rag-debate-agentic-search-code-exploration/ | 技術分析(二次) |
| S61 | FHIR-AgentBench (Verily公式) | https://verily.com/perspectives/Introducing-FHIR-AgentBench | 公式(Verily) |

---

*全72件の完全な参照ソース・検索ログは `workers/researcher-1.md`〜`researcher-5.md` および `01-integrated-findings.md` を参照。*
