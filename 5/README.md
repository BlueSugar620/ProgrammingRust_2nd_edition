# 参照

これまで見てきた、単純なヒープへのポインタである `Box<T>` や、 `String` 値や `Vec` 値の内部にあるポインタは、所有権を持つポインタでした。
所有権であるポインタがドロップされると参照先もドロップされます。
Rust には、所有権を持たないポインタ型、参照 (refrerence) があります。
参照は、参照先の生存期間に何の影響も持ちません。

実際のところ、話はむしろ逆で、参照は参照先よりも長生きをしてはいけません。
プログラマは、参照が参照先よりも長生きすることがないことを、プログラム中で明確にしなければなりません。
これを強調するために、 Rust はある値に対する参照を作ることを借用 (borrowing) と呼びます。

## 値への参照

```
use std::collections::HashMap;

type Table = HashMap<String, Vec<String>>;

fn show(table: Table) {
    for (atrist, works) in table {
        println!("works by {}:", artist);
        for work in works {
            println!(" {}", work);
        }
    }
}

fn main() {
    let mut table = Table::new();
    table.insert("Gesualdo".to_string(), vec!["many madrigals".to_string()]);
    table.insert("Caravaggio".to_string(), vec!["The Musicians".to_string(), "The Calling of St.Matthew".to_string()]);
    table.insert("Cellini".to_string(), vec!["Perseus with the head of Medusa".to_string(), "a salt cellar".to_string()]);

    show(table);
}
```

これはうまく動きます。しかし、 `show(table)` が呼び出されるとで、 `table` はドロップされます。

```
show(table);
assert_eq!(table["Gesualdo"][0], "many madrigals");
// error: borrow of moved value: `table`
```

実際、 `show` の定義を見てみると、外側の `for` ループがハッシュテーブルの所有権をとり、完全に消費してしまいます。
内側の `for` ループは、個々のベクタに対して同じことをします。

このようなことを行いたい場合は、参照を使うのが正しいです。
参照を用いると、値の所有権に影響を与えずに値にアクセスできます。

- 共有参照 (shared reference) を用いると、参照先を読むことはできるが変更することはできません。しかし、ある値に対する共有参照は、同時にいくらでも持つことができます。 `&e` は値 `e` に対する共有参照を返します。 `e` の型が `T` であれば、 `&e` の型は `&T` ([ref T]と発音します)となります。共有参照は `Copy` 型です。

- 値への可変参照 (mutable reference) を用いると、値を読み出すことも変更することもできます。しかし、同じ値に対する可変参照と、他の参照(共有参照と可変参照)とは同時に使用することはできません。`&mut e` は、値 `e` に対する可変参照を返します。この参照の型は `&mut T` ([ref mute T]と発音します)となります。可変参照は `Copy` 型ではありません。

共有参照と可変参照を用意しているのは、複数読み出しか単一書き込みのルールをコンパイル時に強要するためです。
ある値への共有参照が存在する間は、所有者といえども値を変更することはできません。
同様に、ある値に対する可変参照が存在する間は、この参照が値に対する排他的なアクセス権を持ちます。
参照がなくなるまでは所有者を使うことはまったくできません。

値の所有権を移動するような方法で関数へ値を渡すことを値渡し (pass by value) とよび、関数に値の参照を渡すことを参照渡し (pass by reference) とよぶ。

## 参照の使い方

Rust では参照は明示的に `&` 演算子で作られ、参照解決も `*` 演算子で行われます。
参照があまりに一般的に使われるので、 Rust では `.` 演算子が、必要に応じて暗黙に左のオペランドを参照解決するようになっています。
また、 `.` 演算子は、メソッド呼び出すの際に、必要があれば暗黙に左オペランドへの参照を借用します。

変数に参照を代入すると、その変数は新しい場所を指すようになります。

```
let x = 10;
let y = 20;
let mut r = &x;

if v { r - &y; }

assert!(*r == 10 || *r == 20);
```

Rust は参照への参照を許します。

```
struct Point { x: i32, y: i32 }
let point = Point { x: 1000, y: 729 };
let r: &Point = &point;
let rr: &&Point = &r;
let rrr: &&&Point = &rr;

assert_eq!(rrr.y, 729);
```

`.` 演算子は、何段でも参照解決を行って目的の値を取り出します。

`.` 演算sにと同様に、比較演算子も、参照の連鎖を「見通す」ことができます。双方のオペランドは同じ型でなければなりません。

```
let x = 10;
let y = 10;

let rx = &x;
let ry = &y;

let rrx = &rx;
let rry = &ry;

assert!(rrx <= rry);
assert!(rrx == rry);

assert!(rx == rrx); // error: type mismatch
assert!(rx == *rrx) // this is okay
```

どうしても 2 つの参照が同じメモリをさしているかを知りたければ、 `std::ptr::eq` を使えばよいです。
これはアドレスでの比較を行います。

Rust の参照は決して `null` にはなりません。
参照のデフォルト初期値は存在しないし、整数から参照へ変換することもできません。

Rust では任意の式の値に対して参照を借用できます。

```
fn factorial(n: usize) -> usize {
    (1..n + 1).product()
}
let r = &factorial(6);
// Arithmetic operators can see through one level of references.
assert_eq!(r + &1009, 1729);
```

このような場合、 Rust は式の値を保持する無名の変数を作り、参照がそれを指すようにします。
無名変数の生存期間は、参照で何をするかに依存します。

- 参照を `let` 文ですぐに変数に代入する場合（もしくは構造体や配列などの値の一部としてすぐに代入する場合）、無名変数の生存期間は、 `let` が初期化した変数の生存期間と同じになります。上の例では、 `r` の参照先にこれが当たります。

- それ以外の場合には、無名変数の生存期間はそれを包んでいる文の終わりまでになります。上の例では `1009` を指すために作られた無名変数の生存期間は `assert_eq!` 文が終了する時点までとなります。


ここまでに示した3章は全て単純なアドレスでした。
しかし、 Rust には 2 種類のファットポインタ (fat pointer) があります。
ファットポインタは、何らかの値へのアドレスと、その値を使うために必要な情報を持つワードの 2 ワードで構成されます。

スライスへの参照は、スライスの開始点のアドレスと長さを持つファットポインタです。
もう 1 つのファットポインタが、特定のトレイトを実装した値への参照であるトレイトオブジェクト (trait object) です。
トレイトオブジェクトは、トレイトのメソッド実行のために、値のアドレスと、その値に対応するトレイトの実装を持ちます。

## 参照の安全性

ローカル変数の参照を借用して、その変数のスコープの外に持ち出すことはできません。

```
{
    let r;
    {
        let x = 1;
        r = &x;
    }
    assert_eq!(*r, 1);  // bad: reads memory `x` used to occupy
}
```

Rust コンパイラは、プログラム中の全ての参照型に対して、その参照の使われ方によって生じる制約を反映した生存期間 (lieftime) を割り当てます。
生存期間とは、プログラム実行中の、ある参照が安全に利用できる期間を指します。

この例では、 `r` に格納した参照型の生存期間が問題となっています。

参照を関数に渡した場合、関数がその参照を安全に使ってくれることを、 Rust は以下のようにして保証します。

```
// This code has several problems, and doesn't compile.
static mut STASH: &i32;
fn f(p: &i32) { STASH = p; }
```

Rust における、グローバル変数に該当するものは、 `static` と呼ばれます。
この変数は、プログラムが実行開始する際に作られ、プログラムが終了するまで維持されます。
上のコードは以下のルールを守っていません。

- `static` 変数は初期化されなければなりません。
- 可変な `static` 変数は、その性質上、スレッド安全ではありません。シングルスレッドプログラムでも、リエントラント問題に陥る可能性があります。このため、可変な `static` には `unsafe` ブロックの中でしかアクセスできないようになっています。

```
static mut STASH: &i32 = &128;
fn f(p: &i32) { // still not good enough
    unsafe {
        STASH = p;
    }
}
```

ここに書いた `f` のシグネチャは、実は次のように書くべきものを省略した形になっています。

```
fn f<'a>(p: &'a i32) { ... }
```

ここで、生存期間 `'a` ([tick A]と発音します)は、 `f` の生存期間パラメータです。
`<'a>` は「任意の生存期間 `'a` に対して」と読みます。

`'a` として任意の生存期間を許すということなので、可能な限り短い生存期間を想定して考えてみます。
つまり `f` への呼び出すを囲む期間がそれになります。
すると `STASH = p;` の代入で問題が生じます。
プログラム実行全体の生存期間のことを Rust では、「 `'static` 生存期間 」と呼びます。
よって、以下のようにコードを書き直さなければいけません。

```
static mut STASH: &i32 = &10;

fn f(p: &'static i32) {
    unsafe {
        STASH = p;
    }
}
```

次に関数の呼び出す側との関係をみていきます。

```
// This could be written more briefly: fn g(p: &i32),
// but let's write out the lifetimes for now.
fn g<'a>(p: &'a i32) { ... }

let x = 10;
g(&x);
```

使うときには、コンパイラが生存期間を推論してくれます。

関数が、何らかのデータ構造への参照を引数にとり、その構造の一部への参照を返すことはよくあります。

```
// v should have at least one element.
fn smallest(v: &[i32]) -> &i32 {
    let mut s = &v[0];
    for r in &v[1..] {
        if *r < *s { s = r; }
    }
    s
}
```

この関数のシグネチャでは、いつものように生存期間を省略しています。
関数が引数として参照を 1 つだけ取り、 1 つだけ参照を返す場合には、 Rust コンパイラはこれらの参照が同じ生存期間を持つと仮定します。
明示的に書くと以下のようになります。

```
fn smallest<'a>(v: &'a [i32]) -> &'a i32 { ... }
```

`smallest` を以下のように呼び出すことを考えてみましょう。

```
let s;
{
    let parabola = [9, 4, 1, 0, 1, 4, 9];
    s = smallest(&parabola);
}
assert_eq!(*s, 0);  // bad: points to element of dropped array
```

上の例では、引数 `&parabola` は `parabola` そのものよりも長生きすることはできず、 `smallest` の返り値は少なくとも `s` と同じだけ生きていなければなりません。
この 2 つの制約を同時に満たす生存期間 `'a` は存在しないので、このコードはコンパイルできません。

以下のようにして解決できます。

```
let parabola = [9, 4, 1, 0, 1, 4, 9];
let s = smallest(&parabola);
assert_eq!(*s, 0);  // fine: parabola still alive
```

関数シグネチャに生存期間の情報を持たせることによって、関数に渡す参照と関数が返す参照の関係をコンパイラが判断し、安全に使われることを保証することができます。

構造体に格納された参照を見てみます。

```
// This does not complie.
struct S {
    r: &i32
}

let s;
{
    let x = 10;
    s = S { r: &x };
}
assert_eq!(*s.r, 10);   // bad: read from dropped `x`
```

参照型が他の型定義に含まれている場合、その生存期間を明示的に書かなければなりません。

```
struct S {
    r: &'static i32
}
```

これでは制約が強すぎます。
他の方法としては、生存期間パラメータ `'a` を型に追加し、これを参照 `r` に使う方法があります。

```
struct S<'a> {
    r: &'a i32
}
```

生存期間パラメータが指定された型を、他の型に内包した場合をみてみます。

```
struct D {
    s: S    // not adequate
}
```

`S` に生存期間を指定せずに参照をおいた場合と同様です。

```
struct D {
    s: S<'static>
}
```

```
struct D<'a> {
    s: S<'a>
}
```

上の二つのようにして明示的に書く必要があります。

これまでに、関数シグネチャが、それに渡した参照をどう扱うかを明示していることを示しました。
ここでは型に対して同じことをしています。
型の生存期間パラメータは、興味深い(`'static`ではない)生存期間を持つ参照を保持しているかどうか、もし保持しているならばその生存期間がどうなるかを明示します。

例として、以下の関数を考えます。

```
fn parse_record<'i>(input: &'i [u8]) -> Record<'i>
```

`Record` 型の中身を全くみなくとも、 `parse_record` から返却された `Record` に含まれる参照はこの関数に引数として渡した入力バッファの中身を指していて、それ以外を指すことがないことがわかります。

生存期間を持つのは参照や `S` のような型だけではありません。 Rust では `i32` や `String` を含むあらゆる型が生存期間を保ちます。
多くの場合には `'static` つまりいくらでも生きていることができます。

個別の生存期間パラメータを使用することができます。

```
struct S<'a> {
    x: &'a i32,
    y: &'a i32,
}
```

どちらの参照も同じ生存期間を持ちます。
しかし、これは次のようなコードでは問題になります。

```
let x = 10;
let r;
{
    let y = 20;
    {
        let s = S  { x: &x, y: &y };
        r = s.x;
    }
}
println!("{}", r);
```

このコードはダングリングポインタを作るわけではありません。
しかし、 `x` と `y` の生存期間が同じ生存期間 `'a` を持つという条件を満たしません。
`S` の定義を次のように変更するとよいです。

```
struct S<'a, 'b> {
    x: &'a i32,
    y: &'b i32,
}
```

関数シグネチャでも同じようなことができます。

```
fn f<'a>(r: &'a i32, s: &'a i32) -> &'a i32 { r } // perhaps too tight
fn f<'a, 'b>(r: &'a i32, s: &'b i32) -> &'a i32 { r }   // looser
```

生存期間パラメータを省略することができます。

```
struct S<'a, 'b> {
    x: &'a i32,
    y: &'b i32,
}

fn sum_r_xy(r: &i32, s: S) -> i32 {
    r + s.x + s.y
}
```

この関数シグネチャは以下の省略形です。
Rust 処理系が、必要な場所にそれぞれ独立した生存期間を割り当ててくれます。

```
fn sum_r_xy<'a, 'b, 'c>(r: &'a iu32, s: S<'b, 'c>) -> i32
```

参照などの、生存期間パラメータを持つ型を返す場合でも、曖昧さがない場合にはコンパイラが単純に書けるようにしてくれます。
コンパイラは、関数の仮引数に生存期間が 1 つしかない場合には、それを返り値に出現するすべての生存期間と同じだと想定します。

```
fn first_third(point: &[i32; 3]) -> (&i32, &i32) {
    (&point[0], &point[2])
}
```

この関数の生存期間を全て書き下すと以下のようになります。

```
fn first_third<'a>(point: &'a [i32; 3]) -> (&'a i32, &a' i32)
```

仮引数に複数の生存期間があらわれる場合は、どちらかを優先して返り値に用いる理由はないので、コンパイラは、プログラマに明示的に書くように求めます。

関数が何らかの型に対するメソッドで、 `self` 仮引数を参照で撮る場合にはこれを優先し、 `self` の生存期間を、返す値に存在するすべての生存期間にします。

```
struct StringTable {
    elements: Vec<String>,
}

impl StringTable {
    fn find_py_prefix(&self, prefix: &str) -> Option<&String> {
        for i in 0..self.elements.len() {
            if self.elements[i].starts_with(prefix) {
                return Some(&self.elements[i]);
            }
        }
        None
    }
}
```

関数のシグネチャを、全て書くと次のようになります。

```
fn find_py_prefix<'a, 'b>(&'a self, prefix: &'b str) -> Option<&'a String>
```

Rust は、何かを借用するのであれば、 `self` から借用すると予想します。

これらは単なる省略形で、プログラマが驚くような自体を避けつつプログラマを補助するものにすぎません。
プログラマの意図と違うのであれば、いつでも生存期間を明示的に書くことができます。

## 共有と変更

他にもダングリンクポインタを作る場合があります。

```
let v = vec![4, 8, 19, 27, 34, 10];
let r = &v;
let aside = v;  // move vector to aside
r[0];           // bad: uses `v`, which is now uninitlized
```

共有参照は、生存期間中ずっと参照先を読み出しのみ可能な状態にします。
参照先に代入したり、値を別のところに移動することはできません。

このコードでは、ベクタを移動しようとする部分が `r` の生存期間の範囲内なので、コンパイラがエラーをだします。
下のように変更すれば、問題なくコンパイルできます。

```
let v = vec![4, 8, 19, 27, 34, 10];
{
    let r = &v;
    r[0];   // ok: vector is still there
}
let aside = v;
```

他にも以下のような状況が考えられます。

```
fn extend(vec: &mut Vec<f64>, slice: &[f64]) {
    for elt in slice {
        vec.push(*elt);
    }
}
```

これは、標準ライブラリでベクタに対して用意されておる `extend_from_slice` メソッドの、柔軟でないバージョンです。

```
let mut wave = Vec::new();
let head = vec![0.0, 1.0];
let tail = [0.0, -1.0];

extend(&mut wave, &head);   // extend wave with another vector
extend(&mut wave, &tail);   // extend wave with an array

extend(&mut wave, &wave);
assert_eq!(wave, vec![0.0, 1.0, 0.0, -1.0, 0.0, 1.0, 0.0, -1.0]);
```

一見するとこれで良さそうに見えるかもしれませんが、ベクタに要素を追加する際、バッファの容量が足りなくなると容量の大きい新しいバッファを確保しなければな離ません。
`wave` が 4 要素のバッファでできていて、 `extend` が 5 つ目の要素を追加しようとすると、大きいバッファを取り直さなければならないとします。

関数 `extend` の仮引数 `vec` は、呼び出し元が所有する `wave` を借用していて、`wave` は 8 要素格納できる新たなバッファを確保しています。
しかし、 `slice` は、すでにドロップされた古い 4 要素のバッファを指し続けています。
これに対して、 Rust はエラーを返します。

言い換えると、ベクタに対する可変参照を借用することもできるし、その要素に対する共有参照を借用することもできますが、これらの参照の生存期間は重なってはなりません。
この場合は、両方の参照の生存期間が関数 `extend` の呼び出しを包含しているので、 Rust はこのコードを拒絶します。
このエラーは Rust の可変参照と共有参照のルールに対する違反に起因します。

- 共有アクセスは読み出しのみにアクセスになります。
共有参照が借用した値は読み出すことしかできません。
共有参照の生存期間の間、参照先だけではなく、参照先から辿ることのできるものは全て、どのような方法でも変更することはできません。
その構造体内のなんであれ、可変参照で参照することはできません。
所有者が読み出ししかできない状態になっているからです。
共有参照で参照されているものは完全に凍結されます。

- 可変アクセスは排他アクセスになります。
可変参照で借用された値は、その参照を通じてのみアクセスすることができます。
可変参照の生存期間の間、参照先および参照先からたどることのできるもの全てに対して、他のパスから参照することはできません。
可変参照の生存期間と重なっていてよいのは、その可変参照から借用した参照だけです。

いずれの場合も、参照先への所有の経路上にある値は、参照の生存期間中は変更できません。
共有参照の場合は、経路上にある値は読み出すのみ可能となります。
可変参照の場合は、完全にアクセスできなくなります。

```
let mut x = 10;
let r1 = &x;
let r2 = &x;    // ok: multiple shared borrows permitted
x += 10;        // error: cannot assign to `x` because it is borrowed

let m = &mut x; // error: cannot borrow `x` as mutable because it is also borrowed as immutable

println!("{}, {}, {}", r1, r2, m);  // the references are used here, so their lifetimes must last at least this long

let mut y = 20;
let m1 = &mut y;
let m2 = &mut y;    // error: cannot borrow as mutable more than once
let z = y;          // error: cannot use `y` because it was mutably borrowed

println!("{}, {}, {}", m1, m2, z);  // references are used here

let mut w = (107, 109);
let r = &w;
let r0 = &r.0;      // ok: reborrowing shared as shared
let m1 = &mut r.1;  // error: cannot reborrow shared as mutable

println!("{}", r0); // r0 gets used here

let mut v = (136, 139);
let m = &mut v;
let m0 = &mut m.0;  // ok: reborrowing mutable from mutable
*m0 = 137;

let r1 = &m.1;      // ok: reborrowing shared from mutable, and doesn't overlap with m0
v.1;                // error: accwss through other paths still forbidden

println!("{}", r1); // r1 gets used here
```


