# mandelbrot

## Web ページを公開する

Rust の利点の 1 つとして、 `crate.io (https://crate.io)` で公開されている無償のライブラリ群が挙げられます。
`cargo` コマンドを用いると、 `crate.io` のパッケージを自分のコードで簡単に利用することができます。
Rust のパッケージは、ライブラリであれ実行ファイルであれ、クレート (crate) と呼ばれます。


`#[derive(Deserialize)]` マーカーを型定義の前に付与しておくと、 serde クレートはコンパイル時にその型を解析して、 HTML フォームが POST リクエストに使用する形式のデータからその型のデータ値を取り出すコードを、自動的に生成します。
実際には、このように書くだけで POST リクエストだけではなく、 JSON 、 YAML 、 TOML などのさまざまなテキスト形式もしくはバイナリ形式のデータから Rust の値を読み取ることができるようになります。
serde クレートには、これと逆のこと、つまり Rust の値を構造化された形式で書き出すための `Serialize` 属性も用意されています。

## 並列プログラミング

Rust のメリットの 1 つは並列プログラムに対する支援が強力であることです。
Rust は、メモリエラーがないことを保証するのと同じルールによって、データ競合が起きない形でのみ複数スレッドがメモリを共有できないようにしてくれます。
例えば、

- 複数のスレッドが排他ロックを用いて共有データ構造を協調して更新する場合、 Rust はロックを保持しているとき以外にデータにアクセスできないことと、更新が終わったら自動的にロックが解放されることを保証してくれます。

- 複数のスレッドが読み出しのみ可能なデータを共有する場合、 Rust は間違ってそのデータを更新してしまわないことを保証してくれます。

- あるデータ構造の所有権を別のスレッドに移した場合、 Rust はそのデータへのアクセスを全くできないようにしてくれます。



Mandelbrot 集合は、以下のように定義されます。

$$
\begin{cases}
z_{n + 1} = z_n ^2 + c & n \ge 0 \\
z_0 = 0
\end{cases}
$$

で定義される複素数列 $\{ z_n \}_{n \ge 0}$ が $n \to \infty$ の極限で無限大に発散しないという条件を満たす複素数 $c$ 全体が作る集合。


関数定義の上にあるコメントを `///` で書くと、ドキュメントコメント (documentation comments) になります。 rustdoc ユーティリティは、このようなコメントをパースする方法を知っており、コメントが説明しているコードの情報と合わせてオンラインドキュメントを作成します。

`parse_pair` はジェネリック関数となっています。 `<T: FromStr>` の部分は、「 `FromStr` トレイトを実装する任意の型 `T` に対して」と読みます。 Rust プログラマは `T` を `parse_pair` の型パラメータ (type parameter) と呼ぶ。


Rust では失敗し得る関数は Result 型で値を返す必要があります。 Result は、成功時には `Ok(s)` を、失敗時には `Err(e)` を返します。このとき、 `match` 文をいちいち書くことを省略するために `?` 演算子が用意されています。これは以下と等しいです。そのため、その関数が `Result` 型を返さなければ使用できません。

```
match x {
    Ok(s) => s,
    Err(e) => {
        return Err(e);
    }
}
```


`&mut` は、可変参照を借用しています。 `&` は共有参照を渡しています。


プログラムを release モードでビルドし実行することができます。 release モードにすると強力なコンパイル時最適化が行われます。

```
$ cargo build --release
  Compiling mandelbrot v0.1.0 (<パス>/ProgrammingRust_2nd_edition/2/mandelbrot)
    Finished `release` profile [optimized] target(s) in 29.00s
$ time target/release/mandelbrot mandel.png 4000x3000 -1.20,0.35 -1,0.20
target/release/mandelbrot mandel.png 4000x3000 -1.20,0.35 -1,0.20  3.53s user 0.01s system 91% cpu 3.856 total
```

上に示した実行では Unix の time コマンドを使ってプログラムの実行時間を測定しています。
並列化していないので、 3.856 秒かかっています。


`|spawner| { ... }` は Rust のクロージャ (closure) 式で、引数を一つとります。
`into_iter()` で得られるイテレータは、個々の要素の排他的な所有権を、各ループのボティに与えます。
これによって、ある時点では 1 つのスレッドだけがその要素に書き込めることを保証しています。
`move |_| { ... }` の `move` キーワードは、このクロージャが利用する変数の所有権を取得することを意味しています。

```
$ cargo build --release
  Compiling mandelbrot v0.1.0 (<パス>/ProgrammingRust_2nd_edition/2/mandelbrot)
    Finished `release` profile [optimized] target(s) in 1.30s
target/release/mandelbrot mandel.png 4000x3000 -1.20,0.35 -1.0,0.20  4.04s user 0.01s system 270% cpu 1.496 total
```

並列化をすることで、 1.496 秒まで短くなりました。
