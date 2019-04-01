# みんなのGo
## 1. Goによるチーム開発のはじめ方とコードを書く上での心得
### 1.4 Goらしいコードを書く 
- panicを使わない
    - ちゃんとエラーを書け
- 正規表現を避ける
    - Goの標準パッケージのregexpはパフォーマンスが悪い
    - stringsパッケージをうまく使うと良い
    - どうしても使う場合は初期化時に使う(varやinit時)
- mapを避ける
    - 可能な限りstructでtypeを定義する
    - mapが本来の用途で必要な場合、mapに対する操作はスレッドセーフでないことに気をつける
        - goroutineからの同時アクセスは変な値を読み込んだりする可能性がある
        - RW Mutexパッケージを使うのが定石
- reflectを避ける
- 巨大なstructを作らない、継承させようとしない
    - 継承より委譲
- 並行処理を使いすぎない
    - 追いづらい

## 2. マルチプラットフォームで動作する社内ツールの作り方
### goの利点
- 出来上がったバイナリを配布すればいい

### path/filepathを使う
- windowsでよくあるパスの問題を解決してくれる

### deferを使う
呼び出したスコープを抜ける際に、呼び出された順番とは逆の順番で実行される

### 2.4 OS固有の処理
- runtime.GOOSを使う
    - runtime.GOOS == "windows" 的な
- Build Constraintsを使う

### 2.5 シングルバイナリにこだわる
#### シングルバイナリ
シングルバイナリはgoの強みだが、画像ファイル等、依存が増えてしまうと辛い  
それを解決するのが go-bindata というパッケージやgo-assetsを使うと便利  

## 3.実用的なアプリケーションを作るために
### 3.1 実用的なアプリケーションとは
- どのような機能を持っているか容易に調べられること
- パフォーマンスが良いこと
- 多様な入出力ができること
- 人間にとって扱いやすい形式であること
- 想定外の場合に安全に処理を停止できること

### 3.2 バージョン管理
- バージョン番号をバイナリに埋め込む
    - 標準のflagパッケージを使う

### 3.3 効率的なI/O処理
- 標準パッケージのbufioを使う
    - Goでは自動的なバッファリングは行われない

### 3.4 乱数を使う
2種類の標準パッケージがある
- math/rand: 疑似乱数
- crypto/rand: 暗号論的擬似乱数を生成する

### 3.5 人間が扱いやすい形式の数値
- go-humanizeパッケージを使う

### 3.6 外部コマンドを実行する
外部コマンドは os/exec パッケージで実行可能

### 3.7 タイムアウトする
自前で作る場合はgoroutineする

### 3.8 シグナルを扱う
OSが外部からプロセスに割り込みを与えるための機構にシグナルがある。詳細はos/signalのドキュメント
- SIGTERM: プロセスを終了させる
- SIGHUP: ユーザーがプロセスに対して設定ファイルの再読込をさせる
- SIGINT: ユーザーにctrl + C で止められた場合

シグナルを取り扱うには os/signal パッケージのos.Signal型の値を取り扱う channel を作成してsignal.Notifyに与える

### 3.9 goroutineの停止
goroutineの起動はgo doSomething()としてできる  
goroutineに停止にはチャンネルを使用 or contextパッケージ(ver1.7~)がある

#### チャンネルを使用する方法
チャンネルを送信側でcloseするとそのチャンネルから受信した場合に第二の値がfalseになるので、それを見てreturnするようにする

#### contextパッケージを使用する方法
context.withChannelで返されるcontextをgoroutineに渡し、閉じたいタイミングでcancel()を実行する

## 4. コマンドラインツールを作る
### なぜGoを使うのか
- 配布が簡単
- パフォーマンスが良い

### 4.3 flagパッケージ
flagパッケージを使えば、簡単にCLIツール実行時のオプション引数を実装できる
- 例) var port = flag.Int("port", defaultPort, "use")

## 5. The Dark Arts Of Reflection
### 5.1 動的な型の判別
ユーザーから与えられた任意の値をバリデートする関数があった場合、すべての値に対応せざるを得ない場合もある
```
func validation(x interface{}) error {
    ...
}
```

また、interface{}を受け取り、\*os.Fileに変換するような関数で変換に失敗した場合などはこうする
```
func HundleData(x interface{}) {
    f, ok = x.(*os.File)
    if !ok {
        ...
    }
    ...
}
```

型アサーションを使って型ごとのswitch文も書ける
```
func HundleData(x interface{}) {
    switch x.(type) {
        case int:
            ...
        case string:
            ...
        default:
            ...
    } 
}
```

ただし、この方法には欠点がある
- 型アサーションではアサーションする型名を事前に知っておく必要があるので選択肢が限定されている必要がある
- アサーションに利用する型は完全な型でないといけない

この場合、Goで扱われる値・型の構造を動的に調べたり操作できるreflectパッケージを利用する

### 5.2 reflectパッケージ
rv := reflect.ValueOf(p) のような使い方で型情報や格納されている値などを得ることができる

#### reflect.Value
reflectを使って行う操作の基本はすべてreflect.Value型を通して行われる
- reflect.ValueOfはGoで利用できるすべての値に対応するメソッドを備えているが、実行時にそれが不正な呼び出しかどうかは呼び出し側で確認する必要がある

## 6. test
### 6.1 Goにおけるテストのあり方
goのtestへの戦略は「明示」、そして「シンプル」

### 6.2 testingパッケージ
- ファイル名はのサフィックスは \_test.go である必要がある
- Testで始まる関数がテストされる
- - や . で始まるファイルは無視される
- go test -run TestXXX で指定してテストを実行できる

#### Testable Example
- Examplesは実行例をそのままテストコードとして記述する機能
- Exampleから始まる名前で定義し、出力を // Output: から始まるコメントで書くことで標準出力の内容をテストできる

#### Unordered output
- Unordered output(v1.7~)を使うと順不同な結果に対してもマッチできる

### 6.3 ベンチマーク入門
- ベンチマーク用の手続きはBenchmarkXXX という関数名で定義する
- go test -bench で実行可能

### 6.4 テストの実践的なテクニック
- TableDrivenTests
- reflect.DeepEqualを使う
    - 大きい配列などを比較する際にすべてforで回すのは大変なのでreflext.DeepEqualを使うと良い
- Race Detectorを使って競合状態を検出する
    - 複数のgoroutineから同じ変数にアクセスしていて、少なくともどれか一つに書き込みをした場合に発生する競合を検出できる
    - test以外でも使える
        - go run -race mypkg
        - go build -race mypkg
    - Race Detectorは実行されている最中に競合状態が検出された場合のみData Raceを検出する
    - Race Detectorを有効にするとメモリ使用量は5 ~ 10倍、実行時間は2 ~ 20倍になる
- TestMainによるテストの制御
    - testでDB Insertが生じる場合に、先にデータを用意し、実行が終わったらデータを戻すというような作業ができる
    - setup()で何かしらの前準備、shutdown()で終了後の処理ができる
- Build Constraintsを利用したテストの切り替え
    - インテグレーションテストをする際にはBuild Constraintsを利用して単体テスト特別すると便利
    - ファイルの先頭に // +build を宣言すると使える
    - 特別にテストのために使われる機能ではない
        - インテグレーションテスtの対象としたいファイルについて // +build integration とファイルの先頭に書くと go test -tags=integration と実行できる
- テストにおける変数または手続きの置き換え
    - testの関数内で処理を置き換える
- インターフェースを使ったMock
- net/http/httptest パッケージ
    - HTTPに関するテストをする際に便利なパッケージ
    - 特定のアドレスとポートをlistenさせることなくwebアプリケーションのテストができる
        - ローカルのループでバッグデバイスで動作するテスト用のサーバを建てられる
- テストカバレッジ
    - go test -cover / go tool cover -func などが使える



