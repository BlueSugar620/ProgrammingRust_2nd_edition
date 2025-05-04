# hello

`cargo run` コマンドで、プログラムをビルドして実行することができます。

```
$ cargo run
   Compiling hello v0.1.0 (/<ディレクトリへのパス>/ProgrammingRust_2nd_edition/2/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.63s
     Running `target/debug/hello`
Hello, world!
```

実行ファイルはパッケージトップディレクトリ直下の `target` サブディレクトリに置かれます。

```
$ ls -l target/debug
total 880
drwxr-xr-x   2 <ユーザー名>  staff      64 <作成時刻> build
drwxr-xr-x  10 <ユーザー名>  staff     320 <作成時刻> deps
drwxr-xr-x   2 <ユーザー名>  staff      64 <作成時刻> examples
-rwxr-xr-x   1 <ユーザー名>  staff  445304 <作成時刻> hello
-rw-r--r--   1 <ユーザー名>  staff     154 <作成時刻> hello.d
drwxr-xr-x   3 <ユーザー名>  staff      96 <作成時刻> incremental
```

実行ファイルから、実行することができます。

```
$ target/hello/debug
Hello, world!
```

操作が終了したとき、 Cargo によって生成されたファイルを以下のコマンドで削除できます。

```
$ cargo clean
$ target/hello/debug
zsh: no such file or directory: target/debug/hello
```
