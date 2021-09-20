---
title: 6. Enumとパターンマッチング
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

# アウトライン

- この章のトピック：**列挙型(Enum)**
  - Rustでは呼吸をするように列挙型を使います
  - TRPLでも頻出する，大事な機能
- Enumを使う上で便利な機能についても取り上げます
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
- もしIPv4を数値で表現したければ...？

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

# Null値の問題点

- CやJavaでは，データが無いことをNullで表現する
- Nullを適切に扱うのはプログラマの責任
- もしNullの値を有効な値として使ったらどうなるだろう？
  - 変数だったら...予期せぬ値を読み書き
  - 関数だったら...でたらめな処理を始めてしまう
  - e.x. Unix系OSにおけるsegmentation fault, JavaにおけるNullPointerException

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

- **値に直接アクセスできないため**
- そもそもコンパイルエラーに

```
let x: i8 = 5;
let y: Option<i8> = Some(3);

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

error: aborting due to previous error

For more information about this error, try `rustc --explain E0277`.
error: could not compile `playground`

To learn more, run the command again with --verbose.
```

# 【補足】値を取り出すには？

- unwrap()を使う
  - Someなら関連付けられた値を得られる
  - Noneだとそこでpanicに
- 安全に値を取り出すには→6.2以降で

```
let x: i8 = 5;
let y: Option<i8> = Some(3);

let sum = x + y.unwrap();
```

# 6.2 matchフロー制御演算子

- マッチしたパターンに応じてコードを実行するフロー制御演算子
- `match`は式であるため，値を返す

```
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

# matchは包括的

- matchで網羅的にパターンが列挙されていないと，コンパイルエラーに

```
fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        // Coin::Quarter => 25,
    }
}
```

```
    Compiling playground v0.0.1 (/playground)
error[E0004]: non-exhaustive patterns: `Quarter` not covered
 --> src/main.rs:9:11
  |
1 | / enum Coin {
2 | |     Penny,
3 | |     Nickel,
4 | |     Dime,
5 | |     Quarter,
  | |     ------- not covered
6 | | }
  | |_- `Coin` defined here
...
9 |       match coin {
  |             ^^^^ pattern `Quarter` not covered
  |
  = help: ensure that all possible cases are being handled, possibly by adding wildcards or more match arms
  = note: the matched value is of type `Coin`

error: aborting due to previous error

For more information about this error, try `rustc --explain E0004`.
error: could not compile `playground`

To learn more, run the command again with --verbose.
```

# \_というプレースホルダー

- 全てのパターンを列挙したくない場合，\_が便利です
- \_はその他全てのパターンに対してマッチします
- ()はユニット値（何も無い）です

```
let some_u8_value = 0u8;
match some_u8_value {
    1 => println!("one"),
    3 => println!("three"),
    5 => println!("five"),
    7 => println!("seven"),
    _ => (),
}
```

# 値に束縛されるパターン

- `match`はマッチした値を束縛できる
- コインにデザインされた州名を表示してみましょう
  - 一時期のQuaterコインは州ごとにデザインされていた

- ところで原本の題目は`Patterns that Bind to Values`
  - 実際は，値を束縛するパターン？

```
#[derive(Debug)] // すぐに州を点検できるように
enum UsState {
    Alabama,
    Alaska,
    // ... などなど
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        },
    }
}
```

# Option<T>とのマッチ

- Option<T>に関連付けられたデータを取り出すには？
- Optionももちろん列挙型なので，`match`で扱えます

```
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}
    
let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```

# 6.3 if letで簡潔なフロー制御

- 1つの値にしか興味が無い場合，`match`は少し長い
- `if let`はより短い表現で値をマッチさせることができます

```
// matchだとちょっと長い
let some_u8_value = Some(0u8);
match some_u8_value {
    Some(3) => println!("three"),
    _ => (),
}

// if letだと簡潔
if let Some(3) = some_u8_value {
    println!("three");
}
```

# まとめ

- Rustで頻出するEnumについて述べた
- Enumに関する便利な機能について取り上げた
  - Option型: 値の有無を表現する列挙型
  - match式: 網羅的なフロー制御機能
  - if let: 列挙型を使う上で便利な慣用句
 

