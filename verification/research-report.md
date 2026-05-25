# 電子カルテ(EHR)×ブロックチェーン結論文書 — 主張検証レポート

| 項目 | 内容 |
|---|---|
| 調査日 | 2026-05-25 |
| 対象文書 | `source/emr_blockchain_conclusion_verified.md` |
| 検証対象 | 外部事実 3点（F1–F3）＋ 過去会話由来の設計結論 11点（C1–C11） |
| 調査方式 | `research:controller-search`（plan→5並列researcher→integrate→critique→report）。F1はcontrollerが厚労省公式ページを直接補完確認 |
| 品質検証 | critic判定 **PASS**（Coverage/Relevance/Citation Accuracy 合格） |
| 一次/公式ソース | 厚労省・HL7・NIST・WHO・EDPB・理研・e-Gov ほか |
| 学術ソース | 2024–2026 の systematic review / arXiv / PMC / Springer / JMIR を中心に 60+ 件 |

---

## エグゼクティブサマリ

検証した14主張の判定分布は以下の通り。

| 判定 | 件数 | 主張 |
|---|---|---|
| ✅ 裏付けあり | 9 | F2, F3, C1, C2, C3, C5, C6, C7, C9 |
| ✅ 裏付けあり（要注意フラグ付き） | 1 | F1（第6.1版が目前） |
| 🟡 部分的裏付け | 2 | C8, C10（※C10は説明支援=裏付けあり／CC=部分的裏付けの複合） |
| 🟡🔴 複合（裏付けあり＋検証困難）＋要注意 | 1 | C11（診療記録=裏付けあり／対話ログ=検証困難／同意前提に齟齬） |
| — | — | C4（裏付けあり：研究面 高・実用面 中） |

**総括**: 本文書の中核アーキテクチャ主張（ブロックチェーンを記録層に限定し、実データは暗号化オフチェーン保存、FHIRブリッジで段階導入、HSM/分散鍵管理）は、2024–2026の学術・業界・規制動向と**高い整合性**を持ち、おおむね妥当である。特にC1（記録層用途）・C9（価値は標準化データ活用）は、近年の「ブロックチェーン医療hype批判」の学術コンセンサスとむしろ強く一致しており、文書の慎重な姿勢（設計結論と外部事実の峻別、未検証部分の明示）は高く評価できる。

一方、**3点に注意喚起が必要**:
1. **F1**: 厚労省ガイドラインは2026-05-25時点では依然「第6.0版」が最新だが、**第6.1版（案）が2026年3月提示・パブコメ中・5月公表予定**。文書は「第6.1版が目前」の注記を要する。
2. **C8**: 「提供先をIPFSクラスター不参加にできる」点を医療文脈で正面から論じた学術文献は未確認（PRE＋別チャネル鍵配布の技術的妥当性自体は裏付けあり）。
3. **C11**: 日本の次世代医療基盤法は**オプトアウト方式**であり、「患者同意を前提に」が積極的インフォームドコンセントを意味するなら法制度の実態と齟齬。かつLLM対話ログの研究利用は**制度的空白**。

---

## 検証方法

- 文書が明示する情報源区分を尊重し、**外部事実（F1–F3）**は一次資料での裏付け、**設計結論（C1–C11）**は技術的妥当性＋2024–2026動向＋学術/業界コンセンサス度＋既知の反例の4軸で評価した。
- 判定ラベル: **裏付けあり** / **部分的裏付け** / **反証・異論あり** / **要更新（古い）** / **検証困難**。
- 信頼度は一次資料・査読論文・systematic review を上位に置き、業界ブログ・市場予測は補助とした。

---

## Part 1: 外部事実の検証（F1–F3）

### F1 — 厚労省「医療情報システムの安全管理に関するガイドライン 第6.0版」
**判定: ✅ 裏付けあり（⚠️ 要注意：第6.1版が目前）**

- 厚労省公式ページに掲載の最新正式版は**第6.0版（令和5年5月 = 2023年5月公表）**。概説編・経営管理編・企画管理編・システム運用編の4編構成。日本で医療情報システムを扱う際の公的安全管理基準であることは事実として確認（controllerが2026-05-25にmhlw.go.jp公式ページを直接確認）。
- → 文書の「第6.0版が存在し公的基準である」は**裏付けあり**。2026-05-25時点では「6.0版が最新」も依然正しい。
- ⚠️ **ただし**: **第6.1版（案）**が2026年3月17日に「健康・医療・介護情報利活用検討会」で提示され（新規に「保守委託機関編」を追加）、e-Govでパブリックコメント中、**2026年5月公表予定**。近日中に第6.0版は最新でなくなる見込み。文書には「第6.1版が目前」の注記を推奨。
- 出典: [厚労省 第6.0版](https://www.mhlw.go.jp/stf/shingi/0000516275_00006.html) / [第6.1版案 システム運用編](https://www.mhlw.go.jp/content/10808000/001675755.pdf) / [e-Gov パブコメ](https://public-comment.e-gov.go.jp/pcm/detail?CLASSNAME=PCMMSTDETAIL&id=495250498&Mode=0) / [CBnews](https://www.cbnews.jp/news/entry/20260319122428)

### F2 — FHIR（Fast Healthcare Interoperability Resources）
**判定: ✅ 裏付けあり**

- 正式名称 "Fast Healthcare Interoperability Resources"、HL7が策定・維持管理する医療情報交換標準であることをHL7公式で確認。
- Resourceがモジュール単位（"The basic building block in FHIR is a Resource"、157種類）、RESTful設計・HTTP（GET/POST/PUT/PATCH/DELETE）、JSON（`application/fhir+json`）・XML（`application/fhir+xml`）、OAuth推奨（SMART on FHIR）——**全項目をHL7公式一次資料で裏付け**。
- 補足: FHIRはRESTを必須としない（"you do not have to use REST to make use of resources"）。「API重視設計」は正確だが、RESTが唯一の実装方式ではない点に留意。
- 出典: [HL7 FHIR Overview](https://www.hl7.org/fhir/overview.html) / [HTTP仕様](https://www.hl7.org/fhir/http.html) / [Security仕様](https://www.hl7.org/fhir/security.html) / [Resource List](https://www.hl7.org/fhir/resourcelist.html)

### F3 — HSM（Hardware Security Module）のNIST定義
**判定: ✅ 裏付けあり（定義が複数存在）**

- NIST CSRC用語集に2系統の定義:
  - **SP 800-57 Part 2 Rev.1**: 「暗号鍵を保護・管理し暗号処理を提供する物理デバイス」（耐タンパ性の明示なし）
  - **SP 1800-16B/C/D**: 「tamper-evident（耐タンパ性）かつ intrusion-resistant（侵入耐性）の鍵等の保護・管理＋暗号処理を提供する物理デバイス」（耐タンパ性・侵入耐性を明記）
- 文書の記述「鍵保護・管理＋暗号処理＋耐タンパ性・侵入耐性」は、NISTの複数定義の要素を組み合わせたものとして裏付けられる。
- 出典: [NIST CSRC Glossary: HSM](https://csrc.nist.gov/glossary/term/hardware_security_module_hsm)

---

## Part 2: 設計結論の検証（C1–C11）

### C1 — ブロックチェーンは記録層（同意・権限・監査）として使う
**判定: ✅ 裏付けあり** ｜ コンセンサス: **強い（学術＋規制）**

- 2024–2025の複数学術論文（arXiv 2511.17464, PMC 12494854 ほか）が一貫してこの設計を採用。32研究のsystematic review（Blockchain in Healthcare Today）が「同意管理・安全なデータ共有・改ざん耐性監査証跡をオフチェーン＋FHIR/SMART-on-FHIRと組合せ」と確認。
- **規制的裏付け（強力）**: EDPB 2025年4月ガイドラインが「公開ブロックチェーンへの個人データ保存はGDPR第17条（忘れられる権利）と原理的に矛盾」と明示 → 医療記録本体をオンチェーンに置かない設計の最強根拠。
- 反例・課題: スマートコントラクトへの同意エンコード複雑性、Ethereumガス代。
- 出典: [arXiv 2511.17464](https://arxiv.org/pdf/2511.17464) / [systematic review](https://blockchainhealthcaretoday.com/index.php/journal/article/view/471) / [EDPB/OMFIF](https://www.omfif.org/2025/06/european-data-protection-board-puts-blockchain-at-a-gdpr-crossroads/)

### C2 — 実データは暗号化オフチェーン保存、ブロックチェーンにはCID参照と権限のみ
**判定: ✅ 裏付けあり** ｜ コンセンサス: **強い（支配的パターン）**

- 調査した全論文がこのパターンを採用。EHRChain（PMC 12494854, Scientific Reports 2025）はABE暗号化→IPFSにCID→オンチェーンアンカリングで**248 TPS / 412ms**を実測。MDPI 2025がon/off-chainトレードオフを整理し「ハイブリッド最善」と結論。
- 反例・課題: **IPFSはピン留めしないとGCでデータ消失**（「分散保存＝永続保存」の誤解リスク）。集中型IPFS運用時のプライバシー懸念（PMC 11978318）。
- 出典: [PMC 12494854](https://pmc.ncbi.nlm.nih.gov/articles/PMC12494854/) / [MDPI 2025](https://www.mdpi.com/2076-3417/15/6/3225) / [HIPAA Vault](https://www.hipaavault.com/resources/blockchain-integration-healthcare-records/)

### C3 — 患者単独鍵より患者・病院ハイブリッド鍵管理が現実的
**判定: ✅ 裏付けあり** ｜ コンセンサス: **高い**

- 患者単独鍵の「鍵紛失問題」は学術・業界双方で広く認識。MedRec（MIT）の限界として「秘密鍵を失うとアクセス権回復メカニズムがない」と文献明示。
- 日本固有文脈（高齢化65歳以上27.7%、EHR普及率34.4%、ブロックチェーン開発者不足）がハイブリッドの必要性を支持（PMC 6996742, JMIR 2020）。
- 反例: 病院がキーカストディアンになると患者プライバシーが形骸化（実質病院管理）し設計複雑性も増す。患者/病院の具体的割り振りは流動的。
- 出典: [arXiv 2511.17464](https://arxiv.org/pdf/2511.17464) / [PMC 6996742](https://pmc.ncbi.nlm.nih.gov/articles/PMC6996742/)

### C4 — MPC・閾値署名・プロキシ再暗号化で分散管理
**判定: ✅ 裏付けあり** ｜ コンセンサス: **研究面 高 / 実用面 中**

- PRE（プロキシ再暗号化）のEHR応用は2024–2025に集中（ScienceDirect 2024, Springer 2024×2, EURASIP 2025）。患者秘密鍵を開示せず復号権付与、閾値PRE（PB-TPRE）でプロキシ信頼集中を緩和。
- MPC: 2024年11月に欧州で初の国際臨床MPC研究。NISTが2026年1月にNIST IR 8214Cで閾値スキーム標準化を進行（提出期限2026年5月）。
- 反例・課題: MPCは「severely underused・計算的に複雑」で理論-実用ギャップ大。**確認できたMPC医療応用の多くは「共同データ分析」用途で、「患者の鍵管理」への直接適用は限定的** → 文書C4がどちらの用途を想定するかで評価が変わる（後述「残課題」）。
- 出典: [ScienceDirect 2024](https://www.sciencedirect.com/science/article/abs/pii/S1389128624006832) / [EURASIP 2025](https://link.springer.com/article/10.1186/s13635-025-00200-y) / [NIST閾値暗号](https://csrc.nist.gov/projects/threshold-cryptography) / [Bio-IT World 2024](https://www.bio-itworld.com/news/2024/11/20/secret-shares-of-patient-health-data-enable-secure-multiparty-research)

### C5 — 病院側の鍵管理にHSMが有力
**判定: ✅ 裏付けあり** ｜ コンセンサス: **非常に高い（業界標準）**

- HIPAA準拠ガイドで「秘密鍵はHSMから出てはならない」明記、FIPS 140-3 Level 3推奨。HSM市場は2024年37.3億→2033年72.2億USD（CAGR 7.6%）で医療・ライフサイエンスが最高成長セグメント。Futurex CryptoHub（2024）でクラウド/ハイブリッド化進展。
- 反例ほぼなし。課題: オンプレHSMは中小病院にコスト負担（Cloud HSMで緩和）、HA構成が必要。
- 出典: [HIPAA Vault](https://www.hipaavault.com/resources/blockchain-integration-healthcare-records/) / [Healthcare HSM Guide](https://www.accountablehq.com/post/healthcare-hsm-deployment-guide-architecture-hipaa-compliance-and-best-practices) / [HSM市場](https://www.marketsandmarkets.com/Market-Reports/hardware-security-modules-market-162277475.html)

### C6 — 既存EHR導入は全面置換でなくFHIRブリッジ型
**判定: ✅ 裏付けあり** ｜ コンセンサス: **強い（業界・学術・政策の三方向）**

- PubMed 2025論文（ID 40588950）がEAVレガシーEHR向けFHIRファサードのOSS実装を実証「大規模再構築なしの段階的対応」。全面置換は「数百万ドル＋重大ダウンタイム」。
- 米21世紀キュア法・欧eHDSも「既存システムへのFHIR APIレイヤー追加」を要求。FHIR採用率2021年49%→2024年64%→2025年71%見込み。日本も厚労省/デジタル庁の標準型電子カルテ戦略がFHIR API連携型を採用、2030年普及率100%目標。
- 反例・課題: FHIR実装非統一（ベンダー差）、臨床試験データ要素の51%しか完全サポートされず、API利用料（年最大$15,000）、FHIR専門人材不足。
- 出典: [PubMed 40588950](https://pubmed.ncbi.nlm.nih.gov/40588950/) / [標準型電子カルテ（ウィーメックス）](https://www.phchd.com/jp/medicom/park/tech/ehr-karte-standardization) / [FHIR adoption 2025](https://mirth-fhir.com/fhir-adoption-is-surging-what-2025-data-tells-us-about-healthcare-interoperability/)

### C7 — 台帳の分散複製と実データ（IPFS等）冗長化を別レイヤーで分離設計
**判定: ✅ 裏付けあり** ｜ コンセンサス: **高い（暗黙の標準）**

- PMC 9183171（EHRChain）が最も明示的: EHRを「少なくとも3つのIPFSノード」に保存しランサムウェア耐性確保、台帳複製とIPFS冗長化を分離。5層アーキ論文（arXiv 2402.17342）「生データのオンチェーン保存は非効率・非実用的」、Springer 2025が「ブロックチェーン=アクセス制御/ログ、IPFS=実データ冗長保存」と役割分担。
- 「分離すべき」と明言した論文は少数だが、**全論文が当然の前提として分離設計を採用**（台帳ノード数≠IPFSノード数）。
- 反例・課題: CID参照先のIPFSデータ消失時の整合性保証（ピン留め管理）が未解決の実装が多い。
- 出典: [PMC 9183171](https://pmc.ncbi.nlm.nih.gov/articles/PMC9183171/) / [arXiv 2402.17342](https://arxiv.org/html/2402.17342v2) / [Springer 2025](https://link.springer.com/article/10.1007/s44163-025-00564-7)

### C8 — 他院共有はクラスター不参加・ゲートウェイ経由取得＋別チャネル復号権限
**判定: 🟡 部分的裏付け** ｜ コンセンサス: **中**

- **裏付けられる部分**: PRE＋別チャネル鍵配布は複数論文で確認（PMC 9183171, arXiv 2509.08402, ScienceDirect 2024/2025）。中間者に内容を見せず再暗号化で復号権付与。IPFS HTTP Gatewayはクラスター非参加でもCID経由で取得可能（IPFS仕様）。
- **弱い部分**: 「**受信側がIPFSクラスターに参加しない構成**」を**医療文脈で正面から論じた学術論文は未確認**。技術的妥当性は支持されるが、医療実装としての明示的裏付けは弱い。
- 反例・課題: 公開ゲートウェイ使用時のアクセスパターン漏洩。PMC 11513489のクロスチェーン実験でクロスレート1.0時に成功率約40%へ低下。PRE実装は信頼できるプロキシの確保が別途必要。
- 出典: [PMC 9183171](https://pmc.ncbi.nlm.nih.gov/articles/PMC9183171/) / [arXiv 2509.08402](https://arxiv.org/abs/2509.08402) / [Inter-hospital PRE (ScienceDirect 2025)](https://www.sciencedirect.com/science/article/abs/pii/S0010482525008133) / [PMC 11513489](https://pmc.ncbi.nlm.nih.gov/articles/PMC11513489/)

### C9 — 真の価値は保存技術でなく標準化された高品質データの活用
**判定: ✅ 裏付けあり** ｜ コンセンサス: **強い（hype批判の学術コンセンサス）**

- ブロックチェーン医療hype批判は学術界で確立: 複数systematic review（2024-2025）が「大多数はPoC止まり、本番稼働は極めて少ない（実装率38%等）」。「標準化フォーマットなしにはブロックチェーンの核心的優位が損なわれる」「医療業界はブロックチェーンを誇大宣伝にもかかわらず信頼できない技術と見なす」と学術誌が明示。
- FHIR二次利用価値は実証（systematic review 49研究、相互運用性11%→66%）。「ブロックチェーン不要」論も学術界に存在（信頼できる第三者で対応可なら不要）。
- ⚠️ Gartner 2024 Hype Cycle（Web3/ブロックチェーンが幻滅期）は**ペイウォールのため二次情報依存・要確認**。ただしC9判定はGartner単独でなく複数systematic reviewで同結論のため影響軽微。
- 反例・補完: 医薬品サプライチェーン追跡（偽造防止）など保存技術自体に固有価値を持つ領域も存在。「標準化（FHIR）とブロックチェーンは補完的」という統合論もある。→ **C9はEHR文脈では妥当**だが「あらゆる医療領域で保存技術に価値なし」とまでは言えない。
- 出典: [PMC 12071524](https://pmc.ncbi.nlm.nih.gov/articles/PMC12071524/) / [arXiv 2304.04101](https://arxiv.org/pdf/2304.04101) / [PMC 10213639](https://pmc.ncbi.nlm.nih.gov/articles/PMC10213639/) / [Do you need a blockchain? (tertiary review)](https://www.explorationpub.com/Journals/edht/Article/101114) / [FHIR research SR](https://pmc.ncbi.nlm.nih.gov/articles/PMC9346559/)

### C10 — LLMは「医療機関管理下の患者説明支援」に寄せる＋Confidential Computing
**判定: ✅ 裏付けあり（患者説明支援）／🟡 部分的裏付け（Confidential Computing）**

- **患者説明支援限定（裏付けあり・強い国際コンセンサス）**: 複数systematic review（2024-2025）が「LLMは医療専門家の補助、自律診断の代替に不適」。CDS機能のLLMはFDA/EU-MDR規制対象 → 説明支援に留めると規制リスク回避。WHO 2024ガイダンス。診断リスクも実証（退院サマリー変換で18%に安全懸念、ハルシネーション20%超）。
- **Confidential Computing（部分的裏付け）**: arXiv 2509.18886でCPU TEEスループット低下10%以下、GPU TEE（H100）4-8%——技術的に実証済み。Red Hat 2025がTEE内のみ復号を実装。**ただし医療本番デプロイ事例は限定的（2025年は萌芽期〜移行期）**。
- 反例: 一部ベンチでLLM診断精度が医師超え（AMIE等）だが臨床リスク管理では限定的。医師の自動化バイアス。
- 出典: [PMC 11554522](https://pmc.ncbi.nlm.nih.gov/articles/PMC11554522/) / [JMIR e71916](https://www.jmir.org/2025/1/e71916) / [WHO 2024](https://www.who.int/news/item/18-01-2024-who-releases-ai-ethics-and-governance-guidance-for-large-multi-modal-models) / [arXiv 2509.18886](https://arxiv.org/abs/2509.18886) / [Red Hat 2025](https://next.redhat.com/2025/10/23/enhancing-ai-inference-security-with-confidential-computing-a-path-to-private-data-inference-with-proprietary-llms/)

### C11 — 研究利用は同意前提で診療記録＋LLM対話ログを仮名化して段階収集
**判定: ✅ 裏付けあり（診療記録の仮名化収集）／🔴 検証困難（LLM対話ログ）／⚠️ 要注意（同意前提に齟齬）**

- **診療記録の仮名化研究利用（裏付けあり）**: 日本の次世代医療基盤法（2024年4月施行、2023年改正）が「仮名加工医療情報」を創設、希少疾患名削除不要で研究価値高。2025年2月に理研が初認定。脱識別化にLLM併用も技術的に可能。
- ⚠️ **要注意（同意前提の齟齬）**: 同法は**オプトアウト方式**（事前同意不要・停止請求可）。文書C11の「患者同意を前提に」が**積極的インフォームドコンセント**を意味するなら、法制度の実態（オプトアウト）と齟齬する。dynamic consentが推奨されつつある。
- 🔴 **LLM対話ログ（検証困難・制度的空白）**: 次世代医療基盤法の対象「医療情報」に対話ログが含まれるか法文上不明確、個情法上のカテゴリも行政解釈未確立。→ この構想の実装には追加の制度整備が必要。
- 反例・課題: 脱識別化の実施品質にばらつき（98.7%が有効性未評価）。医療AI研究の31.9%が同意状態を未報告。
- 出典: [改正次世代医療基盤法 Deloitte](https://www.deloitte.com/jp/ja/Industries/life-sciences/analysis/jisedaiiryou-202404.html) / [厚労省 改正概要](https://www.mhlw.go.jp/content/10808000/001166476.pdf) / [理研初認定](https://www.riken.jp/pr/news/2025/20250228_1/index.html) / [JMIR e76571](https://www.jmir.org/2025/1/e76571) / [PMC 12350686 脱識別化](https://pmc.ncbi.nlm.nih.gov/articles/PMC12350686/)

---

## Part 3: 横断的所見

1. **規制が文書の中核設計を後押し**: EDPB 2025がGDPR第17条との矛盾を理由に「個人データのオンチェーン保存」を否定 → C1/C2（オフチェーン＋ポインタ）の最強の規制的根拠。日本の次世代医療基盤法はC11（研究二次利用）の制度基盤。
2. **「hype批判」と文書の慎重姿勢が一致**: 学術界は2024-2026に「ブロックチェーン医療はPoC止まり、価値はデータ標準化にある」という批判的コンセンサスに収束。文書のC9（価値は標準化データ活用）・FHIR重視はこの潮流と整合し、むしろ先進的。
3. **技術的妥当性 ≠ 実装成熟度**: C4（MPC等）・C8（PRE/ゲートウェイ）・C10（Confidential Computing）は技術的妥当性は高いが、医療本番運用事例は限定的。文書が「設計結論」「有望な設計方向」と位置づけ、事実と峻別している点は正確。
4. **日本固有の論点**: HSM（C5）・ハイブリッド鍵（C3）は欧米基準（HIPAA/FIPS）での裏付けは強固だが、厚労省ガイドライン上のHSM要件の明示や国内導入事例は本調査では確認できず。

---

## Part 4: 残課題・追加調査候補

| # | 残課題 | 影響する主張 | 備考 |
|---|---|---|---|
| 1 | 第6.1版の正式公表確認（2026年5月以降） | F1 | パブコメ中・5月公表予定。公表後は文書記述の更新が必要 |
| 2 | 「患者同意前提」vs オプトアウト方式の整理 | C11 | 積極的IC前提か否かで法制度適合性が変わる（最重要） |
| 3 | LLM対話ログの「医療情報」該当性・行政解釈 | C11 | 個情委/厚労省の解釈待ち。制度的空白 |
| 4 | クラスター非参加ゲートウェイモデルの医療文脈裏付け | C8 | 技術的妥当性はあるが医療実装の明示論文なし |
| 5 | MPCの「鍵管理用途」vs「データ分析用途」の区別 | C4 | 文書の想定用途を明確化すると評価が確定 |
| 6 | 日本国内のHSM・ハイブリッド鍵の実装事例 | C3, C5 | 欧米基準の裏付けは強固だが国内事例は未確認 |
| 7 | Confidential Computingの医療本番デプロイ事例 | C10 | 2025年は萌芽期。今後の事例蓄積を要観察 |

---

## 主張判定 一覧表（F1–F3, C1–C11）

| 主張ID | 判定 | 根拠の要点 | 主要出典URL |
|---|---|---|---|
| **F1** | ✅ 裏付けあり（⚠️要注意：第6.1版が目前） | 第6.0版（2023年5月）が2026-05-25時点の公式最新版。第6.1版案が2026年3月提示・5月公表予定 | https://www.mhlw.go.jp/stf/shingi/0000516275_00006.html |
| **F2** | ✅ 裏付けあり | 正式名称・HL7策定・Resource単位・REST/JSON/XML/HTTP/OAuthを全項目HL7公式で確認（RESTは必須でない点に補足） | https://www.hl7.org/fhir/overview.html |
| **F3** | ✅ 裏付けあり（定義複数） | NIST CSRC用語集に2系統。SP1800-16で耐タンパ性・侵入耐性を明記、SP800-57は明示なし | https://csrc.nist.gov/glossary/term/hardware_security_module_hsm |
| **C1** | ✅ 裏付けあり | 学術systematic review＋EDPB（GDPR第17条との矛盾）が記録層用途を強く支持 | https://www.omfif.org/2025/06/european-data-protection-board-puts-blockchain-at-a-gdpr-crossroads/ |
| **C2** | ✅ 裏付けあり | 2024-2025論文の支配的パターン（ABE暗号化→IPFS CID→オンチェーン）。IPFSピン留め課題あり | https://pmc.ncbi.nlm.nih.gov/articles/PMC12494854/ |
| **C3** | ✅ 裏付けあり | 患者単独鍵の鍵紛失問題は広く認識。日本の高齢化・低EHR普及率がハイブリッドを支持 | https://pmc.ncbi.nlm.nih.gov/articles/PMC6996742/ |
| **C4** | ✅ 裏付けあり（研究高/実用中） | PREのEHR応用は2024-2025多数、MPCは欧州初臨床研究・NIST標準化中。鍵管理用途への直接適用は限定的 | https://link.springer.com/article/10.1186/s13635-025-00200-y |
| **C5** | ✅ 裏付けあり | HIPAA/FIPS準拠でHSMが事実上の業界標準。医療セクターが最高成長。反例ほぼなし | https://www.accountablehq.com/post/healthcare-hsm-deployment-guide-architecture-hipaa-compliance-and-best-practices |
| **C6** | ✅ 裏付けあり | FHIRファサード実装論文＋米欧規制＋日本政策の三方向で強コンセンサス。実装非統一が課題 | https://pubmed.ncbi.nlm.nih.gov/40588950/ |
| **C7** | ✅ 裏付けあり | 全論文が台帳複製とIPFS冗長化を分離前提で設計（EHRChainは3+ IPFSノード）。明示論文は少数 | https://pmc.ncbi.nlm.nih.gov/articles/PMC9183171/ |
| **C8** | 🟡 部分的裏付け | PRE＋別チャネル鍵配布は裏付けあり。「クラスター不参加」を医療文脈で論じた論文は未確認 | https://www.sciencedirect.com/science/article/abs/pii/S0010482525008133 |
| **C9** | ✅ 裏付けあり | 「hype止まり・価値は標準化データ」が学術コンセンサス。医薬品SCM等の反例も併記。Gartnerは要確認 | https://pmc.ncbi.nlm.nih.gov/articles/PMC12071524/ |
| **C10** | ✅ 裏付けあり（説明支援）/🟡 部分的裏付け（CC） | 説明支援限定は強い国際コンセンサス。Confidential Computingは技術実証済みだが医療本番は萌芽期 | https://www.jmir.org/2025/1/e71916 |
| **C11** | ✅ 裏付けあり（診療記録）/🔴 検証困難（対話ログ）/⚠️同意前提に齟齬 | 仮名加工医療情報は次世代医療基盤法で裏付け（但しオプトアウト方式）。LLM対話ログは制度的空白 | https://www.deloitte.com/jp/ja/Industries/life-sciences/analysis/jisedaiiryou-202404.html |

---

## Evidence Table（主要ソース・抜粋）

| # | Title | URL | Type |
|---|---|---|---|
| 1 | 厚労省 医療情報システム安全管理GL 第6.0版 | https://www.mhlw.go.jp/stf/shingi/0000516275_00006.html | 公式（厚労省） |
| 2 | 第6.1版（案）システム運用編 | https://www.mhlw.go.jp/content/10808000/001675755.pdf | 公式・草案 |
| 3 | e-Gov パブコメ（第6.1版案） | https://public-comment.e-gov.go.jp/pcm/detail?CLASSNAME=PCMMSTDETAIL&id=495250498&Mode=0 | 公式（e-Gov） |
| 4 | HL7 FHIR Overview | https://www.hl7.org/fhir/overview.html | 公式（HL7） |
| 5 | HL7 FHIR Security仕様 | https://www.hl7.org/fhir/security.html | 公式（HL7） |
| 6 | NIST CSRC Glossary: HSM | https://csrc.nist.gov/glossary/term/hardware_security_module_hsm | 公式（NIST） |
| 7 | A Patient-Centric Blockchain Framework for Secure EHR Management (arXiv 2511.17464) | https://arxiv.org/pdf/2511.17464 | 学術（2025） |
| 8 | Toward blockchain based EHR management with fine-grained ABE (PMC 12494854) | https://pmc.ncbi.nlm.nih.gov/articles/PMC12494854/ | 学術（Sci Rep 2025） |
| 9 | On-Chain/Off-Chain Blockchain Storage trade-offs (MDPI Appl Sci 2025) | https://www.mdpi.com/2076-3417/15/6/3225 | 学術（2025） |
| 10 | Addressing the Challenges of EHRs Using Blockchain and IPFS (PMC 9183171) | https://pmc.ncbi.nlm.nih.gov/articles/PMC9183171/ | 学術 |
| 11 | Blockchain Technology for Enhancing Clinical Data Management: SLR (32研究) | https://blockchainhealthcaretoday.com/index.php/journal/article/view/471 | Systematic Review |
| 12 | EDPB puts blockchain at a GDPR crossroads (OMFIF) | https://www.omfif.org/2025/06/european-data-protection-board-puts-blockchain-at-a-gdpr-crossroads/ | 規制（EDPB 2025） |
| 13 | A Scalable Multi-Layered Blockchain Architecture for EHR Sharing (arXiv 2402.17342) | https://arxiv.org/html/2402.17342v2 | 学術（2024） |
| 14 | A secure interoperable method for EHR exchange on cross-platform blockchain (PMC 11513489) | https://pmc.ncbi.nlm.nih.gov/articles/PMC11513489/ | 学術（2024） |
| 15 | Inter-hospital secure data exchange using PRE and blockchain (ScienceDirect 2025) | https://www.sciencedirect.com/science/article/abs/pii/S0010482525008133 | 学術（2025） |
| 16 | Leveraging Blockchain and PRE to secure Medical IoT Records (arXiv 2509.08402) | https://arxiv.org/abs/2509.08402 | 学術（2025） |
| 17 | Examining Blockchain for 21st-Century Japanese Health Care (PMC 6996742) | https://pmc.ncbi.nlm.nih.gov/articles/PMC6996742/ | 学術（2020・日本） |
| 18 | Multi-authority ABE PRE for EHR Sharing in Web 3.0 (ScienceDirect 2024) | https://www.sciencedirect.com/science/article/abs/pii/S1389128624006832 | 学術（2024） |
| 19 | SSX-EHRs: cross-domain EHR sharing with sharding and dynamic PRE (EURASIP 2025) | https://link.springer.com/article/10.1186/s13635-025-00200-y | 学術（2025） |
| 20 | NIST Multi-Party Threshold Cryptography Project | https://csrc.nist.gov/projects/threshold-cryptography | 公式（NIST） |
| 21 | Healthcare HSM Deployment Guide (AccountableHQ) | https://www.accountablehq.com/post/healthcare-hsm-deployment-guide-architecture-hipaa-compliance-and-best-practices | 業界専門 |
| 22 | Blockchain Integration for Healthcare Records: HIPAA-Compliant 2025 (HIPAA Vault) | https://www.hipaavault.com/resources/blockchain-integration-healthcare-records/ | 業界ガイド |
| 23 | HSM Market Report 2025-2030 (MarketsandMarkets) | https://www.marketsandmarkets.com/Market-Reports/hardware-security-modules-market-162277475.html | 市場調査 |
| 24 | Bridging the Legacy Interoperability Gap: FHIR Facade (PubMed 40588950) | https://pubmed.ncbi.nlm.nih.gov/40588950/ | 学術（2025） |
| 25 | 標準型電子カルテの現状と今後（ウィーメックス 2026） | https://www.phchd.com/jp/medicom/park/tech/ehr-karte-standardization | 業界（日本） |
| 26 | FHIR adoption is surging: 2025 data (Mirth) | https://mirth-fhir.com/fhir-adoption-is-surging-what-2025-data-tells-us-about-healthcare-interoperability/ | 業界レポート |
| 27 | Challenges of Blockchain Applications in Digital Health: SR (arXiv 2304.04101) | https://arxiv.org/pdf/2304.04101 | 学術（2023） |
| 28 | A Systematic Literature Review for Blockchain-Based Healthcare Implementations (PMC 12071524) | https://pmc.ncbi.nlm.nih.gov/articles/PMC12071524/ | 学術（2025） |
| 29 | Comprehensive review: healthcare data quality challenges in blockchain (PMC 10213639) | https://pmc.ncbi.nlm.nih.gov/articles/PMC10213639/ | 学術 |
| 30 | FHIR for Interoperability in Health Research: SR (PMC 9346559, 49研究) | https://pmc.ncbi.nlm.nih.gov/articles/PMC9346559/ | 学術（SR） |
| 31 | Do you need a blockchain in healthcare data sharing? (tertiary review) | https://www.explorationpub.com/Journals/edht/Article/101114 | Tertiary review |
| 32 | Hype Cycle for Web3 and Blockchain, 2024 (Gartner)（要確認・ペイウォール） | https://www.gartner.com/en/documents/5623191 | 業界レポート |
| 33 | Large language models in patient education: scoping review (PMC 11554522) | https://pmc.ncbi.nlm.nih.gov/articles/PMC11554522/ | 学術 |
| 34 | Implementing LLMs in Health Care: Clinician-Focused Review (JMIR 2025 e71916) | https://www.jmir.org/2025/1/e71916 | 学術（2025） |
| 35 | Considerations for Patient Privacy of LLMs in Health Care (JMIR 2025 e76571) | https://www.jmir.org/2025/1/e76571 | 学術（2025） |
| 36 | Confidential LLM Inference across CPU and GPU TEEs (arXiv 2509.18886) | https://arxiv.org/abs/2509.18886 | 学術（2025） |
| 37 | Enhancing AI inference security with confidential computing (Red Hat 2025) | https://next.redhat.com/2025/10/23/enhancing-ai-inference-security-with-confidential-computing-a-path-to-private-data-inference-with-proprietary-llms/ | 技術文書 |
| 38 | WHO AI ethics governance guidance for LMMs (2024) | https://www.who.int/news/item/18-01-2024-who-releases-ai-ethics-and-governance-guidance-for-large-multi-modal-models | 公式（WHO） |
| 39 | 改正次世代医療基盤法（2024年4月施行）Deloitte解説 | https://www.deloitte.com/jp/ja/Industries/life-sciences/analysis/jisedaiiryou-202404.html | 専門家解説 |
| 40 | 次世代医療基盤法 改正内容概要（厚労省 2023年11月） | https://www.mhlw.go.jp/content/10808000/001166476.pdf | 公式（厚労省） |
| 41 | 理研が仮名加工医療情報利用事業者に初認定（2025） | https://www.riken.jp/pr/news/2025/20250228_1/index.html | 公的機関 |
| 42 | Leveraging LLMs for deidentification of health information (PMC 12350686) | https://pmc.ncbi.nlm.nih.gov/articles/PMC12350686/ | 学術（2025） |

*（全71件の完全なEvidence Tableは `01-integrated-findings.md` を参照）*

---

## 結論

本文書の最終結論——**ブロックチェーンを主記録媒体ではなく同意・アクセス制御・監査証跡の台帳層として用い、診療記録本体は暗号化オフチェーン保存、患者・病院ハイブリッド鍵管理（病院側HSM）、FHIRブリッジ型導入、価値は標準化データ活用にある**——は、2024–2026の学術・業界・規制動向に照らして**全体として妥当性が高く、近年の批判的コンセンサスとも整合的**である。文書が外部事実と設計結論を峻別し、未検証部分を明示する姿勢も適切。

更新・注意を要するのは、(1) **F1**（第6.1版が目前 → 近く要更新）、(2) **C8**（クラスター不参加の医療文脈裏付けが弱い）、(3) **C11**（オプトアウト方式との「同意前提」の齟齬＋LLM対話ログの制度的空白）の3点である。
