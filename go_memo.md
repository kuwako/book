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

