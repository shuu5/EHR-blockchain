# EHR×ブロックチェーン結論文書 — 要注意4項目 フォローアップ検証レポート

| 項目 | 内容 |
|---|---|
| 調査日 | 2026-05-26 |
| 対象文書 | `source/emr_blockchain_conclusion_verified.md` の要注意4項目（F1・C8・C10・C11） |
| 位置づけ | **差分調査（最新化）**。2026-05-25作成の `verification/research-report.md` を基線とし、それ以降の変化のみを抽出 |
| 調査方式 | `research:controller-search`（plan→4並列researcher→integrate→controller補完→critique→report）。F1は controller が厚労省WGページ・e-Govを直接補完確認 |
| 品質検証 | critic判定 **PASS**（Coverage/Relevance/Citation Accuracy 合格、WARNING 2件は本レポートで反映） |
| 一次/公式ソース | 厚労省（医療等情報利活用WG・e-Gov）・内閣府・個人情報保護委員会・理研・IPFS公式・Microsoft/AWS・Confidential Computing Consortium |
| 学術ソース | 2024–2026 の arXiv / PMC / Scientific Reports / EURASIP を中心に追加確認 |

---

## エグゼクティブサマリ — 4項目の判定変化

| 項目 | 既存判定(2026-05-25) | **今回判定(2026-05-26)** | 変化 |
|---|---|---|---|
| **F1** 厚労省ガイドライン | ✅裏付けあり（⚠️第6.1版が目前） | ✅裏付けあり（⚠️**第7.0版へ版番変更・正式版は5/29 WG審議中**） | 🔄 **最新化（更新あり）** |
| **C8** クラスター不参加共有 | 🟡部分的裏付け | 🟡部分的裏付け（間接裏付け微増・反証なし） | ➖ **変化なし** |
| **C10** 機密計算＋LLM | ✅(説明支援)/🟡部分的(CC) | ✅(説明支援)/🟡→🟢**初期本番期に前進**(CC) | ⬆️ **裏付け強化＋最新化** |
| **C11** オプトアウト/対話ログ | ✅/🔴/⚠️齟齬 | ✅/🔴/⚠️齟齬（**制度見直し議論が進行・空白は継続**） | 🔄 **最新化（更新あり）** |

**総括**: 4項目のうち **3項目（F1・C10・C11）で更新すべき新事実**が確認できた。最も実体的な変化は **C10**（医療機関での機密計算CCの実運用デプロイ事例が出現＝「萌芽期」から「初期本番期」へ）。**F1** は版番が **6.1→7.0** に変わる見込みで文書の注記更新が必須。**C11** はオプトアウトと同意の論点が国の検討会で正面から再検討に入った（2027年法改正に向け流動化）が、現時点で齟齬・空白そのものは解消していない。**C8** のみ実質的変化なし（依然、医療文脈の正面文献は不在だが反証も不在）。

> 信頼度ラベル: `verified`＝一次資料/直接確認で確定 / `deduced`＝公式資料から論理的に導出 / `inferred`＝状況証拠からの推測 / `uncertain`＝未確認。

---

## F1 — 厚労省「医療情報システムの安全管理に関するガイドライン」

### 判定: 🔄 **最新化（更新あり）** ｜ ✅裏付けあり（⚠️版番が第6.1版→第7.0版へ）

**結論**: 文書が依拠する「第6.0版が公的安全管理基準」は引き続き正しい（2026-05-26時点でも公式掲載は第6.0版＝令和5年5月のみ）。ただし改定の進行状況に重要な更新がある。

**確認できた事実と信頼度**:

1. `verified` **正式版（最新の改定版）は2026-05-26時点で未公表**。厚労省ガイドライン掲載ページ（`mhlw.go.jp/stf/shingi/0000516275_00006.html`）を直接確認した結果、掲載は第6.0版のみ。Q&Aは令和7年5月更新が最新。
2. `verified` **2026年3月のパブコメ草案は「第6.1版（案）」**。e-Gov掲載の概要PDF（厚労省医政局医療情報担当参事官室、seqNo=0000310842）のタイトルが「『医療情報システムの安全管理に関するガイドライン第6.1版』（案）について（概要）」。意見募集は2026年3月27日開始、すでに**終了**（結果取りまとめは未公表）。
3. `verified` **5/29のWG議題は「第7.0版（案）」**。第32回 健康・医療・介護情報利活用検討会 医療等情報利活用WG（令和8年5月29日13:00–15:00）の開催案内ページで、議題(5)が「医療情報システムの安全管理に関するガイドライン**第7.0版（案）**について」であることを直接確認。同ページに第6.1版への言及はない。
4. `inferred` **版番が6.1→7.0へ引き上げられた**。3月パブコメ＝6.1版案、5月WG＝7.0版案という一次事実から、パブコメ後に正式版の版番を6.1から7.0へ繰り上げたと推測される（新規「保守委託機関編」追加・編構成の再編という大規模改正に伴うメジャーバージョン化と解釈するのが自然）。ただし「6.1案を7.0に改める」旨を明記した一次文言は取得できておらず、確証ではない。
5. `verified` **新規「保守委託機関編」の内容（草案段階）**。第6.0版の「小規模機関編」「クラウド利用機関編」を統合・再編し、サーバ保守・セキュリティ更新を外部委託する小規模診療所・薬局向けに新設。該当機関は「概説編＋保守委託機関編」のみ対応で全体遵守とみなす。MDS（医療情報取扱機器等セキュリティ確認書）／SDS（サービス仕様開示書）で確認。医療機関の役割を「対策の実行」から「事業者の対策確認・管理」へ転換。
6. `uncertain` **正式告示日・施行日・経過措置は未確定**（5/29 WG後に確定する見込み）。

**文書・HTMLへの反映提案**: F1の注記を「第6.1版が目前」から「**次期版は第7.0版として改定進行中（2026年5月29日のWGで案を審議）。第6.1版は草案段階の呼称で、正式版は第7.0版になる見込み。新規『保守委託機関編』を追加**」へ更新。施行日は未確定と明示。

**出典**: [厚労省GL掲載ページ(第6.0版)](https://www.mhlw.go.jp/stf/shingi/0000516275_00006.html) / [第32回WG開催案内(議題=第7.0版案)](https://www.mhlw.go.jp/stf/shingi/0001210262_00097.html) / [e-Gov 第6.1版(案)概要PDF](https://public-comment.e-gov.go.jp/pcm/download?seqNo=0000310842) / [第29回WG資料](https://www.mhlw.go.jp/stf/newpage_71572.html) / [CBnews(2026-03-19)](https://www.cbnews.jp/news/entry/20260319122428)

---

## C8 — 他院共有＝クラスター不参加・ゲートウェイ経由取得＋別チャネル復号権付与

### 判定: ➖ **変化なし**（🟡部分的裏付けのまま。間接裏付けが微増・反証は依然なし）

**結論**: 2026-05-26時点でも既存判定を変える材料はない。「受信側がIPFSクラスターに参加しない＋HTTPゲートウェイ経由でCID取得＋別チャネルでPRE等により復号権付与」という3要素を**同時に・医療データ共有の文脈で正面から論じた学術文献は依然として不在**（9回検索・8件Fetchで0件）。一方、この構成を否定する反証も見つからなかった。

**確認できた事実と信頼度**:

1. `verified` **核心文献は不在**。3要素を同時に医療文脈で扱う論文は再探索でも0件。
2. `deduced` **間接的な構造的裏付けが微増**:
   - arXiv:2511.17464（2024, "Decoupling Data Storage from Access Control"）＝受信側がクラスター非参加でポインター（IPFS CID／S3 URL）取得する設計。ただし鍵配布はPREでなくECIES公開鍵ラッピング。「受信側クラスター非参加」を医療文脈で扱う最も近い文献。
   - PMC9183171（2022）＝「data requestersはproxy provider経由でのみアクセス可」と明示（フルノード不要を示唆）。
   - PMC11111748（ZK-ROLLUP, 2024）＝「患者ノードは軽量ノード」（参加度の分離を承認）。
3. `verified` **反証は不在**。「クラスター参加が医療で必須」「ゲートウェイ経由は非現実的」とする学術的根拠は見当たらない。
4. `verified` **実装上の留保（裏付けと注意の両面）**: IPFS**公開**ゲートウェイはCID要求のタイムスタンプ・IP・リクエストパターンを可視化する（IPFS公式ドキュメントが明示）。→ 医療応用では**専用（private/dedicated）ゲートウェイ**が前提。専用IPFSゲートウェイ市場は2024年4,080万USD→2032年1.39億USDへ成長予測で、企業採用の60%超を医療・金融・メディアが占めるなど、ゲートウェイ経由アクセスの実用性自体は支持される。

**文書・HTMLへの反映提案**: 判定は「部分的裏付け」を維持。補強情報として「**反証となる文献も不在＝技術的に妥当だが医療実装の正面文献が薄い、という状態が継続**」「**公開ゲートウェイはメタデータ漏洩リスクがあり、医療では専用ゲートウェイが前提**」を追記すると、要注意の理由がより正確になる。

**出典**: [arXiv 2511.17464(2024)](https://arxiv.org/html/2511.17464v1) / [PMC9183171(2022)](https://pmc.ncbi.nlm.nih.gov/articles/PMC9183171/) / [PMC11111748(2024)](https://pmc.ncbi.nlm.nih.gov/articles/PMC11111748/) / [IPFS公式: Privacy & Encryption](https://docs.ipfs.tech/concepts/privacy-and-encryption/) / [Inter-hospital PRE(ScienceDirect 2025)](https://www.sciencedirect.com/science/article/abs/pii/S0010482525008133)

---

## C10 — LLMは「医療機関管理下の患者説明支援」に寄せる＋Confidential Computing

### 判定: ⬆️ **裏付け強化＋最新化** ｜ ✅説明支援限定(不変) / 🟡→🟢 機密計算が「萌芽期」から「初期本番期」へ前進

**結論**: 「LLMは患者説明支援に寄せる」（C10前段）の妥当性は不変（強い国際コンセンサスのまま）。後段の **Confidential Computing/TEE は2025-2026で明確に前進**し、医療機関を巻き込んだ実運用デプロイ事例・クラウドの医療向けCC提供・GPU TEEの本番性能実証が出そろった。ただし「**臨床LLM推論による患者説明支援を機密計算環境で本番運用する**」ピンポイントの事例は依然不在で、現段階の医療CC活用は主に研究用データ共有・モデル訓練/検証である。

**確認できた事実と信頼度**:

1. `verified` **医療機関を巻き込んだCC実運用デプロイの出現（最重要の前進）**: BeeKeeperAIの「EscrowAI」（Azure Confidential Computing + Intel SGX）で、**マウントサイナイ医科大学・モアハウス医科大学**が慢性心不全のマルチモーダルAIモデルを実患者データで検証（2025-05-27発表、**Coalition for Health AI(CHAI)認定下での初の実運用デプロイ**と称する）。データ共有なし・IRB期間を2年超→3-4か月に短縮。ただし段階は「AIモデル性能検証」で、患者向け臨床ワークフロー（患者説明支援等）への直接統合には未到達。
2. `verified` **クラウド医療向けCCの本格化**:
   - Azure: Confidential GPU VMs（NCCads_H100_v5＝AMD SEV-SNP＋NVIDIA H100）が**HIPAA対象サービス**。Confidential Clean Rooms（医療機関間協働）はプレビュー。Azure AI Confidential Inferencing（暗号化プロンプトをTEE内のみで復号）提供（2026-03更新確認）。
   - AWS: 2025-12-12に医療LLM推論（Google MedGemma 4B＋Nitro Enclaves）の**公式リファレンス実装**を公開。ただし「教育・デモ目的のみ、本番・臨床用途には不適」と明記。
3. `verified` **GPU TEE本番性能の実証**: Corvexが2026-03-03、NVIDIA HGX B200上でCC「検証済み本番デプロイ」を達成（Intel TDX＋NVIDIA CC、ネイティブ同等性能）。H100世代の4-8%オーバーヘッドをさらに下回るB200世代で実証。
4. `verified` **学術蓄積**: Scientific Reports Vol.16(2026)に医療連合学習×CC最適化論文（TEEのサイドチャネル脆弱性も指摘）。arXiv:2603.00196(2026-03)が主要クラウド全社のLLM機密推論CCソリューションを臨床ノートデータで網羅レビュー。
5. `verified` **規制・業界**: Confidential Computing Consortiumが2025-12に**規制専門SIG新設**（CCを国際標準へ組込む活動）。GartnerがCCを2026年戦略トレンドに選定。一方、FDA-EMA共同AI 10原則(2026-01-14)は**TEE/CCに直接言及なし**。HIPAA Security Rule改定NPRM（暗号化・MFA事実上必須化）にCCは要件超の手段と業界が位置づけ。
6. `verified` **依然限定的な点**: 「臨床LLM推論を用いた患者説明支援」の直接本番事例は公開情報になし。規制でのTEE義務化もなし。具体的な病院システム（EHR連携）への本番統合の公開事例もなし。

**文書・HTMLへの反映提案**: 判定を「部分的裏付け（医療本番は萌芽期）」から「**移行期〜初期本番期**」へ更新。「医療AIの訓練・検証フェーズではCCの実運用が始まっている（BeeKeeperAI×マウントサイナイ等）が、臨床LLM推論＝患者説明支援の本番運用はなお萌芽段階」と表現するのが正確。

**出典**: [BeeKeeperAI×マウントサイナイ/モアハウス(2025-05-27)](https://www.businesswire.com/news/home/20250527913519/en/) / [AWS Nitro Enclaves医療LLM(2025-12-12)](https://aws.amazon.com/blogs/compute/building-zero-trust-generative-ai-applications-in-healthcare-with-aws-nitro-enclaves/) / [Corvex B200 CC本番(2026-03-03)](https://www.prnewswire.com/news-releases/corvex-among-the-first-companies-to-achieve-verified-production-deployment-of-confidential-computing-for-ai-on-nvidia-hgx-b200-systems-302702992.html) / [Azure Confidential AI](https://learn.microsoft.com/en-us/azure/confidential-computing/confidential-ai) / [CCC 2025-12ニュースレター](https://confidentialcomputing.io/2025/12/31/welcome-to-the-december-2025-newsletter/) / [arXiv 2603.00196(2026-03)](https://arxiv.org/pdf/2603.00196)

---

## C11 — 研究利用は同意前提で診療記録＋LLM対話ログを仮名化収集

### 判定: 🔄 **最新化（更新あり）** ｜ ✅診療記録(不変) / 🔴 LLM対話ログ＝空白継続 / ⚠️ オプトアウトとの齟齬は継続だが制度見直し議論が進行

**結論**: 既存の3層判定（診療記録＝裏付けあり／対話ログ＝検証困難／同意前提に齟齬）は2026-05-26時点でも基本維持。ただし **2025-2026に「同意のあり方」と「AI学習利用の同意要否」の制度見直しが正面から動き出した**ため、流動化として最新化が必要。齟齬・空白そのものは未解消。

**確認できた事実と信頼度**:

**(a) オプトアウト方式と患者同意の齟齬**
1. `verified` **オプトアウト方式は基本維持**（2024-04施行改正でも変更なし。WGでオプトアウト通知の負担軽減は議論されたが方針転換なし）。→ 原典の「患者同意を前提に」が積極的インフォームドコンセントを意味するなら、依然として法制度の実態（オプトアウト）と齟齬する。
2. `verified` **国の検討会で「本人同意の有無」が正面論点化（新規）**: 内閣府が2025-09-03〜12月に「医療等情報の利活用の推進に関する検討会」を計7回開催、2025-12-25に**中間まとめ(案)**提示。論点に「患者の権利保護（本人同意の有無）・セキュリティ確保」が明示（日経報道）。最終まとめは2026年夏、必要なら**2027年通常国会に新法案**。EU EHDS規則（2025-03発効）を対照事例に参照。
3. `verified` **ガイドライン複数改定**: 内閣府ガイドラインは令和7年12月1日・令和8年3月19日・令和8年5月1日（最新）と改定。
4. `verified` **認定の進展**: 2025-02-28に理研が仮名加工医療情報利用事業者に初認定。協力医療機関は2025-02時点152事業者。日医も申請準備。
5. `inferred` Dynamic consentの公式明示はなし（医薬産業政策研究所提言で「マイナポータル連携の双方向コミュニケーション」可能性を示唆する程度）。

**(b) LLM対話ログの研究二次利用**
6. `verified` **次世代医療基盤法上の位置づけは空白継続**: 対話ログが同法の「医療情報」に含まれるかは法文上・ガイドライン上いずれも明示なし（2026-05時点も未解決）。
7. `verified` **個情法上は要配慮個人情報**: 対話ログ中の病歴・診療情報は要配慮個人情報で明示同意が原則（個情委の生成AI注意喚起＝2023-06が最後の公式文書）。
8. `verified/deduced` **個情法改正の方向性（新規）**: 個情委が2025年に「統計情報等の作成（**AI開発等を含む**）にのみ利用が担保される場合は本人同意を不要」とする改正方針案を公表。条件＝委員会規則の公表事項・事業者間合意・目的外利用禁止。「入口規制（同意原則）→出口規制」への転換。**2026年通常国会**提出目標（`uncertain` 実際の提出・審議入りは2026-05-26時点で未確認）。これは医療対話ログのAI学習利用の法的根拠整備に直結しうる。
9. `verified` **実務解釈**（長島・大野・常松, 2025-12）: 「AI学習に保存されないことが契約で担保されればサーバーが国内法適用下にある必要はない」と明確化。
10. `verified` **業界自主ガイドライン**: HAIP-CIP「医療・ヘルスケア分野における生成AI利用ガイドライン第2版」(2025-07)、JADHA「ヘルスケア生成AI活用ガイド第2.0版」(2025-02)。ただし対話ログの研究二次利用の制度的位置づけ明示はなし。

**文書・HTMLへの反映提案**: 「オプトアウトとの齟齬」「対話ログの制度的空白」は**要注意のまま維持**しつつ、「**2025年に国の検討会が同意のあり方を再検討開始（中間まとめ2025-12、最終2026夏、法改正2027目標）**」「**個情法改正でAI学習利用の同意不要化の方向性（2026国会目標）**」という流動化を注記すると、最新の制度状況を正確に反映できる。

**出典**: [内閣府: 医療等情報利活用検討会](https://www8.cao.go.jp/iryou/studygloup/index.html) / [第7回検討会(2025-12-25中間まとめ案)](https://www8.cao.go.jp/iryou/studygloup/20251225/agenda.html) / [次世代医療基盤法 関係法令・GL(内閣府)](https://www8.cao.go.jp/iryou/hourei/hourei.html) / [日経XTECH: AI/医療データ一部同意不要化(個情法改正)](https://xtech.nikkei.com/atcl/nxt/column/18/00001/10256/) / [個情委: 生成AI注意喚起(2023-06)](https://www.ppc.go.jp/news/careful_information/230602_AI_utilize_alert/) / [理研初認定(2025-02-28)](https://www.riken.jp/pr/news/2025/20250228_1/index.html) / [長島・大野・常松(2025-12)](https://www.nagashima.com/publications/publication20251203-1/)

---

## 既存レポートからの差分まとめ（HTML可視化の更新ガイド）

| 項目 | 更新の要否 | 具体的な更新内容 |
|---|---|---|
| **F1** | **要更新** | 「第6.1版が目前」→「次期版は**第7.0版**として改定進行中（5/29 WG審議）、新規『保守委託機関編』追加、施行日未確定」。版番が6.1→7.0に変わる見込みである旨を注記 |
| **C8** | 任意（補強） | 判定は据え置き。「反証文献も不在」「公開ゲートウェイのメタデータ漏洩リスク→医療では専用ゲートウェイ前提」を補強として追記可 |
| **C10** | **要更新** | CC部分を「萌芽期」→「**初期本番期**（医療AI訓練・検証では実運用デプロイ開始、BeeKeeperAI×マウントサイナイ等）」。ただし臨床LLM患者説明支援の本番運用はなお萌芽 |
| **C11** | **要更新** | 齟齬・空白は維持しつつ「2025年に国の検討会が同意のあり方を再検討開始（中間まとめ2025-12/最終2026夏/法改正2027目標）」「個情法改正でAI学習利用の同意不要化の方向性（2026国会目標）」を流動化として注記 |

---

## Evidence Table（本フォローアップで参照した主要ソース）

| # | Title | URL | Type | Item |
|---|---|---|---|---|
| 1 | 厚労省 医療情報システム安全管理GL（現行=第6.0版） | https://www.mhlw.go.jp/stf/shingi/0000516275_00006.html | 一次（厚労省） | F1 |
| 2 | 第32回 医療等情報利活用WG 開催案内（議題=第7.0版案/令和8年5月29日） | https://www.mhlw.go.jp/stf/shingi/0001210262_00097.html | 一次（厚労省） | F1 |
| 3 | 「第6.1版（案）について（概要）」厚労省医政局（e-Gov概要PDF） | https://public-comment.e-gov.go.jp/pcm/download?seqNo=0000310842 | 一次（厚労省/e-Gov） | F1 |
| 4 | 第29回 医療等情報利活用WG 資料（第6.1版案提示） | https://www.mhlw.go.jp/stf/newpage_71572.html | 一次（厚労省） | F1 |
| 5 | CBnews: 医療情報GL第6.1版案を提示（2026-03-19） | https://www.cbnews.jp/news/entry/20260319122428 | 二次（専門メディア） | F1 |
| 6 | Patient-Centric Blockchain: Decoupling Data Storage from Access Control (arXiv 2511.17464, 2024) | https://arxiv.org/html/2511.17464v1 | 学術（arXiv 2024） | C8 |
| 7 | Addressing the Challenges of EHR Using Blockchain and IPFS (PMC9183171, 2022) | https://pmc.ncbi.nlm.nih.gov/articles/PMC9183171/ | 学術（Sensors 2022） | C8 |
| 8 | Integrating blockchain & ZK-ROLLUP via IPFS (PMC11111748, 2024) | https://pmc.ncbi.nlm.nih.gov/articles/PMC11111748/ | 学術（Sci Rep 2024） | C8 |
| 9 | IPFS 公式: Privacy & Encryption | https://docs.ipfs.tech/concepts/privacy-and-encryption/ | 公式（IPFS） | C8 |
| 10 | Inter-hospital secure data exchange using PRE & blockchain (ScienceDirect 2025) | https://www.sciencedirect.com/science/article/abs/pii/S0010482525008133 | 学術（2025） | C8 |
| 11 | SSX-EHRs: cross-domain EHR sharing + dynamic PRE (EURASIP 2025) | https://jis-eurasipjournals.springeropen.com/articles/10.1186/s13635-025-00200-y | 学術（2025） | C8 |
| 12 | BeeKeeperAI×マウントサイナイ/モアハウス 実患者データ検証（2025-05-27） | https://www.businesswire.com/news/home/20250527913519/en/ | 業界PR（2025-05-27） | C10 |
| 13 | Building zero trust GenAI in healthcare with AWS Nitro Enclaves（2025-12-12） | https://aws.amazon.com/blogs/compute/building-zero-trust-generative-ai-applications-in-healthcare-with-aws-nitro-enclaves/ | 公式（AWS） | C10 |
| 14 | Corvex: Verified Production Deployment of CC on NVIDIA HGX B200（2026-03-03） | https://www.prnewswire.com/news-releases/corvex-among-the-first-companies-to-achieve-verified-production-deployment-of-confidential-computing-for-ai-on-nvidia-hgx-b200-systems-302702992.html | 業界PR（2026-03-03） | C10 |
| 15 | Confidential AI - Azure Confidential Computing（2026-03-13更新） | https://learn.microsoft.com/en-us/azure/confidential-computing/confidential-ai | 公式（Microsoft） | C10 |
| 16 | Confidential Computing Consortium December 2025 Newsletter（規制SIG新設） | https://confidentialcomputing.io/2025/12/31/welcome-to-the-december-2025-newsletter/ | 業界コンソーシアム（2025-12） | C10 |
| 17 | Confidential Inference for Cloud-based LLMs (arXiv 2603.00196, 2026-03) | https://arxiv.org/pdf/2603.00196 | 学術（arXiv 2026） | C10 |
| 18 | Cross-institutional medical federated learning driven by CC (Sci Rep 2026) | https://www.nature.com/articles/s41598-026-44843-4 | 学術（Sci Rep 2026） | C10 |
| 19 | FDA-EMA Good AI Practice 10原則（2026-01-14）解説 | https://deepceutix.com/insights/fda-ema-ai-principles | 規制動向解説（2026-01） | C10 |
| 20 | 内閣府: 医療等情報の利活用の推進に関する検討会 | https://www8.cao.go.jp/iryou/studygloup/index.html | 一次（内閣府） | C11 |
| 21 | 第7回検討会 議事次第・配布資料（2025-12-25・中間まとめ案） | https://www8.cao.go.jp/iryou/studygloup/20251225/agenda.html | 一次（内閣府） | C11 |
| 22 | 次世代医療基盤法 関係法令・ガイドライン・通知（内閣府） | https://www8.cao.go.jp/iryou/hourei/hourei.html | 一次（内閣府） | C11 |
| 23 | 次世代医療基盤法に基づく事業者の認定（内閣府） | https://www8.cao.go.jp/iryou/nintei/nintei.html | 一次（内閣府） | C11 |
| 24 | 日経XTECH: AI/医療データ一部同意不要化（個情法改正検討） | https://xtech.nikkei.com/atcl/nxt/column/18/00001/10256/ | 報道（二次） | C11 |
| 25 | 個情委: 生成AIサービスの利用に関する注意喚起（2023-06） | https://www.ppc.go.jp/news/careful_information/230602_AI_utilize_alert/ | 一次（個情委） | C11 |
| 26 | 医療DX・医療データ法制の最新動向（長島・大野・常松, 2025-12） | https://www.nagashima.com/publications/publication20251203-1/ | 専門家解説（二次） | C11 |
| 27 | 理研が仮名加工医療情報利用事業者に初認定（2025-02-28） | https://www.riken.jp/pr/news/2025/20250228_1/index.html | 一次（研究機関） | C11 |
| 28 | HAIP-CIP 医療・ヘルスケア分野における生成AI利用ガイドライン第2版（2025-07） | https://haip-cip.org/news/20250711/ | 業界ガイドライン（二次） | C11 |

---

## 未解決事項・追加調査候補

| # | 未解決事項 | 影響項目 | 備考 |
|---|---|---|---|
| 1 | 第7.0版の正式告示日・施行日・経過措置 | F1 | 5/29 WG後に確定見込み。公表後に要再確認 |
| 2 | 「6.1案→7.0版」版番変更の明示的一次文言 | F1 | 現状 `inferred`。5/29資料・正式告示で確認可能になる見込み |
| 3 | C8核心構成（クラスター不参加＋ゲートウェイ＋PRE）の医療正面文献 | C8 | 反証も不在。技術妥当性はあるが医療実装の正面文献が薄い状態が継続 |
| 4 | 臨床LLM推論（患者説明支援）をTEE内で本番運用する直接事例 | C10 | 現段階の医療CCは研究データ共有・モデル訓練/検証が中心 |
| 5 | 内閣府中間まとめ本文の確定的方向（オプトアウト維持か新同意モデルか） | C11 | PDF未取得。最終まとめ（2026夏）で明確化見込み |
| 6 | 個情法改正法案の2026年通常国会への実際の提出・審議状況 | C11 | 提出目標は確認、提出事実は2026-05-26時点で未確認 |

---

## 結論

要注意4項目の再調査の結果、**3項目（F1・C10・C11）で文書・HTMLの注記更新が必要な新事実**を確認した。

- **F1（最新化）**: 改定は「第6.1版」ではなく「**第7.0版**」として2026年内に進行中（5/29 WGで案を審議）。版番変更の明示的一次文言は未取得だが、6.1版案（3月パブコメ）→7.0版案（5月WG）の一次事実から版番引き上げと判断できる。
- **C8（変化なし）**: 医療文脈の正面文献は依然不在。ただし反証も不在で、間接的な構造的裏付けと専用ゲートウェイの実用化が「部分的裏付け」を下支え。
- **C10（裏付け強化＋最新化）**: 医療機関を巻き込んだ機密計算CCの実運用デプロイが出現し「萌芽期」→「初期本番期」へ前進。ただし臨床LLM患者説明支援の本番運用はなお萌芽段階。
- **C11（最新化）**: オプトアウトとの齟齬・対話ログの制度的空白は継続するが、2025年に国の検討会が同意のあり方を再検討開始し（最終まとめ2026夏／法改正2027目標）、個情法改正でAI学習利用の同意不要化の方向性（2026国会目標）が示され、制度が流動化した。

文書の中核設計（記録層限定・暗号化オフチェーン保存・FHIRブリッジ・HSM/分散鍵）の妥当性評価は本フォローアップでも揺らがない。更新は要注意項目の「最新動向の注記」に限られる。
