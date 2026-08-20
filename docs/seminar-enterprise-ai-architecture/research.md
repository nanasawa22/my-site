# セミナー資料 調査メモ / スライド構成案

**セッション名**
「AIが止まれば業務が止まる」時代へ ― 本番業務で使えるエンタープライズAIのアーキテクチャ

**想定視聴者**: IT Pro / Developer
**調査日**: 2026-08-10

---

## 0. この文書の位置づけ

スライドを書き始める前の材料集め。以下を含む。

1. 章立て・スライド構成案（想定40分・38枚）
2. 各章で使えるファクトと出典
3. ファクトの信頼度ランク（そのまま登壇で言えるか／裏取りが要るか）
4. 未確認事項・要確認リスト

> **調査環境の制約**: 本セッション環境の組織egressポリシーにより、`nutanix.com` を含む外部サイトへの直接アクセスがブロックされている（環境再起動後も同様。`example.com` も同様に遮断される全面的な許可リスト方式で、Nutanix 固有の遮断ではない）。したがって Nutanix 製品の記述は Web 検索の結果および三次情報（IT メディア）に基づく。**登壇前に公式ドキュメント／リリースノートでの裏取りが必須**（→ 第6節）。
>
> 2026-08-10 の追加調査で、要確認項目のうち「対応プロバイダ一覧」「MCPガバナンスの提供ステータス」「デプロイ先」「推論エンジン」は解決済み。残る未確認項目は第6節を参照。

---

## 1. スライド構成案（40分想定 / 38枚）

時間配分は 40 分セッション（Q&A 別枠 5 分）を前提。

**本構成の骨格**: 第2章で定義する4つの非機能要件が、第5章の NAI 2.7 の各機能と1対1で対応する。セマンティックルーティングは、それらを横断して最適化する次の一手としてロードマップ章に置く。

| 第2章で定義する要件 | 第5章で対応する NAI 2.7 の機能 |
|---|---|
| **移植性** | 統合エンドポイント — 単一API・マルチプロバイダ |
| **可用性** | 統合エンドポイント — ロードバランス／フォールバック |
| **コスト予測可能性** | トークンレートリミット（＝トークンのサーキットブレーカ） |
| **データ主権** | MCP接続制御 |
| （4要件を横断する最適化） | **セマンティックルーティング【ロードマップ】** |

この対応表はスライド2（結論の先出し）とスライド30（答え合わせ）で2回提示する。冒頭で地図を渡し、第5章の末尾で回収する構造。

### 用語統一ルール

スライド本文では、平易な言い換え（「止まらない」「読める」等）を用いず、以下の標準的な非機能要件の語彙に統一する。4語をすべて「〜性」で揃えることで、要件一覧としての体裁が保たれる。

| 要件 | 英語 | 定義（スライドに1行で明示する） |
|---|---|---|
| **可用性** | Availability | 個別プロバイダの障害時にも推論要求の処理を継続できること |
| **コスト予測可能性** | Cost Predictability | トークン消費量を計測・按分・制限し、支出を統制可能な状態に置けること |
| **データ主権** | Data Sovereignty | 機密データの所在と準拠法域を組織が統制できること |
| **移植性** | Portability | 特定プロバイダに依存せず、モデル・基盤を代替可能に保てること |

**言い換え対応表**（旧稿・口頭説明との対応）

| 避ける表現 | 用いる表現 |
|---|---|
| 止まらない | 可用性の確保 / サービス継続性 |
| 読める・見える | コスト予測可能性 / 可観測性の確保 |
| 守れる | データ主権の確保 / 機密性の担保 |
| 逃げられる | 移植性の確保 / ベンダー中立性 |
| 〜が刺さる・効く | 〜が有効である |
| アプリを書き換えずに | アプリケーションを改修せず |

> **register を変えたい場合の代替案**: より平易寄りに戻すなら「可用性／経済性／機密性／移植性」（4語すべて2文字で最も簡潔）、より厳密に寄せるなら「可用性／コスト統制性／データ主権性／相互運用性」。ただし「コスト予測可能性」は本セッションの論旨（消費量が実行時に決定されるため事前見積りが困難）を最も正確に表すため、置換は非推奨。

### オープニング（2分 / 2枚）

| # | スライド | 内容 |
|---|---|---|
| 1 | タイトル | セッション名・演者 |
| 2 | 本日の結論（先出し） | 「本番AI基盤の非機能要件は **可用性・コスト予測可能性・データ主権・移植性** の4つ。これらを充足すべき層はアプリケーションではなく Gateway レイヤである」＋**上記の要件×機能マッピング表を提示**して本日の地図とする |

IT Pro / Developer 向けなので、結論を先に置く。ストーリーテリングより情報密度を優先。

### 第1章 AI基盤における可用性要件の再定義（4分 / 4枚）

| # | スライド | 内容 |
|---|---|---|
| 3 | 依存構造の変化 | 補助的ツール（停止しても業務は継続可能）→ 業務プロセスの構成要素（停止が業務停止に直結）。Gartner: 2026年末までにエンタープライズアプリの40%がタスク特化AIエージェントを搭載（2025年は5%未満） |
| 4 | 実際の障害事例 | Azure OpenAI 2026-05-29 の広域レイテンシ・5XX 障害（09:39–17:05 UTC、約7.5時間）。**上流変更が内部リトライトラフィックの急増を招いたことが引き金** ← 第3章・第5章の伏線として重要 |
| 5 | 提供されるSLAの水準 | OpenAI Scale Tier 99.9% / Anthropic Priority Tier 99.5%目標 / Azure OpenAI 99.9%。**単一プロバイダ 99.9% = 年間約8.7時間の停止許容**。基幹業務のSLOと整合するか |
| 6 | 可用性の毀損はサービス停止に限らない | ①サービス停止 ②レート制限（429）③レイテンシ劣化 ④モデルの提供終了・バージョン変更 ⑤リージョン単位の障害。**アプリケーションから見ればいずれも「要求を処理できない状態」として等価** |

**この章の狙い**: 可用性を後付けの検討事項ではなく設計前提として位置づける。スライド4の「リトライ増幅が障害を拡大させた」という事実は、第3章のリトライ設計、第5章のトークンサーキットブレーカの両方に効く伏線なので必ず触れる。

### 第2章 本番AI基盤に求められる非機能要件（4分 / 4枚）

| # | スライド | 内容 |
|---|---|---|
| 7 | 4つの非機能要件 | **可用性**（Availability）／**コスト予測可能性**（Cost Predictability）／**データ主権**（Data Sovereignty）／**移植性**（Portability）。定義を1行ずつ明示する |
| 8 | コスト予測可能性 | エージェントは「推論→行動→観測→再推論」でステップごとにLLM呼び出しが発生。同じ業務でもチャットボット比で数十〜数百倍のトークンを消費しうる。**消費量が実行時に決定されるため、事前見積りが原理的に困難** |
| 9 | データ主権 | 改正個人情報保護法・業法（金融/医療）・GDPR等で越境移転に制約。2026年に日本政府もソブリンAI政策を本格化。機密データを外部LLMへ送出できない前提の業務が実在する。**エージェントは業務ツール・内部DBにも接続するため、統制対象はモデルだけではない** ← MCP接続制御への伏線 |
| 10 | 移植性 | ロックインは**モデル層／データ層／API層／インフラ層**の4層で同時に発生する。単一プロバイダへの集約はこの4層を一括で固定することを意味する |

**この章の狙い**: 非機能要件として整理し、第5章の評価軸を確定させる。スライド9で「統制対象はモデルに限らない」と述べておくと、MCP接続制御が唐突に見えなくなる。

### 第3章 アプリケーション層で実装した場合の限界（5分 / 5枚）

Developer 層の実感に最も接続する章。**第5章の山場（トークンサーキットブレーカ）の伏線を張る章でもある。**

| # | スライド | 内容 |
|---|---|---|
| 11 | 信頼性設計の定石 | 障害の継続時間に応じて担当機構が異なる：**リトライ**=一過性のノイズ（瞬断・瞬間的混雑）／**サーキットブレーカー**=劣化したエンドポイント（数十秒〜数分）／**フォールバック**=長期障害。加えて同時実行リミッタ・タイムアウト |
| 12 | **サーキットブレーカーは「障害」にしか反応しない** | 従来のCBのトリガーはエラー率・レイテンシ。**エージェントの暴走は 200 OK を返し続けながら発生する**——ループ、過剰なリトライ、無駄な再推論。正常応答であるためCBは作動せず、コストだけが増大する ← **第5章スライド27への伏線。この1枚が本セッションの構造上の要** |
| 13 | タイムアウトは2階層 | ステップ単位（個々のLLM呼び出し上限）とセッション単位（1タスク全体の経過時間上限）。エージェントのループはセッション単位でしか停止できない |
| 14 | 全アプリケーションへの個別実装がもたらす問題 | N アプリ × M プロバイダの組合せ爆発／SDK差異の吸収コスト／APIキーの分散管理／トークン計測がアプリごとに分断され全社使用量が不可視／ポリシー変更に全アプリの再デプロイが必要 |
| 15 | 結論：横断レイヤへの責務分離 | アプリケーションが知るべきは単一のエンドポイントのみ。信頼性・ガバナンス・可観測性は基盤層の責務 → **AI Gateway** |

**この章の狙い**: 第4章以降を「ベンダー製品紹介」ではなく「アーキテクチャの必然」として提示する橋渡し。**スライド12を必ず入れること**——ここで「コストに対する遮断器が存在しない」という欠落を可視化しておくと、第5章のトークンレートリミットが「新機能の紹介」ではなく「欠落の充足」として受け取られる。

### 第4章 AI Gateway というアーキテクチャ（4分 / 5枚）

| # | スライド | 内容 |
|---|---|---|
| 16 | **API Gateway との構造的相似** | **10年前の API Gateway と同一の経路**。生の接続性（REST API）は、ガバナンス層を介在させて初めてエンタープライズの本番利用に耐えた。エージェントとモデルの間で同じ構造が再現されている |
| 17 | Gateway の責務定義 | 認証・認可 / **ルーティング・ロードバランス** / フォールバック / **レート制限** / トークン計測 / 監査ログ / **MCP接続制御**。API Gateway の責務と1対1で並べて見せる ← **第5章で扱う4機能をここで責務として先に定義しておく** |
| 18 | 位置づけ図 | アプリケーション・エージェント → Gateway → （外部プロバイダ群 ／ セルフホスト推論 ／ MCPサーバー群） |
| 19 | 市場における一般解 | ①汎用AI Gateway: LiteLLM（自前運用）／Portkey（マネージド）／Kong AI Gateway（既存API管理の延長）／Envoy AI Gateway（OSS標準） ②**Agent Gateway という新カテゴリ**: Nutanix / Palo Alto Networks / AWS が参入、Snowflake も Cortex AI Gateway を投入。**OpenAI互換API がデファクト**となり抽象化が現実的になった |
| 20 | エンタープライズ要件における不足点 | ①セルフホストモデルと外部モデルを同一の統制下に置けるか ②オンプレ／エアギャップで動作するか ③マルチテナント・部門別のコスト按分 ④監査要件 |

**この章の狙い**: 中立的な一般論を先に立てる。ここを丁寧に行うほど、次章の製品パートの説得力が上がる。**スライド17で「ロードバランス」「レート制限」「MCP接続制御」を責務として明示**しておくことで、第5章が「Nutanix固有の機能紹介」ではなく「一般的責務に対する実装」として読める。

### 第5章 Nutanix Agent Gateway による4要件の充足（13分 / 10枚）

**本セッションの中核。** NAI 2.7 の新機能を、第2章の要件に対する回答として順に提示する。

| # | スライド | 内容 |
|---|---|---|
| 21 | Nutanix Agentic AI 全体像 | インフラ（Nutanix Cloud Platform）+ Kubernetes（NKP）+ ストレージ（NUS）+ **Nutanix Enterprise AI（NAI）= 中央AIコントロールプレーン**。NAI は Agent Gateway と Inference Management で構成 |
| 22 | NAI 2.7 と Agent Gateway | 2.6 の「AI Gateway」（統一推論エンドポイント・認証・可観測性・トークンベースのレート制限）から、2.7 で **Agent Gateway として GA**。本日扱う3機能（統合エンドポイント／トークンレートリミット／MCP接続制御）＋ロードマップ1件（セマンティックルーティング）を提示 |
| 23 | **機能① 統合エンドポイント（1/3）単一API** | **OpenAI互換の単一インターフェース**で OpenAI / Anthropic / Google Gemini / Azure OpenAI / AWS Bedrock、さらに Groq / Together / Mistral / Cohere / DeepSeek / SambaNova ＋ セルフホストモデル。→ **移植性の充足**。用途に応じたモデル選択が可能となり、4層のロックインを同時に緩和する |
| 24 | **機能① 統合エンドポイント（2/3）ロードバランス** | 複数のモデルエンドポイント間で負荷を分散。**エンドポイントが複数クラスタや複数のホステッドプロバイダに跨在していても分散可能**。→ 平常時のスループット確保とレイテンシ平準化。単一プロバイダのレート上限に張り付かない設計 |
| 25 | **機能① 統合エンドポイント（3/3）フォールバック** | プライマリが**障害を起こした場合／予算を超過した場合**に、健全なフォールバック先へ自動ルーティング。**アプリケーションコードの改修は不要**。→ **可用性の充足**。第1章スライド5「単一プロバイダ99.9%」への直接の回答 |
| 26 | ロードバランスとフォールバックの関係 | 同じ統合エンドポイントの2つの側面。**LBは平常時の分散、フォールバックは異常時の退避**。両者を1枚の図で整理し、正常時→劣化時→障害時の遷移として示す |
| 27 | **機能② トークンレートリミット＝トークンのサーキットブレーカ** | **本セッションの山場**。第3章スライド12で示した「200 OK を返しながらコストが暴走する」欠落への回答。トークン割当（quota）と上限を中央で強制し、閾値超過時に遮断する。**従来のCBがエラー率を監視するのに対し、これはトークン消費量を監視する遮断器**。エージェント別・チーム別にリアルタイム可視化 → **コスト予測可能性の充足** |
| 28 | トークンサーキットブレーカの設計論点 | 遮断の単位（ユーザー／APIキー／エージェント／チーム／時間窓）、遮断時の挙動（429返却／キューイング／下位モデルへの降格）、閾値設計。**「遮断」と「降格」のどちらを選ぶかは業務要件次第**という設計判断を提示 |
| 29 | **機能③ MCP接続制御** **[Tech Preview]** | エージェントがどのMCPサーバーへ接続できるかを中央で制御。**ゲートウェイ側でのAPIキー注入により、エージェント自身は認証情報を保持しない**。MCPリクエスト単位の監査ログ。→ **データ主権の充足**。※MCPサーバーガバナンスと同梱テストエージェントは Tech Preview であり本番利用は想定外である旨を明示する |
| 30 | 4要件 × 機能の答え合わせ | スライド2で提示したマッピング表を再掲し、各行を充足済みとして回収する。**実行環境**（任意のCNCF準拠Kubernetes：NKP / Rancher / ベアメタル / AWS・Azure・GCP、オンプレ／エッジ／NEO Cloud）もここで併記 |

**補足スライド（時間に余裕がある場合／Developer 比率が高い場合）**

| # | スライド | 内容 |
|---|---|---|
| +a | 可観測性の実装 | **OpenInference 分散トレーシングと OpenTelemetry GenAI semantic conventions に準拠**。chat / embeddings / 画像生成 / 音声 / reasoning の各エンドポイントを横断。既存の APM・可観測性基盤にそのまま接続できる ← **標準準拠は Developer の信頼を得やすい論点** |
| +b | 推論基盤の構成 | NAI の推論エンジンは **vLLM**、NGC検証済みモデルは **NVIDIA NIM**。モデル選択後のエンジン設定・GPUスケジューリング・オートスケール・ヘルスチェックは NAI が自動構成。モデルの重みは Hugging Face / NVIDIA NGC から NFS ストレージへ |

### 第6章 ロードマップ：セマンティックルーティング（4分 / 3枚）

| # | スライド | 内容 |
|---|---|---|
| 31 | 現在地：静的ルーティングの限界 | ここまでの機能はいずれも**モデルの指定はアプリケーション側が行う**前提。しかし「どのモデルが最適か」はリクエスト内容に依存する。全要求を最上位モデルへ送る運用は、品質は満たすがコスト効率が悪い |
| 32 | セマンティックルーティングとは | リクエストの**意味内容・複雑性・意図**に基づき、最適なモデルを動的に選択する方式。単純な要求は小規模モデルへ、困難な要求は高性能モデルへ。**判定をGateway層に置くことで、全アプリケーションに一貫したポリシーが自動適用される** |
| 33 | 何が変わるか | コスト・品質・レイテンシの同時最適化。参考値として、上位モデルと小型モデルの単価差は10倍超の水準にあり、相当割合の要求を小型モデルへ振り分けても品質を維持できるとする研究がある。**※本機能はロードマップであり、提供時期・仕様は未確定である旨を明示する** |

**この章の狙い**: 「今日の話で終わりではない」ことを示し、基盤選定の判断軸に将来性を加える。**ロードマップである旨の明示は必須**——GA機能と混在させると信頼を損なう。

### 第7章 リファレンスアーキテクチャと導入手順（2分 / 3枚）

| # | スライド | 内容 |
|---|---|---|
| 34 | リファレンス構成図 | 業務アプリ／エージェント → Agent Gateway → ①外部LLM ②NAI上のセルフホストモデル ③MCPサーバー群 |
| 35 | 障害時シーケンス | 平常時（LBによる分散）→ プライマリ劣化検知 → フォールバック先へ退避 → アプリケーションから見た挙動（正常応答の継続）をシーケンス図で |
| 36 | 段階的導入ロードマップ | **Step1** 全AIトラフィックを統合エンドポイントへ集約（可観測性の確保のみ）→ **Step2** トークンレートリミットの適用 → **Step3** ロードバランス／フォールバック構成 → **Step4** セルフホストモデルの併用 → **Step5** MCP接続制御。**Step1 を可観測性のみに限定することが導入障壁を下げる** |

### クロージング（2分 / 2枚）

| # | スライド | 内容 |
|---|---|---|
| 37 | まとめ | 4要件の再掲＋「アプリケーションを改修せず、基盤層で充足できる」＋ロードマップとしてのセマンティックルーティング |
| 38 | Next Action / 参考リンク | 評価版・ドキュメント・問い合わせ導線 |

### 30分版への圧縮方針

第5章・第6章は必須要素のため維持し、前段を削る。

| 章 | 対応 |
|---|---|
| 第1章 | 4枚 → 2枚（スライド3と5のみ。障害事例はSLAの枚に統合） |
| 第2章 | 4枚 → 2枚（要件一覧＋最も訴求したい1要件のみ展開） |
| 第3章 | 5枚 → 3枚（**スライド12は必ず残す**。13・14を統合） |
| 第4章 | 5枚 → 2枚（16と17のみ。市場動向は口頭で1文） |
| 第5章 | **10枚を維持** |
| 第6章 | **3枚 → 2枚**（31と32を統合） |
| 第7章 | 3枚 → 2枚（35を省略） |

---

## 2. デモ候補（入れる場合は第5章の後に5〜7分）

必須4要素に対応させた優先順位。

1. **トークンレートリミットの発火デモ** — エージェントを暴走させ（意図的にループさせる）、**200 OK が返り続けている最中に**トークン上限で遮断される様子を見せる。**第3章スライド12の伏線を映像で回収できるため、本セッションで最も効果的**
2. **フォールバックのライブデモ** — プライマリプロバイダを意図的に停止（またはキーを無効化）→ 同一の curl / 同一のアプリケーションが応答を継続することを見せる。セッションタイトルの主張をそのまま実証できる
3. **ロードバランスの可視化** — 複数エンドポイントへ要求が分散する様子をダッシュボードで。フォールバック（2）と連続して見せると「平常時の分散」と「異常時の退避」の違いが明確になる
4. **単一APIでのモデル切替** — `model` パラメータのみを変更して外部モデル → セルフホストモデルへ。base_url もコードも不変であることを示す
5. **MCP接続制御** — 許可されていないMCPサーバーへの接続が拒否される

デモが用意できない場合は、1と2をスクリーンショット＋シーケンス図で代替する。**1は静止画でも「200 OK なのに遮断される」という状態遷移が伝わる図にすること。**

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
| AI Gateway は複数の上流プロバイダに接続するよう設定でき、プライマリが**障害を起こすか予算を超過**すると、健全なフォールバック先へ自動的にトラフィックがルーティングされる。**アプリケーションコードの書き換え不要** | B | 検索スニペット（NAI 2.7 関連） — 「予算超過でも切り替わる」点は障害起因のフェイルオーバーと別軸なので**要一次情報での裏取り** |
| OpenAI互換の単一インターフェースで、OpenAI / Anthropic / Google Gemini / Azure OpenAI / AWS Bedrock、加えて Groq / Together / Mistral / Cohere / DeepSeek / SambaNova 等に接続 | B | 検索スニペット（NAI 2.7 関連） |
| トークン割当・上限を中央で強制し、**すべてのエージェント・チーム**にわたるトークン使用量をリアルタイムに可視化 | B | [Nutanix Blog](https://www.nutanix.com/blog/introducing-nutanix-agent-gateway) 経由 |
| **統合エンドポイントはロードバランスに対応**。複数のモデルエンドポイント間で負荷を分散でき、**エンドポイントが複数クラスタ上や複数のホステッドプロバイダに跨って存在していても分散可能** | B | 検索スニペット（NAI 2.7 関連） — **必須要素のため一次情報での裏取り優先度が高い** |
| 全LLM呼び出しを単一の統合エンドポイント経由とし、認証・レート制限・監視を一貫して適用。Azure上のGPT、AnthropicのClaude、NAI上のセルフホストLlamaのいずれを呼ぶ場合も同一 | B | 検索スニペット（NAI 2.7 関連） |
| 可観測性は **OpenInference 分散トレーシング**と **OpenTelemetry GenAI semantic conventions** に準拠し、chat / embeddings / 画像生成 / 音声 / reasoning の各エンドポイントを横断して透明性を提供 | B | 検索スニペット（NAI 2.7 関連） — 標準準拠はDeveloper向けの訴求点として有効 |
| NAI の推論エンジンは **vLLM**、NGC検証済みモデルには **NVIDIA NIM** を使用。いずれも NAI が自動構成し、エンジン設定・GPUスケジューリング・オートスケーリング・ヘルスチェックを担う。モデルの重みは Hugging Face / NVIDIA NGC から NFS ストレージへ取得 | B | 検索スニペット（NAI アーキテクチャ解説） |
| NAI は任意の CNCF準拠 Kubernetes に展開可能（NKP、Rancher、Docker、ベアメタル、AWS/Azure/GCP のコンテナインスタンス） | B | [Nutanix Enterprise AI 製品ページ](https://www.nutanix.com/products/nutanix-enterprise-ai) 経由 |
| MCPサーバーに対する統一的なセキュリティと RBAC を、**ゲートウェイインターフェースでのAPIキー注入**により実現 | B | 検索スニペット（NAI アーキテクチャ解説） |
| モデルベンダー横断の集中トークン可観測性。使用量トラッキング・コスト按分・過剰なトークン消費の抑制 | B | [iTWire](https://itwire.com/business-it-news/data/nutanix-strengthens-agentic-ai-governance-and-cost-control-with-agent-gateway) |
| 外部プロバイダのモデルとセルフホストモデルに単一APIでアクセス可能 | B | [Nutanix Blog](https://www.nutanix.com/blog/introducing-nutanix-agent-gateway) |
| MCPサーバーへの粒度の細かいアクセス制御（Governance for MCP）。GitHub / Stripe 等のツールへの安全な接続。**MCPサーバーガバナンスと同梱のテストエージェントは Tech Preview であり、本番利用は想定されていない**（Agent Gateway 本体は GA） | B | [tech-critter](https://www.tech-critter.com/nutanix-agent-gateway-general-availability/) ほか複数の報道で一致 |
| Agent Gateway は「エージェントと、それらが呼ぶモデルとの間」および「MCPサーバーと、それらがラップする業務ツールとの間」に位置するコントロールプレーン | B | 検索スニペット（NAI アーキテクチャ解説） |
| リクエスト単位の監査ログ | B | [Security Storage und Channel Germany](https://security-storage-und-channel-germany.de/language/en/nutanix-launches-agent-gateway-to-tame-ai-costs-and-governance/) |
| 可視化により、セルフホストモデルへ移行可能なワークロードを特定でき、外部サービスへの依存とコストを低減できる | B | [SourceSecurity](https://www.sourcesecurity.com/news/nutanix-agent-gateway-centralised-ai-governance-co-1568897613-ga.1783359990.html) |
| NAI 2.6 時点で AI Gateway として、クラウドホスト型/プライベートLLMへの統一セキュア推論エンドポイント、認証、可観測性、トークンベースのレート制限を提供。MCPサーバーとFine Tuningのサポートを追加 | B | [Nutanix Blog (NAI 2.6)](https://www.nutanix.com/blog/orchestrating-the-hybrid-ai-frontier) |
| Nutanix Agentic AI は NAI を中央AIコントロールプレーン（Agent Gateway + Inference Management）とし、インフラ・Kubernetes・データを統合。オンプレ／NEO Cloud／OEMパートナー基盤にまたがるソブリン対応 | B | [Nutanix プレスリリース](https://www.nutanix.com/press-releases/2026/nutanix-unveils-nutanix-agentic-ai) |
| NAI は CNCF準拠のKubernetes（NKP含む）上に展開可能。NVIDIA NIM / NeMo マイクロサービスと統合 | B | [Nutanix Blog](https://www.nutanix.com/blog/nutanix-enterprise-ai-makes-agents) |
| 2026年後半に、マルチテナンシー基盤 Nutanix Service Provider Central と、AIエンジニア向けのセルフサービス型マルチテナントAI管理ポータルを追加予定（neocloud向け） | B | [Nutanix プレスリリース (2026-04-07)](https://www.nutanix.com/press-releases/2026/nutanix-to-extend-nutanix-agentic-ai-empowering-neoclouds-to-deliver-higher-value-ai-services) / [Blocks & Files](https://www.blocksandfiles.com/hci/2026/04/07/nutanix-pushes-agentic-ai-bare-metal-kubernetes-at-next-2026/5214682) |

### セマンティックルーティング（ロードマップ機能の背景知識）

**注意**: Nutanix のロードマップ機能としての仕様は未公表。以下は**セマンティックルーティングという技術一般**の説明であり、スライド32-33の背景知識として用いる。Nutanix 実装の具体的挙動として語らないこと。

| ファクト | 信頼度 | 出典 |
|---|---|---|
| セマンティックルーティングとは、入力クエリの**意味内容・複雑性・意図**に基づき、最も適したモデルを動的に選択する方式 | B | [LLM Semantic Router / Red Hat Developer](https://developers.redhat.com/articles/2025/05/20/llm-semantic-router-intelligent-request-routing) |
| 基本パターンは「単純な要求は小型モデルへ、困難な要求は高性能モデルへ、回答は文字列一致ではなく意味でキャッシュする」 | B | [Maxim AI](https://www.getmaxim.ai/articles/top-5-llm-routing-techniques/) |
| ゲートウェイ層に判定を置くことで、ルーティングポリシーを一度定義すれば全アプリケーションの全LLMトラフィックに一貫して適用される | B | 同上 |
| 上位モデルと小型モデルの単価差は10倍超の水準（例: GPT-4o $2.50/M入力 対 GPT-4o-mini $0.15/M = 約16倍）。要求の相当割合を小型モデルへ振り分けられればコストは大幅に低減する | C | [Maxim AI](https://www.getmaxim.ai/articles/top-5-llm-routing-techniques/) — **単価は変動するため、登壇時点で要再確認。「10倍超の水準」程度の表現が安全** |
| 20B未満の小〜中規模モデルが 52.8% のプロンプトを最適に処理できるとする研究がある | C | 研究引用（[経由](https://www.getmaxim.ai/articles/top-5-llm-routing-techniques/)） — **一次論文の特定を推奨** |

> **⚠️ 用語の混同注意**: 検索すると **OpenTelemetry の "GenAI semantic conventions"**（可観測性のメトリクス命名規約）が「semantic」つながりで多数ヒットするが、**セマンティックルーティングとは全く別の概念**。NAI 2.7 の可観測性機能が semantic conventions に準拠していることと、セマンティックルーティングがロードマップであることは別の話なので、スライド上で混在させないこと。

### 市場カテゴリとしての Agent Gateway

| ファクト | 信頼度 | 出典 |
|---|---|---|
| Agent Gateway は新興の製品カテゴリとして立ち上がっており、エージェントと、それが関わるモデル・API・業務ツールの間に位置するコントロールプレーンとして、集中監査・アクセス管理・セキュリティポリシー適用を担う | B | [Forbes / Janakiram MSV (2026-07-05)](https://www.forbes.com/sites/janakirammsv/2026/07/05/agent-gateways-are-becoming-the-control-plane-for-enterprise-ai/) |
| **このパターンは10年前のAPI Gatewayの再来。生の接続性は、エンタープライズが本番で信頼する前にガバナンスの外皮を必要とした** | B | 同上 |
| エージェントを数個以上本番投入した企業では、集中コントロールプレーンがないと監査可能性が崩壊し、セキュリティポスチャが劣化する | B | 同上 |
| Nutanix、Palo Alto Networks、AWS が Agent Gateway 機能を構築中。Nutanix は2026年5月下旬に NAI 2.7 の一部として Agent Gateway を GA 出荷し、このカテゴリに明確な形を与えた | B | 同上 |
| Snowflake も Cortex AI Gateway を投入し、AIエージェントの統制と暴走コストの抑止を掲げている | B | [VentureBeat](https://venturebeat.com/security/snowflake-launches-cortex-ai-gateway-to-control-ai-agents-and-prevent-runaway-enterprise-costs) |

> **使い方**: 「API Gateway の再来」は本セッションで最も強い説得装置。IT Pro は API Gateway の導入経緯を体で知っているため、Agent Gateway を「新しい何か」ではなく「知っているパターンのAI版」として一瞬で受け取れる。第4章の冒頭（スライド16）に置くことを推奨。
>
> あわせて「Nutanix だけが言っている話ではない」ことの証明にもなる。AWS・Palo Alto・Snowflake が同じ方向に動いている事実を示せば、セッション全体が製品広告ではなく業界動向の解説として成立する。

---

## 4. ⚠️ 混同注意：Nutanix Agent Gateway ≠ agentgateway (Solo.io)

調査中に判明した重要な注意点。**名前が非常に似た別製品が存在する**。

| | Nutanix Agent Gateway | agentgateway（Solo.io / agentgateway.dev） |
|---|---|---|
| 提供元 | Nutanix | Solo.io（OSS、CNCF系エコシステム） |
| 位置づけ | Nutanix Enterprise AI 2.7 の構成要素 | Kubernetes ネイティブの OSS エージェント接続基盤 |
| フェイルオーバー | 「プライマリの障害時／予算超過時に、健全なフォールバック先へ自動ルーティング」 | `AgentgatewayBackend` の priority group による優先度ベースのフェイルオーバー、outlier detection によるアンヘルシーなバックエンドの排除 |
| キー管理 | トークン割当の中央強制、エージェント別・チーム別の可視化 | API キー認証 + トークンベースのレート制限 + 可観測性を組み合わせた「virtual keys」 |

検索結果では両者の記述が混在して返ってくる（2026-08-10 の再調査でも複数回混入を確認）。以下の用語は **Solo.io 側のものであり、Nutanix の機能としてスライドに書いてはいけない**。

- `AgentgatewayBackend`
- priority group（優先度グループによるフェイルオーバー順序）
- outlier detection
- virtual keys

Nutanix 側のフェイルオーバー実装の詳細（検知条件、切替単位、リトライ回数、ヘルスチェック方式など）は公式ドキュメントで確認すること。

---

## 5. 想定Q&A

IT Pro / Developer からは実装の詳細が飛んでくる。必須4要素それぞれに対する想定質問を用意する。

**統合エンドポイント：ロードバランス**
- 分散アルゴリズムは何か（ラウンドロビン／最小接続数／重み付け）。重み付けは可能か
- セルフホストモデルと外部プロバイダを**同一のLBプールに混在**させられるか。その場合の重み設計は
- ヘルスチェックの方式と間隔は。異常判定されたエンドポイントの復帰条件は
- 複数クラスタ跨ぎの分散時、レイテンシの差はどう考慮されるか

**統合エンドポイント：フォールバック**
- 切替の検知条件は（HTTPステータス／タイムアウト／エラーレート閾値）。判定単位は
- フォールバック中のレイテンシへの影響は。リトライ分の遅延はどの程度積まれるか
- **バックアップ先が別ベンダーの場合、出力品質・フォーマットの差はどう吸収するか** ← 最も踏み込まれやすい。**「可用性は担保するが出力の等価性は担保しない」という正直な線引きを準備すること**
- ストリーミング応答の途中で障害が発生した場合の挙動は
- 「予算超過でもフォールバックする」とは、どの通貨単位・どの粒度の予算か

**トークンレートリミット（トークンのサーキットブレーカ）**
- 遮断の単位は（ユーザー／APIキー／エージェント／チーム／時間窓）
- **上限到達時の挙動は選べるか**（429返却／キューイング／下位モデルへの降格）
- 入力トークンと出力トークンを区別して制限できるか
- 遮断はハードリミットか、警告閾値（ソフトリミット）を別に設定できるか
- **業務停止との両立をどう考えるか** ← 「コストを止めたら業務も止まる」という当然の反論。閾値設計と降格運用で答える
- リセット周期は（日次／月次／スライディングウィンドウ）

**MCP接続制御**
- アクセス制御の粒度は（サーバー単位／ツール単位／引数レベル）
- APIキー注入とは具体的にどの層で行われるか。エージェントは本当に認証情報を保持しないか
- 監査ログにプロンプト本文・ツール引数は含まれるか。含まれる場合、機密データの扱いは
- **Tech Preview はいつGAになるか** ← 必ず訊かれる。回答できるよう事前確認
- 未登録のMCPサーバーへの接続はどう遮断されるか

**セマンティックルーティング（ロードマップ）**
- 提供時期は。どのリリースを想定しているか
- ルーティング判定自体のコストとレイテンシは。判定に別モデルを使うのか
- 判定精度が不十分だった場合のフォールバック（上位モデルへのエスカレーション）はあるか
- ルーティングポリシーはユーザーが定義できるか、それとも学習ベースか

**共通・基盤**
- **Gateway 自体が SPOF にならないか。冗長構成はどう組むか** ← 可用性を訴求する以上、必ず訊かれる
- Gateway 経由による追加レイテンシはどの程度か
- 既存アプリの改修範囲は。OpenAI互換なら base_url の差し替えのみで済むか
- LiteLLM / Portkey を既に使用している場合の移行パスは
- エアギャップ環境で動作するか
- 新しいモデルが提供された際の追従はどうなるか

---

## 6. 登壇前の要確認リスト

本調査は外部サイトへの直接アクセスが制限された環境で行っており、Nutanix 製品の記述は報道・検索結果由来。**以下は公式ドキュメント／リリースノート／自社の製品担当で裏取りしてからスライドに確定させること。**

### 解決済み（2026-08-10 の追加調査）

- [x] ~~MCPガバナンスの提供ステータス~~ → **Tech Preview**（Agent Gateway 本体は GA）。同梱のテストエージェントも Tech Preview
- [x] ~~対応する外部プロバイダの一覧~~ → OpenAI互換の単一IFで OpenAI / Anthropic / Google Gemini / Azure OpenAI / AWS Bedrock ＋ Groq / Together / Mistral / Cohere / DeepSeek / SambaNova
- [x] ~~デプロイ先~~ → 任意の CNCF準拠 Kubernetes（NKP / Rancher / ベアメタル / AWS・Azure・GCP）
- [x] ~~推論エンジンの実体~~ → vLLM ＋ NVIDIA NIM（NGC検証済みモデル）

### 未解決（要裏取り）

**必須4要素に関わるもの（最優先）**

- [ ] **ロードバランスの仕様** — 分散アルゴリズム、重み付けの可否、ヘルスチェック方式、セルフホストと外部プロバイダの混在可否
- [ ] **フォールバックの実装詳細** — 検知条件・切替単位・リトライ回数・ヘルスチェック方式・設定方法
- [ ] **「予算超過（exceeds its budget）でもフォールバックする」の正確な仕様** — 障害起因の切替とは別軸の挙動。事実なら訴求点として強い
- [ ] **トークンレートリミットの仕様** — 遮断単位、上限到達時の挙動（429／キューイング／降格の選択可否）、入出力トークンの区別、リセット周期、ソフトリミットの有無
- [ ] **「トークンのサーキットブレーカ」という表現を公式に使ってよいか** — 社内表現か、対外的に使える比喩か。使えるなら本セッションの中核メッセージになる
- [ ] **MCP接続制御の粒度**（サーバー単位／ツール単位／引数レベル）と、APIキー注入の実装層
- [ ] **MCP接続制御の GA 予定時期**
- [ ] **セマンティックルーティングの提供時期・想定仕様** — ロードマップとして開示可能な範囲の確認。判定方式（ルールベース／学習ベース）、判定コスト

**その他**

- [ ] Agent Gateway 自体の冗長構成・HA 構成の推奨（Q&Aで必ず訊かれる）
- [ ] Gateway 経由による追加レイテンシの実測値
- [ ] **「予算超過（exceeds its budget）でもフォールバックする」の正確な仕様** — 障害起因の切替とは別軸の挙動。事実なら訴求点として強いので要確認
- [ ] トークン可観測性のメトリクス粒度と、ダッシュボードの実際の画面（デモ用）
- [ ] MCPアクセス制御の粒度（サーバー単位／ツール単位／引数レベル）
- [ ] NAI 2.7 のリリースノート全体（本資料に載せていない機能があるはず）
- [ ] 監査ログにプロンプト本文が含まれるか
- [ ] 日本国内での提供状況・サポート体制
- [ ] 「2026年に日本政府がソブリンAI政策を本格始動」の一次情報（政府発表の特定）
- [ ] コスト系の統計（信頼度C）を使うかどうかの判断。使うなら出典明記、使わないなら定性表現に置換

### 参考になりそうな未読の情報源

egress 制限で本文を読めていないが、タイトルから有用と判断されるもの。**手元の環境で確認を推奨**。

- [Nutanix Blog: AI Series — Building the Enterprise AI Stack with Nutanix Agentic AI](https://www.nutanix.com/blog/building-enterprise-ai-stack-with-nutanix-agentic-ai) — スタック全体像。第5章スライド22の図の元ネタになる可能性が高い
- [Nutanix Enterprise AI Datasheet](https://www.nutanix.com/library/datasheets/nutanix-enterprise-ai) — 仕様の一次情報
- [Nutanix: Running Agentic AI at Scale Requires Control](https://www.nutanix.com/enterprise-agentic-ai) — 本セッションのメッセージと直結
- [Nutanix Community: NAI 2.7 is Now Available](https://next.nutanix.com/nutanix-cloud-platform-for-ai-180/nutanix-enterprise-ai-nai-2-7-is-now-available-45620) — リリースノートへの導線
- [YouTube: Nutanix AI 2.7 — AI Agent Gateway Explained](https://www.youtube.com/watch?v=fPKjOYKGsf0) — **デモ画面の参考に最有力**
- [Nutanix Enterprise AI — Technical Field Guide](https://naifield.picklesbot.ai/) — 非公式と思われるが技術的な記述が詳しい。出典として使う場合は要注意

---

## 7. 構成上の設計判断メモ

なぜこの構成にしたか。差し替え検討時の参考に。

1. **必須4要素を第2章の非機能要件と1対1で対応させた** — 統合エンドポイント→可用性＋移植性、トークンレートリミット→コスト予測可能性、MCP接続制御→データ主権。**機能を列挙するのではなく、先に要件を定義してから回答として提示する**構造にしたことで、第5章が製品カタログではなく論証になる。マッピング表をスライド2と30の2回提示し、冒頭で渡した地図を末尾で回収する。

2. **「トークンのサーキットブレーカ」を成立させるため、第3章にスライド12を新設した** — 従来のCBはエラー率に反応するが、エージェントの暴走は 200 OK を返しながら発生するためCBが作動しない。この「欠落」を先に可視化しておくことで、トークンレートリミットが新機能紹介ではなく**既知の欠落に対する充足**として受け取られる。**この伏線がないと、レートリミットは単なるコスト管理機能に見えて訴求力を失う。**

3. **統合エンドポイントを3枚に分割した** — 単一API（移植性）／ロードバランス（平常時）／フォールバック（異常時）は充足する要件が異なるため、1枚に押し込むと論点が混ざる。さらにスライド26で「LBは平常時の分散、フォールバックは異常時の退避」と関係を整理する。**LBとフォールバックの混同は、この種のセッションで最も起きやすい誤解。**

4. **第1章スライド4に Azure の障害原因（リトライ増幅）を明示した** — 「上流変更が内部リトライトラフィックの急増を招いた」という一次情報は、第3章のリトライ設計と第5章のレートリミットの両方に効く。**実障害が伏線として二度働く**ため、費用対効果が高い1枚。

5. **セマンティックルーティングを独立章にし、ロードマップであることを明示した** — GA機能と混在させると全体の信頼性を損なう。独立させることで「今日の話で終わりではない」という基盤選定の判断軸を追加できる。ただし提供時期・仕様は未確定である旨を必ずスライドに書く。

6. **製品の登場を後半（第5章）に置いた** — IT Pro / Developer は製品紹介から入ると身構える。第3章でアプリケーション層実装の限界を提示し、第4章で Gateway という一般解を中立に示してから製品に入る。**第4章スライド17で「ロードバランス」「レート制限」「MCP接続制御」を一般的責務として先に定義**しておくことで、第5章が固有機能の自慢ではなく責務への実装として読める。

7. **第4章で競合OSS/SaaSに触れる** — LiteLLM / Portkey / Kong / Envoy AI Gateway に言及することで中立性と技術的信頼を獲得する。触れずに進めると「知らないのか、隠しているのか」と受け取られる。

8. **信頼度Cのコスト統計に依存しない構成にした** — 「エージェントは呼び出し回数が実行時に決まるため、事前見積りが原理的に困難」という**構造の説明**を主軸にすれば、数字が裏取りできなくても論旨は崩れない。セマンティックルーティングの単価差も「10倍超の水準」程度の表現に留める。

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
- [Forbes / Janakiram MSV: Agent Gateways Are Becoming The Control Plane For Enterprise AI (2026-07-05)](https://www.forbes.com/sites/janakirammsv/2026/07/05/agent-gateways-are-becoming-the-control-plane-for-enterprise-ai/)
- [VentureBeat: Snowflake launches Cortex AI Gateway](https://venturebeat.com/security/snowflake-launches-cortex-ai-gateway-to-control-ai-agents-and-prevent-runaway-enterprise-costs)
- [tech-critter: Nutanix Agent Gateway General Availability](https://www.tech-critter.com/nutanix-agent-gateway-general-availability/)
- [Nutanix: AI Series — Building the Enterprise AI Stack with Nutanix Agentic AI](https://www.nutanix.com/blog/building-enterprise-ai-stack-with-nutanix-agentic-ai)
- [Nutanix: Running Agentic AI at Scale Requires Control](https://www.nutanix.com/enterprise-agentic-ai)
- [Nutanix Enterprise AI 製品ページ](https://www.nutanix.com/products/nutanix-enterprise-ai)
- [Red Hat Developer: LLM Semantic Router — Intelligent request routing](https://developers.redhat.com/articles/2025/05/20/llm-semantic-router-intelligent-request-routing)
- [LLM Semantic Router ドキュメント](https://llm-semantic-router.readthedocs.io/en/latest/overview/semantic-router-overview/)
- [Maxim AI: Top 5 LLM Routing Techniques](https://www.getmaxim.ai/articles/top-5-llm-routing-techniques/)
- [NeuralTrust: LLM Model Routing](https://neuraltrust.ai/blog/llm-model-routing)
- [PR Newswire: Envoy AI Gateway Reaches v1.0](https://www.prnewswire.com/news-releases/envoy-ai-gateway-reaches-v1-0--establishing-the-open-source-standard-for-enterprise-ai-traffic-302808088.html)
- [agentgateway.dev — Model failover（※Solo.io の別製品）](https://agentgateway.dev/docs/kubernetes/main/llm/failover/)
- [agentgateway.dev — Virtual keys（※Solo.io の別製品）](https://agentgateway.dev/docs/kubernetes/main/llm/cost-controls/virtual-keys/)
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
