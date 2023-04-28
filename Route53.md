# Route53

## Route53 の概要

### Route53 とはなにか？

IP アドレスを人が読みやすい URL に変換して、住所として利用できるようにしてくれる DNS サーバーの役割を提供

Route53 は AWS が提供する権威 DNS サーバーで、ポート 53 で動作することから Route53 と呼ばれる

- 主要機能はドメイン登録/DNS ルーティング/ヘルスチェックの 3 つ
- ポリシーによるルーティング設定、トラフィックルーティング/フェイルオーバー/トラフィックフローに基づく様々な条件のルーティング設定が可能
- AWS 側で 100%可溶性を保証する SLA
- マネージドサービスとして提供しており、ユーザー側で冗長性などを考慮する必要がない
- ドメインを購入・登録・管理するレジストラーとしても機能する
- 他のレジストラーで購入したドメインを移管することも可能

### ホストゾーン

ドメインとそのサブドメインのトラフィックのルーティングする方法についての情報を保持するコンテナ

#### パブリックホストゾーン

- インターネット上に公開された DNS ドメインレコードを管理するコンテナ
- インターネットの DNS ドメインに対するトラフィックのルーティング方法を定義

#### プライベートホストゾーン

- VPC に閉じたプライベートネットワーク内の DNS ドメインのレコードを管理するコンテナ
- VPC 内の DNS ドメインに対して、どのようにトラフィックをルーティングするかを定義
- 1 つのプライベートホストゾーンで複数 VPC に対応
- VPC が相互アクセス可能であれば複数リージョンの VPC でも、同じホストゾーンを利用可能

### ルーティングポリシーの選択

#### シンプルルーティング

- レコードセットにおいて事前に設定された値のみに基づいて DNS クエリに応答するルーティング方式
- 静的マッピングによりルーティングを決定する

#### 加重ルーティング

- 複数エンドポイントに重みを設定して、重みに応じて DNS クエリに応答するルーティング方式
- 重みづけの高いエンドポイントに多くルーティングする

#### フェールオーバールーティング

- ヘルスチェックに基づいて、利用可能なリソースに DNS クエリを応答するルーティング方式
- 利用可能なリソースにルーティングされる

#### 複数値回答ルーティング

- 複数のリソースにトラフィックをルーティングさせる方式
- ランダムに選ばれた最大 8 つの別々のレコードに IP アドレスを設定して、複数の値を返答するルーティングさせる
- IP アドレス単位でヘルスチェックを実施してルーティングすることで正常なリソースの値を返す。ELB に代わるものではないが、正常を確認して複数の IP アドレスを返す機能により、DNS を使用してアベイラビリティとロードバランシングを向上させることができる

#### レイテンシールーティング

- リージョンのレイテンシーに応じて、DNS クエリに応答するルーティング方式。ユーザーの最寄りのリージョンになることが多い
- リージョン間のレイテンシーが低い方へルーティングされる

#### 位置情報ルーティング

- ユーザーの IP アドレスにより位置情報を特定して、地域ごとに異なるレコードを返すルーティング方式
- ネットワーク構成に依拠しない精度の高いレコード返答の区分けが可能となる

#### 地理的近接性ルーティング

- ユーザーとリソースの場所に基づいて地理的近接性ルールを作成して、トラフィックをルーティングする方式
  - AWS リソースを使用している場合は、リソースを作成した AWS リージョンを場所とする
  - AWS 以外のリソースを使用している場合は、リソースの緯度と経度で位置を場所とする
- 必要に応じてバイアスを設定し、地理的なリソースの配信範囲を調整することができる
- トラフィックフローを利用する必要がある

#### IP ベースのルーティングポリシー（New!）

- トラフィックの送信元の IP アドレスがわかっている場合に、IP アドレスによるユーザーの位置に基づいてトラフィックをルーティングする
- ユーザーの IP アドレスからエンドポイントにマッピングすることで、きめ細やかなルーティング制御が可能
- 顧客ごとに特有な情報に基づいてルーティングを最適化
- ユースケース
  - 特定の ISP から特定のエンドポイントにエンドユーザーをルーティングしたい場合
  - 位置情報ルーティングなど、既存の Route53 ルーティングタイプにオーバーライドを追加

### フェールオーバー構成

`フェールオーバー構成はRoute53のヘルスチェック機能を利用して正常なリソースを利用する構成のこと`

#### フェールオーバー（アクティブ/パッシブ）

- Route53 のプライマリリソースをアクティブなリソースとしてルーティングする。障害が発生した場合、Route53 はセカンダリーのリソースをルーティングする
- フェールオーバーポリシーを使用して設定する

#### フェールオーバー（アクティブ/アクティブ）

- Route53 は複数のリソースをアクティブとして、ルーティングする。障害が発生した場合、Route53 は正常なリソースにフェイルバックする
- フェールオーバー以外のルーティングポリシーを使用して設定する

### フェールオーバー手段の比較

`アクティブ/パッシブなフェールオーバーのRTOを高めるためにリカバリーコントローラーを利用する`

|                              | アクティブ/アクティブ                                                            | リカバリーコントローラー                                                                                                   | アクティブ/パッシブ                                                                                  |
| ---------------------------- | -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| RTO                          | 待機系もアクティブに利用しており、最大の RTO を達成できる                        | 待機系の状態をコントロールして、最適なリソース群にフェールオーバーを実施することで、アクティブ/パッシブの RTO を最大化する | 待機系は利用していないため、リソース状況に応じてフェールオーバーの RTO が決まる                      |
| 利用するルーティングポリシー | レイテンシールーティングなどのフェールオーバールーティングポリシー以外のポリシー | フェールオーバールーティングポリシー                                                                                       | フェールオーバールーティングポリシー                                                                 |
| コストや手間                 | アクティブな構成を冗長化するため最もコストを要する可能性がある                   | ・リカバリーコントローラー自体の利用にはコストは必要ない<br>・人の手を介さずにフェールオーバーを最大化できる               | ・待機系の状態次第でコストを抑えられる<br>・`待機系の状態次第でフェールオーバーが遅れる可能性がある` |

### DNS ファイアーウォール

Route53 Resolver 経由の DNS クエリに基づくサイトへの不正アクセスを制御して、DNS レベルの脅威を防ぐ

#### DNS Firewall ルールグループ

- DNS クエリをフィルタリングするために DNS Firewall ルールのコレクション
- DNS Firewall ルールによってルールグループ内の DNS クエリに対するフィルタリングルールを定義
- 関連づけた VPC の DNS クエリを Resolver が受信すると、そのクエリは DNS Firewall に送信され、フィルタリングを実施

#### ドメインリスト

- DNS フィルタリングを適用するドメインのコレクションを定義
- アクセスを許可するドメイン、アクセスを拒否するドメイン、またはその両方を設定する

## DNS の名前解決

### DNS の役割

DNS はドメイン名を IP アドレスに変換する役割を担っており、ネームサーバーとリゾルバによって構成される。

#### ネームサーバー

- インターネット上でドメインと Web サーバーやメールサーバーを結びつけるための名前解決をするサーバー

#### リゾルバ

- ドメイン名に対応つけられた IP アドレスを問われた際に、ドメイン名に関連つけられたネームサーバーを指定して、名前解決を行なってくれるサーバー