# gRPC
2019年4月のWEB+DB

## gRPCとはなにか
### gRPC
- gRPCを導入することで様々な言語で書かれたクライアントアプリケーションから、自動生成共通のインターフェースを介して、別のマシンのサーバアプリケーションへと通信手段を意識せずに接続できる
- Googleが自社内で使うために開発されたStubbyという汎用RPC技術を10年以上使ってきていた
- HTTP/2やQUICの登場により、Stubbyの持つ機能の多くが標準化されたため、gRPCとして作り直した

### gRPCの特徴
- Protocol Buffersによる強い型付けのインターフェース
  - 定義したメソッドの引数や戻り値の型、やりとりするデータ及びそのフィールドが一目瞭然
  - 様々な言語のコードを自動生成できる
- HTTP/2によるサービス間の柔軟な通信
  - 基本的な構造はHTTP/1.xと同じ
  - 特徴はストリーム
    - 同一のTCP接続においてクライアントとサーバ間で双方向にデータをやり取りできる仕組み
    - 単一のTCP接続の中で複数のストリームを生成し、リクエストとレスポンスを並列で処理できる
- 様々な言語およびプラットフォームで使用可能
  - WindowsでもMacでもLinuxでも、JavaでもGoでも動く

### gRPCの用途
- マイクロサービス間の通信
- モバイルアプリとサーバ間の通信
  - モバイルとサーバ間の通信効率化を期待できる
- ブラウザとサーバ間の通信
  - 2018年10月にgrpc-webが正式にリリースされた
  - 特別なプロキシ(デフォルトはenvoy)を使用して、バックエンドのgRPCサーバへの通信手段を意識せずに接続できる

### gRPCとRESTの比較
- 設計モデル
  - REST
    - リソース指向
  - gRPC
    - 通信相手に操作を要求する手法
    - メソッドやサービスを定義
- データ形式
  - REST: JSON
    - JSONと決まっているわけではないが、デファクトスタンダード
    - サーバとクライントの両方でJSONをパースする必要がある
  - gRPC: バイナリ
    - シームレスに通信可能
- 通信方式
  - REST: リクエスト/レスポンス
    - 1リクエスト/1レスポンス
  - gRPC: 様々なストリーミング
    - HTTP/2の機能を活かした双方向通信が可能

## Protocol BUffersの基礎知識
proto3の情報  

### メッセージの定義
通常のメッセージ
```
message User {
  int32 id = 1;
  string name = 2;
  Setting setting = 3;
}
message Setting {
  ...
}
```
のように型 変数名 = {フィールド番号} の順で振る  

入れ子になったメッセージ
```
message FindTaskResponse {
  message Task {
    int64 id = 1;
    string name = 2;
  }
  Task task = 1;
}

message UpdateTaskRequest {
  FindTasksResponse.Task task = 1;
}
```

#### フィールド番号
- フィールド番号とエンコードサイズ
  - 1 ~ 15までは型と番号を合わせて1バイトで表現できるので、頻繁に使用するフィールドは小さいフィールド番号を振ると良い
- フィールド番号と名前の予約ができる
  - reservedを使える

### メッセージのフィールド型
- スカラ
  - int32とかstringとか
- Enum
  - Enumの値はint32の範囲内
- Repeated
  - 配列, 順序は保存される
- Any
  - 型を明示せずに任意のメッセージを格納する
  - google.protobuf.Any を使う
- Oneof
  - 最大一つのフィールドだけを設定する
    - メッセージに複数のフィールドがあり、同時に最大１つのフィールドしか設定しない場合に使用
```
message Activity {
  unit64 id = 1;
  oneof content {
    CreateTask create_task = 2;
    UpdateTask update_task = 3;
  }
}
```
- Map
  - Mapに対してrepeatedは使用できない
  - Map内のデータの順序は保持されない
  - ``` <string, Project> project = 3; ```

### RPC
サービス内に定義するメソッド
- 引数や戻り値がないRPCを定義する場合、空のメッセージ型としてgoogle.protobuf.Emptyを宣言する

### Empty
引数や戻り値がないRPCを定義する場合、空のメッセージとしてgoogle.protobuf.Emptyを宣言する

## gRPCの基礎知識
Modules
- Go1.11から提供された依存ライブラリを管理する仕組み

### 開発フロー
インタフェースを定義してコードを生成する
- メッセージとサービスを定義する
- コードを定義する
  - .protoファイルに定義したメッセージやサービスをprotocコマンドでコンパイルして、Go用のサーバインタフェースとRPCを実行するためのクライアントであるスタブを生成する
- インタフェースを元にサーバを実装する

### gRPCがサポートする4種類の通信方式
- 単項RPC
  - 1req / 1res
- サーバサイドストリーミングRPC
  - 1req / 複数res
  - 戻り値にstreamを付与すると複数のレスポンスをreturnできる
- クライアントサイドストリーミングRPC
  - 複数req / 1res
- 双方向ストリーミングRPC
  - 複数req / 複数res

## gRPCの実践的な機能
### gRPCのエラー
- ステータスコード
    - 問題なければ0を返し、その他の場合はエラー
- エラーハンドリング
    - status.FromError関数にerrorを渡すとStatusに変換されて、st.Code()やst.Message()で解析できる
- タイムアウト
    - withDeadline（）関数で設定可能

### gRPCのメタデータ
- クラインアント
  - 送信方法
    - metadata.AppendToOutgoingContext()を使用
  - 受信方法
    - ヘッダかトレーラとして受信可能
    - 単項RPCならCallOptionとして受信用の変数を渡す
    - ストリーミングの場合はstreamがもつHeader()とTrailer()で取得できる
- サーバー側
  - 受信方法
    - metadata.AppendToOutgoingContext()にcontextを渡すと取得できる
  - 送信方法
    - SendHandler(): ヘッダとして送信
    - SetHandler(): トレーラとして送信

### gRPCのインタセプタ
インタセプタはクライアントとサーバでRPCの実行前後に任意の処理を挟む仕組み
- 単項RPCに適用するインタセプタ
  - クライアント: UnaryClientInterceptor
  - サーバ: UnaryServerInterceptor
- ストリーミングRPCに適用するインタセプタ
  - クライアント側: StreamClientInterceptor
  - サーバ側: StreamServerInterceptor
  - どちらもストリーミングRPCが呼び出されるタイミングで一度だけ実行される

## タスク管理マイクロサービスの設計
- 各サービスのインタフェースを定義する
- 各サービスのコードを生成する
  - ``` protc -I=proto --go_out=plugins=grpc,paths=source_relative:./proto prot/activity/acitivity.proto ```
