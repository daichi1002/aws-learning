# EC2

## EC2 の特徴

`数分で利用可能となる従量課金（時間〜秒単位）で利用可能な仮想サーバー`

- 起動・ノード追加・削除・マシンスペック変更できる仮想サーバーを提供する
- Windows や Linux などのほとんどの OS をサポート
- OS までは提供されているタイプを選択することで自動設定され、OS より上のレイヤーを自由に利用可能
- 独自の AMI（Amazon Machine Image）にバックアップして、再度起動させることが可能
- EC2 は EBS・ELB・Auto Scaling を含む

## EC2 の利用コスト

| 状態                | コスト                                                                                                                     |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| Running/開始/再起動 | 実行時間に応じて料金が発生する<br>利用中の EBS ボリュームの料金が発生する                                                  |
| 停止/Stop           | `EC2の料金発生は停止する`<br>利用中の EBS ボリュームの料金が発生する                                                       |
| 終了/Terminate      | `EC2の料金発生は停止する`<br>`デフォルト設定ではルートボリュームに設定された EBS ボリュームも削除され、料金発生は停止する` |

- リージョン
  - リージョンごとに価格が異なる
- データ転送
  - データ転送イン：無料
  - インターネットへのデータ転送アウト（GB あたり）
  - S3 から AWS 内でのデータ転送アウト（GB あたり）
- ボリューム
  - アタッチされた EBS でのデータ容量にも課金される。インスタンスを停止しても EBS 分は課金が継続されるため注意が必要
  - インスタンスストアには課金されない

## EC2 の起動方法

### 1. AMI(OS セッティング)を選択

Linux や Windows などの OS イメージを選択

### 2. インスタンスタイプを選択

t2.nano

`t2`：ファミリーと世代 `nano`：インスタンスの容量

**インスタンスファミリー**
ユースケースに応じてインスタンスタイプを選択する
|||
|--|--|
|汎用|ファミリー：A1、M5、T3 など<br>バランスの取れたコンピューティング、メモリ、ネットワークのリソースを提供し、多様なワークロードに使用。ウェブサーバーやコードリポジトリなど、インスタンスのリソースを同じ割合で使用するアプリケーションに最適なインスタンス。|
|コンピューティング最適化|ファミリー：C5、C6g など<br>高パフォーマンスプロセッサが必要なコンピューティングバウンドなアプリケーションに利用。バッチ処理ワークロード、メディアトランスコード、高性能 WEB サーバー、ハイパフォーマンスコンピューティング(HPC)、科学モデリング、専用ゲームサーバーおよび広告サーバーエンジン、機械学習推論|
|メモリ最適化|ファミリー：X1、R5、ハイメモリ、z1d など<br>メモリ内の大きなデータセットを処理するワークロードに対して高速なパフォーマンスに最適なインスタンス|
|ストレージ最適化|ファミリー：H1、D2、I3、I3en など<br>ローカルストレージの大規模データセットに対する高いシーケンシャル読み取りおよび書き込みアクセスを必要とするワークロード用。ストレージ最適化インスタンスは、数万 IOPS もの低レイテンシーなランダム I/O オペレーションに最適|
|高速コンピューティング|ファミリー：P3、Inf1、G4(GPU)、F1(FPGA)など<br>高速コンピューティングインスタンスでは、ハードウェアアクセラレーター（コプロセッサ）を使用して、浮動小数点計算、グラフィックス処理、データパターン照合などの機能を、CPU で実行中のソフトウェアに最適|

### 3. インスタンスタイプの詳細の設定

**ユーザーデータの利用**

ユーザーデータを利用して、EC2 インスタンス起動時に実行されるスクリプトを設定できる
|||
|--|--|
|ユーザーデータ|EC2 インスタンスの詳細設定の自動化に利用<br>Bash スクリプトなどを設定して、インスタンス起動時に実行されるように準備できる|
|ブートストラップ|インスタンスにユーザーデータを渡すことで、起動時に実行される処理のこと|

### 4. ストレージを選択

EC2 で直接利用するストレージは、不可分なインスタンスストアと自分で設定する EBS の 2 つ

|                                    |                                                                                                                                                                                               |
| ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| インスタンスストア                 | ・ホストコンピュータに内蔵されたディスクで EC2 と不可分のブロックレベルの物理ストレージ<br>・EC2 の一時的なデータが保持され、EC2 の停止・終了とともにデータは消去される<br>・無料             |
| Elastic Block Store<br>（EBS）     | ・ネットワークで接続されたブロックレベルのストレージで EC2 とは独立して管理される<br>・EC2 を Terminate しても EBS はデータを保持可能で、Snapshot を S3 に保存する。<br>・別途 EBS 料金が必要 |
| Amazon EFS                         | ・NAS に似たファイルストレージ<br>・ファイルシステムとして利用し、複数の EC2 インスタンスでの共有アクセスが可能                                                                               |
| Amazon FSx For Windows File Server | ・Windows File Server と互換性のあるストレージ<br>・Windows 上に構築され、Windows AD、OS やソフトウェアとの連携が豊富に可能                                                                   |

### 5. タグの追加

タグを追加で設定して、名前を付与することができ、EC2 などのリソースのグループ分けや権限分けに利用する

### 6. セキュリティグループを選択

インスタンスへのトラフィックのアクセス可否を設定するファイアーウォール機能を提供

### 7. キーペアを設定

キーペアを利用して自身がダウンロードした秘密鍵とマッチした公開鍵を有するインスタンスにアクセスする

## インターネットアクセス

起動したインスタンスにアクセスする際はパブリック IP を利用してアクセスする。インターネットからの EC2 インスタンスへのアクセスが不能な場合は以下のような原因が考えられる。

- パブリック IP アドレスがない
- アクセス許可設定
  - セキュリティグループまたはネットワーク ACL により適切なアクセス許可が設定されていない
  - 利用しているオンプレミス側のネットワーク環境の問題
- ネットワークの構成ミス
  - パブリックサブネットにインスタンスを配置していない（サブネットと VPC にインターネットゲートウェイが設定されていない）

## インスタンスの購入方式

`インスタンスの購入方式に応じて割引価格が提供されるため、用途に応じて割引価格を利用することが重要になる`

|                          |                                                                                                                                                                                                                                                                                                 |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| オンデマンドインスタンス | ・通常のインスタンス購入方式<br>・長期契約なしで、コンピューティング性能に対して秒単位または分単位で支払う。そのライフサイクルを完全に制御できるためユーザーが起動、停止、休止、開始、再起動、または終了時点を決定できる。                                                                      |
| リザーブドインスタンス   | ・Amazon EC2 リザーブドインスタンス（RI）は、1 年または 3 年の期間利用を予約することで、通常のオンデマンド料金に比べて大幅な割引価格（最大 72%）が適用されるインスタンスの購入方式。<br>・特定のアベイラビリティゾーンまたはリージョンで使用するキャパシティーを予約できる 2 つのタイプがある。 |
| スポットインスタンス     | ・オンデマンド価格より定価で利用できる AWS 管理用に保持されているが未使用の EC2 インスタンス。ユーザーは未使用の EC2 インスタンスを静止状態割引（最大 90%割引ほど）でリクエストできる。<br>・実行時間に柔軟性がある場合や、中断できる処理に利用する。                                           |

## 物理対応可能なインスタンス

`物理サーバーにインスタンスを起動して制御が可能なタイプのインスタンス`

- ハードウェア専有インスタンス
  - 専用 HW の VPC で実行される EC2 インスタンス
  - ホスト HW のレベルで、他の AWS アカウントに属するインスタンスから物理的に分離する
  - 同じ AWS アカウントのインスタンスとは HW を共有する可能性がある
- Dedicated Host
  - EC2 インスタンス容量を完全にお客様専用として利用できる物理サーバー
  - サーバーにバインドされた既存のソフトウェアライセンスを利用可能
- Bare Matal
  - アプリケーションが基盤となるサーバーのプロセッサーとメモリーに直接アクセス可能なインスタンス
  - AWS の各種サービスとの連携が可能で OS が直接仮想のハードウェアにアクセス可能

## リザーブドインスタンスの特徴

`利用期間を長期指定して利用する形式で、オンデマンドに比較して最大75%割引になる`

|                                                               | スタンダード                       | コンパ＝ティブル                   |
| ------------------------------------------------------------- | ---------------------------------- | ---------------------------------- |
| 利用期間                                                      | 1 年（40%割引）<br>3 年（60%割引） | 1 年（31%割引）<br>3 年（54%割引） |
| AZ/インスタンスサイズ/ネットワークタイプ変更可否              | 有                                 | 有                                 |
| インスタンスファミリー/OS/テナンシー/支払オプションの変更可否 | なし                               | 有                                 |
| リザーブドインスタンスマーケットプレイスでの販売可否          | 可能                               | 今後可能となる予定                 |

- ユースケース
  - 一定した状態または使用量が予測可能なワークロード
  - 災害対策などキャパシティ予測が可能なアプリケーション

## スポットインスタンスの特徴

`予備のコンピューティング容量を、オンデマンドインスタンスに比べて割引（最大90%引き）で利用できるEC2インスタンス`

- 予備容量を利用するためとても安いが、予備用のため途中で削除される可能性がある一時利用用のインスタンス
- 事前に上限価格とインスタンスタイプを設定してリクエストすると、その価格以内のインスタンスを起動する。
- 1 時間ごとの価格が需要と供給で変動して、最大 90%割引となる。価格が上回った場合は停止または終了を選択できる（`一時的な拡張などの用途で利用`）

【リクエストの中断などの挙動】

- リクエストを削除しないと、インスタンスは停止しても、再度起動をし続ける

## スポットフリートの利用

フリートとは、グループみたいな意味合い

`スポットインスタンスのタイプと価格をフリート指定することで、自動でスポットインスタンスのリクエストを最適化する`

- スポットインスタンスのフリートでオンデマンドも設定可能
- 上限価格内で目標容量のインスタンスを起動できるように調整する
- 配分戦略を選択して設定する
  - lowestPrice：スポットインスタンスは、最低価格のプールから取得。デフォルト戦略
  - Diversified：スポットインスタンスは、全プールに分散
  - capacityOptimized：起動中のインスタンスの数に最適な容量を持つプールから選択

## EC2 フリートの利用

`オンデマンド、リザーブド、スポットインスタンスで構成されるインスタンスセットを定義する仕組み`

- オンデマンドとスポットのターゲット容量を別個に定義して、1 時間あたりの支払い上限料金を定義する
- アプリケーションに最適なインスタンスタイプを指定
- 各購入オプション内でフリート容量を Amazon EC2 で分散する方法を指定する

## EC2 のリカバリー

`EC2インスタンスは定期的にバックアップすることが重要`

- 定期的にバックアップ（AMI/スナップショット）をとる
- 定期的にリカバリプロセスを確認する
- 複数の AZ に重要なアプリケーションをデプロイする
- CloudWatch によりインスタンスのステータスをモニタリングする
  - チェック結果が失敗になった場合、CloudWatch アラームアクションを使用してインスタンスを自動的に復旧
  - 自動復旧後のステータスを IP アドレスは元のインスタンスと同じ
- インスタンス起動時に動的 IP アドレス処理の設定を行うこと

## インスタンスの再起動

| 状態          | 意味                                                                                                                            | 料金                                   |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------- |
| pending       | インスタンスは running 状態への移行準備中<br>初回起動時、または pending 状態から起動する場合、インスタンスは stopped 状態になる | 課金されない                           |
| running       | インスタンスは実行中で、使用できる状態                                                                                          | 課金される                             |
| stopping      | インスタンスは停止または停止休止の準備中                                                                                        | 停止準備中は無課金<br>休止準備中は課金 |
| stopped       | インスタンスは停止されているため使用できない<br>いつでも起動できる状態                                                          | 課金されない                           |
| shutting-down | インスタンスは削除準備中                                                                                                        | 課金されない                           |
| terminated    | インスタンスは完全に削除されているため、きどうすることができない                                                                | 課金されない                           |

## ハイバネーションの利用

`ハイバネーションにより、再起動時に停止前の状態を維持することが可能`

- シャットダウン前にメインメモリの内容をハードディスク等に退避させることで、次回起動時にまたメインメモリに読み込んで、シャットダウンする前と同じ状態で起動する
- 再起動時に停止前の状態を維持することで、再起動後のセッティングを容易にする

## 小テストで間違えたところ

- EBS は他のリージョンにおいてスナップショットをコピーすることで、復元することができる
- リザーブドインスタンスのスタンダードはインスタンスファミリー/OS/テナンシー/支払オブションの変更ができない。コンバーティブルはそれらを変更できる柔軟性があるため、割引率がスタンダードより低くなっている
- デフォルトセキュリティグループは全てのインバウンドは同じセキュリティグループに割り当てられたインスタンスからのインバウンドトラフィックのみ許可されている。セキュリティグループに関連付けられたインスタンスの相互通信は、トラフィックを許可するルールを追加するまで許可されない
- AMI にはスナップショットも含まれているため、AMI 作成時点の EBS ボリュームも復元できる
