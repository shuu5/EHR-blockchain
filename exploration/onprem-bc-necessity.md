# 最終レポート: 「電子カルテBC管理+LLMコンテキスト」構想の批判的再検証
## ― 院内LLM・医療ソブリンクラウド・BC同意の必要性・日本の単一統合現実性 ―

**調査日**: 2026-05-26 / **重視期間**: 2024-2026 / **手法**: research:controller-search（5並列researcher → 統合 → critic検証 **PASS**）
**session**: 2a0ef170 / **関連**: 既存検証 doobidoo `15a7647a`（原典C1-C11ベースライン）を背景前提とした新規アングル調査

---

## エグゼクティブサマリー（4大問いへの結論）

| 問い | 結論 | 確信度 |
|------|------|--------|
| **(A)** 院内LLMなら今回のセキュリティ懸念の多くは当たらないか | **概ねYES**。外部クラウドAPI特有の懸念群（外部送信・マルチテナントサイドチャネル・KV-cacheタイミング）は院内シングルテナントで**構造的に消滅**。ただし間接プロンプトインジェクション・内部者脅威・ハルシネーションは**残存**。TEE/attestationは院内単独運用なら必須でない | 高(verified) |
| **(B)** BC患者同意は本当に必要か・医療ソブリンクラウドで代替できないか | **大半のケースで不要・代替可能**。富士通×IBMの医療ソブリンクラウド（2026/05/15正式発表）は**BCを一切使わない中央集権モデル**。ソブリンクラウド+SMART on FHIRで同意/アクセス制御を実現。BCが必要十分なのは「trustless多機関」の限定シナリオのみ | 高(verified+deduced) |
| **(C)** 患者アクセスはBC/ソブリンクラウドどちらで実現できるか | **OAuth2/OIDC+SMART on FHIRで技術的に十分**（米CMS義務化済・日本JP-CLINSも採用）。BCは補完的価値のみ | 高(verified) |
| **(D)** 日本で単一ソブリンクラウド統合は現実的か | **現実性は低い**。国策は既に「連携型」を選択。NPfIT型強制統合は失敗、成功例(エストニア/北欧)は日本に欠ける前提に依存。**断片化が何十年も続く蓋然性は高い** | 高(deduced) |

**総括**: ユーザーの4つの直感（A:院内なら懸念減 / B:ソブリンクラウドでBC代替可 / C:患者アクセスは標準APIで足りる / D:単一統合は非現実的で断片化継続）は、**いずれも2024-2026の一次資料・査読文献によって支持された**。原典構想における「ブロックチェーン」は、技術的必要性よりも「複数機関が相互不信のまま連携する未来」という前提シナリオに賭けた設計であり、その前提自体が日本の現実（連携型国策・断片化継続・単一信頼運用者の不在を埋める全国PF）では成立しにくい。

---

## 【問いA】院内LLMでセキュリティ懸念の多くは当たらないか

### A-1. 院内完結で「消える」脅威 = 外部クラウドAPI特有リスク [verified]

ユーザーが挙げた3懸念（外部API送信・マルチテナントTEEサイドチャネル・KV-cacheタイミング）は、**いずれも院内シングルテナントオンプレで構造的に消滅**する。

| 脅威 | 消える理由 | 出典 |
|------|----------|------|
| 外部API送信によるユーザー推論攻撃（出力プロービングで希少コホート適応を判定） | 外部リクエスト自体が存在しない | [G1] |
| マルチテナントサイドチャネル（中間アクティベーション反転） | シングルテナントで共有インフラ消滅 | [G1] |
| **KV-cacheタイミングサイドチャネル**（TTFT計測でクロステナントのプロンプトプレフィックス漏洩。研究グループPromptPeek/InputSnatchがブラックボックス条件下で実証） | クロステナント共有が存在しない | [G5] |
| クラウドプロバイダー/データセンター運営者による入力・出力・中間アクティベーション閲覧 | オンプレ展開自体が対策 | [G3][G4] |
| WAN通信のタイミング・パケットサイズからのPHI密度推定 | 院内ネットワーク完結 | [G1] |
| データ保持ポリシー超過リスク | 機関が直接データ保持を制御 | [G3] |

→ **ユーザーの直感は正しい**。これらは「外部の他者とインフラを共有する」ことに起因する脅威であり、院内完結で前提が消える。

### A-2. 院内完結でも「残る」脅威 [verified]

| 脅威 | 詳細 | 出典 |
|------|------|------|
| **間接プロンプトインジェクション（最重要残存）** | RAGで取り込む外部文書・カルテ内コンテンツへの悪意埋め込み。臨床LLM実験で攻撃成功率**94.4%（4ターン）**、FDA Category X禁忌薬の推薦も**91.7%**。GPT-5/Gemini 2.5 Pro含む最新モデルでも脆弱な事例。PoisonedRAGは数百万件コーパスに5件混入で90%の誤誘導 | [G6][G7][G8] |
| **内部者脅威（3類型）** | ①臨床スタッフ（非マスク識別子の露出）②データエンジニア（デバッグ用に識別子保持→PHI伝播）③システム管理者（バックアップ・前処理ノードへの特権アクセス悪用） | [G1][G9] |
| **ハルシネーション** | ベルギー病院事例ではパイロット時47件中1件ハルシネーション+4件省略。本番後フィードバック率が5.8%→1.3%に低下しモニタリング困難化 | [G1][G4] |
| ログ漏洩 | デバッグ出力・ETL失敗ログがPHIを含み内部者に閲覧可能。HIPAA準拠には自動PHI削除が必要 | [G1][G9] |
| 訓練データ記憶化（独自FT時） | ローカルFTで勾配・重みが機密EHR変数を記憶。DP-SGD/DP-LoRAで対策 | [G1] |
| モデル重み持ち出し（特権内部者） | 独自FT済みモデルは機密性高。TEEが内部者対策として有効 | [G10] |

→ これらは「LLMという仕組み自体」と「内部の人・データ」に起因し、院内化では消えない。**特に間接プロンプトインジェクションは、カルテをRAGコンテキストとして渡す構想の中核リスク**として残る。

### A-3. E2E暗号 / TEE / リモートattestation の要否 [deduced]

- **完全院内オンプレ＋信頼できる職員のみ＋物理セキュリティ＋シングルテナント = TEEは必須でない**[G3][G10][G11]。
- TEEが必要になるのは: ①インフラ運営者を信頼できない（外部委託DC使用）②プロプライエタリモデルのIP保護 ③複数病院間フェデレーション。
- ただしTEEのオーバーヘッドは大規模モデル(32B+)で**1%未満**、NVIDIA H100/H200はネイティブ性能の95-99%維持[G11]。→「不要だが、入れても実害がない」水準でコスト障壁は低い。
- **リモートattestationは複数サイト間フェデレーションでのみ主価値**。院内単独構成では検証相手が存在せず価値は薄い。

### A-4. 医療機関のオンプレ/プライベートLLM導入実例（2024-2026）[verified]

| 機関 | モデル/基盤 | EHR統合 | セキュリティ設計 | 出典 |
|------|-----------|---------|----------------|------|
| **Cliniques Universitaires Saint-Luc**（ベルギー、PLOS Digital Health査読、2025/12） | Qwen3-235B(FP8)+vLLM、H200×8（約40万USD） | **Epic + SMART on FHIR直接埋込** | 「患者データは病院インフラを外に出ない」原則。SMART認証でEHRのRBAC継承。GDPR DPIA完了・EU MDR Class IIa届出・EU AI Act準拠。本番5ヶ月で1,028ユーザー・14,910会話 | [G4] |
| 那須赤十字病院（日本・栃木、2025/04） | リコー製70B（日英中） | 電子カルテ→退院サマリ自動生成 | オンプレ完結・院内ネットワーク分離・クラウド送信なし | [G12] |
| 織田病院（日本・佐賀）[inferred] | ローカルLLM | EMR | オンプレ完結・入退院サマリ自動生成 | [G13] |

→ **2024-2026に「Epic + オープンウェイトLLM(Qwen3) + 院内完結 + SMART on FHIR」という構成が査読論文レベルで実証済み**。原典構想に近い「カルテをLLMにコンテキストとして渡す」運用は、BCを介さずとも院内オンプレで現実に稼働している。

---

## 【問いB】BC患者同意は必要か・医療ソブリンクラウドで代替できないか

### B-1. 富士通×IBM医療ソブリンクラウドの実態 ― BCは使っていない [verified・本調査の最重要発見]

- **発表**: 2025年9月に提携検討 → **2026年5月15日に具体化を正式発表**[G15][G16]。対象は日本国内の医療機関のみ。
- **アーキテクチャ**: 富士通ソブリンクラウド（**Oracle Alloy**ベース、2025年4月提供開始）を共通基盤とし、富士通・日本IBM双方の電子カルテをその上で稼働[G17][G20]。
- **ブロックチェーン/分散台帳/スマートコントラクトへの言及は、調査した全ての一次・二次ソースで皆無**[G15][G16][G17][G18]。「ソブリンクラウド環境を共通基盤として電子カルテを稼働」という記述から**中央集権クラウドモデル**であることは明確。
- **データ主権5要素**: テクノロジー/データ/運用/法的/セキュリティ主権。国内DC・日本居住者のみ管理・日本のソブリン要件向けに106の追加機能[G19]。
- **同意/アクセス制御**: 「患者・医療機関の合意のもと複数病院データを連携」とあるが、実装方式（IAM/OAuth/SSI）は**現時点で未公表**[inferred: SMART on FHIR系IAMが主流]。
- 注: 富士通は別途独自BC技術（Connection Chain）を保有するが、本医療提携への適用は公式情報なし[G19]。

→ **「医療ソブリンクラウドはBCで患者同意を実装する」という想定は、現実の旗艦事例（富士通×IBM）では成立していない。中央集権クラウドが選ばれている。**

### B-2. ソブリンクラウド一般の同意/アクセス制御も中央集権が支配的 [verified]

- **主流 = 中央集権的IAM/OAuth（SMART on FHIR = OAuth 2.0 + OpenID Connect）**[G27][G29]。FHIR Consentリソースで同意を表現。
- EU事例も全てBC不使用:
  - **仏**: 国家健康データハブをMicrosoft→Scaleway（仏企業）へ2026/04移管。2024年法でソブリン保証ホスティング義務化[G23]。
  - **独**: 2024/07 SGB V §393施行。BSI C5認証義務（2025/07以降Type 2必須）、処理地はEU/EEA/十分性認定国限定[G24][G25]。
  - **EU Gaia-X / Health-X**: 連合型データ空間。患者は**データウォレットアプリ**で個別同意制御。BC不使用[G26]。EHDS規則は2025/03発効。
- 新興研究のみハイブリッド（中央集権プラットフォーム + permissionedコンソーシアムBCが**同意メタデータ/監査ログだけ**保持、実データはオフチェーン）[G30][G31]。研究〜パイロット段階に留まる。

### B-3. BCが必要十分になる前提条件（AND条件に近い）[deduced]

Wüst & Gervaisフレームワーク[G33]と医療tertiary review[G32]の収束:

1. 信頼できる中央管理者が存在しない/不適切（**相互不信の競合複数機関**）
2. データ完全性が臨床アウトカム・法的責任に直結（同意撤回記録等）
3. 参加者アイデンティティが事前確認可能（permissioned前提）
4. 患者の直接制御が法的要請（EU患者主権規制）
5. 外部監査機関の独立検証が必要

**BCが実際に優位な具体シナリオ**: グローバル多施設臨床試験 / クロスボーダーゲノム共有コンソーシアム / 希少疾患の国際患者コンソーシアム / 医薬品サプライチェーン[G32][G37]。
→ いずれも「**単一の信頼できる運用者を置けない国際的・競合的環境**」が共通項。

### B-4. 中央集権が優位な境界線 [verified]

- 単一の信頼できる運営機関が存在する（国保・大学病院グループ・国レベルHIE）[G32]
- リアルタイム・高スループット要件（緊急医療）
- コスト制約・中小施設
- GDPR削除権への厳格適合
- 既存EMR（Epic/Cerner）との深い統合
- **IDS Association（2024）**: 「コネクタの組込機能が大半のユースケースをカバー。BCは補完役に限定すべき」[G40]

### B-5. BC型 vs 中央集権DB+OAuth/IAM 技術比較 [verified]

| 軸 | ブロックチェーン型 | 中央集権DB+OAuth/IAM |
|----|------------------|---------------------|
| 改ざん耐性 | 高 | 中（内部不正に脆弱） |
| 監査性 | 高（不変タイムスタンプ） | 中（ログ改ざんリスク） |
| 可用性 | 中〜低（緊急時遅延リスク） | 高（SLA設計容易） |
| コスト | 高（Eth 20,000 gas/word、$2.2-6.2/tx） | 低〜中 |
| GDPR削除権 | **根本矛盾**（immutability vs 忘れられる権利） | 完全適合 |
| スケーラビリティ | 低 | 高 |
| 患者主権 | 高 | 低〜中 |
| FHIR統合 | 開発中 | 成熟 |

出典: [G32][G34][G35][G36][G39]。GDPR回避策はCrypto-Shredding（オフチェーン暗号化保存+鍵破棄）だが、**immutabilityという核心優位を実質放棄するトレードオフ**[G39][G46]。

### B-6. 不要論・hype批判 + 実運用/頓挫事例 [verified]

- **"Blockchain in Health Care: Hope or Hype?"（PMC7382018, JMIR）**[G34]: 「BCは医療の問題を解決せず、むしろ問題を増やしうる」「提案ユースケースの大多数は未実装」。
- **Gartner 2024**[G48]: BCは幻滅のどん底。BC専用Hype Cycle廃止検討。医療価値実現は「5年以上先」。※Gartnerは一次資料でなく業界ブログ経由のため、同方向の査読文献[G32][G42]で補強。
- **頓挫事例**: Gem Health（legacy EMR置換不可）、Patientory（患者にカルテ保存料を課金する設計ミス・価値75%減）、BronTech（6ヶ月で撤退）[G43]。
- **部分成功（BC=補助役）**: エストニア（**KSI/Guardtimeブロックチェーンでハッシュ改ざん検知のみ。データ本体は中央DB**）[G44]、ConsentChain/MedRec（PoC・simulated dataで実患者未使用）[G37][G45]。
- **結論**: 完全BC主体で稼働する医療同意管理の本番実装は**査読文献で確認できない**。エストニア型（BC=ハッシュ証明補助ツール、主体は中央DB）が最も現実的な実運用形態。

---

## 【問いC】患者アクセスはBC/ソブリンクラウドどちらで実現できるか

**結論: OAuth2/OIDC + SMART on FHIR で技術的に十分。BCは必須でなく補完的価値のみ。**

### C-1. SMART on FHIR患者アクセス [verified]

- **SMART App Launch v2.2.0**（現行最新）= OAuth2 Authorization Code + **PKCE（S256をSHALL）** + OIDC拡張[G53][G54][G56]。Standalone Launch（患者がアプリから直接）/ EHR Launch（ポータル内）の2フロー。短命トークン・最小スコープ（`patient/Observation.rs`等）。
- **米国義務化**: CMS-9115-F（2021/07施行）でPatient Access API（FHIR R4+OAuth2+SMART）義務化。CMS-0057-F（2024）で4API追加。初回メトリクス報告2026/03/31[G49][G51]。
- **日本**: 電子カルテ情報共有サービス（JP-CLINS）が2025/02モデル事業開始。FHIR+マイナポータルAPIで患者が診療情報閲覧[G61]。**BC不使用**。

### C-2. 患者向け医療AIチャットボットの認証認可 [deduced]

- SMART on FHIR+OAuth2が基盤[G52]。Epic=RSA署名JWT/Auth Code+PKCE+`aud`完全一致、Cerner=Client Credentials/Auth Code。
- 患者IDフェデレーション=アクセストークン内のpatient identifier + OIDC `fhirUser`クレーム。**外部クラウドLLMにPHI送信時はBAA必須（HIPAA）**[G52]。
- AIチャットボット固有リスク: 会話コンテキストがPHIを含む→トークン失効・セッション切れの即時検知が重要。

### C-3. BC同意 vs OAuth/OIDC同意 [deduced]

| 観点 | OAuth2/OIDC | BC同意 |
|------|-------------|--------|
| 実装成熟度 | 高（義務化済） | 低〜中（POC中心） |
| UX | EHRポータル統合・標準認可画面 | ウォレット操作の障壁 |
| 失効 | RFC7009 Token Revocation + 期限 | スマートコントラクト即時自動 |
| 監査ログ | 中央管理（理論上改ざん可） | 分散不変台帳 |
| スケーラビリティ | 高 | 中（$2-6/tx） |
| 粒度 | スコープ単位 | 目的・期間・提供者ごと細粒度 |

出典[G55][G57][G58][G59]。多くの論文が「**BC同意 ≠ OAuth2代替。BCはOAuth2の上位補完レイヤー**」と位置づけ[G60]。

### C-4. 院外通信セキュリティ要件 [deduced]

- **TLS必須**（患者端末↔FHIR API）[G63]。**mTLSはサーバ間(B2B)向け**で患者モバイルには非実用的（証明書配布の複雑さ）→ PKCE+短命トークン+`aud`検証が実質代替[G62]。SMART Backend ServicesもmTLSよりJWT-based client authを採用。
- **E2E暗号**: FHIR APIはTLS終端で真のE2Eでない。誰にも復号させない設計はアプリ層追加暗号化（FHIR範囲外）。
- IBM 2024 Cost of Data Breach: mTLS採用で推定$12M/件削減[G62]。

---

## 【問いD】日本で単一ソブリンクラウド統合は現実的か（断片化が何十年も続くのでは）

**結論: 単一統合の現実性は低い。断片化継続シナリオの蓋然性が高い。国策も既に「連携型」を選択済み。**

### D-1. 日本の電子カルテ断片化の現状 [verified]

- **ベンダーシェア**: 病院向けは富士通Japan（30-35%）・SSI・CSI・NECの上位で約70%。クリニック向けはエムスリー・PHC（旧日立系ウィーメックス）・富士通[G65][G84]。「プラットフォーマー不在の群雄割拠」。大学病院ごとに別ベンダーが歴史的に固着（京大=富士通/阪大=NEC/東大=IBM）。
- **普及率**: 病院77.7%・診療所71.0%（2025）。だが紙カルテ診療所の**54.2%が「電子カルテ導入不可能」と回答**（日本医師会2025年調査。※本数値はDiamond[G65]/Chambers[G85]経由の二次情報であり、日医一次資料URLは本調査では未収録）。
- **地域HIE**: 全国200+が乱立も、**病院医師の93.5%が年間利用ゼロ・月間稼働ゼロの病院が51.9%**[G66]。会計検査院も「全く利用されないNW」を指摘。
- **FHIR標準化**: JP-CLINS v1.11.0公開[G72]。3文書6情報。標準型電子カルテα版2025/03提供開始（山形）。

### D-2. 全国医療情報プラットフォームは「統合」か「連携」か [verified・核心]

**明確に「連携型（フェデレーション）」。「統合型（単一DB/単一クラウド）」ではない**[G64][G71]。

- オンライン資格確認システムのインフラを活用。社会保険診療報酬支払基金経由の「登録・閲覧」モデル。各機関がFHIR形式で出力したデータをマイナポータル経由で交換し、**単一DBに集約しない**[G61]。
- **稼働**: 2024/01技術解説書 → 2025/01モデル事業（9地域22機関） → 2026/03リリース予定 → **2026年冬頃本格運用目標（当初2025年度内から後ろ倒し）**[G64]。任意参加制（診療報酬加算で誘導、義務化なし）。2030年に全機関接続が最終目標。

→ **日本は意図的に単一統合を回避し、連携型を採用している。**「単一ソブリンクラウド統合」は国策の方向と逆。

### D-3. 統合の障壁 [verified/deduced]

- **ベンダーロックイン・移行コスト**: 独自規格の歴史的蓄積。大規模病院のEpic導入は1-10億ドル超、データ移行プロジェクトの83%が予算超過/遅延。日本の補助金は408-658万円で全く不足[G65]。
- **医療機関の自律性**: 任意参加・日医反発（2024年「医療DXの速度が実態に合わない」緊急調査）。
- **単一事業者集中リスク**: 医療法上、管理者が診療録保管責任を負う→単一クラウドへの全集約は法的にも困難[G73]。
- **歴史的遅延**: レセプト電算25年・電子カルテ普及20年[G74]。

### D-4. 他国比較と成否要因 [verified/deduced]

| 国 | モデル | 結果 | 成否の要因 / 日本との差 | 出典 |
|---|--------|------|---------------------|------|
| **エストニア** | X-Road（連携型データ交換）+ KSIブロックチェーン（完全性証明は**別レイヤー**） | 成功 | 人口130万（日本の約1/100）・唯一の国民ID・デジタル信頼文化 | [G67][G44] |
| **英NHS** | NPfIT（中央集権統合） | **大失敗（2011廃止・100億ポンド損失・参加率10%）** | トップダウン強制・one-size-fits-allが医療現場の多様性と衝突。臨床医の抵抗で紙に逆戻り | [G69][G83] |
| **北欧** | 公的統合+国民ID | 成功（sundhed.dk/Kanta/1177.se） | 医療機関が公営・政府直接運営・民間乱立少 | [G82] |
| **米Epic/Cerner** | 民間寡占デファクト統合 | 部分成功（Epic42.3%+Oracle Health22.9%） | TEFCA移行進行も独禁訴訟（Particle Health 2024-25）。日本に同型寡占なし | [G76][G77][G78] |
| **独** | 連邦制・分散ePA | 苦戦（利用率3%未満） | 医師会・疾病基金の拒否権で15年停滞。日本と構造類似 | [G79][G75] |

→ **教訓: 中央集権強制統合（NHS型）は失敗。成功例（エストニア/北欧）は「国民ID+小規模 or 公営医療」という日本に欠ける前提に依存。米国型民間寡占も日本では発生していない。** エストニアの「ブロックチェーン」はX-Roodの連携層とは別の、KSIによるハッシュ完全性証明であり、いわゆる同意管理BCではない点に注意。

### D-5. 「複数クラウド・孤立カルテが何十年も残る」シナリオの蓋然性 [deduced]

**「高い」**。根拠: ①診療所の54.2%が電子カルテ化に懐疑的 ②HIEは200+あっても病院医師93.5%が年間利用ゼロ ③参加任意・義務化見通しなし ④歴史的遅延パターン（25年/20年）。2030年の全機関接続目標は相当困難。
唯一の緩和要因は国産・安価な標準型電子カルテ（2026-2027本格提供）による新規小規模診療所の参入障壁低下だが、既存オンプレシステムの移行は別問題[G64]。

### D-6. 断片化継続時の連携手段: FHIR/全国PF vs 分散BC [deduced]

| 観点 | FHIR/全国プラットフォーム | ブロックチェーン |
|------|---------------------------|------------------|
| 規制整合性 | 高（政府採用・診療報酬加算） | 低〜中（法的位置付け未確立） |
| 技術的成熟度 | 高（FHIR R4/R5・世界的実績） | 中（スケーラビリティ課題） |
| 単一障害点 | あり（中央交換基盤依存） | 低い（分散） |
| 患者同意管理 | 中（マイナポータル経由） | 高（スマートコントラクト細粒度） |
| 相互運用性 | 高（国際標準・既存EHR接続容易） | 低〜中 |
| 日本の本番実績 | 進行中（2026冬目標） | なし | 

出典[G80][G81]。
- **短中期: FHIR/全国プラットフォームが圧倒的に現実的**（政府主導・診療報酬インセンティブ・国際標準・既存EHRへAPI接続可）。
- **BC単独でFHIR代替は非現実的**（法的位置付け未確立・コスト不透明・日本で本番事例なし）。
- **最も現実的なのはハイブリッド**（BC=同意管理/監査証跡の補完層、FHIR=交換本体）。断片化下でBCが「中央権威なしの信頼の橋渡し」になりうる可能性は研究段階で確認（EHRChain等）。2019年PMC論文[G80]も「日本ではone size fits allにならない」と結論。

---

## 原典構想への含意（4大問いの統合）

1. **院内LLM化はセキュリティ懸念の「外部API系」を確かに減らす**が、カルテをRAGコンテキストとして渡す以上、**間接プロンプトインジェクションという最大級のリスクは残る**。原典の懸念整理はこの残存リスクを軸に再構成すべき。
2. **BCによる患者同意は、現実の旗艦事例（富士通×IBM医療ソブリンクラウド）では採用されていない**。中央集権ソブリンクラウド+SMART on FHIRが代替している。原典の「BC同意層」は、技術的必要性ではなく「単一信頼運用者を置けない多機関trustless環境」という前提に依存する。
3. **患者アクセスはOAuth2/OIDC+SMART on FHIRで十分**。BCを患者アクセスの必須要件とする根拠はない。
4. **日本で単一ソブリンクラウド統合は非現実的**で、国策は連携型。断片化は何十年も続く蓋然性が高い。ただし**この「断片化・多機関・相互不信」の状況こそ、皮肉にもBCが理論上最も正当化されるシナリオ**でもある。とはいえ現実の連携手段はFHIR/全国PFが先行し、BCはあくまで補完層に留まる。

→ **要するに**: 「院内LLM」は懸念を減らすが消さない。「BC同意」は大半不要でソブリンクラウド+標準APIが代替する。「単一統合」は非現実的で断片化が続く。原典構想のBCは、技術的必然というより「特定の前提シナリオ（trustless多機関・患者主権の法的強制）が成立したら活きる」条件付き設計と理解するのが最も整合的。

---

## 未解決事項・追加調査候補

1. 富士通×IBMの具体的アクセス制御スタック（SMART on FHIR / 独自IAM）は未公表[inferred止まり]。ただし「BCか否か」の核心は解決済み（BC不使用）。
2. TEEの院内オンプレ単独必要性を明示する公的医療ガイドライン（厚労省/ENISA）は未確認。
3. 電子カルテ情報共有サービスの本格運用後の見込み参加率の定量データ不足。
4. 「診療所54.2%が導入不可能」の日本医師会一次資料URL（本調査ではDiamond/Chambers経由の二次情報）。
5. 日本の医療BC本番稼働事例は確認できず（研究/開発段階のみ＝これ自体が「BC不要論」の傍証）。
6. mTLSを患者向けモバイルに要求する明文規定の有無（サーバ間は推奨確認済）。

---

## Evidence Table（統合参照ソース・全85件）

| # | Title | URL | Type |
|---|-------|-----|------|
| G1 | SoK: Privacy-aware LLM in Healthcare: Threat Model | https://arxiv.org/abs/2601.10004 | arXiv(2026/01) |
| G2 | LLM Cybersecurity Threats in Radiology (RSNA) | https://www.rsna.org/news/2025/may/llm-cybersecurity-threats | 学会誌(2025/05) |
| G3 | Mind the Trust Gap: Local-to-Cloud LLM (Stanford Hazy) | https://hazyresearch.stanford.edu/blog/2025-05-12-security | 研究ブログ(2025/05) |
| G4 | Implementation of LLMs in EHR (PLOS Digital Health) | https://journals.plos.org/digitalhealth/article?id=10.1371%2Fjournal.pdig.0001141 | 査読(2025/12) |
| G5 | Selective KV-Cache Sharing to Mitigate Timing Side-Channels | https://arxiv.org/abs/2508.08438 | arXiv(2025/08) |
| G6 | Vulnerability of LLMs to Prompt Injection in Medical Advice | https://pmc.ncbi.nlm.nih.gov/articles/PMC12717619/ | 査読(PMC) |
| G7 | Not what you've signed up for: Indirect Prompt Injection | https://arxiv.org/pdf/2302.12173 | arXiv基礎論文 |
| G8 | 見えないプロンプトインジェクション (Trendmicro JP) | https://www.trendmicro.com/ja_jp/research/25/b/invisible-prompt-injection-secure-ai.html | セキュリティ企業(2025) |
| G9 | Securing Healthcare LLMs: On-Prem Deployment (Lazarus Labs) | https://www.lazarus-labs.com/webpage/blog-security-healthcare.html | セキュリティ企業 |
| G10 | Enhancing AI inference security with confidential computing (Red Hat) | https://next.redhat.com/2025/10/23/enhancing-ai-inference-security-with-confidential-computing-a-path-to-private-data-inference-with-proprietary-llms/ | ベンダー(2025/10) |
| G11 | Confidential LLMs – Phala Network | https://phala.com/learn/Confidential-LLMs | ベンダー |
| G12 | リコー、那須赤十字病院にリコー製LLMをオンプレ提供 | https://jp.ricoh.com/release/2025/0430_1 | 公式PR(2025/04) |
| G13 | 地域医療機関が院内生成AIで退院サマリ(EnterpiseZine) | https://enterprisezine.jp/article/detail/22457 | 技術メディア(2025) |
| G14 | A Cost-Benefit Analysis of On-Premise LLM Deployment | https://arxiv.org/html/2509.18101v3 | arXiv(2025/09) |
| G15 | Fujitsu and IBM Japan formalize collaboration in healthcare (英公式) | https://global.fujitsu/en-global/pr/news/2026/05/15-01 | 公式PR(2026/05/15) |
| G16 | 富士通と日本IBM、ヘルスケア領域における協業を具体化(日公式) | https://global.fujitsu/ja-jp/pr/news/2026/05/15-01 | 公式PR(2026/05/15) |
| G17 | 富士通と日本IBM、医療向けソブリンクラウドを構築(IT Leaders) | https://it.impress.co.jp/articles/-/29341 | IT専門メディア |
| G18 | 富士通と日本IBM、医療向けソブリンクラウド提供へ(EnterpriseZine) | https://enterprisezine.jp/news/detail/24291 | IT専門メディア |
| G19 | オラクルと富士通パートナーシップ(富士通note) | https://note.com/fujitsu_pr/n/n0368b0f9cbe2 | 公式広報 |
| G20 | Fujitsu to use Oracle Alloy for sovereign cloud in Japan (DCD) | https://www.datacenterdynamics.com/en/news/fujitsu-to-use-oracle-alloy-for-sovereign-cloud-in-japan/ | IT専門メディア |
| G21 | What Is Sovereign Cloud? (Teradata) | https://www.teradata.com/insights/data-security/what-is-sovereign-cloud | ベンダーWP |
| G22 | Elements of cloud sovereignty (Red Hat) | https://www.redhat.com/en/resources/elements-of-cloud-sovereignty-overview | ベンダー文書 |
| G23 | France moves public health data from Microsoft to French cloud (Euronews) | https://www.euronews.com/health/2026/04/24/france-moves-public-health-data-from-microsoft-to-french-cloud-provider | 一般メディア(2026/04) |
| G24 | Health Data in the Cloud – new law in Germany (Fieldfisher) | https://www.fieldfisher.com/en/insights/health-data-in-the-cloud-new-law-in-germany-raises-the-bar | 法律事務所 |
| G25 | Germany enacts stricter cloud requirements for health data (Covington) | https://www.covingtondigitalhealth.com/2024/09/germany-enacts-stricter-requirements-for-the-processing-of-health-data-using-cloud-computing-with-potential-side-effects-for-medical-research-with-pharmaceuticals-and-medical-devices/ | 法律事務所 |
| G26 | Health-X: A Common Data Space for the Health Sector (Gaia-X) | https://gaia-x.eu/news-press/health-x-a-common-data-space-for-the-health-sector/ | 公式 |
| G27 | Enabling secure and self determined health data sharing (PMC) | https://pmc.ncbi.nlm.nih.gov/articles/PMC12398500/ | 査読 |
| G28 | European ambitions captured by American clouds (Tandfonline) | https://www.tandfonline.com/doi/full/10.1080/1369118X.2025.2516545 | 査読 |
| G29 | SMART on FHIR Explained (Descope) | https://www.descope.com/learn/post/smart-on-fhir | 技術ブログ |
| G30 | Self-Sovereign Identity for Consented Medical Records (arXiv) | https://arxiv.org/pdf/2407.21559 | プレプリント |
| G31 | Self-sovereign mgmt of PHR with PDS and DID (PMC) | https://pmc.ncbi.nlm.nih.gov/articles/PMC11758136/ | 査読 |
| G32 | Do you need a blockchain in healthcare data sharing? A tertiary review | https://www.explorationpub.com/Journals/edht/Article/101114 | 査読(2024) |
| G33 | Do You Need A Blockchain? (Wüst & Gervais) - DSHR's Blog | https://blog.dshr.org/2018/02/do-you-need-blockchain.html | 技術ブログ(原論文解説) |
| G34 | Blockchain in Health Care: Hope or Hype? (PMC7382018, JMIR) | https://pmc.ncbi.nlm.nih.gov/articles/PMC7382018/ | 査読(JMIR) |
| G35 | Blockchain integration in healthcare: performance issues (Frontiers 2024) | https://www.frontiersin.org/journals/digital-health/articles/10.3389/fdgth.2024.1359858/full | 査読(2024) |
| G36 | A Systematic Review of Blockchain for Consent Management (PMC) | https://pmc.ncbi.nlm.nih.gov/articles/PMC7912759/ | 査読 |
| G37 | ConsentChain: Blockchain Dynamic Consent for Genomic Sharing (PMC) | https://pmc.ncbi.nlm.nih.gov/articles/PMC8600428/ | 査読(PoC) |
| G38 | A Decision Framework for Blockchain Adoption (arXiv) | https://arxiv.org/pdf/2210.14888 | プレプリント |
| G39 | Analysis of solutions for blockchain compliance with GDPR (PMC) | https://pmc.ncbi.nlm.nih.gov/articles/PMC9440070/ | 査読 |
| G40 | Is blockchain needed for consent management in data spaces? (IDS) | https://internationaldataspaces.org/is-blockchain-needed-for-consent-management-in-data-spaces/ | 業界団体(2024) |
| G41 | Identifying Barriers to Blockchain-Based Patient-Centric Data Mgmt (PMC) | https://pmc.ncbi.nlm.nih.gov/articles/PMC10855174/ | 査読(2024) |
| G42 | Blockchain adoption in healthcare: Overcoming barriers (ScienceDirect 2025) | https://www.sciencedirect.com/science/article/pii/S0040162525000629 | 査読(2025) |
| G43 | Why are so many Companies Exiting the Healthcare Blockchain Market? | https://medium.com/@Connected_Dots/why-are-so-many-companies-exiting-the-healthcare-blockchain-market-43cddbea4431 | Medium |
| G44 | Blockchain and healthcare: the Estonian experience (e-Estonia) | https://e-estonia.com/blockchain-healthcare-estonian-experience/ | 公式 |
| G45 | MedRec - MIT Media Lab | https://www.media.mit.edu/publications/medrec-blockchain-for-medical-data-access-permission-management-and-trend-analysis/ | 公式 |
| G46 | Immutable Yet Compliant: Harmonizing Blockchain with GDPR | https://emildai.eu/immutable-yet-compliant-harmonizing-blockchain-with-gdpr/ | 技術ブログ |
| G47 | Enabling secure self determined health data sharing (npj Digital Medicine 2025) | https://www.nature.com/articles/s41746-025-01945-z | 査読(Nature) |
| G48 | Gartner Blockchain Hype Cycle (imiblockchain) | https://imiblockchain.com/gartner-blockchain-hype-cycle/ | 業界ブログ |
| G49 | CMS Patient Access API FAQ | https://www.cms.gov/priorities/burden-reduction/overview/interoperability/frequently-asked-questions/patient-access-api | 政府公式 |
| G50 | Understanding the Patient Access API (fire.ly) | https://fire.ly/blog/understanding-the-patient-access-api/ | 技術ブログ |
| G51 | CMS Interoperability and Prior Authorization Final Rule CMS-0057-F | https://www.cms.gov/newsroom/fact-sheets/cms-interoperability-prior-authorization-final-rule-cms-0057-f | 政府公式 |
| G52 | How to Integrate AI Agents with Epic & Cerner (capminds) | https://www.capminds.com/blog/how-to-build-an-ai-agent-that-integrates-with-epic-or-cerner-technical-guide/ | 技術ブログ |
| G53 | SMART App Launch v2.2.0 Overview (HL7) | https://hl7.org/fhir/smart-app-launch/ | 公式(HL7) |
| G54 | SMART App Launch v2.2.0 app-launch.html | https://build.fhir.org/ig/HL7/smart-app-launch/app-launch.html | 公式(HL7) |
| G55 | With FHIR in Place is There Room for Blockchain? (Huron) | https://www.huronconsultinggroup.com/insights/fhir-blockchain-healthcare | コンサル分析 |
| G56 | SMART on FHIR Authorization Best Practices | https://docs.smarthealthit.org/authorization/best-practices/ | 公式(SMART) |
| G57 | Blockchain-Based Dynamic Consent (PMC10877491) | https://pmc.ncbi.nlm.nih.gov/articles/PMC10877491/ | 査読 |
| G58 | Blockchain-enabled EHR access auditing (PMC11381610) | https://pmc.ncbi.nlm.nih.gov/articles/PMC11381610/ | 査読 |
| G59 | Privacy preservation blockchain healthcare (PMC12534302) | https://pmc.ncbi.nlm.nih.gov/articles/PMC12534302/ | 査読 |
| G60 | FHIR and Blockchain Healthcare (oodles.io) | https://blockchain.oodles.io/blog/fhir-and-blockchain-healthcare-data-management/ | 技術ブログ |
| G61 | 電子カルテ情報共有サービス（厚労省公式） | https://www.mhlw.go.jp/stf/seisakunitsuite/bunya/kenkou_iryou/iryou/johoka/denkarukyouyuu.html | 政府公式 |
| G62 | HealthTech API Security mTLS OAuth 2.0 (dev.to/wellallytech) | https://dev.to/wellallytech/healthtech-api-security-protecting-patient-data-with-mtls-and-oauth-20-4k7k | 技術ブログ |
| G63 | FHIR API Security Best Practices (socialroots.ai) | https://www.socialroots.ai/blog/fhir/fhir-api-and-security | 技術ブログ |
| G64 | 電子カルテ情報共有サービス、2026年度冬頃の本格運用目指す(GemMed) | https://gemmed.ghc-j.com/?p=71681 | ニュースメディア |
| G65 | 富士通・エムスリーが高シェア「電子カルテ」市場に異変(Diamond) | https://diamond.jp/articles/-/347214 | 経済メディア |
| G66 | Health Care Worker Usage of Large-Scale HIEs in Japan (PMC) | https://www.ncbi.nlm.nih.gov/pmc/articles/PMC11481819/ | 査読 |
| G67 | X-Road Technology Overview | https://x-road.global/x-road-technology-overview | 公式 |
| G68 | 全国地域医療情報連携ネットワークの概況2023(日医総研WP485) | https://www.jmari.med.or.jp/wp-content/uploads/2024/09/WP485.pdf | 公的研究 |
| G69 | A Call to Reconsider a Nationwide EHR System - NPfIT (PMC) | https://pmc.ncbi.nlm.nih.gov/articles/PMC10958994/ | 査読 |
| G70 | 電子カルテ標準化のゆくえ（LINQUA） | https://linqua.jp/news/electronic-medical-record-standardization/ | 専門メディア |
| G71 | 電子カルテ共有で厚労省がベンダー向け技術解説書を公開(Nikkei xTech) | https://xtech.nikkei.com/atcl/nxt/column/18/00001/08828/ | 技術専門メディア |
| G72 | JP-CLINS FHIR実装ガイド v1.11.0 | https://jpfhir.jp/fhir/clins/igv1/ | 技術標準 |
| G73 | Japan's Sovereign Cloud Strategy (IT Business Today) | https://itbusinesstoday.com/tech/cloud/japans-sovereign-cloud-strategy-balancing-innovation-with-national-security/ | 専門メディア |
| G74 | 医療情報のデジタル化における現状と課題(日医総研RR124) | https://www.jmari.med.or.jp/wp-content/uploads/2022/03/RR124.pdf | 公的研究 |
| G75 | Germany EHR Market Disruption Report (pharmiweb) | https://www.pharmiweb.com/press-release/2025-04-01/germanys-ehr-market-faces-disruption-amid-ai-caution-regulatory-shifts-and-vendor-realignments-re | 市場調査 |
| G76 | Epic controls 42% of US EHR market (TechTarget) | https://www.techtarget.com/searchhealthit/feature/Epic-controls-42-of-the-US-EHR-market-Does-that-help-or-hurt-interoperability | 技術専門メディア |
| G77 | Epic aims to bring all clients live on TEFCA by 2025 (TechTarget) | https://www.techtarget.com/searchhealthit/news/366605772/Epic-aims-to-bring-all-clients-live-on-TEFCA-by-2025 | 技術専門メディア |
| G78 | Epic, Cerner growing EHR market share (Fierce Healthcare) | https://www.fiercehealthcare.com/tech/epic-cerner-growing-ehr-market-share-increased-hospital-consolidation-klas | ヘルスケア専門メディア |
| G79 | Implementing EHR in Germany: Lessons (IJIC) | https://ijic.org/articles/10.5334/ijic.6578 | 査読 |
| G80 | Blockchain for Japanese Healthcare (PMC Viewpoint 2019) | https://pmc.ncbi.nlm.nih.gov/articles/PMC6996742/ | 査読 |
| G81 | EHR interoperability using FHIR and blockchain bibliometric (PMC) | https://pmc.ncbi.nlm.nih.gov/articles/PMC10679575/ | 査読 |
| G82 | Nordic digital health infrastructure (Norden 2025) | https://pub.norden.org/temanord2025-573/2-theme-1-infrastructure-for-digital-health.html | 公的報告 |
| G83 | The rise and fall of England's NPfIT (PMC) | https://pmc.ncbi.nlm.nih.gov/articles/PMC3206716/ | 査読 |
| G84 | 電子カルテのベンダーシェア（SSI公式） | https://www.softs.co.jp/e-map/share.html | ベンダー公式 |
| G85 | Digital Healthcare 2025 - Japan (Chambers) | https://practiceguides.chambers.com/practice-guides/digital-healthcare-2025/japan/trends-and-developments | 専門法律ガイド |

---

**品質**: critic検証 PASS（Coverage/Relevance/核心結論の根拠強度=十分。Citation AccuracyのWARNINGは本レポートで本文インライン引用付与・「54.2%」二次情報注記により対応済み）。
