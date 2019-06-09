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
