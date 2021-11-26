---
marp: true
paginate: true
footer: TRPL読書会 #10
---
# 18. パターンとマッチング

- Kenta Arai
---
# Copyright

- The Rust trademark is owned and protected by The Rust Foundation
- [The Rust Programming Language](https://github.com/rust-lang/book) is licensed under [Apatch 2.0 and MIT terms](https://github.com/rust-lang/book/blob/main/COPYRIGHT)
- This slide is a derivative work of TRPL, and is licensed under Apatch 2.0 and MIT terms
---
# 概要

- Rustのパターンとは，型の構造に一致する記法
- `match`式や他の構文と組み合わせることで，プログラムをコントロールできる

- この章では改めてRustのパターンとは何かを説明する
---
# 18.1 パターンが使用されることのある箇所全部

- パターンが使用される箇所をすべて説明する
    - matchアーム
    - if let式
    - while letループ
    - forループ
    - let文
    - 関数の引数
---
# matchアーム

- `match`式はVALUEと一致するパターンのEXPRESSIONを返す
- パターンはVALUEが取りうる値の全てを網羅する

```rs
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
```
---
# if let式

- `if let`は1つの値とパターンが一致する場合にのみ括弧内の処理を実行する
- `match`とは異なり，値の網羅性については確認されない

```rs
if let Some(color) = favorite_color {
    ...
} else if let Ok(age) = age {
    ...
}
```
---
# while let条件分岐ループ

- `while let`はパターンが合致し続ける限り括弧内の処理を実行する

```rs
let mut stack = Vec::new();

stack.push(1);
stack.push(2);
stack.push(3);

while let Some(top) = stack.pop() {
    println!("{}", top);
}
```
---
# forループ

- `for`ループは`for`に続く部分がパターン

```rs
let v = vec!['a', 'b', 'c'];

for (index, value) in v.iter().enumerate() {
    println!("{} is at index {}", value, index);
}
// a is at index 0
// b is at index 1
// c is at index 2
```
---
# let文

- 実はこれもパターンマッチが使われている
- パターンと値の型が異なるとコンパイルエラーとなる
    - 下記の例ではパターンが(x, y, z)，値が(1, 2, 3)
    - 型は(i32, i32, i32)

```rs
let (x, y, z) = (1, 2, 3);
```
---
# 関数の引数

- 引数もlet文と同じようにパターンを使える

```rs
fn print_coordinates(&(x, y): &(i32, i32)) {
    ...
}
```
---
# 18.2 論駁可能性：パターンが合致しないかどうか

- 論駁：ろんばく，refutation
    - 相手の論や説の誤りを論じて攻撃すること（[goo辞書](https://dictionary.goo.ne.jp/word/%E8%AB%96%E9%A7%81/)より）
- TRPLでは（？）あらゆる値にパターンが合致するか否かを**論駁可能性**という
    - `let x = 5;`は5の部分が他の値でもパターンxに合致する→論駁不可能
    - `if let Some(x) = a_value`はa_valueが`None`の場合パターンxに合致しない→論駁可能
---
# 問題を含む論駁可能なコード例

- let式は論駁不可能でなければならない
- `some_option_value`が`None`場合，xに値を束縛できない
- つまり論駁可能なのでコンパイルできない

```rs
let Some(x) = some_option_value;
```
```
error[E0005]: refutable pattern in local binding: `None` not covered
(エラー: ローカル束縛に論駁可能なパターン: `None`がカバーされていません)
 -->
  |
3 | let Some(x) = some_option_value;
  |     ^^^^^^^ pattern `None` not covered
```
---
# 問題を含む論駁不可能なコード例

- `if let`式は論駁可能でなければならない
- `x`は必ず`5`に束縛される
- 論駁不可能なのでコンパイルできない

```rs
if let x = 5 {
    println!("{}", x);
};
```
```
error[E0162]: irrefutable if-let pattern
(エラー: 論駁不可能なif-letパターン)
 --> <anon>:2:8
  |
2 | if let x = 5 {
  |        ^ irrefutable pattern
```
---
# 18.3 パターン記法

- 18.1ではパターンが使われる箇所を述べた
- 本節ではパターンの書き方を述べる
---
# リテラルにマッチする

- パターンを直接リテラルにマッチさせられる

```rs
match x {
    1 => println!("one"),       // 1
    2 => println!("two"),       // 2
    3 => println!("three"),     // 3
    _ => println!("anything"),  // なんでも
}
```
---
# 名前付き変数にマッチする

- もちろん変数をマッチさせることもできる

```rs
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        // 50だったよ
        Some(50) => println!("Got 50"),
        // マッチしたよ
        Some(y) => println!("Matched, y = {:?}", y),
        // 既定のケース
        _ => println!("Default case, x = {:?}", x),
    }

    // 最後にはx = {}, y = {}
    println!("at the end: x = {:?}, y = {:?}", x, y);
}
```
---
# 複数のパターン

- `match`式では`|`記法で複数のパターンに合致させることができる

```rs
let x = 1;

match x {
    // 1か2
    1 | 2 => println!("one or two"),
    // 3
    3 => println!("three"),
    // なんでも
    _ => println!("anything"),
}
```
---
# ...で値の範囲に合致させる

- `...`記法は限界値を**含む**値の範囲にマッチさせることができる
- `...`は数値か`char`型にのみ使うことが出来る

```rs
let x = 5;

match x {
    // 1から5まで
    1 ... 5 => println!("one through five"),
    // それ以外
    _ => println!("something else"),
}
```
---
# ...を使う便利な例

```rs
let x = 'c';

match x {
    // ASCII文字前半
    'a' ... 'j' => println!("early ASCII letter"),
    // ASCII文字後半
    'k' ... 'z' => println!("late ASCII letter"),
    // それ以外
    _ => println!("something else"),
}
```
---
# 分配して値を分解する

- 構造体，enum，タプルはそれらを構成する型にパターンマッチで分解することが出来る
- 以降ではパターンマッチで分配する方法を述べる
    - ここでは`分解`なのに，以降は`分配`と書かれている
    - 分解→`destructing to break apart`，分配→`destructing`
---
# 構造体を分配する

- 構造体pの要素xとyを，aとbに分配する

```rs
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x: a, y: b } = p;
    // let Point { a, b } = p; でもOK
    // let Point { y: b, x: a } = p; でもOK
    assert_eq!(0, a);
    assert_eq!(7, b);
}
```
---
# enumを分配する

- enumも構造体と同様に分配できる

```rs
enum Message {
    Quit,
    Coordinate(i32, i32),
}

fn main() {
    let msg = Message::Coordinate(10, 5);

    match msg {
        Message::Quit => {
            println!("The Quit variant has no data to destructure.")
        },
        Message::Coordinate(x, y) => {
            println!("Change the coordinate to x {} and y {}", x, y)
        }
    }
}
```
---
# 参照を分配する

- ある型への参照を，その型を構成する型への参照に分配できる

```rs
let points = vec![
    Point { x: 0, y: 0 },
    Point { x: 1, y: 5 },
    Point { x: 10, y: -3 },
];

let sum_of_squares: i32 = points
    .iter()
    .map(|&Point { x, y }| x * x + y * y)
    .sum();
```
---
# 構造体とタプルを分配する

- 構造体とタプルが混ざった型ももちろん分解できる

```rs
let ((feet, inches), Point {x, y}) = ((3, 10), Point { x: 3, y: -10 });
```
---
# パターンの値を無視する

- パターンの中で値を無視する方法は`_`と`..`
---
# _で値全体を無視する

- `_`にマッチした値は無視される

```rs
fn foo(_: i32, y: i32) {
    // このコードは、y引数を使うだけです: {}
    println!("This code only uses the y parameter: {}", y);
    // "This code only uses the y parameter: 4"と表示される
}

fn main() {
    foo(3, 4);
}
```
---
# ネストされた_で値の一部を無視する

- パターンの構造体やタプルのメンバも`_`で無視することが出来る

```rs
let mut setting_value = Some(5);
let new_setting_value = Some(10);

match (setting_value, new_setting_value) {
    (Some(_), Some(_)) => {
        // 既存の値の変更を上書きできません
        println!("Can't overwrite an existing customized value");
    }
    _ => {
        setting_value = new_setting_value;
    }
}
```
---
# タプルのメンバの一部を無視する

```rs
let numbers = (2, 4, 8, 16, 32);

match numbers {
    (first, _, third, _, fifth) => {
        // 何らかの数値: {}, {}, {}
        println!("Some numbers: {}, {}, {}", first, third, fifth)
    },
}
```
---
# 名前を_で始めて未使用の変数を無視する

- 名前を`_`で始めると，コンパイラはその変数を未使用変数として警告しなくなる


```rs
fn main() {
    let _x = 5;
    let y = 10;
}
```
---
# ..で値の残りの部分を無視する

- 複数のメンバをまとめて無視したい場合は`..`を使う

```rs
struct Point {
    x: i32,
    y: i32,
    z: i32,
}

let origin = Point { x: 0, y: 0, z: 0 };

match origin {
    Point { x, .. } => println!("x is {}", x),
}
```
---
# ..で値を無視する場合は位置が明確でなければならない

```rs
let numbers = (2, 4, 8, 16, 32);

match numbers {
    (.., second, ..) => {
        println!("Some numbers: {}", second)
    },
}
```
```
error: `..` can only be used once per tuple or tuple struct pattern
(エラー: `..`は、タプルやタプル構造体パターン1つにつき、1回しか使用できません)
 --> src/main.rs:5:22
  |
5 |         (.., second, ..) => {
  |                      ^^
```
---
# refとref mutでパターンに参照を生成する

- 通常のパターンマッチでは，値を束縛される際にムーブされる
- マッチさせる値の所有権を失いたくない場合は`ref`で参照にマッチするようにする
- 可変参照にマッチさせたい場合は`mut ref`を使う

```rs
let robot_name = Some(String::from("Bors"));

match robot_name {
    Some(ref name) => println!("Found a name: {}", name),
    None => (),
}

println!("robot_name is: {:?}", robot_name);
```
---
# マッチガードで追加の条件式

- マッチガードは`match`アームのパターンの後に指定される`if`条件
- 最初のパターンに合致すると，次の`if`条件で比較する

```rs
let num = Some(4);

match num {
    // 5未満です: {}
    Some(x) if x < 5 => println!("less than five: {}", x),
    Some(x) => println!("{}", x),
    None => (),
}
```
---
# @束縛

- ある範囲に値がマッチするかを検査する


```rs
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    Message::Hello { id: id_variable @ 3...7 } => {
        // 範囲内のidが見つかりました: {}
        println!("Found an id in range: {}", id_variable)
    },
    Message::Hello { id: 10...12 } => {
        // 別の範囲内のidが見つかりました
        println!("Found an id in another range")
    },
    Message::Hello { id } => {
        // それ以外のidが見つかりました
        println!("Found some other id: {}", id)
    },
}
```
---
# まとめ

- Rustのパターンは異なる種類のデータを区別してプログラムを制御する
- パターンが使用される箇所
    - matchアーム
    - if let式
    - while letループ
    - forループ
    - let文
    - 関数の引数
- あらゆる値にパターンがマッチするか？：論駁可能性
- パターン記法
    - 構造体，enum，タプルを分解
    - マッチガード
    - @束縛
