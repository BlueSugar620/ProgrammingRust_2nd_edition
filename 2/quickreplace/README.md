# quickreplace

## ファイルシステムとコマンドラインツール

Rust はコマンドラインツールの世界に、大きなニッチがあることを見出しました。

バージョンが 1 に達すると、「セマンティックバージョン」に従います。
すなわち、メジャーバージョン番号が変わらない限り、新しいバージョンは古いバージョンと互換性を持たなくてはなりません。

以下のようにテストできます。

```
$ cargo run "world" "Rust" test.txt test-modified.txt
  Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.01s
    Running `target/debug/quickreplace world Rust test.txt test-modified.txt`
```


