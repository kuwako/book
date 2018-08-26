# Chapter1: ツールの導入
## ビットコインを扱うソフトウェア
ビットコインの実装はC++で開発されたがその後他の言語でも実装されている
- Bitcoin Core: C++
- NBitcoin: C#
- bitcoinj: Java
- BitcoinJS: JavaScript
- btcd: Go
- bitcoin-ruby: Ruby
- python-bitocinlib: Python

## Bitcoin Core
Bitcoin Coreはビットコインの公式クライアントソフトウェア

#### インストール方法
パッケージファイルからインストール
    - https://bitcoin.org/ja/download

### BitcoinCoreでネットワークに接続
- ビットコインの参照実装であるBitcoinCoreで実装されているビットコインのノードbitcoindを起動する
- ビットコインネットワーク
    - 実際に送金できるmainnet
    - 開発や実験のためのtestnet
        - ビットコインアドレスや秘密鍵の形式が少し異なっている
        - 標準のトランザクションのチェック等で制限が緩められている
    - 他のランダムなノードやブロックと連携しないregtest
        - 任意のタイミングで新しいブロックを追加することができる

### Bitcoin Coreのディレクトリ
BitcoinCoreの設定ファイルや、ブロックチェーンの実態が格納されるDBなどはデフォルトでいか存在する
- MacOSX: ~/Library/Application\ Support/Bitcoin
- Linux: ~/.bitcoin

#### bitcoin.confファイルの設定方法
bitcoin coreの詳細の設定ははbitcoind -h を使って見ることができる
```
testnet=3
txindex=1
server=1
rpcuser=ユーザー名
rpcpassword=パスワード
rpcport=18322
```

- mainnetに接続する場合のデフォルトのポート番号は8332、testnetに接続する場合は184332
- txindexはブロクチェーンのデータベースを作成するときに任意のトランザクションを検索可能にするためにインデックスを作成するというオプション
    - OpenAssetsProtocolを利用する際に必要

### bitcoindの起動と停止
- GUIを利用する場合は bitcoin-qt & 
- デーモンだけで良い場合は bitcoind -daemon
- regtestモードで起動する場合は -regtestオプションをつけて起動

### bitcoin-cliコマンドを使ってbitcoindと対話する
- bitcoin-cli getinfo でjson形式で返ってくる

### RESTインターフェースを使ってbitcoindと対話する
bitcoindにはhttpdとして外部と対話するためのRESTfulなwebAPIの機能が備わっていて以下の方法がある
- bitcoindの設定ファイルでrest=1を指定する
- bitcoind -restで起動する

ただし、このインターフェースを利用しているとXSSを受ける脆弱性がある  
bitcoindのREST APIに関する詳細は https://github.com/bitcin/bitcoin/bolb/master/doc/RESTinterface.md を参照

### ビットコイン・ネットワークのノードとの接続
- bitcoindコマンドなどでビットコインのノードが起動すると、ノードはP2P型ネットワークであるビットコイン・ネットワークと接続しようとする
- Bitcoin CoreのソースコードにはDNSシードと呼ばれる5つのドメイン名が埋め込まれている
- 接続が安定すると8ノード程度と接続された状態になる
    - bitcoin-cli getpeerinfo コマンドで接続している相手を確認することができる

### regtestもー0度でのブロック生成
- regtestモードではマイニングの代わりにgenerateコマンドを実行することで任意のタイミングでブロックを生成することができる
     - bitcoin-cli -regtest generate <作成するブロック数>

## bitcoin-ruby
### インストール方法
- gemを使う
    - sudo gem install bitcoin-ruby

## openassets-ruby
- openassets-rubyはビットコインのオーバーレイプロトコルであるOpenAssetsProtocolのRuby版実装
- OpenAssetsProtocolではビットコインのブロックチェーン上でビットコイン以外の任意のアセットを発行・取引することができる

### インストール方法
- sudo gem install openassets-ruby


