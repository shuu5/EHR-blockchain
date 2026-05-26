# ブロックチェーン管理EHRをLLMチャットボットに「患者本人の個人カルテ」として渡す仕組みの設計とセキュリティ懸念 — 調査レポート

| 項目 | 内容 |
|---|---|
| 調査日 | 2026-05-26 |
| トピック | ブロックチェーンで管理された電子カルテ(EHR)を、LLMチャットボットに患者個人カルテとしてコンテキスト注入する設計と、そのセキュリティ懸念 |
| 調査方式 | `research:controller-search`（plan→5並列researcher→integrate→critique→report）。重視期間2024-2026 |
| 品質検証 | critic判定 **PASS**（Coverage/Relevance/Citation Accuracy 合格、WARNING 2件＝焦点4は学術中心・焦点5の一部が解説経由、いずれも許容範囲） |
| 一次/公式ソース | OWASP GenAI(LLM Top 10 2025)・NVIDIA/Azure/AWS Confidential AI・HL7 FHIR/SMART on FHIR・個人情報保護委員会・EHDS規則・NIST AI RMF |
| 学術ソース | 2024–2026 の PMC/arXiv/JMIR/medRxiv/Springer/Oxford を中心に 40+ 件 |
| 接続する既存結論 | C1(BC=同意/権限/監査の記録層)・C2(暗号化オフチェーン+CID)・C3(患者/病院ハイブリッド鍵)・C10(LLMは患者説明支援+機密計算が初期本番期)・C11(研究利用は同意前提・対話ログは制度的空白) |

> 信頼度ラベル: `verified`＝一次資料/実証で確認 / `deduced`＝公式資料・複数学術から論理導出 / `inferred`＝状況証拠からの推測 / `uncertain`＝未確認。

---

## エグゼクティブサマリ

1. **患者の全カルテをそのまま渡すことは技術的に不可能**（FHIR全体で平均約300万トークン）であり、**RAG（検索拡張生成）による選択的・最小コンテキスト注入**が支配的設計である。これは偶然にも、HIPAA「最小必要原則」・GDPR「データ最小化」・攻撃面の縮小という**規制・セキュリティ要請と同方向**に収束する（本テーマ最大の設計指針）。[verified]

2. **本テーマ固有の最重要セキュリティ懸念は「取得した患者カルテ本文そのものが攻撃ベクターになる」こと**（間接プロンプトインジェクション）。ブロックチェーンで正当に復号・取得したカルテであっても、その本文にゼロ幅文字や白色テキストで悪意ある指令が仕込まれうる。「取得データ＝信頼境界の外側」として扱う多層防御が必須。[verified]

3. **ブロックチェーンの役割は「コンテキストの供給」ではなく「同意ゲート＋アクセス制御＋改ざん耐性ある監査証跡」**（既存 C1 と一致）。LLMとの最も自然な統合点は「**誰がいつどの患者のカルテをどのLLMに渡したか**」をオンチェーンに不変記録すること。ただし「患者がスマートコントラクトで同意→LLMがオフチェーン暗号カルテを復号してコンテキスト取得」を明示実装した文献は**現時点で不在＝研究フロンティア**。[verified]

4. **推論環境の選択がPHI保護を左右する**。外部消費者向けAPIへのPHI送信は即・規制違反。TEE/Confidential Computing（NVIDIA H100で性能低下4–8%）が「クラウド推論＋PHI保護＋規制当局へのアテステーション証明」を架橋し、C10「2026年初期本番期」と整合。[verified]

5. **日本では制度的空白（C11）が最大の非技術的障壁**。対話ログの法的性質が未定義で、次世代医療基盤法はリアルタイム診療支援に適用されない。当面は院内オンプレ/TEEでPHIを院外に出さず、対話ログを診療録準拠で厳格管理するのが現実的。[verified]

---

## 1. コンテキスト注入の技術

### 1.1 全量注入は不可能、RAGが支配的

患者1人分のFHIRレコードをJSON表現すると**平均約300万トークン**に達し、128K級コンテキストでも収まらない。FHIR-AgentBench（arXiv 2509.19319）は「全FHIRをそのままコンテキストに入れるナイーブ手法は一貫して失敗」と実証し、現状の最高性能エージェントでも正答率は50%止まりである。[verified]

主要2方式とトレードオフ:

| 方式 | 内容 | 適性 |
|---|---|---|
| **コンテキストウィンドウ直接注入** | フィルタ済みサブセットをプロンプトに埋込 | 大規模では「lost in the middle」発生。複雑推論（診断生成）ではRAGとほぼ同等 |
| **RAG（選択的取得）** | 関連チャンクのみベクトル検索/function callingで取得 | 情報抽出・検索で大幅優位（3.75倍超）。トークン削減 |

RAG vs フルコンテキストは**タスク依存**（arXiv 2508.14817）: 情報抽出はRAG優位、時系列分析は中間、複雑推論はフルコンテキストも有効。実務では両者のハイブリッド（識別子リストを注入しLLMが必要分を取得）が有力。[verified]

### 1.2 FHIRのテキスト化・ベクトル化・患者分離

- **JSON→擬似文変換**: FHIR木構造を「パス is 値」形式に変換（例 `Resource type is Observation. ... Value quantity value is 123.6.`）。患者識別子はメタデータとして付加（RAG on FHIR）。[verified]
- **埋め込みモデル**: BGE-M3（多言語・1024次元・512トークン/128オーバーラップ）で97.6%精度、OpenAI text-embedding-3-small（MediGRAF）。[verified]
- **患者ごとデータ分離パターン**（後述のクロスリーク防止に直結）:
  - メタデータフィルタ（共有ストア+患者IDフィルタ。簡易だが混入リスク）
  - グラフノード分離（Neo4jで患者を中心ハブ化＝MediGRAF）
  - アクセス制御付きRAG（権限のある情報のみ使用）
  - フェデレーション型（各機関内にデータ保持）

### 1.3 実装・研究事例（2024-2026）

| 事例 | 種別 | 要点 |
|---|---|---|
| **LLMonFHIR**（2025, JACC Advances） | 臨床医向け | iOS+GPT-4、SMART on FHIR接続、function callingでRAG。28医師・400+会話・64%が毎日使用。医師検証済 |
| **MediGRAF**（2026, Frontiers Digital Health） | 患者QA | ハイブリッドGraph RAG。複雑推論4.25/5、**安全性違反ゼロ** |
| **MCP-FHIR Framework**（2025, arXiv 2506.13800） | OSS | FHIRサーバーをMCP接続、臨床医/介護者/患者の3ペルソナUI |
| **欧州大学病院Epic統合**（2025, PMC12716761） | 臨床医向け | Qwen3-235B+RAG、**GDPR準拠オンプレミス**、20文書/5万文字バジェット |
| **ChatGPT Health**（2025, OpenAI） | 患者向け | Apple Health/EHR接続、FHIR API基盤(b.well)、260+医師協力。技術詳細非公開 |

→ **C10との接続**: これら実装は大半が「患者説明・カルテ理解支援・臨床推論補助」に用途を限定しており、C10「LLMは患者説明支援に寄せる」と整合する。自律診断ではなく補助ツールとしての位置づけが業界標準。[verified]

---

## 2. 主要セキュリティ懸念と緩和策（本テーマの核心）

### 2.1 プロンプトインジェクション（OWASP LLM01:2025、2年連続1位）

**(a) 直接インジェクション** — 攻撃成功率が極めて高い。医療助言文脈の実証（PMC12717619, 2025）で商用LLM全体**94.4%**、極度危害シナリオ（FDA Category X薬）91.7%。GPT-4o-mini/Gemini-2.0-flash-lite=100%脆弱、フラグシップ（GPT-5/Gemini 2.5 Pro/Claude 4.5 Sonnet）でも脆弱性確認。手法はコンテキスト認識インジェクション＋証拠捏造（偽メタ分析で危険処方を正当化）。[verified]

**(b) 間接インジェクション（本テーマ固有・最重要）** — RAGが取得するカルテ・医療文書に悪意ある指令を隠蔽。ユーザーは攻撃ペイロードを入力しない。
- **PDF/カルテポイズニング**: 紹介状・文献にゼロ幅文字や白色テキストで指令を埋込→AI要約時にPHI窃取が発動。
- 腫瘍学AIシステム侵害（2024）: 医療画像・検査結果の隠しプロンプトで診断・治療推奨を改ざん。
- **EchoLeak**（2025, arXiv 2509.10540）: M365 Copilotでゼロクリック PII漏洩を実証。細工メールを送るだけでCopilotが自律的にデータを外部漏洩。医療版では患者紹介情報の無断転送が可能。
- → **ブロックチェーンEHRでも回避できない**: 正当に復号・取得したカルテ本文に攻撃が仕込まれうるため、取得データを「信頼できないコンテンツ」として扱う必要がある。[verified]

**緩和策**: 入力サニタイズ（"ignore previous"検出・ゼロ幅文字除去）／Spotlighting・デリミタ（`---USER TEXT FOLLOWS---`でシステム指示と分離）／コンテンツ分類ゲートキーパー（単純注入の約80%ブロック）／出力検証（許可リスト照合）／最小権限／高影響アクションは人間承認ゲート／配備前レッドチーミング／RAG外部コンテンツの明示的分離。[verified]

### 2.2 推論環境の選択と外部API送信リスク（焦点b）

| 環境 | PHI制御 | コスト | 主リスク |
|---|---|---|---|
| 外部API（BAA未締結・消費者向け） | 最低 | 最低 | 即・規制違反、対話ログの学習利用 |
| マネージド+BAA（Azure OpenAI/AWS Bedrock） | 中 | 中 | 提供者依存、適格エンドポイント確認必須 |
| オンプレ/プライベートVPC | 最高 | 高(CAPEX) | データが境界外に出ない |
| **TEE / Confidential Computing** | 最高(HW保証) | 高+α | 5–15%の「プライバシー税」 |

**TEE実証性能**（C10「2026年初期本番期」と整合）[verified]:
- NVIDIA H100 GPU TEE: スループット低下**4–8%**（70B級でほぼ0%）、TTFT約19%増。
- CPU TEE（Intel TDX/AMD SEV-SNP）: スループット<10%、レイテンシ<20%。
- **AWS Nitro Enclaves**: 医療向けゼロトラスト生成AIの公式実装ガイド（2025）。PHIを暗号化、KMS+アテステーション統合。
- **Azure Confidential Inferencing**: アテステーション付きECDH鍵交換。医療機関が**規制当局にPHI機密性を証明可能**。
- NVIDIA B200（Blackwell）: HBM・NVLINK暗号化でH100の残課題を解消。

→ **推奨**: PHIを含む高リスク推論はオンプレ or TEE。非機密クエリのみ外部API（ハイブリッド）。TEEのアテステーションは「学習に使われない」「指定ポリシーでのみ推論される」をクライアント側で検証する数少ない技術的手段。[deduced]

### 2.3 患者本人認証・なりすまし防止（焦点c）

- **SMART on FHIR**（OAuth2.0+OpenID Connect）が解。スコープ文法（`patient/Observation.read`等）でFHIRリソース単位の粒度認可。LLMをスコープ制限付きSMART on FHIRクライアントとして登録すれば、**認可された患者のカルテのみ**取得できる。PKCE・短命トークン・CSRF保護が必須。[verified]
- 2025-2026動向: 生成AI詐欺損失が2023年123億→2027年400億ドル見込みで患者ポータルが主要標的。1Kosmos×Epic MyChart（NIST IAL2準拠の政府ID+顔バイオメトリクス）、Zoom×World ID（テレヘルス本人認証）が本番導入。MCP仕様に2025-03でOAuth2.1スタイルのAIエージェント認証が追加。[verified]
- **原則**: 最小権限（「現セッションの患者レコードのみクエリ可能なツール」）、セッション属性を認可レイヤーに組込み。エージェントに全EHR書込権限を与えない。[verified]

### 2.4 マルチテナント分離＝クロスパティエント・リーク防止（焦点d）

他患者データの混入は本テーマ固有の致命的懸念。失敗モード: ①`tenant_id`フィルタ未適用、②ノイジーネイバー（他テナント処理がフィルタ優先度を低下）、③共有推論インフラの境界漏洩、④テナント別暗号鍵の隔離不備、⑤ナレッジベース汚染の波及。[verified]

隔離3モデル: **Silo型**（テナント別専用インデックス・最強隔離・高コスト）／**Pool型**（共有+IDフィルタ・効率的だがリスク高）／**Bridge型**（ハイブリッド）。

**緩和策**: テナントIDフィルタの強制（無フィルタクエリを完全禁止）、文書/行レベルアクセス制御を**取得前**に検証、テナント別暗号鍵の隔離、ポリシーベースアクセス制御+監査証跡（MediRAG）。医療では患者単位のSilo型または厳格なフィルタ強制が推奨。[verified]

### 2.5 対話ログ・推論ログの扱い（焦点e、C11と接続）

- 外部APIへのPHI送信は「公開の場で患者ケースを議論するようなもの」。BAA未締結は違反。OpenAIはAPIゼロ保持エンドポイント・ChatGPT Enterprise/for HealthcareがBAA対象、消費者版は対象外。[verified]
- BAA確認事項: スコープ付きBAA・訓練除外条項・プロンプト/出力/**埋め込み(derivative artifacts)**の保持期間と削除保証・データ保存場所。
- HIPAA監査ログ（45 CFR 164.312(b)）: セッションID・ツール名/パラメータ・アクセス患者レコードID・タイムスタンプ・読取/変更区別。PHIを含むLLMログは暗号化・アクセス制御が必要。
- **メンバーシップ推論攻撃**: EHRファインチューニングLLMは特定患者の訓練データ包含を推定されうる。差分プライバシー+フェデレーション学習が推奨だが、464件の医療LLM研究中FL使用1件・準同型暗号0件、約38%がPHI保護措置を未報告。
- → **C11との接続**: 日本では対話ログの法的性質が未定義（診療録か業務ログか不明）。**ログをオンチェーンに本文記録してはならない**（GDPR忘れられる権利・要配慮個人情報と衝突）。記録するのはハッシュ・アクセスメタデータのみ。[verified]

### 2.6 Hallucinationによる医療誤情報（焦点f）

発生率は研究間で**1.47%〜91.4%**と広範（タスク・モデル・評価基準の差。単一値で断定できない）。GPT-4の医療要約で1.47%だが**うち44%がmajor（診断・管理に影響）**、敵対的攻撃下では50〜82%まで上昇。種類は捏造43%・否定30%（計画/評価セクションで最多＝患者安全に直結）。[verified]

**緩和策（ガードレール）**: RAGグラウンディング（検証済み知識ベースで根拠付け）／粒度の細かいFact-checking（CHECK手法で31%→0.3%、※独立検証は未確認）／引用元・根拠表示の義務化／**人間レビュー必須（Human-in-the-loop）**／継続的免責表示／不確実性定量化。GDPR Art.22・C10からも、患者向け医療判断は臨床医レビューを介すべき。[verified]

---

## 3. ブロックチェーン管理との統合（焦点3、C1/C2/C3と接続）

### 3.1 最重要所見: 「同意→復号→コンテキスト取得」ループは研究フロンティア

「**患者がスマートコントラクトでアクセス権を認可→LLMがオフチェーンの暗号化カルテを復号してコンテキスト取得**」という、本テーマがまさに問うパターンを明示的に実装した論文は、2026-05時点で**見つからない**。現状のLLM×ブロックチェーンEHR統合は次の3類型に留まる:[verified]
- (i) LLMを**属性抽出エンジン**として使う（コンテキスト受信者でない）— ClinicalBERTでABE属性生成（PMC12979818）
- (ii) AIは**フェデレーション学習でデータを動かさない**（Oxford JLB 2025）
- (iii) LLMが**MCP経由でEHRに直接アクセス**するがブロックチェーン非統合（EHR-MCP, arXiv 2509.15957）

→ これは「設計上の未解決課題」であり、本プロジェクトの構想が研究の最前線にあることを意味する。

### 3.2 2025-2026の主要統合事例（接続点となる構成要素）

| 事例 | 鍵管理 | オンチェーン | オフチェーン | 接続 |
|---|---|---|---|---|
| 患者中心FW（arXiv 2511.17464） | 公開鍵ラッピング、EIP-712患者署名 | 暗号コミットメント＋時間制限付き許可 | 暗号化FHIR(S3/IPFS) | C1/C2 |
| EHRChain（PMC12494854） | ABE | SHA256ハッシュ＋CID | IPFS暗号化記録 | C2 |
| **MediChainAI**（PMC12650700） | AES-256-GCM+RSA-2048、同意更新時に鍵ローテーション | 暗号ハッシュ＋AIアクセスイベント | 暗号化記録、TEE推論 | C1/C3/C10 |
| **BXHF**（arXiv 2509.14987） | 準同型暗号 | AI予測＋説明(SHAP)の暗号証明 | 機関内暗号化記録 | C1（監査） |
| MedBeads（arXiv 2602.01086） | DID署名、SHA-256 | （将来インフラ） | Merkle DAG、CAS | 監査・分離 |

特に **MediChainAI** は「ZKPでデータ非開示のままアクセス妥当性検証→AI推論前の同意ゲート」「スマートコントラクトがHIPAA/GDPR準拠を検証してからモデル呼出（BXHF）」という、**本テーマに最も近い同意ゲーティング**を示す。[verified]

### 3.3 オン/オフチェーン分担と鍵管理（C1/C2/C3を裏付け）

- **オンチェーン**（記録層）: 暗号コミットメント(ハッシュ)、CID/IPFS参照、患者同意・許可イベント（時間制限付）、**アクセスログのハッシュ（誰がいつどのカルテをLLMに渡したか）**、属性識別子・アクセスポリシー、AI説明の暗号証明。← **C1を直接裏付け**。
- **オフチェーン**: 暗号化EHR本体、**LLM推論処理**、鍵生成・管理の一部、ABE/ZKP等の重い暗号演算。← **C2を直接裏付け**。
- **鍵管理**: Multi-Authority ABE（複数KGCで部分鍵分散・エスクローフリー）が「患者/病院の権限分離」を概念的に実現するが、**C3の「患者鍵+病院鍵の二者分散」をLLM統合と組合せた厳密実装は未確認**[deduced]。HSMの明示的なEHR×ブロックチェーン学術統合事例も不在（業界製品では存在）。

---

## 4. 規制・プライバシー上の留意点（焦点4、C11と接続）

### 4.1 HIPAA（米）
- **最小必要原則**: プロンプトがPHIを含めば規制対象。APIゲートウェイ・推論キュー・キャッシュ・ベクトルDB・出力チャネル全層がPHI保護対象。
- **BAA必須**: PHIを扱うAIベンダー全員。2025年改正で暗号化が「mandatory」(FIPS 140-3)、監査ログ6年保存、AIエージェントに固有IDトークン必須（共有APIキー不可）。

### 4.2 GDPR/EU
- Art.9（健康データ=特別カテゴリ、明示的同意）、**Art.22（自動処理のみの医療決定を禁止＝Human-in-the-loop必須）**、Art.17（忘れられる権利→対話ログ・埋め込み・FTデータの削除が難題）。EU AI Act（2025-08施行）はハイリスク医療AIに透明性・監査証跡・説明可能性。EHDS（2025/327）は二次利用オプトアウト・AI訓練制限・2029-03適用。

### 4.3 日本（C11と直結）
- **個情法**: 病歴・診療事実=要配慮個人情報。ChatGPT等への入力は27条1項違反の第三者提供（本人同意なし）。海外サービスは28条も適用。「名前を消せば匿名化」は誤り（提供元基準説）。
- **次世代医療基盤法**: オプトアウト方式だが**対象は認定事業者の研究開発目的**。リアルタイム診療支援LLMへの直接適用は明示なし＝**制度的空白**。
- **対話ログの法的性質が未定義**（診療録か個人データ取扱記録か業務ログか）。個情委2023-06通達は「訓練利用されないことの確認」を求めるが保存・開示・削除指針は未整備。2027年「医療情報特別法」まで空白継続見込み。
- → **C11を全面的に裏付け**。当面の現実解: (a)院内オンプレ/TEEでPHIを院外に出さない、(b)対話ログを診療録準拠で厳格管理、(c)研究二次利用は明示同意orオプトアウト要件を個別確認。

---

## 5. 設計考察 — 推奨アーキテクチャと多層防御

### 5.1 推論場所の3類型とトレードオフ

| 案 | PHI安全性 | コスト | 機能性 | 規制適合 |
|---|---|---|---|---|
| **セルフホスト型**(LLaMA/Mistral/MedAlpaca) | 最高 | 最大($50K-200K+) | 中 | 最適 |
| **HIPAAクラウド+BAA**(Azure/AWS/GCP) | 高(テナント分離) | 中 | 高(最新モデル) | 良(日本法要確認) |
| **TEE/Confidential Computing** | 最高(HW保証) | 高+α(5-15%税) | 高 | 良＋アテステーション証明 |
| 非識別化+外部API | 中(再識別リスク) | 低 | 低(精度低下) | 条件付き |
| 公開LLM(BAA無) | 最低 | 最低 | 高 | 不可 |

**本テーマの推奨**: 日本の制度的空白とPHI機微性を踏まえ、**院内オンプレ または TEE付きプライベートクラウド**で推論し、**Metadata-First設計**（生PHIでなく構造化コード・統計・タスク限定コンテキストを渡す）を基本とする。[deduced]

### 5.2 推奨統合アーキテクチャ（ブロックチェーン＋LLM＋多層防御）

```
[患者] --SMART on FHIR/OAuth2(本人認証・スコープ認可)--> [認可レイヤー]
   |                                                          |
   |  EIP-712署名で同意                          スマートコントラクト(同意/権限検証ゲート)
   v                                                          v  (HIPAA/GDPR/同意をSCで検証=BXHF型)
[ブロックチェーン: 記録層=C1]  <--ハッシュ/CID/許可イベント/アクセスログハッシュ--
   |  オンチェーンには本文を置かない(=C2)
   v
[オフチェーン暗号化EHR(IPFS/S3)=C2] --PRE/ABEで復号権付与(C3)--> 
   v
[TEE内 推論環境(=C10機密計算)]
   ├ Layer0 入力前: PHIマスキング/最小必要フィルタ(ABAC)
   ├ RAG: 患者単位Silo分離 + 取得前アクセス制御 + Distance threshold
   ├ Layer2 推論時: 取得カルテを"信頼境界外"扱い(spotlighting) + PII入出力フィルタ
   └ Layer3 出力・監査: PHI漏洩監視 + Human-in-the-loop + アクセス事実をオンチェーン監査(=C1)
```

### 5.3 多層防御（Defense-in-Depth）チェックリスト

| レイヤー | 対策 | 対応する懸念 |
|---|---|---|
| Layer 0（入力前） | PHIマスキング、ポリシープロキシで最小必要コンテキスト、ABAC | 2.2/4.1 最小必要 |
| Layer 1（RAG） | 患者単位Silo分離、取得前アクセス制御、Provenance、Distance threshold、埋め込み逆変換対策 | 2.4 クロスリーク |
| Layer 2（推論時） | 取得データを信頼境界外扱い、spotlighting、Jailbreak検出、PII入出力フィルタ | 2.1 間接PI |
| Layer 3（出力・監査） | PHI漏洩監視、改ざん防止監査ログ（オンチェーンハッシュ）、Human-in-the-loop、インシデント対応 | 2.5/2.6 ログ・hallucination |
| 認証・認可 | SMART on FHIRスコープ、最小権限、AIエージェント固有ID | 2.3 なりすまし |
| 推論環境 | オンプレ/TEE、BAA、ZDR、アテステーション | 2.2 外部送信 |
| 参照FW | NIST AI RMF、OWASP LLM Top 10、ISO/IEC 42001、仮名化ワークフロー(arXiv 2505.08728) | 全般 |

### 5.4 各セキュリティ懸念への緩和策マッピング（要約）

| 懸念 | 第一防御 | 補完防御 |
|---|---|---|
| (a)プロンプトインジェクション | 取得データの信頼境界外扱い+spotlighting | 入力サニタイズ、出力検証、レッドチーミング |
| (b)外部送信 | オンプレ/TEE推論 | BAA、ZDR、PrivateLink、アテステーション |
| (c)なりすまし | SMART on FHIRスコープ認可 | NIST IAL2本人確認、MFA、最小権限 |
| (d)クロスリーク | 患者単位Silo分離+取得前アクセス制御 | テナントフィルタ強制、暗号鍵隔離 |
| (e)ログの扱い | ハッシュのみオンチェーン+本文はオフチェーン暗号化 | 訓練除外契約、削除保証、診療録準拠管理 |
| (f)hallucination | RAGグラウンディング+Human-in-the-loop | 引用根拠表示、Fact-checking、免責表示 |

---

## 6. 未解決事項・追加調査候補

| # | 未解決事項 | 影響 | 備考 |
|---|---|---|---|
| 1 | 「患者SC同意→LLM復号→コンテキスト取得」の明示実装 | 焦点3 | **真性の研究フロンティア**。2026以降の設計課題 |
| 2 | C3「患者鍵+病院鍵の二者分散」をLLM統合と組合せた厳密実装 | C3 | MA-ABEで概念的には可能だが実装事例なし |
| 3 | 対話ログの法的性質（日本）・診療録該当性 | C11 | 個情委/厚労省の公式判断待ち。2027年特別法を要観察 |
| 4 | 日本語EHR特有の埋め込み手法・攻撃ベクター | 焦点1,2 | 学術的に希薄 |
| 5 | 「学習に使わない」ベンダー主張の顧客側技術検証 | 焦点3 | TEEアテステーションが部分的に該当。決定的ガイドライン未確立 |
| 6 | HSMのEHR×ブロックチェーン×LLM統合の実装 | C5/C3 | 業界製品はあるが学術統合事例なし |
| 7 | クロスパティエントリークの実医療システム実証 | 焦点2 | 研究倫理上非公開。原理的に入手困難 |

---

## 結論

ブロックチェーン管理EHRをLLMチャットボットに患者個人カルテとして渡す構想は、**技術的には RAG＋最小コンテキスト＋TEE推論＋SMART on FHIR認可** で実装可能性が見えており、これは規制（最小必要・データ最小化）とセキュリティ（攻撃面縮小）の要請と同方向に収束する。**ブロックチェーンは既存 C1 のとおり「同意ゲート＋アクセス制御＋改ざん耐性ある監査証跡」の記録層**として機能し、実データと推論はオフチェーン（C2）・院内TEE（C10）で扱うのが妥当である。

最大のセキュリティ懸念は **「正当に取得したカルテ本文そのものが間接プロンプトインジェクションの媒体になる」** ことであり、取得データを信頼境界の外側として扱う多層防御が不可欠。最大の非技術的障壁は **日本の制度的空白（C11：対話ログの法的性質未定義・リアルタイム診療支援への法不適用）** であり、当面はPHIを院外に出さない院内完結型が現実解である。

そして、本構想の核心である「患者の同意に基づきLLMが暗号化カルテを復号してコンテキスト取得する」エンドツーエンドの統合は、**現在の学術文献に明示実装が存在しない研究フロンティア**である。構成要素（EIP-712同意・ZKP同意ゲート・ABE/PRE復号権付与・TEE推論・オンチェーン監査）は出揃っており、それらを統合する設計こそが本プロジェクトの貢献余地となる。

---

## Evidence Table（主要ソース・抜粋）

| # | Title | URL | Type |
|---|-------|-----|------|
| 1 | Implementation of LLMs in EHR（欧州Epic/Qwen3, GDPR準拠オンプレ） | https://pmc.ncbi.nlm.nih.gov/articles/PMC12716761/ | 学術PMC 2025 |
| 2 | LLMonFHIR: Physician-Validated LLM Mobile App for Querying EHR | https://pmc.ncbi.nlm.nih.gov/articles/PMC12144420/ | 学術PMC/JACC 2025 |
| 3 | MediGRAF: Hybrid Graph RAG for Safe Clinical AI Patient QA | https://arxiv.org/html/2602.00009 | 学術 Frontiers/arXiv 2026 |
| 4 | FHIR-AgentBench: Benchmarking LLM Agents for EHR QA | https://arxiv.org/html/2509.19319v2 | arXiv 2025 |
| 5 | Evaluating RAG vs Long-Context Input for Clinical Reasoning over EHRs | https://arxiv.org/html/2508.14817v1 | arXiv 2025 |
| 6 | RAG on FHIR (Sam Schifman) | https://medium.com/@samschifman/rag-on-fhir-29a9771f49b6 | 実装ブログ |
| 7 | RAG-Based EMR Chatbot System | https://pmc.ncbi.nlm.nih.gov/articles/PMC12370418/ | 学術PMC 2025 |
| 8 | Open-Source MCP-FHIR Framework | https://arxiv.org/html/2506.13800 | arXiv 2025 |
| 9 | OpenAI launches ChatGPT Health | https://www.fiercehealthcare.com/ai-and-machine-learning/openai-launches-chatgpt-health-connect-data-health-apps-medical-records | 業界ニュース 2025 |
| 10 | OWASP LLM01:2025 Prompt Injection（公式） | https://genai.owasp.org/llmrisk/llm01-prompt-injection/ | 公式 OWASP |
| 11 | OWASP LLM Top 10 2025 解説 (Oligo) | https://www.oligo.security/academy/owasp-top-10-llm-updated-2025-examples-and-mitigation-strategies | 技術解説 |
| 12 | Vulnerability of LLMs to Prompt Injection in Medical Advice（成功率94.4%） | https://pmc.ncbi.nlm.nih.gov/articles/PMC12717619/ | 学術PMC 2025 |
| 13 | AI Prompt Injection in Healthcare (Clearwater) | https://clearwatersecurity.com/blog/ai-prompt-injection-in-healthcare/ | 技術ブログ |
| 14 | Indirect Prompt Injection in RAG Systems (AquilaX) | https://aquilax.ai/blog/indirect-prompt-injection-rag-agents | 技術ブログ |
| 15 | EchoLeak: Zero-Click Prompt Injection Exploit | https://arxiv.org/html/2509.10540 | arXiv 2025 |
| 16 | Privacy Challenges & Solutions in RAG-Enhanced LLMs for Healthcare | https://arxiv.org/pdf/2511.11347 | arXiv 2025 |
| 17 | Multi-Tenant RAG 2026: Secure Implementation (Mavik Labs) | https://www.maviklabs.com/blog/multi-tenant-rag-2026 | 技術ブログ |
| 18 | Security Challenges of LLM in Multi-Tenant SaaS | https://cybersecurityjournal.info/archive/security-challenges-of-llm-integration-in-multi-tenant-saas-threats-vulnerabilities-and-mitigations | 技術論文 |
| 19 | Framework for Clinical Safety & Hallucination Rates | https://pmc.ncbi.nlm.nih.gov/articles/PMC12075489/ | 学術PMC |
| 20 | Adversarial Hallucination Attacks in Clinical Decision Support | https://www.medrxiv.org/content/10.1101/2025.03.18.25324184v1 | medRxiv 2025 |
| 21 | Mount Sinai: Hallucination comparison across 6 LLMs | https://www.healthcareitnews.com/news/garbage-garbage-out-mount-sinai-experts-compare-hallucinations-across-6-llms | 医療ITメディア |
| 22 | Harm Reduction for LLM Use in Medicine | https://www.jmir.org/2025/1/e75849 | 学術JMIR 2025 |
| 23 | Mitigating Hallucinations in Healthcare LLMs | https://arxiv.org/pdf/2512.16189 | arXiv 2025 |
| 24 | Confidential LLM Inference: Performance & Cost Across CPU/GPU TEEs | https://arxiv.org/abs/2509.18886 | 学術 arXiv 2025 |
| 25 | Confidential Computing on NVIDIA H100 GPU: Benchmark Study | https://arxiv.org/html/2409.03992v2 | 学術 arXiv |
| 26 | Building zero trust GenAI in healthcare with AWS Nitro Enclaves（公式） | https://aws.amazon.com/blogs/compute/building-zero-trust-generative-ai-applications-in-healthcare-with-aws-nitro-enclaves/ | 公式AWS 2025 |
| 27 | Confidential AI - Azure Confidential Computing（公式） | https://learn.microsoft.com/en-us/azure/confidential-computing/confidential-ai | 公式Microsoft |
| 28 | SMART on FHIR Explained (Descope) | https://www.descope.com/learn/post/smart-on-fhir | 技術ブログ |
| 29 | Healthcare builds new identity infrastructure (Biometric Update) | https://www.biometricupdate.com/202605/healthcare-builds-new-identity-infrastructure-as-fraud-and-interoperability-pressures-grow | ニュース 2026 |
| 30 | Agentic AI in healthcare: secure LLMs with tool access (Aptible) | https://www.aptible.com/hipaa-ai-security/agentic-ai-security | 技術ガイド |
| 31 | Is OpenAI HIPAA Compliant? BAAs & Secure Alternatives | https://www.accountablehq.com/post/is-openai-hipaa-compliant-current-status-baas-and-secure-alternatives | 技術ブログ |
| 32 | Considerations for Patient Privacy of LLMs in Health Care: Scoping Review | https://pmc.ncbi.nlm.nih.gov/articles/PMC12680930/ | 学術PMC |
| 33 | Exploring Membership Inference Vulnerabilities in Clinical LLMs | https://arxiv.org/pdf/2510.18674 | 学術 arXiv 2025 |
| 34 | Secure EHR access control via blockchain, dual-ABE, LLM attribute extraction | https://pmc.ncbi.nlm.nih.gov/articles/PMC12979818/ | 学術PMC/SciRep 2025 |
| 35 | MedBeads: Agent-Native Immutable Data Substrate for Medical AI | https://arxiv.org/html/2602.01086v1 | arXiv 2026 |
| 36 | A Patient-Centric Blockchain Framework (Decoupling Storage from Access Control) | https://arxiv.org/abs/2511.17464 | arXiv 2025 |
| 37 | SSX-EHRs: cross-domain EHR sharing w/ sharding & dynamic PRE | https://jis-eurasipjournals.springeropen.com/articles/10.1186/s13635-025-00200-y | 学術Springer 2025 |
| 38 | Toward blockchain EHR mgmt w/ fine-grained ABE (EHRChain) | https://pmc.ncbi.nlm.nih.gov/articles/PMC12494854/ | 学術PMC/SciRep 2025 |
| 39 | Ethical AI in Healthcare: ZKP + Smart Contracts (MediChainAI) | https://pmc.ncbi.nlm.nih.gov/articles/PMC12650700/ | 学術PMC 2025 |
| 40 | Patient-centric federated learning: consent automation w/ smart contracts | https://academic.oup.com/jlb/article/12/1/lsaf003/8123125 | 学術Oxford JLB 2025 |
| 41 | Blockchain-Enabled Explainable AI for Trusted Healthcare (BXHF) | https://arxiv.org/html/2509.14987v1 | arXiv 2025 |
| 42 | EHR-MCP: Clinical Info Retrieval by LLMs via Model Context Protocol | https://arxiv.org/pdf/2509.15957 | arXiv 2025 |
| 43 | HIPAA Compliance AI: Using LLMs Safely in Healthcare (TechMagic) | https://www.techmagic.co/blog/hipaa-compliant-llms | 技術ブログ |
| 44 | Best Practices for HIPAA Compliant AI & LLMs (EdenLab) | https://edenlab.io/blog/hipaa-compliant-ai-best-practices | 技術ブログ |
| 45 | AI Agents and HIPAA: Solving the PHI Access Challenge (Kiteworks) | https://www.kiteworks.com/hipaa-compliance/ai-agents-hipaa-phi-access/ | 技術ブログ |
| 46 | Navigating Medical GDPR in the Age of AI (Questa AI) | https://www.questa-ai.com/privacy-cafe/navigating-medical-gdpr-in-the-age-of-ai | 技術ブログ |
| 47 | GDPR's Right Not to Be Subject to Automated Decision-Making | https://pmc.ncbi.nlm.nih.gov/articles/PMC11347939/ | 学術PMC |
| 48 | EHDS: Five Key Takeaways on Secondary Use of Health Data | https://www.insideprivacy.com/digital-health/ehds-series-1-five-key-take-aways-on-secondary-use-of-health-data/ | 法律事務所 |
| 49 | European Health Data Space Regulation Published (Arnold & Porter) | https://www.arnoldporter.com/en/perspectives/advisories/2025/03/european-health-data-space-regulation-published | 法律事務所 2025 |
| 50 | ChatGPT等の生成AIに患者個人情報を入力してはいけない理由 | https://note.com/taichi_endoh/n/n3392180a0f0f | 専門家記事 |
| 51 | 生成AIを医療機関で使うときの情報セキュリティ上の注意点 (Henry) | https://note.com/henry_app/n/nc1e01817365c | 実務ブログ |
| 52 | 医療DX・医療データ法制の最新動向（長島・大野・常松） | https://www.nagashima.com/publications/publication20251203-1/ | 法律事務所 2025 |
| 53 | オプトインからオプトアウトへ—次世代医療基盤法 (CIO) | https://www.cio.com/article/4098568/ | 技術メディア |
| 54 | 生成AIサービスの利用に関する注意喚起（個人情報保護委員会・公式） | https://www.ppc.go.jp/news/careful_information/230602_AI_utilize_alert/ | 公式・一次 2023 |
| 55 | Securing RAG: A Risk Assessment and Mitigation Framework | https://arxiv.org/html/2505.08728v2 | 学術 arXiv 2025 |
| 56 | Enterprise AI Security Framework 2025 (Enkrypt AI) | https://www.enkryptai.com/blog/enterprise-ai-security-framework-2025-securing-llms-rag-and-agentic-ai | 技術ブログ |
| 57 | AI Risk Management in Healthcare: NIST AI RMF (Meditology) | https://www.meditologyservices.com/ai-risk-management-in-healthcare/ | 技術ブログ |
| 58 | AI Chatbots and HIPAA Compliance Challenges | https://pmc.ncbi.nlm.nih.gov/articles/PMC10937180/ | 学術PMC |

*（全72件の完全なEvidence Tableは `01-integrated-findings.md` を参照）*
