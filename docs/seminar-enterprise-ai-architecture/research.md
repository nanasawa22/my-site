# セミナー資料 調査メモ / スライド構成案

**セッション名**
「AIが止まれば業務が止まる」時代へ ― 本番業務で使えるエンタープライズAIのアーキテクチャ

**想定視聴者**: IT Pro / Developer
**調査日**: 2026-08-10

---

## 0. この文書の位置づけ

スライドを書き始める前の材料集め。以下を含む。

1. 章立て・スライド構成案（想定40分・約34枚）
2. 各章で使えるファクトと出典
3. ファクトの信頼度ランク（そのまま登壇で言えるか／裏取りが要るか）
4. 未確認事項・要確認リスト

> **調査環境の制約**: 本セッション環境のネットワークポリシーにより `nutanix.com` を含む外部サイトへの直接アクセス（WebFetch）がブロックされている。Nutanix 製品の記述は検索結果のスニペットおよび三次情報（IT メディア）に基づく。**登壇前に公式ドキュメント／リリースノートでの裏取りが必須**（→ 第6節）。

---

## 1. スライド構成案（40分想定 / 約34枚）

時間配分は 40 分セッション（Q&A 別枠 5 分）を前提。30 分の場合は第3章と第6章を圧縮する。

### オープニング（2分 / 2枚）

| # | スライド | 内容 |
|---|---|---|
| 1 | タイトル | セッション名・演者 |
| 2 | 本日の結論（先出し） | 「AI基盤の要件は "止まらない・読める・守れる・逃げられる" の4つ。それを実現する場所はアプリではなく Gateway レイヤ」 |

IT Pro / Developer 向けなので、結論を先に置く。ストーリーテリングより情報密度を優先。

### 第1章 なぜ今「止まらないAI」なのか（6分 / 5枚）

| # | スライド | 内容 |
|---|---|---|
| 3 | 依存の質が変わった | チャット補助（落ちても困らない）→ 業務プロセスの構成要素（落ちると業務が止まる） |
| 4 | 数字で見る現在地 | Gartner: 2026年末までにエンタープライズアプリの40%がタスク特化AIエージェントを搭載（2025年は5%未満）／導入済みは17%、2年以内に60%超が予定 |
| 5 | 実際に止まっている | Azure OpenAI 2026-05-29 の広域レイテンシ・5XX 障害（09:39–17:05 UTC、約7.5時間）、OpenAI 2026-07-21 の長時間障害など |
| 6 | SLAの現実 | OpenAI Scale Tier 99.9% / Anthropic Priority Tier 99.5%目標 / Azure OpenAI 99.9%。**単一プロバイダ 99.9% = 年間約8.7時間の停止許容**。基幹業務のSLOと釣り合うか？ |
| 7 | 「止まる」は落ちるだけではない | ①ダウン ②レート制限（429）③レイテンシ劣化 ④モデルの廃止・バージョン変更 ⑤リージョン単位の障害。**アプリから見ればすべて同じ「使えない」** |

**この章の狙い**: 「可用性は AI 基盤の"あとで考える話"ではなく前提条件」という土台を作る。

### 第2章 本番AI基盤に求められる4条件（5分 / 4枚）

| # | スライド | 内容 |
|---|---|---|
| 8 | 4条件の提示 | **止まらない**（可用性）／**読める**（コスト予測可能性）／**守れる**（データ主権・秘匿）／**逃げられる**（ロックイン回避） |
| 9 | 読める：コストが読めない構造 | エージェントは「推論→行動→観測→再推論」でステップごとにLLM呼び出しが発生。同じ業務でもチャットボット比で数十〜数百倍のトークンを消費しうる。事前見積りが原理的に難しい |
| 10 | 守れる：データ主権 | 改正個人情報保護法・業法（金融/医療）・GDPR等で越境移転に制約。2026年に日本政府もソブリンAI政策を本格化。秘匿データはクラウドLLMに出せない前提の業務が実在する |
| 11 | 逃げられる：ロックインは4層で効く | モデル層／データ層／API層／インフラ層。1つのプロバイダに全部預けると同時に4つ固定される |

**この章の狙い**: 非機能要件として整理し、次章以降の評価軸にする。ここで表を1枚作っておくと第5章の答え合わせに効く。

### 第3章 アプリ側で頑張ると何が起きるか（5分 / 4枚）

Developer に最も刺さるパート。ここで共感を取る。

| # | スライド | 内容 |
|---|---|---|
| 12 | 信頼性設計の定石 | 障害の時間スケールで担当が違う：**リトライ**=一過性のノイズ（瞬断・瞬間的混雑）／**サーキットブレーカー**=劣化したエンドポイント（数十秒〜数分）／**フォールバック**=長期障害（プロバイダ長時間ダウン）。加えて同時実行リミッタ・タイムアウト |
| 13 | タイムアウトは2階層 | ステップ単位（個々のLLM呼び出し上限）とセッション単位（1タスク全体の経過時間上限）。エージェントのループ暴走はセッション単位でしか止まらない |
| 14 | それを全アプリに実装すると | N アプリ × M プロバイダの組合せ爆発／SDK差異の吸収コスト／APIキーの分散管理／トークン計測がアプリごとにバラバラ＝全社の使用量が見えない／ポリシー変更に全アプリ再デプロイ |
| 15 | 結論：横断レイヤに外出しする | アプリは「1つのエンドポイント」だけを知る。信頼性・ガバナンス・可観測性は基盤側の責務 → **AI Gateway** |

**この章の狙い**: 第4章以降を「ベンダー製品紹介」ではなく「アーキテクチャの必然」として受け取ってもらう橋渡し。ここが本セッションの技術的な背骨。

### 第4章 AI Gateway というアーキテクチャ（8分 / 6枚）

| # | スライド | 内容 |
|---|---|---|
| 16 | Gateway の責務定義 | 認証・認可 / ルーティング / フォールバック / レート制限 / トークン計測 / 監査ログ / MCPガバナンス |
| 17 | 位置づけ図 | アプリ・エージェント → Gateway → （外部プロバイダ群 ／ セルフホスト推論） |
| 18 | データプレーンとコントロールプレーンの分離 | ポリシー定義と実際のトラフィック処理を分ける。運用者が触る面と開発者が触る面 |
| 19 | 一般解としての市場 | OSS/SaaS 各種（LiteLLM＝自前運用向け、Portkey＝マネージド、Kong AI Gateway＝既存API管理の延長）。**OpenAI互換API がデファクト**になったことで抽象化が現実的になった |
| 20 | エンタープライズで足りなくなる点 | ①セルフホストモデルと外部モデルを同一の統制下に置けるか ②オンプレ／エアギャップで動くか ③マルチテナント・部門別のコスト按分 ④監査要件 |
| 21 | 評価軸まとめ | 第2章の4条件 × Gateway 機能のマッピング表 |

**この章の狙い**: 中立的な一般論を先に立てる。ここを丁寧にやるほど、次章の製品パートの説得力が上がる。IT Pro/Developer は製品紹介から入ると身構える。

### 第5章 Nutanix Agentic AI と Nutanix Agent Gateway（10分 / 8枚）

| # | スライド | 内容 |
|---|---|---|
| 22 | Nutanix Agentic AI 全体像 | インフラ（Nutanix Cloud Platform）+ Kubernetes（NKP）+ ストレージ（NUS）+ **Nutanix Enterprise AI（NAI）= 中央AIコントロールプレーン**。NAI は Agent Gateway と Inference Management で構成 |
| 23 | Agent Gateway とは | エージェント / LLM / 業務ツール間のやり取りを束ねる「中央の玄関口（front door）」。エージェント活動の統制・アクセスポリシー・トークン消費監視を単一のコントロールポイントで |
| 24 | 機能① 単一APIでマルチプロバイダ | 外部プロバイダのモデルとセルフホストモデルを単一APIで。ユースケースごとに最適なモデルを選べる＝ロックイン回避 |
| 25 | 機能② 自動フォールバック | プライマリがダウン／レート制限に当たると、設定済みバックアップへ自動フェイルオーバー。**アプリのコード変更なし** |
| 26 | 機能③ トークン可観測性とコスト統制 | モデルベンダー横断の集中トークン可観測性。使用量トラッキング・コスト按分・トークンベースのレート制限。**「見える → 減らせる／移せる」** |
| 27 | 機能④ MCPガバナンス | MCPサーバーへの粒度の細かいアクセス制御。エージェントが業務ツール・プライベートデータに安全に接続。（※提供ステータス要確認：Tech Preview との情報あり） |
| 28 | 機能⑤ 監査ログ | リクエスト単位の監査ログ。誰の・どのエージェントが・どのモデルに・何トークン |
| 29 | 動く場所 | オンプレ／エッジ／NEO Cloud／OEMパートナー基盤／パブリッククラウド。ソブリン対応 |

**押さえどころ**: NAI 2.7 で Agent Gateway が GA。NAI 2.6 時点で「AI Gateway」として統一推論エンドポイント・認証・可観測性・トークンベースのレート制限が入り、2.7 で Agent Gateway に発展という流れ。

### 第6章 実践：どう組み、どう効くか（5分 / 4枚）

| # | スライド | 内容 |
|---|---|---|
| 30 | リファレンス構成図 | 業務アプリ／エージェント → Agent Gateway → ①外部LLM ②NAI上のセルフホストモデル ③MCPサーバー群 |
| 31 | 障害時シーケンス | 平常時 → プライマリ障害検知 → バックアップへ切替 → アプリから見た挙動（正常応答継続）をシーケンス図で |
| 32 | コスト最適化の流れ | 可視化 → 部門/エージェント別に按分 → 大量・定型のワークロードを特定 → セルフホストへシフト。判断材料として「自前ホストの損益分岐はトークン量とGPU稼働率で決まる／GPU費用は総コストの3〜4割で残りは運用・人件費」を添える |
| 33 | 段階的導入ロードマップ | Step1 全AIトラフィックをGateway経由に集約（可視化のみ）→ Step2 レート制限・ポリシー適用 → Step3 フォールバック構成 → Step4 セルフホストモデル併用 → Step5 MCPガバナンス |

**この章の狙い**: 「明日から何をするか」を持ち帰らせる。Step1 が可視化だけ、という設計が現実的で刺さる。

### クロージング（2分 / 2枚）

| # | スライド | 内容 |
|---|---|---|
| 34 | まとめ | 4条件の再掲＋「アプリを書き換えずに、基盤側で満たせる」 |
| 35 | Next Action / 参考リンク | 評価版・ドキュメント・問い合わせ導線 |

---

## 2. デモ候補（入れる場合は第5章の後に5〜7分）

優先度順。

1. **フォールバックのライブデモ** — プライマリプロバイダを意図的に落とす（またはキーを無効化）→ 同じ curl / 同じアプリが応答し続けることを見せる。**このセッションで最も効くデモ**。タイトルの主張をそのまま証明できる
2. **単一APIでのモデル切替** — `model` パラメータだけを変えて外部モデル → セルフホストモデルへ。base_url もコードも変わらないことを見せる
3. **トークンダッシュボード** — エージェントを1本走らせて、消費トークンがリアルタイムに積み上がる様子。「エージェントは数十〜数百倍」を体感させる
4. **MCPアクセス制御** — 許可されていないMCPサーバーへの接続が弾かれる

デモが用意できない場合は、1と3をスクリーンショット＋シーケンス図で代替。

---

## 3. 使えるファクト一覧（出典・信頼度つき）

信頼度: **A** = 一次情報／権威ある調査。そのまま引用可。**B** = 報道・ベンダーブログ。出典明記の上で引用可。**C** = SEOブログ等の二次情報。**登壇では数字を出さず定性的な表現に留めるのが安全**。

### 市場・採用動向

| ファクト | 信頼度 | 出典 |
|---|---|---|
| 2026年末までにエンタープライズアプリの40%がタスク特化AIエージェントを搭載（2025年は5%未満） | A | [Gartner プレスリリース (2025-08-26)](https://www.gartner.com/en/newsroom/press-releases/2025-08-26-gartner-predicts-40-percent-of-enterprise-apps-will-feature-task-specific-ai-agents-by-2026-up-from-less-than-5-percent-in-2025) |
| AIエージェントを導入済みの組織は17%、60%超が2年以内に導入予定（2026 Gartner CIO調査） | A | [Gartner Hype Cycle for Agentic AI](https://www.gartner.com/en/articles/hype-cycle-for-agentic-ai) |
| 2029年までに70%の企業がITインフラ運用の一部としてエージェンティックAIを導入（2025年は5%未満） | A | [Gartner](https://www.itential.com/resource/analyst-report/gartner-predicts-2026-ai-agents-will-reshape-infrastructure-operations/) |
| エージェンティックAIプロジェクトの40%が2027年末までに中止される（Gartner予測） | A | [Gartner](https://www.gartner.com/en/articles/hype-cycle-for-agentic-ai) |
| 少なくとも1つのAIエージェントを本番稼働させている企業は31%、銀行・保険は約47% | B | S&P Global Market Intelligence / McKinsey（[経由](https://www.beri.net/article/ai-agent-adoption-enterprise-2026-gartner-idc)） |

> **使い分け**: 「40%が中止される」は本セッションの文脈で強い。**「PoCが失敗するのではなく、本番運用の設計で失敗する」**という論の補強に使える。

### 障害・可用性

| ファクト | 信頼度 | 出典 |
|---|---|---|
| Azure OpenAI 2026-05-29、09:39–17:05 UTC（約7.5時間）にレイテンシ増大・断続的リクエスト失敗・タイムアウト・5XX。欧州とAustralia Eastで顕著。上流変更が内部リトライトラフィックの急増を招いたことが引き金 | A | [Azure status history](https://azure.status.microsoft/en-us/status/history/) |
| Azure 2025-09-26、証明書デプロイ不良により Switzerland North で Azure OpenAI を含む複数サービスが停止 | A | [Azure status history](https://azure.status.microsoft/en-us/status/history/) |
| OpenAI 2026-07-21 に795分間の障害記録 | C | [downforai.com](https://downforai.com/openai) — 第三者計測。登壇では「Azureの一次情報」を主に使い、こちらは補足に留める |
| OpenAI Scale Tier は 99.9% SLA（全体標準ではなくオプションtier）／Anthropic Priority Tier は 99.5% アップタイム目標 | B | [OpenAI Help Center](https://help.openai.com/en/articles/5008641-is-there-an-sla-for-latency-guarantees-on-the-various-engines) ほか |
| Azure OpenAI の SLA は 99.9% | B | [Redress Compliance](https://redresscompliance.com/azure-openai-sla-and-support) |

> **登壇での使い方**: 「99.9% = 年間8.76時間」は自分で計算した値なので安全に出せる。ここに「実際の障害は1回で7.5時間（Azure 2026-05-29の一次情報）」を重ねると、**単一プロバイダ構成のリスクが計算ではなく実測で示せる**。

### コスト

| ファクト | 信頼度 | 出典 |
|---|---|---|
| エージェンティックなワークフローは、同じ業務をチャットボットで行う場合に比べ 50〜500倍のトークンを消費しうる | C | [The Deployment Layer](https://www.thedeploymentlayer.com/p/the-token-economy-why-llm-cost-architecture-will-define-enterprise-ai-winners) |
| エンタープライズ agentic AI プロジェクト127件のレビューで73%が予算超過、一部は2.4倍、影響を受けたプロジェクトあたり約230万ドルの計画外コスト | C | [AI CERTs News](https://www.aicerts.ai/news/enterprise-token-costs-spiral-why-ai-budgets-are-under-siege/) |
| エンタープライズのトークン予算の40〜60%が無駄 | C | 同上 |
| 正式なAIガバナンスポリシーを持つ組織は43%、エージェンティックガバナンスが成熟しているのは21% | C | 同上 |
| セルフホストの損益分岐はおよそ日次200万〜500万トークン（12ヶ月のリザーブドGPU前提）。GPU実費は真のインフラ投資の30〜40%に過ぎず、残りはエンジニアリング | C | [Spheron](https://www.spheron.network/blog/ai-inference-cost-economics-2026/) / [Developers Digest](https://www.developersdigest.tech/blog/self-hosting-open-weights-models-break-even-math) |

> **注意**: このカテゴリは信頼度Cが多い。**具体的な数値を断定的に出すのは避け**、「エージェントは呼び出し回数が動的に決まるので、原理的に事前見積りが難しい」という**構造の説明**に寄せるほうが安全かつ説得力がある。数字を出す場合は必ず出典をスライドに明記する。

### データ主権・規制

| ファクト | 信頼度 | 出典 |
|---|---|---|
| 2026年に日本政府がソブリンAI政策を本格始動 | C | [navi-dx.com](https://navi-dx.com/what-is-sovereign-ai-japan-data-sovereignty) — **要一次情報での裏取り** |
| GDPR・改正個人情報保護法・医療/金融の業法など、データ越境移転に制約がかかる領域ではソブリンAIが事実上の前提条件 | C | 同上 |
| 医療・金融向けにエアギャップ構成のオンプレミス生成AI製品が国内で商用提供されている（例: FIXER「Sovereign GaiXer」2026-04正式受注開始） | B | [PR TIMES](https://prtimes.jp/main/html/rd/p/000000138.000009536.html) |

### MCP・ガバナンス

| ファクト | 信頼度 | 出典 |
|---|---|---|
| MCPの主要リスクはプロンプトインジェクション／コンテキスト操作によるツールの不正利用・データ持ち出し。攻撃ベクタは confused deputy / token passthrough / tool poisoning / SSRF via tool connectors / rogue server registration | A | [Cloud Security Alliance](https://labs.cloudsecurityalliance.org/agentic/agentic-mcp-security-best-practices-v1/) / [CoSAI](https://www.coalitionforsecureai.org/wp-content/uploads/2026/03/model-context-protocol-security-1.pdf) |
| 2026年1〜2月にMCPサーバー／クライアント／基盤コンポーネントを狙う30件超のCVEが登録 | B | [CSA Research Note](https://labs.cloudsecurityalliance.org/research/csa-research-note-mcp-security-crisis-20260504-csa-styled/) |
| MCPはガバナンスについて中立であり、認証・認可・ログのないMCPサーバーは「サービスアカウントの権限で何でもできるAI」を生む | A | [Wiz](https://www.wiz.io/academy/ai-security/model-context-protocol-security) / [arXiv 2511.20920](https://arxiv.org/html/2511.20920v1) |
| ベストプラクティス: 承認済みツールのみ公開、最小権限、機微な操作にはポリシー統制、監査可能性の確保。すべてのMCPサーバーを信頼できない第三者として扱う | A | [CSA](https://labs.cloudsecurityalliance.org/agentic/agentic-mcp-security-best-practices-v1/) |

> **使い方**: 第5章の機能④（MCPガバナンス）の直前に1枚挟むと、機能の必要性が一気に立つ。「MCPは便利だが、プロトコル自体はガバナンスを規定していない。だから統制レイヤが要る」という論法。

### 信頼性設計パターン

| ファクト | 信頼度 | 出典 |
|---|---|---|
| LLM呼び出しの信頼性は5層で守る：同時実行リミッタ／サーキットブレーカー／タイムアウト付き呼び出し／リトライ／フォールバック | B | [Qiita](https://qiita.com/akira_papa_AI/items/d3303248aae3d87e0a6a) |
| 障害の時間スケールで担当が異なる：リトライ=一過性のノイズ、サーキットブレーカー=数十秒〜数分の劣化、フォールバック=長時間のプロバイダ障害 | B | 同上 |
| タイムアウトはステップ単位とセッション単位の2階層で設ける | B | 同上 |

> この整理は**第3章のスライド12の骨格そのもの**。出典はコミュニティ記事だが、内容は分散システム設計の標準的な定石であり、登壇者自身の言葉として語って問題ない。

### Nutanix 製品

| ファクト | 信頼度 | 出典 |
|---|---|---|
| Nutanix Agent Gateway は Nutanix Enterprise AI 2.7 の一部として GA | B | [Nutanix Blog](https://www.nutanix.com/blog/introducing-nutanix-agent-gateway) / [ITdaily](https://itdaily.com/news/software/nutanix-launches-agent-gateway/) |
| エージェント／LLM／業務ツール間のやり取りを管理する「中央の front door」。エージェント活動の統制・アクセスポリシー管理・トークン消費監視の単一コントロールポイント | B | [iTWire](https://itwire.com/business-it-news/data/nutanix-strengthens-agentic-ai-governance-and-cost-control-with-agent-gateway) / [IT Brief AU](https://itbrief.com.au/story/nutanix-launches-agent-gateway-to-govern-ai-agents) |
| プライマリプロバイダのダウン／レート制限時に、設定済みバックアップへ自動フェイルオーバー。**アプリケーションコードの書き換え不要** | B | 検索スニペット（NAI 2.7 関連） — **要一次情報での裏取り** |
| モデルベンダー横断の集中トークン可観測性。使用量トラッキング・コスト按分・過剰なトークン消費の抑制 | B | [iTWire](https://itwire.com/business-it-news/data/nutanix-strengthens-agentic-ai-governance-and-cost-control-with-agent-gateway) |
| 外部プロバイダのモデルとセルフホストモデルに単一APIでアクセス可能 | B | [Nutanix Blog](https://www.nutanix.com/blog/introducing-nutanix-agent-gateway) |
| MCPサーバーへの粒度の細かいアクセス制御（Governance for MCP）。GitHub / Stripe 等のツールへの安全な接続。**Tech Preview との記載あり** | B | 検索スニペット — **提供ステータス要確認** |
| リクエスト単位の監査ログ | B | [Security Storage und Channel Germany](https://security-storage-und-channel-germany.de/language/en/nutanix-launches-agent-gateway-to-tame-ai-costs-and-governance/) |
| 可視化により、セルフホストモデルへ移せるワークロードを特定でき、外部サービス依存とコストを下げられる | B | [SourceSecurity](https://www.sourcesecurity.com/news/nutanix-agent-gateway-centralised-ai-governance-co-1568897613-ga.1783359990.html) |
| NAI 2.6 時点で AI Gateway として、クラウドホスト型/プライベートLLMへの統一セキュア推論エンドポイント、認証、可観測性、トークンベースのレート制限を提供。MCPサーバーとFine Tuningのサポートを追加 | B | [Nutanix Blog (NAI 2.6)](https://www.nutanix.com/blog/orchestrating-the-hybrid-ai-frontier) |
| Nutanix Agentic AI は NAI を中央AIコントロールプレーン（Agent Gateway + Inference Management）とし、インフラ・Kubernetes・データを統合。オンプレ／NEO Cloud／OEMパートナー基盤にまたがるソブリン対応 | B | [Nutanix プレスリリース](https://www.nutanix.com/press-releases/2026/nutanix-unveils-nutanix-agentic-ai) |
| NAI は CNCF準拠のKubernetes（NKP含む）上に展開可能。NVIDIA NIM / NeMo マイクロサービスと統合 | B | [Nutanix Blog](https://www.nutanix.com/blog/nutanix-enterprise-ai-makes-agents) |
| 2026年後半に、マルチテナンシー基盤 Nutanix Service Provider Central と、AIエンジニア向けのセルフサービス型マルチテナントAI管理ポータルを追加予定（neocloud向け） | B | [Nutanix プレスリリース (2026-04-07)](https://www.nutanix.com/press-releases/2026/nutanix-to-extend-nutanix-agentic-ai-empowering-neoclouds-to-deliver-higher-value-ai-services) / [Blocks & Files](https://www.blocksandfiles.com/hci/2026/04/07/nutanix-pushes-agentic-ai-bare-metal-kubernetes-at-next-2026/5214682) |

---

## 4. ⚠️ 混同注意：Nutanix Agent Gateway ≠ agentgateway (Solo.io)

調査中に判明した重要な注意点。**名前が非常に似た別製品が存在する**。

| | Nutanix Agent Gateway | agentgateway（Solo.io / agentgateway.dev） |
|---|---|---|
| 提供元 | Nutanix | Solo.io（OSS、CNCF系エコシステム） |
| 位置づけ | Nutanix Enterprise AI 2.7 の構成要素 | Kubernetes ネイティブの OSS エージェント接続基盤 |
| フェイルオーバー | 「プライマリ障害時に設定済みバックアップへ自動フェイルオーバー」 | `AgentgatewayBackend` の priority group による優先度ベースのフェイルオーバー、outlier detection によるアンヘルシーなバックエンドの排除 |

検索結果では両者の記述が混在して返ってくる。**`AgentgatewayBackend`、priority group、outlier detection といった具体的な実装用語は Solo.io 側のものであり、Nutanix の機能としてスライドに書いてはいけない。** Nutanix 側のフェイルオーバー実装の詳細（検知条件、切替単位、リトライ回数、ヘルスチェック方式など）は公式ドキュメントで確認すること。

---

## 5. 想定Q&A

IT Pro / Developer からは実装の詳細が飛んでくる。準備しておく。

**可用性まわり**
- フェイルオーバーの検知条件は？（HTTPステータス／タイムアウト／エラーレート閾値）切替の判定単位は？
- フェイルオーバー中のレイテンシへの影響は？リトライ分の遅延はどれくらい積まれる？
- バックアップ先のモデルが別ベンダーの場合、出力品質・フォーマットの差はどう吸収する？（**これは踏み込まれやすい。「可用性は担保できるが出力の等価性は担保しない」という正直な線引きを準備**）
- Gateway 自体が SPOF にならないか？ Gateway の冗長構成は？
- ストリーミング応答の途中で障害が起きた場合の挙動は？

**コスト・運用**
- トークン計測はどの粒度で取れる？（ユーザー／エージェント／アプリ／部門）
- コスト按分のためのタグ付け・メタデータはどう渡す？
- レート制限に当たったリクエストの扱いは？（429返す／キューイング／降格モデルへ回す）

**互換性・移行**
- 既存アプリの改修範囲は？ OpenAI互換なら base_url の差し替えだけ？
- 対応プロバイダの一覧は？ 新しいモデルが出たときの追従は？
- LiteLLM や Portkey を既に使っている場合の移行パスは？

**セキュリティ**
- MCPガバナンスのアクセス制御の粒度は？（サーバー単位／ツール単位／引数レベル）
- 監査ログにプロンプト本文は含まれる？ 含まれる場合、秘匿データの扱いは？
- エアギャップ環境で動作するか？

---

## 6. 登壇前の要確認リスト

本調査は外部サイトへの直接アクセスが制限された環境で行っており、Nutanix 製品の記述は報道・検索スニペット由来。**以下は公式ドキュメント／リリースノート／自社の製品担当で裏取りしてからスライドに確定させること。**

- [ ] Agent Gateway のフェイルオーバー実装の詳細（検知条件・切替単位・設定方法）
- [ ] MCPガバナンスの提供ステータス（GA か Tech Preview か）と、アクセス制御の粒度
- [ ] トークン可観測性のメトリクス粒度と、ダッシュボードの実際の画面
- [ ] 対応する外部プロバイダの一覧
- [ ] NAI 2.7 のリリースノート全体（本資料に載せていない機能があるはず）
- [ ] Agent Gateway 自体の冗長構成・HA 構成の推奨
- [ ] 日本国内での提供状況・サポート体制
- [ ] 「2026年に日本政府がソブリンAI政策を本格始動」の一次情報（政府発表の特定）
- [ ] コスト系の統計（信頼度C）を使うかどうかの判断。使うなら出典明記、使わないなら定性表現に置換

---

## 7. 構成上の設計判断メモ

なぜこの構成にしたか。差し替え検討時の参考に。

1. **製品の登場を後半（第5章）に置いた** — IT Pro / Developer は製品紹介から入ると身構える。第3章で「アプリ側実装の限界」を体験的に共感させ、第4章で「Gateway という一般解」を中立に提示してから製品に入ると、製品が「答え合わせ」として機能する。

2. **第4章で競合OSS/SaaSに触れる** — あえて LiteLLM / Portkey / Kong に言及することで、中立性と技術的信頼を獲得する。そのうえで「エンタープライズで足りなくなる点」（セルフホスト統合・オンプレ・マルチテナント・監査）を提示すれば、差別化が自然に立つ。触れずに進めると「知らないのか、隠しているのか」と受け取られる。

3. **アブストラクトの3つの訴求（フォールバック／トークン可視化／単一API）を第5章の機能①②③に正確に対応させた** — 集客時の期待と中身がずれないようにする。

4. **第6章に段階的導入ロードマップを置いた** — Step1 を「可視化だけ」にすることで、導入のハードルを下げる。IT Pro が上司に説明するときの筋書きをそのまま渡す。

5. **信頼度Cのコスト統計に依存しない構成にした** — 「エージェントは呼び出し回数が動的に決まるので原理的に見積りが難しい」という**構造の説明**を主軸にすれば、数字が裏取りできなくても論旨は崩れない。

---

## 8. 出典一覧

- [Gartner: 40% of enterprise apps will feature task-specific AI agents by 2026](https://www.gartner.com/en/newsroom/press-releases/2025-08-26-gartner-predicts-40-percent-of-enterprise-apps-will-feature-task-specific-ai-agents-by-2026-up-from-less-than-5-percent-in-2025)
- [Gartner: 2026 Hype Cycle for Agentic AI](https://www.gartner.com/en/articles/hype-cycle-for-agentic-ai)
- [Gartner Predicts 2026: AI Agents Will Reshape Infrastructure & Ops](https://www.itential.com/resource/analyst-report/gartner-predicts-2026-ai-agents-will-reshape-infrastructure-operations/)
- [Azure status history](https://azure.status.microsoft/en-us/status/history/)
- [OpenAI Help Center: SLA for latency guarantees](https://help.openai.com/en/articles/5008641-is-there-an-sla-for-latency-guarantees-on-the-various-engines)
- [Azure OpenAI SLA 2026: 99.9% Uptime, What It Misses](https://redresscompliance.com/azure-openai-sla-and-support)
- [Nutanix Blog: Introducing Nutanix Agent Gateway](https://www.nutanix.com/blog/introducing-nutanix-agent-gateway)
- [Nutanix Press Release: Nutanix Unveils Nutanix Agentic AI](https://www.nutanix.com/press-releases/2026/nutanix-unveils-nutanix-agentic-ai)
- [Nutanix Press Release (JA): Nutanix Agentic AI 発表](https://www.nutanix.com/ja/press-releases/2026/nutanix-announces-nutanix-agentic-ai)
- [Nutanix Press Release: Extending Agentic AI for Neoclouds](https://www.nutanix.com/press-releases/2026/nutanix-to-extend-nutanix-agentic-ai-empowering-neoclouds-to-deliver-higher-value-ai-services)
- [Nutanix Blog: Nutanix Enterprise AI 2.6 — Orchestrating the Hybrid AI Frontier](https://www.nutanix.com/blog/orchestrating-the-hybrid-ai-frontier)
- [Nutanix Blog: Nutanix Enterprise AI with NVIDIA](https://www.nutanix.com/blog/nutanix-enterprise-ai-makes-agents)
- [Nutanix Community: NAI 2.7 is Now Available](https://next.nutanix.com/nutanix-cloud-platform-for-ai-180/nutanix-enterprise-ai-nai-2-7-is-now-available-45620)
- [iTWire: Nutanix Strengthens Agentic AI Governance and Cost Control with Agent Gateway](https://itwire.com/business-it-news/data/nutanix-strengthens-agentic-ai-governance-and-cost-control-with-agent-gateway)
- [IT Brief AU: Nutanix launches Agent Gateway to govern AI agents](https://itbrief.com.au/story/nutanix-launches-agent-gateway-to-govern-ai-agents)
- [ITdaily: Nutanix launches Agent Gateway](https://itdaily.com/news/software/nutanix-launches-agent-gateway/)
- [Blocks & Files: Nutanix pushes agentic AI, bare-metal Kubernetes at .NEXT 2026](https://www.blocksandfiles.com/hci/2026/04/07/nutanix-pushes-agentic-ai-bare-metal-kubernetes-at-next-2026/5214682)
- [SiliconANGLE: Nutanix expands agentic AI infrastructure for neoclouds](https://siliconangle.com/2026/04/10/nutanix-expands-agentic-ai-infrastructure-power-neoclouds-nutanixnext/)
- [agentgateway.dev — Model failover（※Solo.io の別製品）](https://agentgateway.dev/docs/kubernetes/main/llm/failover/)
- [Cloud Security Alliance: Agentic MCP Security Best Practices](https://labs.cloudsecurityalliance.org/agentic/agentic-mcp-security-best-practices-v1/)
- [CSA Research Note: MCP Security Crisis](https://labs.cloudsecurityalliance.org/research/csa-research-note-mcp-security-crisis-20260504-csa-styled/)
- [Wiz: Understanding Model Context Protocol Security in 2026](https://www.wiz.io/academy/ai-security/model-context-protocol-security)
- [CoSAI: Model Context Protocol Security (PDF)](https://www.coalitionforsecureai.org/wp-content/uploads/2026/03/model-context-protocol-security-1.pdf)
- [arXiv 2511.20920: Securing the Model Context Protocol](https://arxiv.org/html/2511.20920v1)
- [Qiita: 本番でAIが「たまに落ちる」を許す設計](https://qiita.com/akira_papa_AI/items/d3303248aae3d87e0a6a)
- [TrueFoundry: A Definitive Guide to AI Gateways in 2026](https://www.truefoundry.com/blog/a-definitive-guide-to-ai-gateways-in-2026-competitive-landscape-comparison)
- [TrueFoundry: AI model gateways vendor lock-in prevention](https://www.truefoundry.com/blog/vendor-lock-in-prevention)
- [AI CERTs News: Enterprise Token Costs Spiral](https://www.aicerts.ai/news/enterprise-token-costs-spiral-why-ai-budgets-are-under-siege/)
- [Spheron: AI Inference Cost Economics in 2026](https://www.spheron.network/blog/ai-inference-cost-economics-2026/)
- [Developers Digest: Self-Hosting Open-Weights Models — Break-Even Math](https://www.developersdigest.tech/blog/self-hosting-open-weights-models-break-even-math)
- [PR TIMES: FIXER「Sovereign GaiXer」正式受注開始](https://prtimes.jp/main/html/rd/p/000000138.000009536.html)
- [navi-dx: ソブリンAIとは？日本企業のデータ主権戦略](https://navi-dx.com/what-is-sovereign-ai-japan-data-sovereignty)
