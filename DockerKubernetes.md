# Docker/Kubernetes 実践コンテナ開発入門
# 1章: Dockerの基礎
## Dockerの苦手な部分
- Dockerのコンテナ内部はLinux系OSっぽい構成をしているが、厳密にLinuxとしてふるまうわけではない
- FreeBSDのような非Linux環境を動作させたい場合は実現できない
- あくまでもアプリケーションをデプロイするための箱だと思うべき

## 1.1.2 Dockerの基礎概念
コンテナ型仮想化技術
- 仮想化ソフトウェアなしにOSのリソースを隔離し、仮想OSにする
    - この仮想OSをコンテナと呼ぶ
- OS上にインストールした仮想化ソフトウェアを利用し、ハードウェアを円座員により再現しゲストOSを作り出す仕組みはホストOS型の仮想化と呼ぶ
    - コンテナ型仮想化に比べると、仕組み上のオーバーヘッドが大きい

## 1.1.3 Dockerの考え方に触れる
出来上がったimageはDockerさえ実行されていればホスト環境を問わず実行でき、ホストにNode.jsやnpmをインストールする必要さえない

### LinuxKit
WindosやMacでも使えるLinuxサブシステムの名前
- WindowsやMacではLinuxがサポートされていないため、当初はvirtual boxからDockerを操作していた
- オーバーヘッドがあり使いにくいため、より簡素な環境構築のために作成した

## 1.2.3 本番環境に導入してこそDocker
実際に使用されている例
- AbemaTV
- FRESH LIVE
- アメーバブログ
- Pokemon Go

 クラウドプラットフォーム上での利用例
 - GKE(GCP)
 - ECS(AWS)
 - ACS(AZURE)

 不向きなもの
 - データストア

# 2章: Dockerコンテナのデプロイ
## 2.1 コンテナでアプリケーションを実行する
- Dockerイメージ
    - Dockerコンテナを構成するファイルシステムや、実行するアプリケーションや設定をまとめたもの
    - コンテナを作成するときのテンプレートとなるもの
- Dockerコンテナ
    - Dockerimageをもとに作成され、具現化されたファイルシステムとアプリケーションが実行されている状態

## 2.3.1 Dockerコンテナのライフサイクル
実行中・停止・破棄というライフサイクルになっている

- 実行中
    - Dockerimageをもとにコンテナが作成され、DockerfileのCMDやENTRYPOINTで定義されているアプリケーションの実行を開始する
- 停止
    - ディスクにコンテナ終了時の状態は保持されているので、再実行可能
- 破棄
    - 停止したコンテナは明示的に破棄しない限りディスクに残り続ける
        - 頻繁に実行と停止をするような場合はどんどんディスクを専有していくので不要なコンテナは削除することが望ましい

## 2.5 Docker Composeでマルチコンテナを実行する
- Dockerはアプリケーションのデプロイに特化したコンテナ
    - Dockerコンテナ=単一のアプリケーション
    - 仮想サーバとは対象とする粒度が違う
- docker-composer.ymlにコンテナ同士の依存関係を定義する

## 2.6 Composeによる複数コンテナの実行
- docker_compose.ymlのvolumeというキーでホストや他コンテナとの共有領域を指定できる
- linksキーを使うと他のコンテナとの通信を簡単にできる(IPを調べたりせずに名前解決できる)

# 3 実用的なコンテナの構築とデプロイ
## 3.1.1 1コンテナ = 1プロセス?
### 定期的にジョブを実行するアプリケーション
スケジューラを持たないアプリケーションの場合(cronを使用する場合)、1コンテナ=1プロセスとして行うならば以下の2つの方法が考えられる
- ジョブコンテナ側にジョブの実行トリガーとなるようなAPIを用意し、cronコンテナからコンテナ間通信でこのAPIをコールする
- cronコンテナ上にDockerを構築して、その上でジョブコンテナを実行する

実現不可能ではないが、複雑すぎるため、無理せずに1つのコンテナで複数のプロセスを実行するこの方式のほうがシンプルに完結するユースケースも多い

### 子プロセスまで意識するのは本末転倒
プロセスを過剰に意識するとコンテナをうまく扱えない
- Apacheはクライアントからリクエストを受けるたびにmasterプロセスが子プロセスをforkすることもある
- Nginxの場合はmasterプロセスに加え、workerプロセスやキャッシュマネジメントプロセスも存在している

## 3.1.2 1コンテナに1つの関心事
1プロセス=1コンテナは無理があるが、Docker社はこのことについてどう考えているのか  
Docker公式ドキュメントの'Best practice for writing Dockerfiles'において以下のように述べている
- Each container should have only one concern.

アプリケーションとコンテナの粒度について考える場合、「それぞれのコンテナが担うべき役割を適切に見極め、かつそれがレプリカとして複製された場合でも副作用なくスタックとして正しく動作できる状態になるか」という考えに基づいて設計すると良い

## 3.2 コンテナのポータビリティ
Dockerの大きな利点はポータビリティだが、それは完璧なものではなく、例外が存在する  

### 3.2.1 Kernel・アーキテクチャの違い
- Dockerコンテナが実行できるホストは、ある特定のCPUアーキテクチャやOSの前提に成立している
    - RaspberryPiなど

### 3.2.2 ライブラリ・ダイナミックリンクの課題
- アプリケーションが利用しているライブラリによっても、ポータビリティが損なわれる可能性がある
    - ネイティブライブラリをダイナミックリンクさせるような場合
    - CI側とコンテナ側で採用しているライブラリが違う
        - よくあるケースではCI側がglibcを使い、コンテナ側がmuslを使用しているなど

## 3.3 Dockerフレンドリーなアプリケーション
### 3.3.1 環境変数を活用する
Dockerコンテナとして実行されるアプリケーションを制御する方法は以下4つある
- 実行時引数として値を渡す
    - 実行時引数が増えるとアプrケーション側での引数から任意の変数へのマッピングが増えたりして煩雑になってしまう傾向がある
- 設定ファイルとして渡す
    - 実行するアプリケーションにdevelopmentやproductionといった環境名を与えることで利用する設定ファイルを切り替える方法
    - ポータビリティを下げてしまう
- 環境変数で制御
    - 設定ファイルと比べ、ちょっとした変更で毎回imageをビルドしなくて良い
    - KeyValueの形式でしかデータを保持させられないのがデメリット
- 設定ファイルに環境変数を埋め込む
    - 設定ファイルと環境変数の両方のメリットを活かしているが、環境変数だけでできるのがベスト

Dockerフレンドリに作れるかどうかが、技術選定の軸になりうる

## 3.4 永続化データをどう扱うか
Dockerコンテナ実行中に書き込まれたファイルは、ホスト側にファイル・ディレクトリをマウントしない限りコンテナを破棄したタイミングでディスクから消去される

### 3.4.1 DataVolume
Dockerコンテナ内のディレクトリをディスク内に永続化する仕組み
- docker container run に -v [マウント先] を指定することで実行可能
- ポータビリティは下がる

### 3.4.2 DataVolumeコンテナ
DataVolumeコンテナを利用する方法ではコンテナ間でディレクトリを共有する
- DataVolumeコンテナではホスト側の/var/lib/docker/volumes/ 以下に配置されるようになっており、コンテナに与える依存性を下げることができる

## 3.5 コンテナ配置戦略
### 3.5.1 DockerSwarm
DockerSwarm: 複数のDockerホストを束ねてクラスタ化するためのツールで、コンテナオーケストレーションシステムの一つ  
- Serviceはレプリカ数(コンテナ数)を制御することで容易にコンテナを複製でき、複数のノードに配置できるためスケールアウトへの親和性が高い
- Serviceによって管理される複数のレプリカはService名で名前解決でき、かつServiceへのトラフィックはレプリカへ分散される
- Swarmクラスタの外からSwarmのServiceを利用するには、Serviceにトラフィックを分散するためのプロキシを用意する
- Stackは複数のServiceをグルーピングでき、複数のServiceで形成されるアプリケーションのデプロイに役立つ

# 4 Swarmによる実践的なアプリケーション
TODOアプリ作成のためのコンテナ配置戦略  
SERVICEに対するリクエストを所属する複数のコンテナに分散させられるので耐久性の高いシステムを作ることができる

# 5 Kubernetes入門
## 5.1 Kubernetesとは
Google主導で開発されたコンテナの運用を自動化するためのコンテナオーケストレーションシステム
- APIやCLIツールも提供されている
- コンテナを用いたアプリケーションのデプロイはじめ、様々な運用管理の自動化を実現する
    - Dockerホストの管理、サーバリソースの秋具合を考慮したコンテナ配置、スケーリングや複数のコンテナ郡へのアクセスを取りまとめるロードバランサー、死活監視など

### 5.1.1 Dockerの隆盛とKubernetesの誕生
Dockerの盛り上がりとともに、エコシステムの不足を補おうと様々なオーケストレーションシステムが登場した

#### クラウドプラットフォームのKubernetesサポート
GCPにはGKEというコンテナマネージドサービスがあるが、KubernetesはGKE専用ではない  
AWSやAzureにもKubernetesのマネージドサービスがある   
=> クラウド事業者にとってKubernetesとそれぞれのプラットフォームをシームレスに連携させ、効率的に開発できるサービスを提供していくことが不可欠になった

### 5.1.2 Kubernetesの位置付け
- Swarm: 複数のホストを束ねて基本的なコンテナオーケストレーションを実現するためのDockerの関連技術  
- Kubernetes: Swarmより機能が充実したコンテナオーケストレーションシステムでDockerを始めとする様々なコンテナランタイムを扱える
    - Compose/Stack/Swarmの機能を統合しつつ、より高度に管理できるもの

## 5.3 Kubernetesの概念
Kubernetesで実行されるアプリケーションは様々なリソースと協調して動作することで成立している (リソース: Node, Namespace,Podといった構成要素。コンテナとリソースは別の粒度)  
リソース
- Node: Kubernetesクラスタで実行するコンテナを配置するためのサーバ
- Namespace: Kubernetesクラスタ内で作る仮想的なクラスタ
- Pod: コンテナの集合単位でコンテナを実行する方法を定義する
- Service: Podの集合にアクセスするための経路を定義するy
- その他: いろいろ

## 5.4 KubernetesクラスタとNode
KubernetesクラスタはKubernetesの様々なリソースを管理する集合体
- クラスタが持つリソースの中で最大のモノがNode
- NodeはKubernetesのクラスタの管理下に登録されているDockerホストのことで、Kubernetesでコンテナをデプロイするために利用されている
- Kubernetesクラスタには全体を管理するサーバーであるMasterが一つ以上存在し、MasterとNode郡で構成されている

KubernetesはNodeの使用リソース状況や配置戦略によって適切にコンテナを配置する
- クラスタに配置されているNodeの数、Nodeのマシンスペックによって配置できるコンテナの数は変わってくる

## 5.5 Namespace
クラスタの中に入れ子となる仮想的なクラスタのこと  
クラスタを構築するとあらかじめ、default, docker, kube-public, kube-systemというNamespaceが用意されている  
Namespaceごとに操作権限を設定できる

## 5.6 Pod
Containerの集合単位で、少なくとも1つのPodを持つ  
- NginxコンテナとGoのアプリケーションコンテナのように密結合な関係で有ることのほうが都合が良いケースは1Podにする
- 同一Pod内のコンテナは全て同一のNodeに配置される = 1つのPod内のContainerが複数Nodeに渡って配置されることはない

Podをどのような粒度にすべきかと悩む場合は同一Nodeに配置されるという特性から逆算してみると良い
- リバースプロキシとしてのNginxとその背後のアプリケーションで1つのPodにするのはよくあるパターン
- 同時にデプロイしないと整合性が保てないようなケースにおいて、同一のPodでコンテナをひとまとめにしてしまうのはデプロイ戦略として有効

## 5.7 ReplicaSet
Podを定義したマニフェストファイルからは1つのPodしか作成できない  
しかしある程度の規模になると同一のPodを複数実行することで可用性を高めることが必要になる  
そのような場合ReplicaSetを利用するReplicaSetは
- 同じ仕様のPodを複数作成、管理するためのリソース
- ReplicaSetを操作して削除されたPodは復元できないため、Webアプリケーションのようなステートレスな性質を持つPodの利用に向いている

## 5.8 Deployment
- ReplicaSetよりも上位のリソース
- アプリケーションデプロイの基本単位となるリソース
- ReplicaSetを管理・操作するためのリソース

実はDeploymentの定義はReplicaSetと変わらないが、DeploymentはReplicaSetの世代管理ができる

### 5.8.1 ReplicaSetのライフサイクル
KubernetesはDeploymentを1つの単位としてアプリケーションをデプロイする  
- 実運用では、ReplicaSetを直接用いることは少なく、Deploymentのマニフェストファイルを扱うことがほとんど
- ロールバックが可能なのでなにかあればすぐに戻せる

## 5.9 Service
Podの集合(主にReplicaSet)に対する経路やサービスディスカバリを提供するためのリソース  

Serviceには様々な種類がある
- ClusterIP Service
    - デフォルトはClusterIP Service
    - クラスタ上の内部IPアドレスにServiceを公開でき、Service名で名前解決できる
    - 外からはアクセスできない
- NodePort Service
    - クラスタ外からアクセスできる
    - ClusterIPを作成できる点はClusterIP Serviceと同じ
- LoadBalancer Service
    - ローカル開発環境では利用できない
    - クラウドプラットフォームで提供されているロードバランサーと連携するためのもの
- ExternalName Service
    - selectorもport定義も持たない特殊なService
    - Kubernetesクラスタ内から外部のホストを解決するためのエイリアスを提供

## 5.10 Ingress
Serviceでの外部公開はL4層レベルまでしか扱えないため、HTTP/HTTPSのようにパスベースで転送先のServiceを切り替えるといったL7層レベルの制御はできない  
IngressであればServiceのKubernetesクラスタの外への公開とVirtualHostやパスベースでの高度なHTTPルーティングを両立する  

# 6章: Kubernetesのデプロイ・クラスタ構築
コンテキストをスイッチするのにkubectxが便利

## 6.3 Master Slave構成のMySQLをGKE上に構築する
Kubernetesではホストから分離可能な外部ストレージをボリュームとして使用する
- Podが別のホストに再配置された場合は外部ストレージとしてのボリュームはデプロイされたホストに自動的に割り当てられる
- これによってホストとデータボリュームが別れて便利になる

この仕組を実現するのが以下のリソース
- PersistentVolume
- PersistentVolumeClaim
- StorageClass
- StatfulSet

### persistentVolumeとPersistentVolumeClaim
ストレージ確保のためのリソース  
- クラスタが構築されているプラットフォームに対応した永続ボリュームを作成する

PersistentVolume
- ストレージの実体
- GCPではGCEPersistentDiskがそれにあたる

PersistentVolumeClaim
- ストレージを論理的に抽象化したリソース
- PersistentVolumeに対して必要な容量を動的に確保

### StorageClass
PersistentVolumeが確保するストレージの種類を定義できるリソース  
GCPのストレージには標準とSSDがある

### StatefulSet
データストアのように継続的にデータを永続化するステートフルなアプリの管理に向いているリソース
- deploymentと似ているが、一意性をもつPodや永続化データをもつ必要のないステートレスなアプリに向いている
    - deploymentと違い、連番で一意な識別子が振られる
- ステートフルなReplicaSetという位置付け

# 7章 Kubernetesの発展的な利用
## 7.1 Kubernetesの様々なリソース
### 7.1.1 Job
1つ以上のPodを作成し、指定された数のPodが正常に完了するまでを管理するリソース
- JobによるすべてのPodが正常に終了しても、Podは削除されずに保持されるため、終了後にPodのログや実行結果を分析できる
    - 大規模な計算やバッチ指向のアプリケーションに向いている
- Podを複数並列で実行することで容易にスケールアウトできる
- Serviceと連携した処理を行いやすい

### 7.1.2 CronJob
スケージューリングして定期的にPodを実行できる
- コンテナと親和性を持ったままスケジューリングできるのがメリット

### 7.1.3 Secret
TLS/SSL証明書や秘密鍵やパスワードを平文で扱うのは危ないので、機密情報をbase64エンコードした状態で扱える

## 7.2 ユーザー管理とRole-Based Access Control(RBAC)
Kubernetesのユーザーは以下2つの概念がある
- 認証ユーザー
    - クラスタ外からKubernetesを操作するためのユーザーで、様々な方法で認証される
    - kubectl等でKubernetesを操作するためのもの
    - ユーザーをグループ化するグループという概念も存在
- ServiceAccount
    - Kubernetes内部で管理され、Pod自身がKubernetesAPIを操作するためのユーザー
    - Kubernetesリソースとして提供
    - ServiceAccountと紐付けられたPodは与えられた権限の範囲内でKubernetesの操作が可能

### 7.2.1 RBACを利用して権限制御を実現する
RBACでの権限制御は以下の2つで成立している
- KubernetesAPIのどの操作が可能であるかを定義したロール
- 認証ユーザー・グループ・ServiceAccountとロールのヒモ付け

### 7.2.2 ServiceAccount
クラスタ内で実行されているPodからクラスタ内の他のリソースへのアクセスを制御するためのリソース
- Kubernetesクラスタ内でほかリソースに対して何らかの操作を施すBot入のPod作成などにも使える

## 7.3 Helm
HelmはChartsを管理するツール、Chartsは設定済みのKubernetesリソースのパッケージ
- 実際の開発ではローカルや開発・本番環境のクラスタを問わず、複数環境にデプロイする用途のアプリケーションであればChartsでパッケージングし、kubectlではなくHelmでデプロイやアップデートを完結させる
- kubectlはデプロイされたリソースの運用上の修正で引き続き利用する

### 7.3.2 Helmの概念
クライアントとサーバーで構成される  
- クライアントはサーバーに命令するのみ
- サーバーはKubernetesクラスタに対して、パッケージのインストールや更新、削除の処理をKubernetesクラスタ上で行う

マニフェストを構築するためのテンプレート郡をパッケージとしてまとめたのがChart  
ChartはHelmリポジトリにtgzファイルとして格納され、Tillerがマニフェストを構築するのに利用される  

Helmリポジトリ
- local
- stable
    - 安定した品質のもの
- incubator
    - チャレンジングな設定

## 7.4 Kubernetesにおけるデプロイ戦略
DockerContainerでのデプロイは書くサーバーがDockerイメージを取得するぷる型のデプロイであるため、デプロイやスケールアウトは容易  
デプロイの基礎的な部分がDockerとContainerオーケストレーションに含まれているため、これまでのような手間を変える必要がなくなってきた  

### 7.4.1 RollingUpdate
Deploymentでは新しいポッドに置き換えるための戦略を .spec.strategy.type で指定する(RollingUpdate or Recreate)  
- RollingUpdateは古いバージョンのアプリケーションを実行した状態で新しいバージョンのアプリケーションを起動し、準備ができたものから順にサービスインする仕組み

#### RollingUpdateの挙動を制御する
maxUnavailable
- Rollingupdateの際に同時に削除できるPodの最大数
- 割合(%)で指定することも可能

maxSurge
- Rollingupdate実施時に新しいPodを作る個数
- defaultはreplicasの25%
- 増やせばすぐに切り替えできて良いが、瞬間的に必要なリソースは増える

### 7.4.2 Container実行時のヘルスチェックを設定する
アプリケーションによっては起動が完了していてもクライアントからリクエストを受けられる状態になるまでに多少の時間を要することもある
- PodがRunningになっていても正しいレスポンスが返せないことになる

この問題を避けるために以下が存在する
- livenessProbe
    - アプリケーションの死活チェック
    - Container内でアプリケーションが依存しているファイルや実行ファイルといったものの存在をチェックする用途
- readinessProbe
    - Containerの外からHTTPリクエスト等のトラフィックを受けられる状態になっているかのチェックを設定

### 7.4.3 BlueGreen Deployment
RollingUpdateは強力な仕組みだが、新旧Podが混在する瞬間が生じてしまう  
- これを解決する仕組みの一つとしてBlueGreen Deploymentが存在する
    - 新旧バージョンそれぞれ2系統のサーバ郡を切り替えてデプロイする方法

# 8章: Containerの運用
## 8.1 ロギングの運用
### 8.1.1 Containerにおけるロギング
従来型ロギング
- ログがどんどんたまり、ログローテートが必須

Docker
- ファイルではなく標準出力に出し、Fluentdなどのログコレクタで収集することが多い
- Container内の標準出力が自動的にjsonファイルになっているので、ログ出力を完全にDockerに任せることも可能
- logging driverのおかげで出力できている(json以外のフォーマットも選択可能)
    - json
    - syslog
    - journald
    - awslogs
    - gcplogs
    - fluentd

### 8.1.2 Containerログの運用
Container内でログを出力する方法は、障害で予期せずContainerが止まった場合にログごと消えてしまうので良くない  
Containerのログローテートをするために --max-size や --max-file を指定できる

### 8.1.4 fluentd logging driverの運用イメージ
#### fluentdは各ホストへ配置し、常に健康な状態に保つ
fluentdを各Dockerホストのエージェント的に配置し、Docker logging driverで利用すると良い
- 分散型のメリット
    - ホスト障害で影響を受ける範囲を限定できる
- 集約型のデメリット
    - 冗長化を手厚くする必要がある
    - ホスト数やログの数に見合ったfluentdを用意するのは大変

fluentd監視で中止すべき点
- buffer_queue_length: バッファに保持されているchunkの数
- buffet_total_queued_size: バッファに保持されているchunkの数
- retry_count: リトライの回数

### 8.1.5 Kubernetesにおけるログの管理
- 基本的にはDocker composeと同じ
- どのNodeにどうPodが配置されるかはKubernetesのスケジューラ次第なので、それぞれのコンテナで独自にログを管理するのは難しい
- Elasticsearch/Kibanaはkube-systemというNamespaceに構築すると良い
    - kube-systemはコアコンポーネントが含まれており、ログを横断的に集約するのにはkube-systemに配置するのが便利
- DaemonSetでfluentdを構築する
  - DaemonSetはKubernetesクラスタで管理されている全てのNodeに対して、必ず1つ配置されるPodを管理するためのリソース
  - fluentdのDaemonSetもElasticsearch, Kibana同様にkube-systemに配置される
- sternを使うと普通にログを見ることもできる

#### Docker/Kubernetesでのロギングの王道
- アプリケーションのロギングはファイル出力ではなく、全て標準出力する
- nginx等のミドルウェアのログも全て標準出力できるようにDockerイメージを構築する
- 標準出力するログは全てJSON形式で出力し、それぞれの属性で検索や集計をしやすくする
- Kubernetesにおいてはfluent/fluentd-kuberbetes-daemonsetで構成されるPodをDaemonSetで各ホストに配置する
- Kubernetesのリソースにはラベルを適切に設定することで、ログの検索性を確保する

## 8.2 Dockerホストやデーモンの運用
### 8.2.2 dockerdのチューニング
Linux系OSでは/etc/docker/daemon.jsonに記述し、再起動することで反映できる
- max-concurrent-downloads
  - docker image pullにおけるイメージダウンロードの並列数
  - デフォルトは3
- max-concurrent-uploads
  - docker image push におけるイメージダウンロードの並列数
  - デフォルトは5

## 8.3 障害対応
### Docker運用での障害対策
- イメージの運用起因での障害対応
    - 本番環境でlatestでコンテナを実行しており、コンテナオーケストレーションによるコンテナ再起動で最新イメージになり実行される
    - latest以外のバージョン名でタグ付けされているイメージを上書きしてしまった
    - 別のタグをつけて上書きしてしまった
- イメージのテスト
    - イメージのテストはcontainer-structure-testがよく使われる
      - Googleが出しているDockerイメージに対するテストフレームワーク
- ディスク容量の枯渇に注意
  - docker system prune -a で利用していないイメージやコンテナを一括削除できるのでcronに仕込んでおくと良い

### Kubernetes運用での障害対策
- Node障害時のKubernetesの挙動
  - Podをデプロイする際にReplicaSetが自動で配置してくれるのでNodeを気にする必要がなさそうに見えるがそうではない
  - Nodeがダウンした際にPodがどうなるか
    - 正常に可動している別のNodeに再配置される(Auto-Healing機能)
    - ReplicaSetを管理するDeploymentやStatefulSetを利用してPodを作成することが最初の障害対策になる
- Pod Anti Affinityによる耐障害性の強いPod配置戦略
  - Auto-Healingは便利だが、replicas=1のときなどは再配置までの間がダウンタイムとなる
  - Podを複数のNodeに配置できるようにreplicasの数を調整する
    - replicas=2にしても同じNodeに乗ってしまう可能性はある
    - これを解決する仕組みがPod AntiAffinity
      - Deploymentの定義でspec.affinity.podAntiAffinityを設定する
      - replicas=3でPodが2つの場合、1つのpodはPend状態になり配置されない

### CPUを多く利用するPodをNodeAffinityで隔離する
- CPUを多く利用するPodをNodeAffinityで隔離する
  - CPUリソースを多く必要とするPodだけ他のNodeに隔離することで他のPodのパフォーマンスに影響を与えないようにする
  - Deploymentのspec.affinity.nodeAffinityでPodをどのNodeに配置するか指定できる
    - 例えばBatchのPodにinstancegroup: batchというラベルをつけておき、NodeのDeployment側でbatchがついているものだけ配置させることができる

### Horizontal Pod Autoscaler(HPA)を利用したPodのオートスケール
HPAはPodのシステム使用率に応じて、Pod数を自動で増減させるKubernetesリソース
```yml
resouce:
  name: cpu
  targetAverageUtilization: 40
```
のような設定でcpuリソースが40%を上回ったら自動で新しいPodが作成される。ただし、maxReplicasが設定されていた場合それを超えない。

### Cluster Autoscalerを利用したNodeのオートスケール
- HPAはPodをオートスケールする仕組みだが、PodをスケールしようにもNodeがない場合がある
- Cluster AutoscalerはGCP/AWS/Azureそれぞれに対応している
- 最大/最小node数も指定可能

### Helmのリリース履歴を制限する
Helmでインストール・アップデートを繰り返すとその分だけConfigMapが残り、レスポンスが遅くなったり、デプロイ不可能になる
- helm initでTillerをデプロイする際にリリース当たりの最大履歴保持数を設定する--history-maxでコントロールする

# 9章: より軽量なDockerイメージを作る
## 9.1 なぜ軽量なイメージを作るべきか
イメージサイズの増大で発生する弊害
- イメージのビルド時間
- イメージをDocker Registryにプッシュする時間
- コンテナを実行したいホスト・ノードへのイメージダウンロード時間

上記は以下の課題を発生させる
- Kubernetes等のコンテナクラスタを構成するNodeのディスクの消費
- CI時間の増大
- トライアンドエラーのしにくさ、生産性の低下
- オートスケールでコンテナがサービスinされるまでの時間が長くなる(Nodeにイメージが存在しない場合に新たにダウンロードするため)

## 9.2 軽量なベースイメージ
scratch
- scratchは空のDockerイメージで特殊なもの
  - scratchにバイナリファイルだけを置くということもできる
  - 現実的な選択肢としては難しいことが多い

BusyBox
- BusyBoxは組み込み系システムで多く利用されるLinuxディストリビューションで、非常に小さいOS
  - たった1.13MB

AlpineLinux
- AlpineLinuxはBusyBoxをベースに作られたディストリビューション
  - イメージサイズは4MB弱
  - デファクトスタンダード
    - apkが使える点が強み

## 9.3 軽量なDockerイメージを作る
デプロイするアプリケーションのサイズを削減する
- アプリケーションのサイズをチューニングする
  - 不要なファイルや依存ライブラリ、assets(特に画像)の削除
- .dockerignore
  - Dockerfileと同じディレクトリに作る
  - コンテナに含めないファイルやディレクトリを定義できる

レイヤーを減らす
- コマンドの数だけレイヤー(=イメージ)も増えるので、コマンドは && でつなぐと一気にサイズが小さくなる
- 可読性とのトレードオフ

## 9.4 multi-stage builds
ビルドコンテナと実行コンテナを分ける
- 例えば、goは700MBもあるが、ビルド後のバイナリさえあれば実行はできる
- FROMを2回使うことで一気に軽量化することができる

## 言語にフォーカスしたdistrolessイメージ
distrolessはOSを含まずに言語にフォーカスしたDockerイメージで、Googleによって公開されている
- glibcをベースとしている
- 16MBでAlpineLinuxほどではないが十分な軽さ

# 10章 Dockerの様々な活用方法
DockerはVagrantの代替となるか
- 筆者はVagrantの代替としてDockerを使うべきとは考えていない
- Dockerは完全仮想化ではない
- Dockerで運用していないアプリケーションはOSの状態に依存していることがほとんど
  - Dockerは完全なLinuxではないので、何かしら期待をした挙動をしないことが考えられる
- Vagrant -> Dockerへ載せ替えるならば、本番環境までもDockerへ移行する前提で検証/テストした方が良い  

負荷テスト
- LOCUSTを使うと簡単に負荷テストができる

# Appendix-A セキュリティ
## 公開Dockerイメージの安全性
DockerHubで見るべきポイント
- pull数やstar数は信用できない
- OFFICIALに[OK]がついているか

Quay.io
- DockerHubとは別のレジストリサービス
- アナリティクスが充実している
- 無料でも十分使える

## 安全なDockerイメージと運用体制を作る
### DockerBenchForSecurity
- Dockerコンテナのセキュリティリスク発見に役に立つ
    - docker/docker-bench-securityというイメージで使える

### コンテナへのファイル追加におけるリスク
- Dockerfileでのコンテナにファイルを追加する命令としてCOPYとADDがある
    - COPYはシンプルなコピー
    - ADDは指定したURLのファイルをダウンロードして追加できたりする
        - 指定したURLのファイルがなかったり、悪意のあるものに置き換えられた場合に問題が発生する
    - wgetで外部ファイルをDLする場合もADDと同じリスクがある
- Terraformには実行ファイルを含み時pファイルと共にSHA256のチェックサム情報が提供されているので、確認することが可能

### 適切なアクセス制御
- /var/run/docker.sock をコンテナに共有しない
    - docker.sockを使えると、自由にコンテナを作成できたりする
- コンテナ内のアプリケーション実行ユーザーを用意する
    - dockerコンテナ内の実行ユーザーはデフォルトでroot

### クレデンシャルの扱い
- 環境変数をもたせる方法
    - human readableなので、少し危険だが、よく使われる方法
- クレデンシャルを外部から取得する
    - container内部から外部のS3などにアクセスさせる方法
    - 適切に権限を設定すればセキュアな方法

## Docker開発を支援するツール
### 独自のDokcerレジストリの構築
Docker HubやQuay.ioのようなpublicレジストリサービスが利用できない場合がある

- Registry(Docker Distribution)
    - プライベートレジストリ構築ツール
    - library/registry

### DockerとCI/CDサービスの連携






















# Telepresenceについて
単一のサービスをリモートのKubernetesクラスタに接続しながら、そのサービスをローカルに実行できるオープンソースのツール  
これにより、開発者はマルチサービスアプリケーションで作業することができる  

- もしクラスター内の別のサービスに依存していたとしても、迅速にローカル開発ができる。あなたのServiceを変更して保存したら、すぐに変更を確認できる
- ローカルのツールを利用できる(例えば、debuggerやIDE)
- ローカル開発環境をKubernetesクラスタの一部のように使える

## どうやって動いているのか
TelepresenceはKubernetesクラスターで動いているPodに総方向のネットワークプロキシを配置する  
- このPodはKubernetes環境からのデータ(TCP接続、環境変数、ボリューム)をローカルプロセスにプロキシする
- ローカルプロセスはDNS呼び出しとTCPコネクションがプロキシ経由でKubernetesクラスタにルーティングされるように透過的にオーバーライドされる

つまり、ローカルのServiceはリモートクラスタ内の他のサービスにフルアクセスできるて、環境変数や秘密情報、設定にアクセスできる  
また、リモートサービスもあなたのローカルサービスにフルアクセスできるようになる

## telepresenceのゴール
- 透明性：ローカルプロキシプロセスをKubernetes環境にできるだけ一致させる。
- 分離：プロキシプロセスだけが環境を変更します。この目標はコンテナのそれとよく似ています。それは独立したプロセス固有の環境です。
- クロスプラットフォーム：可能であれば、LinuxとmacOSは同じように機能します。 LinuxはmacOSよりはるかに多くの機能（マウントネームスペース、バインドマウント、ネットワークネームスペース）を提供します。
- 互換性：任意のプログラムで動作します。
