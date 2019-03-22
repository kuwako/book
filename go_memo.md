# Goの理解しきっていないところを調べる
## ポインタ
pはポインタ型なので&で渡す必要がある
```
var p *Person

p = &Person{
  Name: "太郎",
  Age:  20,
}
```
pにはアドレスが入っているので0x1234abcd的な値になる  
pがポインタ型変数で \*Person がポインタ型  

**ポインタ→実体は暗黙で置換されるが、実体→ポインタは無理なので注意**

### デリファレンス
& を使うことで、ポインタ型を生成することができる  
- Person型の変数pを &p とすると、Personへのポインタである \*Person型 の値を生み出すことができる
- &p は、pのアドレスという。

変数名の前に \* をつけることで実体を取得できる
- 紛らわしいが、\*p自体も変数なのでこれに代入することも可能

## インターフェース
Jack氏: "Accept interfaces, return structs"
- パッケージでは処理する側でinterfaceを定義し引数として受け取り、New関数の戻り値など外部にわたすものはstructのように具体的な型にすべき
- オブジェクト指向ではカプセル化だが、Goのインターフェースは実装を隠すのでなく実装に依存しない
- https://qiita.com/weloan/items/de3b1bcabd329ec61709

### 意図せずinterfaceのポインタを使わないように気をつける
ついなんでもpointerでやってしまいがちだが、interfaceのpointerを使おうとすると不便なことになる
- 例) [[教えて]Go言語:なぜインターフェイスはポインタにできない?](https://qiita.com/suin/items/68ed7020d21dca047a73)
- https://eel3.hatenablog.com/entry/20140915/1410788174
- http://otiai10.hatenablog.com/entry/2014/05/27/223556


## 構造体
a := new(Animal) は a := &Animal{} と同じ  

ただし、&Animal{}は一行で初期化できるが new(Animal)はできない  
- a1 := &Animal{"hoge", 5} => ok  
- a2 := new(Animal{"hoge", 5}) => error (Animal literal is not a type)


### 初期化
- a := Animal{"hoge", 5} => ok  
- a := Animal{"hoge"} => error  
- a := Animal{Name: "hoge", Age: 5} => ok  
- a := Animal{Name: "hoge"} => ok (Ageは0で初期化される)  

## 配列/スライス

## golint
エクスポート(phpでいうpublic)されたメソッドや変数、型には必ずその名前から始まる説明コメントを書かなければならない  
Get: abc みたいなやつもダメ  
```
// Get is hogehoge
func Get() 
```

## ポインタの暗黙変換について
- Go の関数レシーバは値型とポインタ型の間で暗黙の型変換が行える場合と行えない場合がある
- 暗黙的型変換を使うメリットはあまりないので、明示的に型を指定して関数定義をしていったほうが安全
- Go の関数のレシーバは呼び出し元がアドレス指定可能であれば、値型とポインタ型の間で暗黙の型変換が行われ、簡略な記述で呼び出すことができる
- 型変換が行われないのは、主に呼び出し元が値型でマップやインターフェイスの要素であった場合
- 暗黙的型変換によって関数呼び出しが多少便利になる代わりに、呼び出しできないケースについて気をつけながらコードを書いていくのは割に合わない
- 実体がポインタレシーバのメソッドfunc (ap \*a) hogeHoge ()を呼び出すときって
    - a.hogeHoge() は暗黙変換されて、実際は
    - (&a).hogeHoge() を実行している

[詳細](https://qiita.com/nirasan/items/02e88c3ba64c444fa527)

## Goの同じディレクトリの中に別のパッケージは入れられない (\_test.goを除く)
https://stackoverflow.com/questions/14416275/error-cant-load-package-package-my-prog-found-packages-my-prog-and-main
- それぞれのpackageは各自のディレクトリで定義されなければならない
- パッケージは、公開、インポート、URLからの取得など、複数のプログラムで使用できるコンポーネントです。したがって、プログラムがディレクトリを持つことができるのと同じくらい独自のディレクトリを持つことは理にかなっています。 
