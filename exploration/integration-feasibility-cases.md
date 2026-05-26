# 調査レポート: 日本で医療カルテを単一ソブリンクラウドに統合するのは現実的か — 海外事例からの根拠

**調査日**: 2026-05-26 / **セッションID**: 1ec45572 / **品質検証**: PASS（critic評価）
**重視期間**: 2024-2026（歴史的失敗事例は適宜遡及） / **ワーカー数**: 4 / **引用ソース**: 94件（査読論文・政府公式統計・公式PR・権威技術媒体）

---

## エグゼクティブサマリー（核心的問いへの回答）

**問い**: 「日本で医療カルテを単一ソブリンクラウドに統合するのは現実的か（断片化が何十年も続くのでは）」

**回答（信頼度: deduced〜verified）**: 海外事例は「**単一クラウド/単一DBへの統合は構造的に失敗しやすく、断片化は『安定均衡』として持続する**」という見立てを強く支持する。日本が短中期に単一ソブリンクラウド統合を達成するのは**非現実的**であり、断片化が長期継続する蓋然性が高い。根拠は3点に集約される:

1. **「単一中央DBへの統合」を試みた国はほぼ失敗した**（英NPfIT＝約£100〜127億の損失で廃止、初期豪My Health Record、加オンタリオ州）。成功したのは**連携型(federation)**（エストニアX-Road・北欧）か、**小国/単一州での強制統一**（フィンランド・加アルバータ州）に限られる。日本は人口1.26億の大国で、医療機関が独立法人として自律しており、エストニア型の成功条件をほぼ満たさない。

2. **「ソブリンクラウド」自体が成立困難な概念**である。仏Health Data Hubは「主権を標榜してMS Azure採用」が裁判で違憲とされ国産回帰、EU Gaia-Xは「紙の怪物」と評され失敗、欧州クラウド市場の約70%を米ハイパースケーラーが支配。データ所在地≠データ主権であり、外国法（CLOUD Act）リスクを技術・契約で完全排除できた例は僅少。

3. **ブロックチェーンは解にならない**。患者カルテをBC上に格納して実患者データで本番稼働したシステムは事実上存在しない（学術システマティックレビュー2024-2025が一致）。「唯一の本番」エストニアKSIすら、実態はカルテ格納ではなく監査ログのハッシュ証明にすぎない。

ただし重要な留保: 海外の失敗は主に「**国家を1つのシステムに統合する**」試みの失敗であり、「**連携(federation)による相互運用**」や「**単一州/単一医療圏での統一**」は成功例がある。したがって日本でも、単一ソブリンクラウド一極統合ではなく、標準化＋連携アーキテクチャ（日本の電子カルテ情報共有サービス/全国医療情報プラットフォーム構想はこの方向）が現実的な落としどころとなる。

---

## サブ質問1: 大規模国家EHR統合の詳細事例研究

### 結論: 「単一統合」は失敗、「連携」が成功する（明確なパターン）

#### ① 英国 NPfIT — 「単一集中統合」最大の失敗事例（最重要反例）｜信頼度: verified
- **規模/アーキテクチャ**: 全イングランドNHS。GP約3万人・病院約300施設を、中央「Spine」を核とする**単一電子患者記録**に接続する世界最大の民間ITプログラム（集中型クラスターモデル）。
- **コスト**: 初期予算 約**£23億**（£2.3 billion）→ 最終 約**£100〜127億ポンド**（£10〜12.7 billion。NAO/議会報告で£12.7 billionが最頻引用、IEEE Spectrumは「真のコストは永遠に判明しない」と評）。ユーザー認識の「約100億ポンド損失」と整合。生んだ便益は約£2.6億分のみとされ、費用対効果は極端に負。
- **期間**: 2002年開始 → 2011年9月廃止発表 → 2013年3月正式廃止（約9〜11年）。
- **利用率/達成度**: 医師支持率が2005年70%→2008年41%へ低下、約3分の2が利用に消極的。中核のCare Records Serviceは4年以上遅延し事実上未達成。
- **廃止理由（6点）**: ①政治主導トップダウン（現場臨床家を排除）②利害関係者の同意不足 ③要件不確定のまま10年の大型契約 ④「一つのシステムで全て解決」の過大な野心 ⑤複数ベンダー連携の崩壊（Accentureが2006年撤退）⑥説明責任構造の欠如。
- **教訓**: **単一中央DB＋トップダウン＋大型ベンダー一括契約の三点セットは大規模国家EHRで機能しない。**

#### ② エストニア — 連携型(X-Road)＋KSIの成功（NPfITとの決定的対比）｜信頼度: verified
- **規模/アーキテクチャ**: 人口約130万人。**連携型(federation)。中央DBは存在せず「重複レコードは法律で禁止」**。データは源泉（各医療機関）に留まり、X-Roadは安全な交換プロトコル層のみを提供（2008年開始）。
- **X-Road実績**: 450超の機関が接続、2,800超のサービス、1,300超の情報システム連携。医療データの99%がデジタル化、電子処方箋ほぼ100%。
- **KSIブロックチェーン**: Guardtime製。**カルテ本体は格納せず、アクセスログ/改変記録のハッシュ＋Merkleルート＋タイムスタンプで整合性を証明するのみ**（後述サブ質問3と完全一致）。
- **コスト**: 全国EHR構築コスト約1,000万ユーロ（市民1人あたり約7.5ユーロ）。新規ソリューションは通常4ヶ月以内に費用回収。
- **成功要因**: ①全市民eIDによる統一識別 ②X-Roadの分散設計（中央集権の回避）③段階的な法的義務化 ④ベンダー中立 ⑤医療単独でなく国家デジタル基盤(eID/X-Road)への相乗り ⑥高い市民信頼（全記録閉鎖は年0.04%のみ）。Bertelsmann Digital Health Index 2024で1位。

#### ③ 北欧 — 連携/ハイブリッドで成功｜信頼度: verified
- **デンマーク sundhed.dk**: 連携型。分散データを単一プレゼンテーション層で統合（単一DBではない）。CPR番号（1968年〜）＋MedCom標準が基盤。150システム間で毎月550万件交換。EHRカバレッジ100%。欧州保健IT責任者の94%がモデルと評価（Black Book Survey 2025）。
- **フィンランド Kanta**: 集中型に近い全国FHIRプラットフォーム。小国＋強い政治意思で成功。2024年にMyKantaを330万人が4,110万回利用、毎日200万件超を保存。公的機関100%・薬局100%・私的医療機関2/3が参加。
- **スウェーデン 1177**: ハイブリッド。21地域が自律しつつ患者アクセスは1177.seに集約。「疎結合・フェデレーション」を国家参照アーキテクチャの原則に明記。人口90%超が利用。

#### ④ 米国 — 連携型(TEFCA)＋Epic/Oracle寡占｜信頼度: verified
- **アーキテクチャ**: 連携型（QHINネットワーク）。単一国家DBは存在しない。
- **寡占**: Epic 42.3%＋Oracle Health（旧Cerner）22.9%＝入院EHR市場の62%超。Epicは2024年に176病院純増（史上最大）、新規契約の約70%を獲得、HHI指数2,500超（高度集中）。
- **TEFCA**: 2023年12月に8 QHIN指定、2025年1月に最終規則発効。Epic Nexusが625病院接続済（2024末）、2025年末に全Epic顧客のTEFCA移行を目標。ただし**自発的参加**。
- **示唆**: 技術的に相互運用可能でも運用的に断片化しうる。Epic寡占が事実上の標準を形成。

#### ⑤ ドイツ gematik ePA — 集中インフラだが利用率激低｜信頼度: verified
- **アーキテクチャ**: gematik管理のTelematikinfrastruktur(TI)上。2025年1月にオプトアウト方式「ePA für alle」で約7,200万人に自動付与、7,000万アカウントが自動作成。
- **アクティブ利用率: 2026年1月時点でわずか3.6%**（Techniker 7.4%/Barmer 5.5%/AOK連合 1.4%）。医療提供者のTI接続は歯科97%・薬局96%・医師93%と高いのに、患者の実利用が伸びない。
- **低利用の理由**: ①自動登録で所有意識が低下 ②ID作成/アクセス権管理の複雑さ ③プライバシー懸念58% ④追加便益への疑問55% ⑤デジタルリテラシー不足48%。
- **教訓**: **アカウント自動付与（数の達成）と実利用は別物。「箱を配っても使われない」。**

#### ⑥ 豪 My Health Record — 集中型＋オプトアウト論争｜信頼度: verified
- **アーキテクチャ**: 集中型。ADHA管理の国家中央DB。当初は分散型計画だったが実装は中央集権に。「静的サマリー文書の中央集中DB」と批判。
- **参加率**: 約90%（オプトアウト約10%・250万人超）。ただし「90%は積極同意でなく不作為」と批判。GPの約1/3が未使用。2018年のオプトアウト転換時、予測脱退1.9%に対し実際は250万人超が脱退。
- **論争**: 令状なしの法執行アクセス（MHR Act §70）、約90万人の医療従事者が比較的制限なくアクセス可。
- **2025年改正**: 病理・画像のアップロードを法的義務化（使われないので強制に転換）。

#### ⑦ カナダ — 連邦制の構造的断片化｜信頼度: verified
- **アーキテクチャ**: 単一国家システムなし。各州が独自。Canada Health Infoway（2001〜）が標準化を推進するも**拘束力なし**。
- **投資と失敗**: 連邦初期$5億。オンタリオ州だけで2002-2016に$80億超を投資（Smart Systems for Health $6.5億失敗・eHealth Ontario $10億超失敗）。
- **現状**: 医師EHR使用率95%だが**電子健康情報の70%超が共有されていない**。共有の重要性認識90% vs 実際共有40%。
- **唯一の成功例**: アルバータ州がEpic Connect Careを州全体に強制導入（$4.59億・125,000人超・1,000サイト超）＝「カナダ唯一の州統一システム」。一人あたりコスト半減。**統一は州単位の強制でしか達成できなかった**。

### 集中 vs 連携 総括
| 国 | アーキテクチャ | 成否 | 主因 |
|---|---|---|---|
| 英NPfIT | 集中(Spine単一DB) | **失敗** | トップダウン・スケール過大・医師不支持・コスト超過 |
| エストニア | 連携(分散・各機関保有) | **成功** | eID基盤・法的義務化・ベンダー中立・KSI完全性 |
| デンマーク | 連携(分散+単一UI) | 成功 | CPR・MedCom・政治統一 |
| フィンランド | 集中に近い | 成功 | 小国・強い政治意思・段階実装 |
| スウェーデン | ハイブリッド | 成功 | 地域自律+国家基準 |
| 米TEFCA | 連携(QHIN) | 進行中 | Epic寡占が事実上標準形成 |
| 独ePA | 集中インフラ(TI) | **普及低迷3.6%** | 自動付与でアカウントのみ増・実利用伸びず |
| 豪MHR | 集中(ADHA中央DB) | 論争中 | 参加90%だがGP1/3未使用・プライバシー批判 |
| カナダ | 州別断片化 | 課題継続 | 連邦制+任意参加。アルバータのみ強制で統一 |

→ **「単一中央DB/クラウドへの統合」を試みた例（英NPfIT・初期豪MHR・オンタリオ州）はほぼ失敗。成功は連携型か、小国/単一州での強制統一に限られる。**

---

## サブ質問2: 国家規模ソブリンクラウドの前例

### 結論: 「主権」を標榜しても実態はハイパースケーラー依存（ソブリンウォッシング）

#### ① 仏 Bleu（Capgemini+Orange+Microsoft）— 「管理されたAzure」｜信頼度: verified
- 2025年1月に商業活動開始発表、本格稼働は2026後半〜2027予定。仏国内2DC（300km離隔）、顧客鍵管理オプション、Level3サポートはMicrosoftが担当。
- **決定的証言**: 2025年6月、Microsoft France GMのAnton Carniaux氏がフランス上院で「**フランス市民データが外国当局に渡らないとは保証できない**」と証言（CIO.comで独立確認、Nextcloudブログが一次引用）。アナリスト評：「主権は保証でなくシミュレート」。

#### ② 仏 S3NS（Thales+Google）— 最も技術的に強固｜信頼度: verified
- 2025年12月17日にSecNumCloud 3.2取得（IaaS/CaaS/PaaS 3スコープ同時＝ANSSI史上初）。
- S3NSがGoogle Cloudソースコードへのアクセス権を保有し、約500万コンポーネントを毎年隔離環境でリバースエンジニアリング後に本番展開＝**ソースコードレベルの監査権でBleu/Delosより深い主権**。医療・保険・金融で早期採用。ただしGoogle技術依存は残る。

#### ③ 仏 Health Data Hub — 「主権標榜でAzure採用」矛盾の公式決着（最重要事例）｜信頼度: verified
- SNDS（国民健康保険DB）含む医療データハブをMS Azureでホスト → 2020年CNIL警告 → **2022年に最高行政裁判所がEUデータ保護原則違反と判決**。
- MSの「機密クリーンルーム」提案を「いかなる契約もCLOUD Actを上書きできない」として拒否 → 仏国産Scaleway（HDS+SecNumCloud認証）へ移行。2024年に敏感データの「sovereign-guaranteed」ホスティングを法律で義務化。完全移行は2026後半〜2027。
- **教訓: CLOUD Actは技術的・契約的手段では回避不可能という公式確認。欧州最重要の「医療ソブリンクラウド失敗→国産回帰」事例。**

#### ④ 独 Delos Cloud（SAP+Microsoft+Arvato）— 10〜20%割高の「管理Azure」｜信頼度: verified
- 2024年9月最終契約、2025年前半からAzure基礎サービス。セキュリティクリアランス付き独人スタッフが管理、BSI準拠目標。
- **コスト: 通常Azureより10〜20%高い**。少なくとも6州が安価なパブリッククラウドを代替検討中。2024年調査で多くの州が「価格・機能が不明確で評価困難」と回答。Azure技術への根本的依存は変わらず。

#### ⑤ 独 Sovereign Cloud Stack（SCS）— 数少ない非ハイパースケーラー依存｜信頼度: verified
- BMWK資金提供終了（2024/12）後はOSBA e.V.が継続管理。7プロバイダーが本番採用、第8次リリース公開済。Gaia-X Lighthouseとして数少ない現実的成果。ただし政府資金終了後の持続可能性が課題。

#### ⑥ EU Gaia-X — 「野心と現実の乖離」の典型的失敗｜信頼度: verified
- 2019年に独仏主導で「欧州クラウドのエアバス」を目指す → **対抗すべきMS/Amazonを加盟企業に招聘＝「トロイの木馬」批判**。CISPE「Gaia-Xは相互運用性を認証するがCloud Act保護は認証しない」。
- 具体的失敗: Agdatahub（初日メンバー・480万ユーロ調達）が清算。創設国の独自身がOracle Cloud（非欧州）に30億ユーロ超を拠出。
- Nextcloud CEO（2024年1月）「Gaia-Xに未来はない。影響のない紙の怪物」。前CEO「野心的すぎた。紙の怪物となった」。欧州クラウド市場シェアはGaia-X期間中に3/4を喪失。

#### ⑦ ソブリンウォッシングの構造的問題｜信頼度: verified
- **データ所在地(Residency)≠データ主権(Sovereignty)**。制御プレーン・課金・サポート・更新サイクルが非欧州親会社に依存する限り、外国法の強制コンプライアンスは回避不可能。
- 2025年、MS France GM・AWS・Google・Salesforce担当が各国議会/法廷で「米国法廷命令があれば欧州顧客データを提出する」と認める。
- **Amazon・Microsoft・Googleが欧州クラウド市場の約70%を支配。欧州プロバイダーは2%未満**。ソブリンクラウドはパブリック比10〜30%割高。

#### ⑧ 医療ソブリンクラウド世界事例（富士通×IBM以外）
| 国/機関 | プロジェクト | 実態 |
|---|---|---|
| 仏HDH | Scalewayへ移行中 | Azure→国産移行（CLOUD Act問題が決定打）|
| 仏S3NS | Thales+Google | SecNumCloud取得。医療・保険向け |
| 独T-Systems | Google Cloud | 大学病院シュレスヴィヒ＝ホルシュタインで患者データ管理採用 |
| 英NHS | Azure+AWS+GCP | マルチクラウド。CLOUD Actリスク継続 |
| 加 | Epic/Cerner/MEDITECH | 全て米国企業＋米クラウド依存。主権クラウド投資をCMAJが提言 |
| 印TCS | SovereignSecure | 政府/規制産業向け国産クラウド |
| サウジ | Vision 2030 | AWS/Azure/Google現地DC。Tier1データは国内保存義務 |
| 豪ADHA | Accenture(入札中) | US CLOUD Actリスク抱え運用継続 |

→ **「真の主権」を技術的に実現できている例はごく僅か（S3NS/SCS程度）。大半は『主権クラウド』を名乗りつつハイパースケーラー依存で、外国法リスクを完全には排除できていない。** 富士通×IBMの医療ソブリンクラウド（前回検証でBC不使用の中央集権Oracle Alloyと判明）も、この「管理されたハイパースケーラー」型に分類される。

---

## サブ質問3: ブロックチェーンEHRプロジェクト詳細

### 結論: 患者データで本番稼働した「カルテ格納」BCシステムは事実上存在しない

#### 最重要結論｜信頼度: verified（複数の独立した学術レビューが一致）
**「患者カルテをブロックチェーン上に格納し、実患者データで本番稼働しているシステム」の確認事例は現時点でなし。** 2024-2025年の複数の独立した学術システマティックレビュー（PMC 82研究・Wiley・PMC 73研究）が一致して「ほぼ全実装がシミュレーション環境/PoC段階」と結論。

#### ① Guardtime/エストニアKSI — 唯一の「本番」だが実態は監査ログのハッシュ証明｜信頼度: verified
- **技術的区別（最重要）**: KSI（Keyless Signature Infrastructure）は**ハッシュ証明のみ**。「ブロックチェーンには暗号証拠（署名）のみが格納され、患者データやPIIは直接オンチェーンには存在しない」。カルテ本体はOracle DBに保管し、BCはアクセスログ/改変記録のハッシュで改ざん検知・監査追跡を行う。
- 規模: 全国民約130万人を対象、2011年〜本番稼働、100万件超の記録を保護（2016時点）。
- **誇大広告との乖離**: メディアは「BCで100万件カルテを保護」と報道するが、実態は「カルテ格納」ではなく「監査証跡のハッシュ証明」（researcher-1とresearcher-3が独立に同一結論＝相互検証）。

#### ② 撤退・廃業・ピボットしたプロジェクト群
| プロジェクト | 実装 | 規模 | 現状 | 撤退/不在理由 |
|---|---|---|---|---|
| MedRec(MIT) | EthereumスマートコントラクトでEHRアクセス権制御、本体オフチェーン | BIDMCで40名パイロット | **プロトタイプ止まり**（サイト消失） | 規制負担・既存EMR統合困難 |
| Gem Health | PhilipsとプライベートEthereum(2016) | 本番データなし | **医療撤退**、2022 Blockdaemonが暗号企業として買収 | EMR置換不可能・患者インセンティブ欠如 |
| Akiri | BC活用NaaS(AMA系Health2047発) | $10M調達、AMA自身が最初の加入者 | **廃業（Out of Business・PitchBook確認）** | 医療参入障壁・ビジネスモデル立証困難 |
| Hashed Health | DLT医療コンサル/ベンチャースタジオ(2016) | UPMC等出資 | **BC言及消滅**、汎用デジタルヘルスへ完全転換 | BC固有価値が現場で立証できず |
| Patientory | プライベートBC(PTOYMatrix)、2017 ICO | 45,000購読者(アプリ登録)・24,000機関 | 名目稼働だが**PTOYトークン約99.9%超下落**（市場評価ほぼゼロ） | ICO後の事業継続性に疑問 |
| BurstIQ | ヘルスケア特化BC | 売上$3M・顧客10件(2024) | 小規模稼働継続 | 大規模本番の証拠なし |
| Medicalchain | Hyperledger+Ethereum、2018 ICO $24M | NHS承認サプライヤー登録 | 創業者がテレメディシン(MyClinic)へ注力、**ピボットの様相** | EHR事業の現状不透明 |
| IBM(Watson Health/Blockchain) | Hyperledger Fabric | — | Watson Health 2022売却(Merative化)、**Blockchain Platform 2023/4 EOL** | 医療BC分野からの撤退 |

#### ③ 国家BC医療｜信頼度: medium
- **ドバイ/UAE NABIDH**: 9.47百万件・1,300施設・医師81%接続の大規模稼働だが、**EHR本体のBC格納の証拠なし**。BCは主に医師免許/資格管理用途に限定の可能性。
- **中国**: 2024年に医療BC国家ガイドライン策定（EHR共有を「最も実装しやすいシナリオ」と評価）だが、国家規模の本番稼働の具体事例は未確認＝研究/パイロット段階。

#### ④ 撤退の構造的理由｜信頼度: deduced
①既存EMR（Epic/Cerner）の置換不可能性 ②患者インセンティブ設計の失敗 ③規制コンプライアンスコスト過大 ④より参入障壁の低い市場へのピボット。「既存EHRベンダーが支配する市場でのスイッチングコストが障壁」。

---

## サブ質問4: 断片化が「安定均衡」として持続する5層の構造的理由

### ① 技術層｜信頼度: high(verified)
- 米病院の60%超がレガシー（クラウド非対応）で基幹アプリ運用。HL7v2インターフェース50〜200本の年間維持費10〜20万ドル、だが完全移行はその10〜100倍（Partners HealthCareは予算6億→実績12億ドル超）。
- **FHIRは「容器」を標準化するが「意味」は解決しない**。R4は92%のEHRベンダーが対応済だが、ICD-10とSNOMED CT混用の意味的等価性は未解決。「国家レベルの用語管理の欠如」が断片化を永続化。

### ② 組織層｜信頼度: high(verified)
- **Epic寡占とネットワーク効果**: HHI 1,452(2012)→2,500超(2018)。同一ベンダー間相互運用0.68 vs 異ベンダー間0.22。教育病院60%がEpic→医学生90%超がEpicで訓練＝「人材ロックイン」。
- **情報ブロッキング**: HIE 89機関調査で55%が「ベンダーの情報ブロッキングを確認」、最多手法は不合理な高価格（42%）。競争市場ほど多い（47% vs 16%）。48%の病院がデータ送信するが受信しない。
- **コンウェイの法則**: 「組織のコミュニケーション構造がシステム構造を写像」。医療機関が自律組織である限りITも組織境界を反映した孤立構造に（※医療×コンウェイの直接的定量実証は限定的＝信頼度inferred）。

### ③ 政治・ガバナンス層｜信頼度: high(verified)
- 5カ国比較（米分散/英ミドルアウト/独集権・遅延/イスラエル・ボトムアップ→集権/ポルトガル義務移行）から「健康システムタイプに関わらず**中央戦略的計画と関与が不可欠**」。
- 米国: 「医療情報インフラの根本的決定が公的機関の関与なしに行われる」制度的空白。TEFCAも自発的参加に留まる。
- JMIR 2024: 断片化を維持する6障壁＝技術企業の収益優先・VC影響・インフラ不平等・リテラシー過少投資・償還不確実性・正当な不信感。

### ④ 経済層（ナッシュ均衡）｜信頼度: deduced（理論的説明。一次根拠は技術ブログ/Medium中心、査読経済学文献での直接定式化は限定的）
- スイッチングコストは「先行・確実」、便益は「将来・不確実」→ 現状維持が支配戦略。
- **フィーサービス制度ではデータ共有（重複検査削減）が医療機関の収入を直接削減**＝共有しないことが経済合理的。
- **パス依存の自己強化**: 2009 HITECH法の390億ドル補助金がEpic/Cernerへの早期ロックインを引き起こし、その後の移行コストを天文学的に。「補助金による加速的ロックイン」。

### ⑤ 法的層｜信頼度: high(verified)
- 「病院/スタートアップが二次利用・データ共有・説明責任の法的明確性なしに運営」。GDPR域外適用が越境共有の法的コストを増大。
- HIPAAの「誤解」: 治療目的のPHI共有は同意不要だが、多くが過度な制限を課す。「法的リスク懸念が実質的に経済的決定の正当化に使われる」。

### ⑥ なぜ断片化がナッシュ均衡か（統合的説明）｜信頼度: deduced
- EHRベンダー: 共有容易化は競合への乗り換え容易化 → 非共有が支配戦略
- 病院: 患者データ共有は患者流出リスク → 非共有が支配戦略
- 政府: 強制標準化はベンダー・医療機関双方の抵抗を招く → 任意参加が政治均衡
- **3者全員にとって現状維持が合理的で、誰も一方的逸脱のインセンティブを持たない＝「断片化」が安定したナッシュ均衡**。セマンティック相互運用の持続的ガバナンス不在（コモンズの悲劇）も同型。

### ⑦ 統合成功国の必要条件（6条件）と日本の充足評価｜信頼度: high（条件）/ deduced〜uncertain（日本評価の一部）
**成功国（エストニア/デンマーク/フィンランド）の共通条件**:
1. 普遍的国民識別子（デンマークCPR 1968〜、エストニアeID）
2. 小規模人口/高い社会的均質性（エストニア137万人、構築コスト市民1人7.5ユーロ）
3. 公営医療/強力な単一支払者（競争的情報戦略が生まれにくい）
4. 長期的政治統一・一貫ガバナンス（10年超）
5. 高い政府信頼（エストニアは全記録閉鎖が年0.04%のみ）
6. 先行的デジタル国家基盤（X-Road、MedCom）

**日本の充足評価**:
| 条件 | 日本の状況 | 充足 |
|---|---|---|
| 国民ID | マイナンバー存在も誤登録事故・プライバシー不信が深刻（「別人と紐付け」複数確認） | ✕（現状） |
| 人口規模 | 1.26億人（エストニアの約90倍）→ 統合コスト桁違い | ✕ |
| 医療制度 | 公的保険はあるが医療機関は独立法人（国公私混在）→ 競争的情報戦略が生じやすい | △ |
| 政府信頼 | マイナンバー問題等で低下中 | ✕（現状） |
| 政治統一 | — | データ不足 |
| デジタル基盤 | EMR導入率が診療所16.5%〜主要病院62.5%と極端に不均一、外部NW接続57.8%のみ ※出典は2012年/COVID期のPMC論文で古く、現況は改善している可能性に留意 | ✕（古いデータ） |

→ **日本は「統合成功の必要条件」の大部分を（少なくとも従来は）欠いており、断片化が安定均衡として継続する構造的条件が揃っている。** ただし日本のEMR/ネットワークデータは2012年・COVID期の古い論文由来で、近年の電子カルテ情報共有サービス等で改善が進んでいる可能性があり、最新の国内統計での再評価が望ましい。

---

## 総合的含意: 日本への示唆（信頼度: deduced）

1. **「単一ソブリンクラウドへの一極統合」は海外で繰り返し失敗しており、日本でも非現実的**。英NPfIT（約£100〜127億の損失）が示す通り、大国での単一中央DB統合はトップダウン・コスト超過・現場不支持で崩壊する。

2. **断片化は「怠慢」ではなく「合理的均衡」**。ベンダー・医療機関・政府の三者にとって現状維持が支配戦略であり、誰も一方的に動かない。これを崩すには中央の強い戦略的関与（法的義務化・標準強制・インセンティブ再設計）が必要だが、日本の医療機関の自律性と政府信頼の現状はそれを難しくする。

3. **現実的な道は「連携(federation)＋標準化」**。エストニア・北欧・米TEFCAの成功/進行例はいずれも「単一DB」ではなく「各機関がデータを保持しつつ相互運用する連携層」。日本の電子カルテ情報共有サービス/全国医療情報プラットフォーム構想はこの方向であり、海外事例に照らして妥当。

4. **「ソブリンクラウド」も「ブロックチェーン」も統合の本質的解にはならない**。前者はハイパースケーラー依存と外国法リスクを完全には排せず（仏HDHの教訓）、後者は患者カルテ格納の本番稼働実績が皆無。両者は「統合できない問題」を解決しない。

5. **したがって「断片化が何十年も続くのでは」という懸念は、海外事例からは妥当**。完全統合ではなく「連携による段階的相互運用の向上」が、達成可能で現実的な目標となる。

---

## 未解決事項・追加調査候補
1. **NPfIT実支出の確定値**: £10〜12.7 billionと幅。NAO報告原典の精査で確定可能。
2. **日本の現行EMR採用率/ネットワーク接続率**: 本調査の根拠は2012年・COVID期の古い論文。厚労省の最新統計（電子カルテ情報共有サービス進捗等）での再評価が必要。
3. **ナッシュ均衡の査読経済学的根拠**: 理論的説明は妥当だが、一次根拠が技術ブログ/Medium中心。医療経済学の査読文献での裏付けが望ましい。
4. **ドバイNABIDHのBC使用の技術詳細**: EHR本体格納か免許管理のみかの一次ソース（ScienceDirect 403で未取得）。
5. **中国の本番稼働病院数**: ガイドライン策定は確認も具体的本番事例の数値なし。
6. **Carniaux仏上院証言の一次議事録**: CIO.com等で確認済だが、上院議事録原典の直接確認が望ましい。

---

## Evidence Table（参照ソース94件）

| # | Title | URL | Type |
|---|-------|-----|------|
| 1 | The £10 Billion IT Disaster at the NHS | https://www.henricodolfing.ch/en/case-study-1-the-10-billion-it-disaster-at-the-nhs/ | 技術ブログ |
| 2 | NPfIT Dismantled (£12.7B) - IEEE Spectrum | https://spectrum.ieee.org/npfit-dismantled-uk-government-announces-end-of-its-127-billion-national-electronic-health-record-program | 権威技術誌 |
| 3 | Six reasons why NHS NPfIT failed - Computer Weekly | https://www.computerweekly.com/opinion/Six-reasons-why-the-NHS-National-Programme-for-IT-failed | 権威技術媒体 |
| 4 | NHS Connecting for Health - Wikipedia | https://en.wikipedia.org/wiki/NHS_Connecting_for_Health | 二次情報 |
| 5 | Reasons Behind NHS IT System Failure - Panorama | https://www.panorama-consulting.com/nhs-it-system-failure/ | コンサルブログ |
| 6 | Digital healthcare - e-Estonia (公式) | https://e-estonia.com/programme/digital-healthcare/ | 公式 |
| 7 | Estonia's Interoperable Health Records - Digital Care Hub | https://www.digitalcarehub.co.uk/case-study/estonias-interoperable-health-records/ | 専門機関 |
| 8 | X-Road: Estonia's digital backbone - Nortal | https://nortal.com/insights/x-road-estonias-digital-backbone | 実装企業 |
| 9 | Digital Health in Estonia - G_NIUS(仏国家数字保健庁) | https://gnius.esante.gouv.fr/en/decode-ehealth-internationally/digital-health-in-estonia | 政府機関 |
| 10 | Inside Estonia's pioneering digital health service - Sifted | https://sifted.eu/articles/estonia-digital-health | 専門メディア |
| 11 | Danish e-Health Portal - NCBI | https://www.ncbi.nlm.nih.gov/books/NBK543679/ | 学術(NCBI) |
| 12 | Kanta Statistics (公式) | https://www.kanta.fi/en/statistics | 公式 |
| 13 | Swedish Patient Portal & National Reference Architecture - NCBI | https://www.ncbi.nlm.nih.gov/books/NBK543680/ | 学術(NCBI) |
| 14 | European Health IT Leaders: Denmark/Finland Models - Black Book | https://www.accessnewswire.com/newsroom/en/healthcare-and-pharmaceutical/european-health-it-leaders-look-to-denmark-and-finland-as-models-for-1036600 | 調査会社PR |
| 15 | Germany's ePA: From Quiet Launch to Mass Adoption - Joep Lange | https://www.joeplangeinstitute.org/data_and_health/electronic-patient-record-germany/ | 医療シンクタンク |
| 16 | Germany's ePA Sees Slow Adoption | https://evrimagaci.org/gpt/germanys-electronic-patient-record-sees-slow-adoption-522855 | 専門メディア |
| 17 | Consumer perspectives on ePA in Germany - BMC | https://link.springer.com/article/10.1186/s12913-024-12175-6 | 査読誌 |
| 18 | My Health Record Statistics December 2024 - ADHA(公式) | https://www.digitalhealth.gov.au/sites/default/files/documents/my-health-record-statistics-december-2024.pdf | 政府公式統計 |
| 19 | My Health Record - Australian Privacy Foundation | https://privacy.org.au/campaigns/myhr/ | 市民団体 |
| 20 | One in ten Australians opt out of MHR - Healthcare IT News | https://www.healthcareitnews.com/news/asia/one-ten-australians-opt-out-my-health-record-system-adha-says | 専門メディア |
| 21 | Why Canada's health records remain fragmented - Policy Options | https://policyoptions.irpp.org/2026/05/canada-health-records-fragmentation/ | 政策シンクタンク |
| 22 | Connected Care for Canadians Act - PMC | https://pmc.ncbi.nlm.nih.gov/articles/PMC11627560/ | 査読(NCBI) |
| 23 | Alberta expands Epic EHR across province - Canadian Healthcare Tech | https://www.canhealth.com/2024/11/06/alberta-expands-epic-ehr-across-the-province/ | 専門メディア |
| 24 | Epic TEFCA QHIN status - Healthcare IT News | https://www.healthcareitnews.com/news/epic-says-providers-are-sharing-widely-through-its-tefca-qhin | 専門メディア |
| 25 | KLAS: Epic Dominates 2024 EHR Market Share - HIT Consultant | https://hitconsultant.net/2025/05/05/klas-epic-dominates-2024-ehr-market-share/ | 専門調査 |
| 26 | TEFCA - ONC(公式) | https://www.healthit.gov/topic/interoperability/policy/trusted-exchange-framework-and-common-agreement-tefca | 政府公式 |
| 27 | Orange & Capgemini launch Bleu - DataCenterDynamics | https://www.datacenterdynamics.com/en/news/orange-capgemini-to-finally-launch-bleu-sovereign-cloud-service/ | 技術メディア |
| 28 | Capgemini and Orange announce Bleu(公式PR) | https://www.capgemini.com/news/press-releases/capgemini-and-orange-are-pleased-to-announce-the-launch-of-commercial-activities-of-bleu-their-future-cloud-de-confiance-platform/ | 公式PR |
| 29 | Orange Business turns 70% blue with Bleu - SDxCentral | https://www.sdxcentral.com/news/orange-business-turns-70-blue-with-bleu-cloud-sovereignty/ | 技術メディア |
| 30 | Bleu: trusted French cloud - Jint | https://www.jint.co/blog-posts/bleu-trusted-french-cloud | 技術ブログ |
| 31 | How sovereign is Microsoft Sovereign Cloud really? - CIO | https://www.cio.com/article/4009314/how-sovereign-is-microsofts-sovereign-cloud-really.html | 技術メディア |
| 32 | Big Tech's Sovereign Cloud promises collapsed - Nextcloud | https://nextcloud.com/blog/big-techs-sovereign-cloud-promises-just-collapsed-in-their-own-words/ | Nextcloud公式 |
| 33 | S3NS & Sovereignty - Futurum | https://futurumgroup.com/insights/s3ns-sovereignty-can-thales-google-venture-make-ai-sovereignty-work-at-scale/ | アナリスト |
| 34 | S3NS receives SecNumCloud qualification - Thales(公式) | https://www.thalesgroup.com/en/news-centre/insights/group/s3ns-receives-secnumcloud-qualification-turning-point-trusted-cloud | 公式 |
| 35 | S3NS Announces SecNumCloud for PREMI3NS - BusinessWire | https://www.businesswire.com/news/home/20251218817208/en/ | 公式PR |
| 36 | France moves health data from Microsoft to French cloud - Euronews | https://www.euronews.com/health/2026/04/24/france-moves-public-health-data-from-microsoft-to-french-cloud-provider | ニュース |
| 37 | France Health Data Hub Switches to Scaleway | https://windowsnews.ai/article/france-health-data-hub-switches-to-scaleway-sovereign-cloud-signals-after-azure.414865 | ニュース |
| 38 | First Sovereign Cloud Platform for German Admin - Bertelsmann | https://www.bertelsmann.com/en/news-and-media/news/first-sovereign-cloud-platform-for-the-german-administration-on-the-home-straight.jsp | 公式 |
| 39 | Delos Cloud & Arvato Systems | https://us.arvato-systems.com/industries/public/delos-cloud | 公式 |
| 40 | Delos: Sovereign cloud 10-20% more expensive - Heise | https://www.heise.de/en/news/Delos-Sovereign-cloud-10-to-20-more-expensive-than-Microsoft-s-public-cloud-10102043.html | Heise |
| 41 | Sovereign Cloud Stack Release 8 | https://sovereigncloudstack.org/announcements/release8/ | SCS公式 |
| 42 | Gaia-X: A trojan horse for Big Tech - EuroStack | https://euro-stack.com/blog/2020/11/2020-gaia-x-trojan-horse | EuroStack |
| 43 | Gaia-X doesn't have a future - The Register | https://www.theregister.com/2024/01/08/gaiax_future/ | The Register |
| 44 | Gaia-X - Chronicle of a Failure Foretold - EuroStack | https://euro-stack.com/blog/2025/2/gaia-x-failure | EuroStack |
| 45 | T-Systems Sovereign Cloud Powered by Google Cloud | https://www.t-systems.com/de/en/sovereign-cloud/solutions/sovereign-cloud-powered-by-google-cloud | T-Systems公式 |
| 46 | Sovereignty Washing - VSHN | https://www.vshn.ch/en/blog/sovereignty-washing-when-sovereign-cloud-isnt-really-sovereign/ | 技術ブログ |
| 47 | UK NHS Cloud Infrastructure 2025 - CompareTheCloud | https://www.comparethecloud.net/articles/nhs-cloud-infrastructure-digital-health-2025/ | 技術メディア |
| 48 | Ensuring sovereignty/security of Canadian health data - CMAJ/PMC | https://pmc.ncbi.nlm.nih.gov/articles/PMC12316698/ | 査読(CMAJ) |
| 49 | TCS SovereignSecure Cloud | https://www.tcs.com/what-we-do/industries/public-services/solution/tcs-sovereignsecure-cloud | TCS公式 |
| 50 | Sovereign Cloud Transformations in Middle East - Cloud4C | https://www.cloud4c.com/blogs/sovereign-cloud-transformations-in-middle-east | 技術ブログ |
| 51 | Sovereign Cloud Market Size - Grand View | https://www.grandviewresearch.com/industry-analysis/sovereign-cloud-market-report | 市場調査 |
| 52 | MIT Media Lab - MedRec Overview | https://www.media.mit.edu/projects/medrec/overview/ | 公式(研究機関) |
| 53 | MIT DCI - MedRec Case Study | https://www.dci.mit.edu/dci-news/medrec-a-case-study-for-blockchain-in-healthcare | 公式(研究機関) |
| 54 | Open Health News - Guardtime Estonia 1M Records | https://www.openhealthnews.com/content/estonian-government-guardtime-accelerate-adoption-blockchain-technology-secure-1m-patient-he | ニュース |
| 55 | Guardtime Blog - Estonian eHealth Partnership | https://guardtime.com/blog/estonian-ehealth-partners-guardtime-blockchain-based-transparency | 公式(企業) |
| 56 | luckystar.ai - Estonia e-Health Blockchain Lessons | https://luckystar.ai/blogs/blockchain/futureproofing-blockchain-infrastructure-lessons-from-estonia-s-e-health-system | 技術ブログ |
| 57 | Dr. Hempel Network - Estonia Medical Blockchain | https://www.dr-hempel-network.com/digital-health-technolgy/medical-blockchain-technology-implementation/ | 専門家ブログ |
| 58 | PitchBook - Akiri 2025 (Out of Business) | https://pitchbook.com/profiles/company/223951-24 | 企業DB |
| 59 | Healthcare IT News - Akiri $10M funding | https://www.healthcareitnews.com/news/blockchain-network-service-platform-scores-10-million-ama-backed-health2047 | 業界ニュース |
| 60 | CBInsights - Gem (acquired by Blockdaemon) | https://www.cbinsights.com/company/bitvault | 企業DB |
| 61 | Hashed Health 公式サイト | https://www.hashedhealth.com/ | 公式(企業) |
| 62 | Medium - Why companies exiting healthcare blockchain | https://medium.com/@Connected_Dots/why-are-so-many-companies-exiting-the-healthcare-blockchain-market-43cddbea4431 | Medium |
| 63 | HIT Consultant - Patientory 2023 launch | https://hitconsultant.net/2023/04/05/patientory-launches-blockchain-app/ | 業界ニュース |
| 64 | CoinMarketCap - PTOY price | https://coinmarketcap.com/currencies/patientory/ | 市場データ |
| 65 | Latka - BurstIQ $3M revenue 2024 | https://getlatka.com/companies/burstiq | 企業データ |
| 66 | PMC - Systematic Literature Review 2025 (82 studies) | https://pmc.ncbi.nlm.nih.gov/articles/PMC12071524/ | 学術(査読) |
| 67 | PMC - Blockchain in Health Information Systems 2024 | https://pmc.ncbi.nlm.nih.gov/articles/PMC11593537/ | 学術(査読) |
| 68 | Kyndryl - Dubai NABIDH 9.47M records | https://www.kyndryl.com/us/en/about-us/news/2024/11/digitizing-healthcare-services-for-dubai-health-authority | 公式(企業) |
| 69 | ScienceDirect - 2024 Chinese guideline on medical blockchain | https://www.sciencedirect.com/science/article/pii/S2667102624000640 | 学術 |
| 70 | IBM Watson Health sold to Francisco Partners - Healthcare IT News | https://www.healthcareitnews.com/news/ibm-sell-watson-health-assets-francisco-partners | 業界ニュース |
| 71 | Blockchain Council - UAE health data platform | https://www.blockchain-council.org/blockchain/uae-launches-health-data-platform-powered-by-blockchain/ | 専門メディア |
| 72 | Healthcare Dive - blockchain huge promise largely unproven | https://www.healthcaredive.com/news/blockchain-in-healthcare-huge-promise-but-largely-unproven/529666/ | 業界ニュース |
| 73 | Medicalchain 公式 / Dr. Albeyatti(Doctorpreneurs) | https://medicalchain.com/en/ | 公式(企業) |
| 74 | HL7 to FHIR Migration - Roving Health | https://www.rovinghealth.com/articles/hl7-to-fhir-migration-modernizing-legacy-integrations | 技術ブログ |
| 75 | A problem of Epic proportion - PLOS Digital Health | https://journals.plos.org/digitalhealth/article?id=10.1371%2Fjournal.pdig.0001143 | 査読論文 |
| 76 | Towards real-world clinical data standardization - ScienceDirect | https://www.sciencedirect.com/science/article/pii/S0010482525000952 | 査読論文 |
| 77 | Commoning Semantic Interoperability in Healthcare - Commons Journal | https://thecommonsjournal.org/articles/10.5334/ijc.1157 | 査読論文 |
| 78 | EHR Interoperability in 2026: FHIR, TEFCA - EHRSource | https://www.ehrsource.com/articles/ehr-interoperability-guide/ | 技術記事 |
| 79 | Information blocking remains prevalent - PMC | https://pmc.ncbi.nlm.nih.gov/articles/PMC7973451/ | 査読論文 |
| 80 | Conway's Law - Wikipedia | https://en.wikipedia.org/wiki/Conway's_law | 参考資料 |
| 81 | Health Information Exchange: Policy Landscape - PMC | https://pmc.ncbi.nlm.nih.gov/articles/PMC10751121/ | 査読論文 |
| 82 | Barriers to Health Information Exchange - AJMC | https://www.ajmc.com/view/barriers-to-health-information-exchange | 査読誌 |
| 83 | Interoperability - ONC(公式) | https://www.healthit.gov/topic/interoperability | 公式 |
| 84 | Political Economy of Digital Health Equity - PMC(JMIR) | https://pmc.ncbi.nlm.nih.gov/articles/PMC11005444/ | 査読論文 |
| 85 | Vendor Lock-In Economics: Nash Equilibrium - SoftwareSeni | https://www.softwareseni.com/vendor-lock-in-economics-understanding-the-nash-equilibrium-you-are-already-in/ | 技術ブログ |
| 86 | Legal Barriers to HIE: Boulders or Pebbles? - PMC | https://pmc.ncbi.nlm.nih.gov/articles/PMC5835678/ | 査読論文 |
| 87 | Game Theory and Why Healthcare Keeps Losing - Medium | https://medium.com/@jrichardscc25/game-theory-and-why-healthcare-keeps-losing-f03d7a55e289 | 分析記事 |
| 88 | Who Owns Your Health Data? - Modern Diplomacy | https://moderndiplomacy.eu/2025/05/15/who-owns-your-health-data-inside-the-global-battle-between-regulation-and-innovation/ | 分析記事 |
| 89 | Estonia launches $10 EHR - Healthcare IT News | https://www.healthcareitnews.com/news/estonia-launches-10-ehr | 業界ニュース |
| 90 | E-Health in Denmark - Healthcare Denmark | https://healthcaredenmark.dk/national-strongholds/digitalisation/digital-infrastructure/ | 公式 |
| 91 | Finland-Estonia centralisation/decentralisation - ESPON | https://archive.espon.eu/estonia-and-finland-question-centralisation-and-decentralisation-of-digital-healthcare | 政策研究 |
| 92 | Japanese EMRs and IT in Medicine - PMC (2012・要更新) | https://pmc.ncbi.nlm.nih.gov/articles/PMC3212745/ | 査読論文 |
| 93 | COVID-19 Japanese healthcare data system - PMC | https://pmc.ncbi.nlm.nih.gov/articles/PMC8944184/ | 査読論文 |
| 94 | Japan's My Number ID backlash - Unseen Japan | https://unseen-japan.com/japan-my-number-backlash/ | 報道記事 |

---

*本レポートは research:controller-search ワークフロー（plan→parallel-research×4→integrate→critique[PASS]→report）で生成。信頼度ラベル: verified=一次資料/公式統計で確認 / deduced=複数ソース・理論から導出 / inferred=推測 / uncertain=不確実。*
