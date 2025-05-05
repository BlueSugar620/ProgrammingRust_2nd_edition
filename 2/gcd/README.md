# gcd

## ユニットテスト

Rust ではテスト機構が言語に含まれています。
テスト関数は通常のコンパイル時にはスキップされますが、 `cargo test` コマンドでプログラムを実行すると、自動的にコンパイルされ、実行されます。

`#[test]` マーカーは属性 (attribute) の一例です。これが関数の上に付いていると、その関数がテスト関数であることを意味します。

```
$ cargo test
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.03s
     Running unittests src/main.rs (target/debug/deps/gcd-d835dde4d73bb0be)

running 1 test
test test_gcd ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```
