---
marp: true
paginate: true
footer: TRPL読書会 #6
---
# 11. 自動テストを書く

- Kenta Arai
---
# Copyright

- The Rust trademark is owned and protected by The Rust Foundation
- [The Rust Programming Language](https://github.com/rust-lang/book) is licensed under [Apatch 2.0 and MIT terms](https://github.com/rust-lang/book/blob/main/COPYRIGHT)
- This slide is a derivative work of TRPL, and is licensed under Apatch 2.0 and MIT terms
---
# はじめに

- プログラムのテストは、バグの存在を示すには非常に効率的な手法であるが、バグの不在を示すには望み薄く不適切である
    - 謙虚なプログラマ，エドガー・W・ダイクストラ
- Rustの型システムは，プログラムの正当性の多くを肩代わりしてくれるが，不当性は補足しない
    - 正当性：どこまで自分のコードが意図していることをしているか
- Rustは言語内で自動化されたソフトウェアテストを書くことをサポートする

- 本章では，Rustのテスト機構を説明する
---
# 11.1. テストの記述法

- テストでは，テスト対象のコードが想定通りに動作することを検証する
- Rustのテストコードは一般に，下記の3つを実行する
    - 必要なデータと状態を準備する
    - テスト対象のコードを動作させる
    - 結果が想定通りであることを表明する(assert)

- 本節では，テスト関数の記述方法を述べる
- キーワード：`test`属性，`should_panic`属性
---
# テストの構成

- `test`属性で注釈された関数がテストターゲット
- ソースコードと同じファイル内にテストを記述する

```rs
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```
---
# テストを実行する

- テスト関数は単一のバイナリとしてコンパイルされる
- `cargo test`でテストプログラムが実行される

```
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.57s
     Running target/debug/deps/adder-92948b65e88960b4

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```
---
# テストのお約束

- `Running`の後にテストの結果が表示される
    - テストがパスしたら`ok`，失敗すると`FAILED`
- `test result:`の行では，テストの結果が要約される
    - `passed`及び`failed`はテストがパス／失敗した数
    - `ignored`は無視したために実行されなかったテスト（後述）
    - `measured`はベンチマークテスト向け（nightlyのみ）
    - `filtered out`はフィルタされたテストの数（後述）
```
$ cargo test
（省略）
running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```
---
# テスト関数の作り方

- テスト関数内でマクロや属性を使い，関数の振る舞いを検証する
- 戻り値を検証する
    - 論理型：`assert!`
    - 同値性：`assert_eq!`
    - 非同値性：`assert_ne!`
    - それ以外：`Result<T, E>`
- `panic!`を起こすことを検証する
    - `should_panic`属性
---
# assert!

- `true`なら成功，`false`なら失敗

```rs
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_adds_two() {
        assert!(4 == add_two(2));
    }
}
```
---
# assert_eq!/assert_ne!

- 同じ値なら成功，違ったら失敗
- 比較する値は`PartialEq`と`Debug`トレイトをderiveした型である必要がある

```rs
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_adds_two() {
        assert_eq!(4, add_two(2));
    }
}
```
---
# 非同値な値をassert_eq!/assert_ne!で検証する

- `assert`と異なり，非同値の場合に値を出力する

```
$ cargo test
（省略）
running 1 test
test tests::it_adds_two ... FAILED

failures:

---- tests::it_adds_two stdout ----
thread 'main' panicked at 'assertion failed: `(left == right)`
  left: `4`,
 right: `5`', src/lib.rs:11:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace.


failures:
    tests::it_adds_two
```
---
# Result

- `Ok`なら成功，`Err`ならテストが失敗

```rs
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
}
```
---
# panicする関数をテストする

- panicする関数は，戻り値から結果を判定できない
- `should_panic`属性によって，panicした場合にテストが成功として判定する

```rs
pub fn add_two(a: i32) -> i32 {
    panic!("panic! panic! panic!");
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn run_panic() {
        add_two(1);
    }
}
```
---
# カスタムの失敗メッセージを追加する

- `assert!`の第2引数，`assert_eq!`及び`assert_ne!`の第3引数に文字列を追加すると，エラーメッセージとして出力される

```rs
pub fn greeting(name: &str) -> String { String::from("Hello!") }

#[cfg(test)]
mod tests {
    use super::*;
    #[test]
    fn greeting_contains_name() {
        let result = greeting("Carol");
        assert!(
            result.contains("Carol"),
            "Greeting did not contain name, value was `{}`", result
        );
    }
}
```
---
# カスタムの失敗メッセージを確認する

```
$ cargo test
（省略）
running 1 test
test tests::greeting_contains_name ... FAILED

failures:

---- tests::greeting_contains_name stdout ----
thread 'main' panicked at 'Greeting did not contain name, value was `Hello!`',
src/lib.rs:12:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace.

failures:
    tests::greeting_contains_name

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out

error: test failed, to rerun pass '--lib'
```
---
# 11.2. テストの実行のされ方を制御する

- `cargo test`はコードをテストモードでコンパイルし，全てのテストを並行に実行する
- テスト実行中に生成された出力はキャプチャされ，表示されるのを防ぐ

- 本節ではテストの実行のされ方について述べる
- キーワード：`--test-threads`，`--nocapture`，`ignore`属性
---
# テストを逐次に実行する

- `cargo test`は複数のテストを並列に実行する
- 共通の資源を利用する機能をテストする場合，注意が必要
- テストを逐次に実行したい場合は`--test-threads`を使用する

```
$ cargo test -- --test-threads=1
```
---
# 関数の出力を表示する

- テスト実行時は標準出力に出力されたものを全てキャプチャされる
- テストに失敗した場合にのみキャプチャされた内容が出力される
- 成功した場合も標準出力の内容を確認したい場合は`--nocapture`を使用する

```
$ cargo test -- --nocapture
```
---
# 単独のテストを実行する

- 名前を指定することで，特定のテストを実行できる
- `one_hundred`を実行したい場合は`cargo test one_hundred`を実行

```
$ cargo test one_hundred
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/adder-06a75b4a1f2515e9

running 1 test
test tests::one_hundred ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 2 filtered out
```
---
# 複数のテストを実行する

- テスト名の一部を指定することで，その名前を含む関数を実行できる

```
$ cargo test add
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/adder-06a75b4a1f2515e9

running 2 tests
test tests::add_two_and_two ... ok
test tests::add_three_and_two ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out
```
---
# テストを無視する

- テストしたくない関数がある場合，`ignore`属性で除外できる

```rs
#[test]
#[ignore]
fn expensive_test() {
    // code that takes an hour to run
}
```
```
$ cargo test
（省略）
running 2 tests
test expensive_test ... ignored
test it_works ... ok

test result: ok. 1 passed; 0 failed; 1 ignored; 0 measured; 0 filtered out
```
---
# 無視したテストだけ実行する

- `ignore`属性を指定したテストのみ実行することも可能

```
$ cargo test -- --ignored
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/adder-ce99bcc2479f4607

running 1 test
test expensive_test ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out
```
---
# 11.3. テストの体系化

- 単体テスト：モジュール内のあらゆる関数をテストする
- 結合テスト：ライブラリと独立したバイナリで，公開された関数のみをテストする

- 本節では，単体テストと結合テストの記述方法
- キーワード：`tests`ディレクトリ
---
# 単体テスト

- 各ファイルに`tests`モジュールを作成し，テスト関数を含ませ，`cfg(test)`で注釈する
- `cargo test`を実行した際にテスト関数が実行される
- 非公開関数もテストできる

```rs
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```
---
# 結合テスト

- `src`ディレクトリと同じ階層に`tests`ディレクトリを作成すると，`Cargo`はそこに結合テストのファイルが置かれると認識する

```rs
// tests/integration_tests.rs
extern crate adder;

#[test]
fn it_adds_two() {
    assert_eq!(4, adder::add_two(2));
}
```
---
# 結合テストだけを実行する

- 結合テストだけを実行したい場合，`--test`でテスト名を指定する

```
$ cargo test --test integration_test
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/integration_test-952a27e0126bb565

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```
---
# 結合テスト内のサブモジュール

- 結合テスト用のサブモジュールは，さらに1つ深い階層に置くことで利用できる

```rs
// tests/common/mod.rs
pub fn setup() {
    // setup code specific to your library's tests would go here
}

// tests/integration_test.rs
extern crate adder;

mod common;

#[test]
fn it_adds_two() {
    common::setup();
    assert_eq!(4, adder::add_two(2));
}
```
---
# まとめ

- Rustのテスト機能は，機能が想定通りに動作することを確認する手段を提供する
- テストの記述法
    - `assert!`，`assert_eq!`，`assert_ne!`，`Result<T, E>`，`should_panic`
- テストの制御
    - 並列度を設定する：`cargo test -- --test-threads=<number of threads>`
    - 出力をキャプチャさせない：`cargo test -- --nocapture`
    - 特定の名前を含むテストだけを実行する：`cargo test <test name>`
    - 無視されたテストだけを実行する：`cargo test -- --ignored`
- テストの構成法
    - 単体テスト：モジュール内に`#[cfg(test)]`でテスト関数を記述する
    - 結合テスト：`tests`ディレクトリ内にテスト関数を記述する
