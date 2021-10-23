---
marp: true
paginate: true
footer: TRPL読書会 #8
---
# 15. スマートポインタ（前半）

- Kenta Arai
---
# Copyright

- The Rust trademark is owned and protected by The Rust Foundation
- [The Rust Programming Language](https://github.com/rust-lang/book) is licensed under [Apatch 2.0 and MIT terms](https://github.com/rust-lang/book/blob/main/COPYRIGHT)
- This slide is a derivative work of TRPL, and is licensed under Apatch 2.0 and MIT terms
---
# スマートポインタとRust

- スマートポインタは，ポインタとして振る舞うだけでなく，ライフタイムと所有権も管理する
    - 今までに学んできた（実は）スマートポインタな型：`String`，`Vec<T>`など
- Rustのスマートポインタは通常，`Deref`と`Drop`トレイトを実装する構造体のメモリを指す
    - `Deref`トレイト：参照外し`*`でオブジェクトを参照できるようにする
    - `Drop`トレイト：スコープから外れたらメモリを解放できるようにする
- 本章ではRustに実装されているいくつかのスマートポインタを取り扱う
    - `Box<T>`，`Rc<T>`，`RefCell<T>`，`RefMut<T>`

- 前半では，スマートポインタの概要と前述したトレイトについて述べる
---
# 15.1 ヒープのデータを指すBox<T>を使用する

- ヒープを使う目的は以下の通り
    - コンパイル時にサイズを決定不能なオブジェクトを扱いたい
    - 多量のデータの所有権をムーブしたい（⇔コピーしたくない）
    - 特定のトレイトを実装した型を所有したい（17章で詳説）
- 上記を達成するために，スマートポインタ`Box<T>`を題材としてメモリを動的に管理する
---
# Box<T>を使ってヒープにデータを格納する

- `i32`型をヒープに置く例
- `Box::new()`はヒープにメモリを確保し，引数を格納する
- スコープを抜けるタイミングでメモリを解放
    - この場合は`main`関数が終了するタイミング

```rs
fn main() {
    let b = Box::new(5);
    println!("b = {}", b);
}
```
---
# ボックスで再帰的な型を可能にする

- コンパイラはある型が占有する領域の大きさを把握する必要がある
- サイズが分からない型の例：再帰的な型
    - 値の一部に同じ型の他の値を持つ可能性がある
- `Box<T>`のサイズ（=ポインタ）はサイズが明らかなので，再帰的な型を作ることができる
---
# コンスリストについてもっと詳しく

- コンスリスト（`cons`，construct functionの略称）は2つの引数から新しいペアを構成する
    - 引数は通常，単独の値と別のペアからなる
    - 最後の要素は`Nil`のみを含む
- 今までの知識で実装しようとするとコンパイルエラーになる
    - 型のサイズが明らかでないため

```rs
// コンパイルできないコンスリストの実装
enum List {
    Cons(i32, List),
    Nil,
}
```
---
# Box<T>で既知のサイズの再帰的な型を得る

- 別のペアを`Box<T>`で指すことで，型のサイズがコンパイル時に決定する

```rs
// コンパイルできるコンスリストの実装
enum List {
    Cons(i32, Box<List>),
    Nil,
}

fn main() {
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
}
```
---
# 15.2 Derefトレイトでスマートポインタを普通の参照のように扱う

- `Deref`トレイトは，**参照外し演算子**の振る舞いをカスタマイズする
- スマートポインタは`Deref`を実装することで，今までの参照と同様に扱うことができる

- 本節では下記の内容を取り扱う
    - `Box<T>`に対して参照外し演算子を作用させる
    - 独自のスマートポインタに`Deref`を実装する
---
# 参照外し演算子で値までポインタを追いかける

- 参照は一種のポインタであるため，データを直接読み書きすることができる
- この内容は今までに学んだ通り
    - `x`の型は`i32`
    - `y`の型は`&i32`
    - `*y`の型は`i32`

```rs
fn main() {
    let x = 5;
    let y = &x;

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```
---
# Box<T>を参照のように使う

- `Box<T>`も参照のように使うことができる
    - `x`の型は`i32`
    - `y`の型は`Box<i32>`
    - `*y`の型は`i32`

```rs
fn main() {
    let x = 5;
    let y = Box::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```
---
# 独自のスマートポインタを定義する

- `new`関数：引数の値をヒープに置く
- このままでは値を参照できない

```rs
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

// let var = MyBox::new(12);
// println!("var: {}", *var); // まだ参照外しできない！
```
---
# Derefトレイトを実装して型を参照のように扱う

- `Deref`トレイトは標準ライブラリで提供されており，`deref()`の実装を要求する
- 参照外しをすると，コンパイラが`*`を`deref()`に解釈する
    - つまり`*x`は`*(x.deref())`として扱われる

```rs
use std::ops::Deref;

impl<T> Deref for MyBox<T> {
    type Target = T; // ※第19章で詳説

    fn deref(&self) -> &T {
        &self.0
    }
}

// let var = MyBox::new(12);
// println!("var: {}", *var); // deref()によって参照外しができる！
```
---
# 関数やメソッドで暗黙的な参照外し型強制

- 参照外し型強制:`Deref`を実装する型への参照を，元の型への参照に変換する機能
- 下記の例では，`&MyBox<String>`を`&String`に変換している

```rs
fn hello(name: &str) {
    println!("Hello, {}!", name);
}

fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&m);
}
```
---
# 暗黙的な参照外し型強制が無いと

- `*m`：`MyBox<String>`を`String`に参照外ししている
- `(*m)[..]`：文字列全体と等しい文字列スライスをとる

```rs
fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&(*m)[..]);
}
```
---
# 参照外し型強制が可変性と相互作用する方法

- コンパイラは下記のルールに則り参照外し型強制を行う
    - `T: Deref<Target=U>`->`&T`を`&U`に
    - `T: DerefMut<Target=U>`->`&mut T`を`&mut U`に
    - `T: Deref<Target=U>`->`&mut T`を`&U`に
---
# 15.3 Dropトレイトで片付け時にコードを走らせる

- `Drop`トレイトは，**値がスコープを抜けるときに起こること**をカスタマイズできる
    - `Box<T>`は`Drop`をカスタマイズして，ヒープ領域のメモリを解放する
- ファイルやネットワーク接続などのリソースを解放するのに活用できる

- 本節では下記の内容を取り扱う
    - `Drop`の使い方
    - 明示的に`Drop`を起動させる
---
# Dropトレイトの使い方

- 変数を生成した順序と逆の順序でドロップされる

```rs
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        // CustomSmartPointerをデータ`{}`とともにドロップするよ
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer { data: String::from("my stuff") };      // 俺のもの
    let d = CustomSmartPointer { data: String::from("other stuff") };   // 別のもの
    println!("CustomSmartPointers created.");                           // CustomSmartPointerが生成された
}
```

```
$ cargo run
CustomSmartPointers created.
Dropping CustomSmartPointer with data `other stuff`!
Dropping CustomSmartPointer with data `my stuff`!
```
---
# std::mem::dropで早期に値をドロップする

- `Drop`トレイトの`drop()`を手動で呼ぶことができない
- 代わりに`std::mem::drop()`で強制的にドロップさせる

```rs
fn main() {
    let c = CustomSmartPointer { data: String::from("some data") };
    println!("CustomSmartPointer created.");
    drop(c);
    // CustomSmartPointerはmainが終わる前にドロップされた
    println!("CustomSmartPointer dropped before the end of main.");
}
```
```
CustomSmartPointer created.
Dropping CustomSmartPointer with data `some data`!
CustomSmartPointer dropped before the end of main.
```
---
# 前半のまとめ

- スマートポインタはメモリを安全かつ効率的に扱うことができる
    - `Deref`トレイトを実装することで，参照外しすることでヒープ領域の値にアクセスすることができる
    - `Drop`トレイトを実装することで，スコープから外れたタイミングでメモリを解放する
- `Box<T>`はヒープにメモリを確保して，動的にメモリを管理できる
    - コンパイル時にサイズが確定しない，コンスリストのような構造を扱える
