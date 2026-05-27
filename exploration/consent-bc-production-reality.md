---
title: 同意・アクセス権・監査をBCで管理しカルテ本体をオフチェーンに置く設計の「本番実態」調査レポート
topic: consent/access-control/audit on-chain, chart-body off-chain — production reality (MedRec/ConsentChain/NABIDH/Kanta/EHDS/SSI, 2024-2026)
created: 2026-05-27
type: research
session: /tmp/research-search-f9177ab5/
critique: PASS
tags:
  - ehr-blockchain
  - consent-management
  - access-control
  - off-chain
  - production-vs-pilot
  - smart-contract
  - fhir-consent
  - ssi-did-vc
  - ehds
  - research-report
---

# 同意・アクセス権・監査をBCで管理しカルテ本体をオフチェーンに置く設計の「本番実態」調査レポート

調査日: 2026-05-27 ｜ 重視期間: 2024-2026 ｜ 品質検証: PASS
調査規模: researcher 5名 × 計50回WebSearch × 49件Fetch成功 + controller補完3検索 ｜ 査読系統的レビュー4本＋横断研究/レビュープロトコル2本 + EU/WHO/W3C等の一次公式情報を参照

---

## 0. 論点の確認（最重要・混同禁止）

本調査の対象は次の**狭く厳密な設計**である:

> **患者の【同意・アクセス権・監査】をブロックチェーン（スマートコントラクト）で管理し、カルテ本体は暗号化してオフチェーンに置く**（consent / access-control / audit on-chain, chart-body off-chain）。

以下は**論点対象外**として明確に区別した:
- ❌ 「カルテ本体をブロックチェーンに格納する」設計（誰も提案しておらず、藁人形）。
- ❌ エストニア KSI ブロックチェーン = 「監査ログのハッシュ完全性証明」。患者同意の付与/剥奪をオンチェーンで行うものではない[24][23]。
- ❌ SUSMED（日本）= 治験のデータ改ざん防止・SDV代替。一般カルテの患者同意・アクセス権制御ではない[10][11][75]。
- ❌ サプライチェーン追跡・保険請求（Avaneer等）・医薬品トレーサビリティ等の医療BC一般[29][33]。

この区別を全ての判定で維持した。

---

## 1. 核心的知見（Executive Summary）

1. **【最重要・verified】当該設計で「実患者データを用いて継続的に本番商用稼働」しているシステムは、2024-2026時点・本調査範囲では世界的に1件も確認できなかった。** これは「確認できた本番稼働ゼロ件」という反証可能な否定的結論であり、独立した複数researcherが同一結論に到達し、複数の査読系統的レビュー（2021[16]・2024[32]・2025[15]）が「BC医療実装の大半はPoC/概念段階、実患者検証済み実装は希少」と明示することで裏付けられる。

2. **【精密な再フレーミング・deduced】「BCで医療は無理」という一般論ではない。** 当該設計は研究レベルでは技術的に成熟しつつある（arXiv 2511.17464 は実装可能な EIP-712 Permission 構造体・ガスコスト実測まで具体化[6]）。問題は**本番商用への「ラスト1マイル」が世界中どこでも未達成**という一点に集約される。

3. **【決定的反例・verified】中央集権型の同意管理は国民規模で本番稼働している。** フィンランド Kanta は330万ユーザー・480万件の同意記録・15年稼働[46][47][48]。日本の全国医療情報プラットフォームも2025年から本格展開で「医療機関ごとの閲覧範囲制限」という粒度の細かい同意を**中央集権・BC不使用**で実装[78][79]。

4. **【最新動向・verified】患者主権アクセス制御の規制・標準の主流（EU EHDS / EUDI Wallet / WHO GDHCN / 米 TEFCA / W3C VC 2.0）はすべて BC を使用していない。** SSI/DID/VC も技術的に BC を必須とせず（did:web/did:key 等）、近年は「BC脱却」傾向が顕著[58][61][67][27]。

5. **【障壁・verified】本番化を阻む構造的障壁が明確。** 最大の壁は GDPR 第17条（削除権）× BC 不変性で、EDPB Guidelines 02/2025 が「暗号化・ハッシュ化データも個人データ」と明示し「ハッシュだけ置けば対象外」論を否定[43][35]。加えて HIPAA 修正権との衝突[39]、Epic/Cerner 統合コスト[41]、鍵管理UX[34]、スケーラビリティ[45]、コンソーシアム・ガバナンス[40]、そして**成熟した代替（SMART on FHIR + OAuth2.0）の存在**[19]。

---

## 2. サブ質問への回答

### Q1. 同意・アクセス権管理BC（カルテ本体オフチェーン型）の具体プロジェクトと到達点

**回答**: 該当プロジェクトは研究レベルで多数存在するが、**到達ステージはPoC・シミュレーション・論文が圧倒的多数**で、実患者データを使ったのは治験特化の2例のみ、本番商用稼働は皆無。

| プロジェクト | 実装方式 | 到達ステージ | 実患者データ | 2024-2026現状 | 信頼度 |
|---|---|---|---|---|---|
| **MedRec (MIT)** [1][2] | Ethereum PoA / Solidity / 各機関ローカルDB（分散オフチェーン）/ go-ethereum | **PoC → 撤退・更新停止** | 不明（BIDMCテストDB接続レベル） | GitHub「NOT MAINTAINED」明記・最終コミット2019年7月・主導者は別分野へ転向 | verified |
| **ConsentChain (豪 JMIR 2021)** [5] | Hyperledger Besu / Solidity 6本 / MongoDB / 公開鍵暗号+ワンタイムトークン | **PoC・論文のみ** | なし（架空データ） | 継続証拠なし | verified |
| **arXiv 2511.17464 (患者中心FW, 2025/11)** [6] | Ethereum Sepolia / Solidity / IPFS or S3 / AES-256-GCM+ECIES。`Permission{expiration,revoked,wrappedKey}`+EIP-712署名、付与78,331gas、L2で10-13倍削減 | **シミュレーション・論文のみ** | なし（Synthea合成FHIR） | 論文発表のみ | verified |
| **Dwarna (マルタ大 2019)** [7][8] | Hyperledger Composer / PostgreSQL（同意変更のみBC）/ WordPress | **PoC・プロトタイプ** | 不明 | 活動証拠なし | deduced |
| **SUSMED (日本)** [10][11][75][76] | 治験IC・SDV不変記録（詳細非公開） | **実患者パイロット→治験領域で部分商用** | あり（治験患者） | 2024年12月NCNP医師主導治験で稼働・DTx治験採用。※治験SDV用途で論点対象外 | verified |
| **カナダHyperledger治験パイロット** [12] | Hyperledger Fabric v1.4 / IBM Blockchain Platform / consent状態+監査ログのみBC | **実患者パイロット（同意管理特化）** | あり（乾癬患者12名/5施設） | 監視時間475→7分・継続/商用化証拠なし | verified |
| **MediChainAI (Jouf大 2025)** [13] | SSI+BC+Merkle tree（詳細限定的） | **シミュレーション・論文のみ** | なし | 論文のみ | deduced |
| **arXiv 2407.21559 (SSI+ABE, 2024)** [14] | BCベース / SSI+DID / コンソーシアムIPFS / 属性ベース暗号化 | **シミュレーション・論文のみ** | なし | 論文のみ | deduced |
| **Medicalchain (英)** [17][18][19] | Ethereum ERC-20(MedToken)/Dual chain | **商用スタートアップ継続も実患者EHR本番展開証拠なし** | 不明（Mayo Clinicは「ごく初期探索」と表明） | 法人存続・実態不明 | uncertain |

**含意**: 実患者で動いた2例（SUSMED・カナダ）はいずれも**治験インフォームドコンセント/SDV特化**であり、一般カルテのアクセス権管理ではない。最も技術的に具体的な arXiv 2511.17464 でも合成データのテストネット評価止まり。フィールド全体として「BC同意管理＋EHRオフチェーン」設計の本番商用稼働は2024-2026時点で確認不能[15]。

### Q2. 実患者データで本番商用稼働した同意管理BCは存在するか（地域別・厳密検証）

**回答: 確認できなかった（ゼロ件）。** 主要候補とされる地域はいずれも (a) BC を使っていない、(b) BC だが同意管理でない（監査のみ）、(c) PoC/パイロット止まり、のいずれかだった。

| 地域/システム | 実態 | 判定 | 信頼度 |
|---|---|---|---|
| **ドバイ NABIDH** [20][21][22] | HL7/FHIR + AI(Imprivata)ベースのHIE。9.47M記録・1,300施設稼働だが、DHA/Kyndryl/DHA公式いずれにも"blockchain"記述ゼロ | **BC不使用**（同意管理BCではない。blockchain-washing疑い） | verified |
| **エストニア KSI** [23][24] | Guardtime KSIで監査ログ完全性証明。スマートコントラクト/患者同意のオンチェーン操作なし | **監査ハッシュ証明であり同意管理BCではない** | verified |
| **韓国 MyHealthWay** [27] | 2023年9月国家稼働のPHR・860機関・8M目標。技術基盤にBC記述なし | **本番稼働だがBC不使用** | verified |
| **中国** [25][26] | Alibaba(常州)/Baidu(重慶)/Tencentが2017-19試験導入。2024年国家ガイドラインは推奨・規格化段階 | **本番継続稼働の一次情報なし** | deduced |
| **米国** [28][29] | MedRec=パイロット止まり。Avaneer Health=保険請求/行政特化（患者同意管理でない）。Gem OSは医療撤退 | **同意管理BCの本番稼働なし** | verified |
| **欧州・豪州・NZ・日本** [15][32][78] | 系統レビューが「本番稼働の広範な証拠なし」。日本のPHRは中央集権でBC不使用 | **該当事例なし** | deduced |

**反証可能性の明示**: 本否定的結論は「50回検索・49件Fetch・査読系統的レビュー複数（2021[16]・2024[32]・2025[15]）・各国公式情報」の範囲で「本番商用稼働の同意管理BCを発見できなかった」という意味である。中国語一次資料・インド Ayushman Bharat 等は未踏査であり、ここから本番事例が将来見つかる可能性は排除しない（後述「未解決事項」）。ただし、英語圏の網羅的系統レビューが一貫して同じ否定的結論を出していることは、結論の頑健性を強く支持する。

### Q3. 本番定着の障壁（なぜPoC/パイロット止まりか）

**回答**: 規制・統合・UX・性能・ガバナンスの5層の障壁が相互に強化し合い、加えて「成熟した非BC代替の存在」が決定打となっている。

1. **GDPR第17条（削除権）× BC不変性 [verified・最重要]** — EDPB Guidelines 02/2025（2025年4月14日採択）が「可能な限りBCに個人データを保存しない」を推奨[43]。決定的に重要なのは EDPB が**「暗号化・ハッシュ化されたデータもGDPR上の個人データであり、GDPRが適用される」**と明示した点で、「ハッシュだけ置けばGDPR対象外」という一般的主張を否定した[35][36]。**Crypto-Shredding（鍵破棄による論理削除）を「削除」と認めるとは明示しておらず**、法的有効性は **uncertain**（2025年6月のコンサルテーション終了後の最終版待ち）[37]。違反制裁は最大2000万ユーロ／全世界売上4%。
2. **HIPAA（米国）[verified]** — 修正権(45 CFR §164.526)がBC不変性と直接衝突[39]。「最小限必要」原則によりパーミッションレス型は不可、パーミッション型のみ現実的。
3. **既存EMR統合 [verified/deduced]** — Epic（Care Everywhere重視・外部統合に消極的）/Cerner(Oracle Health)とのカスタムミドルウェアが必須[41]。実装コストは単一ユースケースで25-75万ドル・4-8ヶ月、マルチ組織は12-24ヶ月超[40]。病院の70%が「統合が最大障壁」と回答[41]。
4. **鍵管理UX [verified]** — 鍵紛失＝永続的データ損失（パスワードリセット不可）[42]。高齢・低デジタルリテラシー層の自己鍵管理は非現実的（GCC医療従事者30名IV研究[34]）。カストディアル/ソーシャルリカバリーで緩和可能だが「分散型自己主権」思想と矛盾。
5. **運用コスト・スケーラビリティ [verified]** — Hyperledger Fabric実測で37-40 TPS、500txでレイテンシ11.2秒[45]。Ethereum系は256ビットごと20,000gas。ただし「同意・アクセス記録のみなら低頻度で影響限定的」との解釈も成立（deduced）。
6. **ガバナンス [verified]** — 多組織合意交渉が技術実装より長期化。初期に参加機関が少なくネットワーク効果が出ない「冷始動問題」[40]。Avaneer Health(Aetna/Anthem/Cleveland Clinic等)も大規模本番稼働の証拠なし。
7. **構造的トリレンマと代替の存在 [deduced]** — 規制適合→設計複雑化→組織間合意コスト増の悪循環。決定打は**SMART on FHIR + OAuth2.0 が既存EMRに組み込み済みで成熟**しており、「わざわざBCを使う必要のある trustless 多機関ユースケース」が実際には限定的なこと。

**両論併記**: BC支持派は「パーミッション型小規模コンソーシアム＋オフチェーン参照設計＋Crypto-Shreddingでこれらは緩和可能」と主張する[35]。懐疑派は「緩和策が分散型の利点を打ち消し、結局中央集権で十分」と反論する[54]。2024-2026の規制動向（EDPB）と本番不在の実態は、後者により有利な証拠を加えている。

### Q4. 中央集権型の同意管理との比較 — BCでなければ不可能なことは何か

**回答**: 国民規模の同意管理・標準化アクセス委譲・動的同意はすべて中央集権型で本番実現済み。**BC が中央集権で代替不能な領域は「相互不信の多者間・中立第三者を設置できない・クロスボーダー」という限定的条件のみ**であり、それすら「改ざん耐性監査」は透明性ログ（Trillian）で代替可能。

| 観点 | 中央集権型（FHIR Consent / SMART on FHIR / Kanta） | ブロックチェーン型 |
|------|------|------|
| 本番実績 | **大規模**（Kanta 330万人・15年[46][48]、米国病院FHIR 90%[19]） | **ほぼPoC**（同意管理分野で実稼働なし[15]） |
| trustless 性 | なし（運営者への信頼が前提） | あり（分散コンセンサス） |
| 改ざん耐性監査 | 運営者依存だが **Trillian/Certificate Transparency 型で代替可能**[56] | 暗号学的保証 |
| 単一障害点 | あり | 理論上なし（ただしパーミッション型はノード管理者信頼が残る） |
| GDPR適合 | **実績あり**（Kantaが証明） | **構造的矛盾**（不変性 vs 削除権） |
| 動的同意・粒度 | FHIR R5 + IHE PCF で向上中[52][53]、Kantaは粒度制限へ移行中[49] | SC で柔軟設定可能 |
| UX・コスト | 実用的 | 複雑・高コスト |

- **フィンランド Kanta [verified・決定的反例]**: MyKanta 330万ユーザー（人口550万のカバー率60%超）・年間4,110万アクセス[48]、2022年末で480万件の同意・16万件の同意制限[47]。2010年開始・全国実装に5.5年。2024年Client Data Actで粒度の細かい同意モデルへ更新中[49]。→「中央集権型同意管理が国民規模で15年以上本番稼働」。
- **SMART on FHIR [verified]**: 米国病院90%以上がFHIR対応（ONC Cures Act/CMS義務化）、Epic App Orchard 790+アプリ[19][57]。OAuth2.0スコープ(patient/*等)でアクセス委譲を標準化。限界＝IdP(EHRベンダー)への集中信頼。
- **HL7 FHIR Consent リソース [verified]**: マチュリティLv2(Trial Use)。モデル化完成はプライバシー同意のみ。IHE PCF v1.1.0(2024年2月)がTrial-Implementation[52][53]。執行はOAuth/UMA/XACML依存。
- **CTRL/RUDY [verified]**: 非BCのDynamic Consent Webアプリ（Ruby on Rails+PostgreSQL）。豪Genomicsで600人中91人登録(15%)[50][51]。→**動的同意はBCなしで実現可能**。研究コホート向けで国民規模ではない。
- **「BCでなければ不可能か」の批判的検討 [deduced]**: Trillian（Google製・Certificate Transparencyの汎用版）はMerkleツリー+暗号ハッシュで**BCなしに append-only・改ざん検出可能なログ**を実現し、PKI全体に本番展開済み[56]。tertiaryレビュー(2024)は「信頼できる第三者が存在するならBCは不要な可能性が高い」と結論[54]。

### Q5. 患者主権アクセス制御の最新動向（2024-2026）— BC使用/不使用の別

**回答**: SSI/DID/VC を含む患者主権の潮流は活発だが、**規制・標準の主流はことごとくBC不使用**。BCが残るのは研究PoC（Hyperledger Fabric系）とEU EBSI連携の機関向け資格証明に限定。

| 動向 | BC使用/不使用 | 到達ステージ | 信頼度 |
|---|---|---|---|
| **W3C VC 2.0**[61] | **不使用**（JSON-LD、医療デジタルウォレットが主要UC） | 2025年5月勧告化（標準完成） | verified |
| **DIDメソッド**[74] | 二分（使用: did:indy/did:ebsi/did:ethr ／ 不使用: did:web/did:key/did:jwk/did:peer/did:self）。EBSIは機関=did:ebsi+自然人=did:keyの二層 | DID 1.0/1.1標準完成 | verified |
| **EU EHDS 規則(EU)2025/327**[58][69][73] | 中核MyHealth@EUは**不使用**（NCPeH相互接続） | 2025/3発効。患者権利は**2029/3適用開始**（→2031画像・遺伝子→2035第三国） | verified |
| **EUDI Wallet / eIDAS 2.0**[63][68] | ウォレット自体は**不使用**（中央トラストアンカー）。DC4EUパイロットのみEBSI連携 | 2024/5発効・2026/12加盟国提供義務 | verified |
| **Gaia-X / HEALTH-X dataLOFT**[65][66] | **不使用**（Gaia-X連携+Gematik標準）。※「ソブリンウォッシング」批判あり | 2024/10実証完了・本番移行不明 | deduced |
| **WHO GDHCN**[67][72] | **不使用**（PKIベース・WHOトラストアンカー） | **本番稼働中**。2026/3に個人ヘルスサマリーへ拡張 | verified |
| **SMART Health Cards / TEFCA(米)**[57] | **不使用**（PKI/JWT/FHIR） | SMART Health Cardsは**本番**。TEFCA v2.1(2024/11)・Epic 2025末参加計画 | deduced |
| **医療SSI研究実装**[62][14][71] | 主にBC使用（Hyperledger Fabric系・コンソーシアム型） | 研究/PoC段階 | deduced |

**EHDSの患者制御権の具体**（2029年から適用[58][69]）: 自己データへの無料・迅速アクセス／特定部分のアクセス制限設定／アクセスログ閲覧／訂正請求／委任／クロスボーダー交換のopt-out。二次利用は「簡単かつ可逆的」にopt-out可能。**いずれもBC不使用の中央連携基盤で実装予定**。

**★傾向 [deduced]**: 「患者向けアクセス制御・同意管理」の実用化は **BCなし設計が主流化**。BCは機関間の不変ログ・スマートコントラクトとして特定機能に限定使用される傾向。

---

## 3. 横断分析

### 3-1. なぜ研究は豊富なのに本番がゼロなのか（収束した説明）

「本番商用稼働ゼロ」と「論文の豊富さ」の同居は、次の3点で説明できる（deduced）:
- **パイロット成功要件 ≠ 本番要件**: 実験環境の成功は、規制監査証跡・24/365稼働・既存EHR統合・患者UX という本番要件を回避していることが多い[34][40]。
- **ネットワーク効果の逆説**: 本番価値は参加機関が多いほど高いが、多機関ネットワーク構築コストは初期段階で最大（冷始動問題）[40]。
- **代替の十分性**: 「BCが本当に必要な trustless 多機関」ユースケースが実務上限定的で、SMART on FHIR + 中央集権同意管理（Kanta型）が先に要件を満たしてしまった[19][54]。

### 3-2. BCの正当な居場所（条件付き肯定）

調査結果は「BCは医療同意管理で常に不要」とまでは結論しない。**相互不信の多機関が中立的第三者を設置できずクロスボーダーで同意状態を共同管理する**という限定条件では、BCの trustless 性・改ざん耐性が固有価値を持ちうる[54][71]。ただし (a) その条件が実務でどれだけ出現するか不確実、(b) 改ざん耐性監査だけなら Trillian 型透明性ログで代替可能[56]、という二重の留保が付く。原典C1の「BC=同意/権限/監査の記録層」という位置づけは、この条件付き肯定の範囲では妥当だが、「本番で実証済み」とは2024-2026時点で言えない。

---

## 4. 反証・最新化サマリー（古い記述・通説の是正）

- **「エストニアは医療ブロックチェーンの先進国＝同意管理BCが稼働」という通説 → 是正**: KSIは監査ハッシュ証明であり患者同意管理ではない[24]。
- **「ドバイ NABIDH はブロックチェーンHIE」という記述 → 反証**: 公式情報にBC記述ゼロ。HL7/FHIR+AIベース[20][22]。
- **「MedRec が実用化への道を開いた」という2016-2018年の楽観 → 最新化**: 2019年以降リポジトリ放置・「NOT MAINTAINED」明記[1]。
- **「2025年までに医療アプリの55%がBC商用採用」「市場$7B・年63%成長」等の市場予測 → 注意喚起（inferred:誇大予測）**: 市場調査会社の予測であって稼働実績ではない。実態は本番不在。
- **「GDPRはハッシュをオンチェーンに置けば回避できる」という設計通説 → 反証**: EDPB 02/2025が「暗号化・ハッシュ化データも個人データ」と明示[35][43]。
- **「SSI/DID には必ずブロックチェーンが要る」という誤解 → 是正**: did:web/did:key 等はBC不要。EU EBSI も自然人にはdid:key採用[74]。

---

## 5. 未解決事項・追加調査候補

1. **SUSMED の設計詳細**: 治験SDV/IC用途は確定だが、内部のオンチェーン/オフチェーン分離アーキテクチャの一次技術仕様が非公開（uncertain）。
2. **中国の継続稼働**: Alibaba(常州)/Baidu(重慶)の2018-19開始分の2022-2025追跡が英語圏資料で取得できず。中国語一次資料調査の余地。
3. **インド**: 研究論文は最多（系統レビューで29件[32]）。Ayushman Bharat等の国家プログラムとBC連携の有無は未踏査。
4. **Crypto-Shredding の最終法的判断**: EDPB Guidelines 02/2025最終版（2025年6月コンサル終了後）でCrypto-Shreddingが「削除」として認容されるか未確定。判例・エンフォースメント事例なし。
5. **Avaneer Health の現況**: 2023年ネットワーク開始後の実稼働規模が不明（廃止の明確証拠もなし）。
6. **EUDI Wallet × EHDS の技術統合**: SSI原則との整合を含む公式アーキテクチャ文書が未取得。

---

## 6. 参照ソース一覧（Evidence Table）

| # | Title | URL | Type |
|---|-------|-----|------|
| 1 | GitHub mitmedialab/medrec (NOT MAINTAINED) | https://github.com/mitmedialab/medrec | 一次(公式リポジトリ) |
| 2 | MedRec — MIT Media Lab publication | https://www.media.mit.edu/publications/medrec-blockchain-for-medical-data-access-permission-management-and-trend-analysis/ | 公式 |
| 3 | Ariel Ekblaw — Wikipedia | https://en.wikipedia.org/wiki/Ariel_Ekblaw | 百科事典 |
| 4 | BIDMC Health Technology Exploration Center | https://www.bidmc.org/centers-and-departments/health-technology-and-exploration-center | 公式 |
| 5 | ConsentChain PoC (JMIR Med Inform 2021, PMC8600428) | https://pmc.ncbi.nlm.nih.gov/articles/PMC8600428/ | 査読(PMC) |
| 6 | arXiv 2511.17464 — Patient-Centric Blockchain Framework (EIP-712) | https://arxiv.org/html/2511.17464 | プレプリント |
| 7 | Dwarna — dynamic consent biobanking (PMC7170942) | https://pmc.ncbi.nlm.nih.gov/articles/PMC7170942/ | 査読(PMC) |
| 8 | GitHub NicholasMamo/dwarna | https://github.com/NicholasMamo/dwarna | 一次(公式リポジトリ) |
| 9 | Blockchain Dynamic Consent: Integrative Review Protocol (PMC10877491) | https://pmc.ncbi.nlm.nih.gov/articles/PMC10877491/ | 査読(プロトコル) |
| 10 | SUSMED Brings Efficiencies in Clinical Trials (BusinessWire 2020) | https://www.businesswire.com/news/home/20201206005036/en/SUSMED-Brings-Greater-Efficiencies-in-Clinical-Trials-by-Using-Blockchain-Technology | 公式PR |
| 11 | Aculys Pharma + SUSMED world's first corporate trial (BusinessWire 2022) | https://www.businesswire.com/news/home/20220712005584/en/ | 公式PR |
| 12 | Blockchain for Informed Consent: Clinical Trial Pilot (PMC9907430) | https://pmc.ncbi.nlm.nih.gov/articles/PMC9907430/ | 査読(PMC) |
| 13 | MediChainAI: ZKP & Smart Contracts (Bioengineering 2025, PMC12650700) | https://pmc.ncbi.nlm.nih.gov/articles/PMC12650700/ | 査読(PMC) |
| 14 | arXiv 2407.21559 — SSI for Consented Access to Medical Records | https://arxiv.org/abs/2407.21559 | プレプリント |
| 15 | SLR for Blockchain-Based Healthcare Implementations (PMC12071524, 2025) | https://pmc.ncbi.nlm.nih.gov/articles/PMC12071524/ | 査読系統レビュー |
| 16 | SR of Blockchain for Consent Management (PMC7912759, 2021) | https://pmc.ncbi.nlm.nih.gov/articles/PMC7912759/ | 査読系統レビュー |
| 17 | Medicalchain — Crunchbase | https://www.crunchbase.com/organization/medicalchain | ビジネスDB |
| 18 | Mayo Clinic exploring blockchain EHR — Healthcare IT News | https://www.healthcareitnews.com/news/mayo-clinic-exploring-blockchain-ehr-use-cases-uk-startup | 業界メディア |
| 19 | Medicalchain.com Ltd — GOV.UK Companies House | https://find-and-update.company-information.service.gov.uk/company/10840710 | 一次(政府登記) |
| 20 | DHA's NABIDH connects 9.47M records (Kyndryl) | https://www.kyndryl.com/us/en/about-us/news/2024/11/digitizing-healthcare-services-for-dubai-health-authority | 公式PR |
| 21 | NABIDH — Forte Healthcare | https://www.forte-healthcare.com/nabidh-network-analysis-backbone-for-integrated-dubai-health/ | 産業解説 |
| 22 | DHA Strengthens Patient Data Security (DHA公式) | https://dha.gov.ae/en/media/news/970 | 公式発表 |
| 23 | Estonian eHealth + Guardtime (e-Estonia) | https://e-estonia.com/ehealth-authority-partners-with-guardtime-to-accelerate-transparency-and-auditability-in-health-care/ | 公式 |
| 24 | KSI blockchain provides truth over trust (e-Estonia) | https://e-estonia.com/ksi-blockchain-provides-truth-over-trust/ | 公式技術解説 |
| 25 | 2024 Chinese guideline on medical blockchain (Intelligent Medicine) | https://mednexus.org/doi/abs/10.1016/j.imed.2024.09.002 | 査読ガイドライン |
| 26 | Baidu/Alibaba/Tencent China healthcare blockchain (Forkast) | https://forkast.news/baidu-alibaba-tencent-china-health-care-blo/ | 産業メディア |
| 27 | Korea link 8M records to MyHealthway (Healthcare IT News) | https://www.healthcareitnews.com/news/asia/korea-link-health-records-8-million-patients-myhealthway | 専門メディア |
| 28 | MedRec: A Case Study (MIT DCI) | https://dci.mit.edu/dci-news/blockchain-medical-records | 研究機関 |
| 29 | Avaneer Health launches decentralized network | https://avaneerhealth.com/press/avaneer-health-launches-its-decentralized-network-and-platform-to-transform-healthcare-administration/ | 公式PR |
| 30 | Why companies exit healthcare blockchain (Medium) | https://medium.com/@Connected_Dots/why-are-so-many-companies-exiting-the-healthcare-blockchain-market-43cddbea4431 | コミュニティ |
| 31 | Blockchain in healthcare: largely unproven (Healthcare Dive) | https://www.healthcaredive.com/news/blockchain-in-healthcare-huge-promise-but-largely-unproven/529666/ | 専門メディア |
| 32 | Blockchain in Health Information Systems: SR (PMC11593537) | https://pmc.ncbi.nlm.nih.gov/articles/PMC11593537/ | 査読系統レビュー |
| 33 | Health Care Sector's Experience of Blockchain (PMC8726042) | https://pmc.ncbi.nlm.nih.gov/articles/PMC8726042/ | 査読横断研究 |
| 34 | Barriers to Blockchain Patient-Centric Data Mgmt (PMC10855174) | https://pmc.ncbi.nlm.nih.gov/articles/PMC10855174/ | 査読(GCC 30名IV) |
| 35 | From Blocks to Rights: EU DPAs on Blockchain (PrivacyWorld 2025/05) | https://www.privacyworld.blog/2025/05/from-blocks-to-rights-privacy-and-blockchain-in-the-eyes-of-the-eu-data-protection-authorities/ | 法律専門ブログ |
| 36 | Blockchain and GDPR: EDPB Guidelines Explained (Xenia Tech) | https://xenia.tech/blog/blockchain-and-gdpr-edpb-guidelines-explained | 技術解説 |
| 37 | When Blockchain Meets Right to be Forgotten (SecurePrivacy) | https://secureprivacy.ai/blog/blockchain-immutability-vs-gdpr-article-17-right-to-be-forgotten | 法律技術ブログ |
| 38 | How GDPR Affects Blockchain (GDPR Local) | https://gdprlocal.com/gdpr-blockchain/ | GDPR専門 |
| 39 | HIPAA Blockchain Integration (HIPAA Vault) | https://www.hipaavault.com/resources/blockchain-integration-healthcare-records/ | HIPAA専門 |
| 40 | Blockchain in Healthcare: Use Cases & Implementation Guide (TactionSoft) | https://www.tactionsoft.com/ideas/blockchain-in-healthcare/ | 実装ガイド |
| 41 | Integrate EHR like Epic/Cerner/MEDITECH (Binariks) | https://binariks.com/blog/how-to-integrate-an-ehr-system-like-epic-cerner-or-meditech/ | 技術ブログ |
| 42 | Blockchain in healthcare: performance issues (Frontiers fdgth 2024.1359858) | https://www.frontiersin.org/journals/digital-health/articles/10.3389/fdgth.2024.1359858/full | 査読 |
| 43 | EDPB adopts guidelines on blockchains (EDPB公式PR) | https://www.edpb.europa.eu/news/news/2025/edpb-adopts-guidelines-processing-personal-data-through-blockchains-and-ready_en | 一次(EDPB公式) |
| 44 | EDPB Guidelines on Blockchain (activemind.legal) | https://www.activemind.legal/guides/edpb-blockchain/ | 法律専門解説 |
| 45 | AguHyper: Hyperledger EHR framework (PMC11157618) | https://pmc.ncbi.nlm.nih.gov/articles/PMC11157618/ | 査読(性能測定) |
| 46 | Over 89% Adoption Nationwide Patient Portal Finland (PubMed 35612157) | https://pubmed.ncbi.nlm.nih.gov/35612157/ | 査読 |
| 47 | Implementation/Adoption of Kanta 2010-2022 (PubMed 37203652) | https://pubmed.ncbi.nlm.nih.gov/37203652/ | 査読 |
| 48 | Statistics — Kanta.fi | https://www.kanta.fi/en/statistics | 公式統計 |
| 49 | Client Data Act and Kanta Services — Kanta.fi | https://www.kanta.fi/en/professionals/client-data-act-and-kanta-services | 公式 |
| 50 | Evaluation of CTRL: dynamic consent web app (PMC10772119) | https://pmc.ncbi.nlm.nih.gov/articles/PMC10772119/ | 査読 |
| 51 | CTRL: online Dynamic Consent platform (PMC8115139) | https://pmc.ncbi.nlm.nih.gov/articles/PMC8115139/ | 査読 |
| 52 | Consent — FHIR v4.0.1 (HL7公式仕様) | https://hl7.org/fhir/R4/consent.html | 公式仕様 |
| 53 | Privacy Consent on FHIR (PCF) v1.1.0 — IHE | https://profiles.ihe.net/ITI/PCF/index.html | 公式実装ガイド |
| 54 | Do you need a blockchain in healthcare? tertiary review | https://www.explorationpub.com/Journals/edht/Article/101114 | 査読 |
| 55 | Blockchain-enabled EHR access auditing (PMC11381610) | https://pmc.ncbi.nlm.nih.gov/articles/PMC11381610/ | 査読 |
| 56 | Trillian: open-source append-only ledger | https://transparency.dev/ | 技術ドキュメント(OSS) |
| 57 | FHIR Interoperability: Adoption Trends 2025 (Helixbeat) | https://helixbeat.com/fhir-interoperablity-adoption-trends-2025/ | 業界レポート |
| 58 | EU EHDS Regulation Official Page (EU委員会) | https://health.ec.europa.eu/ehealth-digital-health-and-care/european-health-data-space-regulation-ehds_en | 一次(EU公式) |
| 59 | EHDS Regulation Published (Arnold & Porter) | https://www.arnoldporter.com/en/perspectives/advisories/2025/03/european-health-data-space-regulation-published | 法律事務所解説 |
| 60 | EHDS Governance & Timelines (Inside Privacy) | https://www.insideprivacy.com/health-privacy/ehds-series-5-european-health-data-space-governance-enforcement-and-timelines/ | 専門メディア |
| 61 | W3C VC 2.0 Press Release (2025-05-15) | https://www.w3.org/press-releases/2025/verifiable-credentials-2-0/ | 一次(W3C公式) |
| 62 | Empower Healthcare through SSI (arXiv 2501.12229) | https://arxiv.org/html/2501.12229v1 | プレプリント |
| 63 | eIDAS 2.0 & EUDI Wallet Guide (wwpass) | https://www.wwpass.com/blog/eidas-2-0-the-eudi-wallet-a-practical-guide-for-enterprise-iam-2025-2027/ | 技術ブログ |
| 64 | SSI-Compliant EUDI Wallet via TEE & ZKP (arXiv 2601.19893) | https://arxiv.org/html/2601.19893v1 | プレプリント |
| 65 | HEALTH-X dataLOFT (Fraunhofer ISST) | https://www.isst.fraunhofer.de/en/departments/healthcare/projects/HEALTH-X-dataLOFT.html | 公式(研究機関) |
| 66 | Health-X Data Space (Gaia-X) | https://gaia-x.eu/health-x-a-common-data-space-for-the-health-sector/ | 公式(Gaia-X) |
| 67 | WHO Global Digital Health Certification Network | https://www.who.int/initiatives/global-digital-health-certification-network | 一次(WHO公式) |
| 68 | EUDI POTENTIAL Pilot Results (Biometric Update) | https://www.biometricupdate.com/202511/eudi-wallet-needs-common-standards-applied-rigorously-potential-pilot-finds | 専門メディア |
| 69 | My Rights Over My Health Data (EU Commission) | https://health.ec.europa.eu/ehealth-digital-health-and-care/my-rights-over-my-health-data_en | 一次(EU公式) |
| 70 | Electronic Cross-Border Health Services — MyHealth@EU | https://health.ec.europa.eu/ehealth-digital-health-and-care/digital-health-and-care/electronic-cross-border-health-services_en | 一次(EU公式) |
| 71 | Enabling Secure Health Data Sharing/Consent (npj Dig Med, PMC12398500) | https://pmc.ncbi.nlm.nih.gov/articles/PMC12398500/ | 査読 |
| 72 | WHO Digital Health Wallets Initiative (2026-03) | https://www.who.int/news/item/23-03-2026-who-and-partners-launch-new-initiative-to-expand-use-of-digital-health-wallets | 一次(WHO公式) |
| 73 | EHDS Regulation (EU) 2025/327 (Digital Policy Alert) | https://digitalpolicyalert.org/change/3347 | 政策分析 |
| 74 | Survey on DIDs and VCs (arXiv 2402.02455) | https://arxiv.org/html/2402.02455v1 | プレプリント |
| 75 | Aculys Pharma + SUSMED world's first BC clinical trial (BioSpectrum Asia) | https://www.biospectrumasia.com/news/50/20685/aculys-pharma-susmed-to-conduct-worlds-first-clinical-trial-using-blockchain.html | 業界メディア |
| 76 | SUSMED, Inc. 会社資料 (Fisco) | https://fisco.jp/cms/contents/data/6/4199/INFO_FILE_4199.pdf | 投資家向け資料 |
| 77 | Smart Patient Consent Mgmt for HIE Based on Blockchain (J. Computer Science 2024) | https://thescipub.com/abstract/jcssp.2024.730.741 | 査読(フレームワーク提案) |
| 78 | 医療DXの更なる推進について(第109回社保審医療部会 R6.7.12) 厚労省 | https://www.mhlw.go.jp/content/10801000/001274832.pdf | 一次(厚労省公式) |
| 79 | 医療DX：全国医療情報プラットフォームの概要 (Deloitte) | https://www2.deloitte.com/jp/ja/pages/life-sciences-and-healthcare/articles/hc/hc-iryoplatform.html | 専門解説 |

---

## 7. 原典（emr_blockchain_conclusion_verified.md / site/index.html C1-C11）への接続メモ

- **C1（BC=同意/権限/監査の記録層、カルテ本体オフチェーン）**: 設計思想として整合的・研究レベルで具体化済み（arXiv 2511.17464[6]）だが、**本番商用稼働は2024-2026時点で世界的に不在**という現実チェックを付すべき。
- **要注意の最新化**: 「エストニア＝同意管理BC稼働」「NABIDH＝BC HIE」は誤りとして是正（KSI=監査のみ[24]、NABIDH=BC不使用[20]）。
- **比較軸の追加**: 中央集権型（Kanta 330万人[48]・日本の全国医療情報プラットフォーム[78]・SMART on FHIR[19]）が本番の本命であり、BCは「相互不信多機関・クロスボーダー」の限定条件のみで固有価値。改ざん耐性監査はTrillian型で代替可能[56]。
- **患者主権の潮流**: SSI/DID/VC・EHDS・EUDI Wallet はBC不使用が主流[58][61][74]。BCを使う/使わないの軸でHTML可視化すると正確。
