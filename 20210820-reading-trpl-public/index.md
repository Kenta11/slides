---
title: The Rust Programming Language 読書会 #3
author: Kenta Arai
date: "2021/8/20"
theme: sky
slideNumber: true
center: false
---

# Copyright

- The Rust trademark is owned and protected by The Rust Foundation
- [The Rust Programming Language](https://github.com/rust-lang/book) is licensed under [Apatch 2.0 and MIT terms](https://github.com/rust-lang/book/blob/main/COPYRIGHT)
- This slide is a derivative work of TRPL, and is licensed under Apatch 2.0 and MIT terms

# 6. Enumとパターンマッチング

- この章のトピック：**列挙型(enum)**
  - Rustでは呼吸をするように列挙型を使います
  - TRPLでも頻出する，大事な機能
- enumを使う上で便利な機能についても取り上げます
  - Option型: そもそも値があるか？を表現する列挙型
  - match式: switch文ライクで，網羅的なフロー制御機能
  - if let: 列挙型を使う上で便利な慣用句

# 6.1 Enumを定義する

- Rustでは表現したい内容によって，構造体よりも列挙型が好ましいケースがある
- 例題：IPアドレスの規格を表現してみましょう
  - IPアドレスの規格：IPv4とIPv6
- 列挙型の各列挙子はインスタンスとして生成できます

```
enum IpAddrKind {
    V4,
    V6,
}

let four = IpAddrKind::V4;
let six = IpAddrKind::V6;
```

# Enumの値

- 列挙子は列挙型の名前で名前空間分けがされている
  - `列挙型名::列挙子名`
- 列挙型は型なので，変数宣言や関数の引数の型として使える

```
let four = IpAddrKind::V4;
let six: IpAddrKind = IpAddrKind::V6;

fn route(ip_type: IpAddrKind) { /* 省略 */ }
```

# Enumのメリット

- 列挙型を使うことで，IPアドレスの種類を分類できました
- ではIPアドレスそのもの（データ）はどうやって保持したらいいでしょうか？
- **列挙型にはデータを紐付けることができます**

```
enum IpAddr {
    V4(String),
    V6(String),
}

let home = IpAddr::V4(String::from("127.0.0.1"));
let loopback = IpAddr::V6(String::from("::1"));
```

# Enumに紐付けるデータは異なる型でもOK

- IPv4とIPv6は表現構造が違います
  - IPv4: 0から255までのいずれかの値を持つ，4つの数値
  - IPv6: 0から65535までのいずれかの値を持つ，8つの数値
- もし数値で表現したければ...？

```
enum IpAddr {
    V4(u8, u8, u8, u8), // IPv4は個別の値を保持したいな
    V6(String),         // IPv6は長いから文字列でいいや
}

let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));
```

# 【参考】標準ライブラリのIPアドレスはこんな実装

```
struct Ipv4Addr { /* 省略 */ }

struct Ipv6Addr { /* 省略 */ }

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

# 列挙型 v.s. 構造体

- より複雑なデータを扱うとどうなるだろう？

```
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

```
struct QuitMessage; // ユニット構造体
struct MoveMessage {
    x: i32,
    y: i32,
}
struct WriteMessage(String); // タプル構造体
struct ChangeColorMessage(i32, i32, i32); // タプル構造体
```

# Enumに関数を関連付ける

- 構造体と同様にメソッドを定義できる

```
impl Message {
    fn call(&self) {
        // メソッド本体はここに定義される
    }
}

let m = Message::Write(String::from("hello"));
m.call();
```

# Option enumとNull値に勝る利点

- CやJavaでは，データが無いことをNullで表現する
- Nullを適切に扱うのはプログラマの責任
  - 有効な値は有効な値，無効な値はNullとして処理する
- もしNullの値を有効な値として使ったらどうなるだろう？
  - 変数だったら...予期せぬ値を読み書き
  - 関数だったら...でたらめな処理を始めてしまう
  - e.x. Unix系OSにおけるsegmentation fault, JavaにおけるNullPointerException
- **RustではOptionを使いましょう！**

# Option<T>型

- 値の有無を表現する型
- とても便利なので，初期化処理(prelude)でも使われている
  - 明示的にスコープ内へ導入する必要が無い

```
enum Option<T> {
    Some(T),
    None,
}
```

```
let some_number = Some(5);
let some_string = Some("a string");

let absent_number: Option<i32> = None;
```

# Nullと比べてなぜ便利？

- **値が無い（=None）だったら，値にアクセスできないため**
- Noneから値を読み出そうとすると，そもそもコンパイルエラーに

```
let x: i8 = 5;
let y: Option<i8> = None;

let sum = x + y;
```

```
   Compiling playground v0.0.1 (/playground)
error[E0277]: cannot add `Option<i8>` to `i8`
 --> src/main.rs:6:17
  |
6 |     let sum = x + y;
  |                 ^ no implementation for `i8 + Option<i8>`
  |
  = help: the trait `Add<Option<i8>>` is not implemented for `i8`
```

# 6.2 matchフロー制御演算子

- 

# 値に束縛されるパターン





# Option<T>とのマッチ





# マッチは包括的





# _というプレースホルダー





# 6.3 if letで簡潔なフロー制御





# まとめ






