# 所有権と移動

メモリの管理に関しては、プログラミング言語に次の 2 つの特徴があることが望ましいです。

- メモリがプログラマが選んだタイミングで適切に解放されること。これによってプログラムのメモリ消費を制御できること。

- 解放済みのオブジェクトへのポインタを使ってしまうことがないこと。このようなことをすると未定義動作になり、クラッシュやセキュリティホールにつながる。

しかしこれらは相反するように思えます。
ポインタが存在するうちに値を解放すればそのポインタの参照する先がなくなってしまいます。
これをダングリングポインタといいます。

Rust は安全でかつ高性能な言語を目指しているので、これらの 2 つの両方を成立させようとします。

## 所有権

Rust では所有権は言語そのものに組み込まれていてコンパイル時チェックによって強制されます。
Rust ではすべての値はその生存期間を決定する唯一の所有者を持ちます。
所有者が解放 (drop) された場合、所有されていた値もドロップされます。
このルールは、任意の値の生存期間をコードを見るだけで知ることを容易にし、システム記述用の言語が提供するべきオブジェクトの生存期間の完全な制御を実現するためのルールです。

変数はその値を所有します。その変数が宣言されたブロックから、プログラムの実行が離れたときに、変数はドロップされ、それに伴って値もドロップされます。

```
fn print_padovan() {
    let mut padovan = bec![1, 1, 1];    // allocated here
    for i in 3..10 {
        let next = padovan[i - 3] + padovan[i - 2];
        padovan.push(next);
    }
    println!("P(1..10) = {:?}", padovan);
}   // drop here
```

最初の時点では、スタックフレーム上に `padovan` 変数が バッファ、容量、長さの要素を持っており、ヒープ上に 実際の `Vec<i32>` があります。
変数 `padovan` がベクタを所有しているので、関数の最後でスコープから外れると、このベクタはドロップされます。
そしてベクタがバッファを所有しているのでバッファも同時にドロップされます。

Rust の `Box` 型は、所有の別の例です。
`Box<T>` はヒープ上の型 `T` の値へのポインタです。
`Box::new(v)` を呼ぶと、ヒープ上にメモリを確保し、値 `v` をそこに移動し、そのヒープ空間を指す `Box` を返します。
`Box` がその空間を所有しているので、 `Box` がドロップされると、その空間も解放されます。

```
{
    let point = Box::new((0.625, 0.5)); // point allocated here
    let label = format!("{:?}", point); // label allocated here
    assert_eq!(label, "(0.625, 0.5)");  
}   // both dropped here
```

プログラムが `Box::new` を呼び出すと 2 つの `f64` 値がヒープ上に作られ、引数の `(0.625, 0.5)` がその空間に移動されます。
`assert_eq!` に制御が到達した時点では以下のようになります。
スタックフレームに、変数 `point, label` が保持されており、それぞれ所有するヒープ上のメモリを参照しています。
これらの変数がドロップされると、これらに所有されているヒープ上に確保されたメモリも同時に解放されます。

変数が値を所有するのと同じように、構造体はそのフィールドを、タプルや配列やベクタは、その要素を所有します。

```
struct Person { name: String, birth: i32 }

let mut composers = Vec::new();
composers.push(Person { name: "Palestrina".to_string(), birth: 1525 });
composers.push(Person { name: "Dowland".to_string(), birth: 1563 });
composers.push(Person { name: "Lully".to_string(), birth: 1632 });

for composer in &composers {
    println!("{}, born {}", composer.name, composer.birth);
}
```

ここで `composers` は、フィールドとして文字列と数値を持つ構造体のベクタ `Vec<Person>` です。
ここにはいくつのもの所有関係が出てきていますが、それぞれはごく単純です。
`composer` がベクタを所有し、ベクタがその要素を所有しています。
ベクタの要素はそれぞれ `Person` 構造体で、個々の `Person` 構造体は、それぞれフィールドを所有しています。
文字列フィールドはテキストを所有しています。
制御が `composers` を宣言したスコープを離れると、プログラムは `composers` の値をドロップし、それに伴ってツリー全体をドロップします。

所有者と所有される値はツリー構造をなします。
所有者を親とするなら、所有される値は子になります。
個々のツリーの究極のルートは変数です。
変数がスコープから外れると、ツリー全体が削除されます。
ツリーよりも複雑な構造にしてしまう可能性のある再接続が Rust の単一所有者のルールによって禁止されています。
Rust プログラム中のすべての値は、何らかの変数がルートとなっているツリーのメンバです。

Rust での値のドロップは、何らかの形で所有権のツリーから取り除かれること、例えば変数がスコープから外れたり、ベクタの要素を削除することによって行われます。
この時点で Rust は、その値とそれが所有している全てが適切にドロップされていることを保証します。

ここにいくつかの方法で考え方を拡張しています。

- 値を 1 つの所有者から別の所有者に移動することができます。これによってツリーを構築し、変更し、分割することができます。

- 整数、浮動小数点数、文字などの非常に単純な型には、所有権のルールが適用されません。これらの型は `Copy` 型と呼ばれます。

- 標準ライブラリに、参照カウントポインタ型の `Rc, Arc` が用意されています。これらを用いると、一定の制限下で 1 つの値に対して複数の所有者を設けることができます。

- 値への「参照の借用」ができます。参照は所有権のないポインタで、生存期間が限定されています。


## 移動

Rust では、ほとんどすべての型が、変数への代入、関数への引数の受け渡し、関数からの返り値の返却の際にコピーされず、移動 (move) されます。
受け渡し元は、値の所有権を受け渡し先に引き渡し、未初期化状態になります。
以降、受け渡し先が値の生存期間を制御することになります。

```
let s = vec!["udon".to_string(), "ramen".to_string(), "soba".to_string()];
let t = s;
let u = s;
```

`s` の初期化が終わった時点で、スタックフレーム上には `s` のバッファ、容量、長さの要素があり、ヒープ上にベクタがあり、さらにヒープ上に `String` があります。
`t` を初期化した時点で、 `s` から `t` へ移動がおき、 `s` は未初期化状態になり、 `t` がベクタの所有権を持ちます。
`u` を初期化しようとすると、初期化されていない値 `s` を `u` に代入しようとするのでエラーを出します。

これによって、代入は安価であり、所有権も明確になっています。
プログラマが支払う代償は、それぞれの変数が構造体の独立したコピーを持つようにしたければ、ベクタの `clone` メソッドを呼んで、ベクタとその要素をディープコピーする必要があることです。

```
let s = vec!["udon".to_string(), "ramen".to_string(), "soba".to_string()];
let t = s.clone();
let u = s.clone();
```

Rust の参照カウントを用いることで、このようにしなくても解決する方法があります。

変数への代入は少し様子が異なります。
すでに初期化されている変数に値を移動すると、代入される変数に以前は言っていた値はドロップされます。

```
let s = "Govinda".to_string();
s = "Siddhartha".to_string();   // value "Govinda" dropped here
```

このコードでは、 `s` に文字列 `Siddhartha` を代入される際に、 `s` に入っていた `Govinda` がまずドロップされます。

```
let mut s = "Govinda".to_string();
let t = s;
s = "Siddhartha".to_string()    // nothing is dropped here
```

この場合、元の文字列の所有権を `t` が `s` から引き継ぐので、 `s` に代入しようとする時点で `s` は未初期化状態になっています。
この場合にはドロップされる文字列はないです。

Rust はこの移動という仕掛けをほとんどすべての値の使用に適用します。
関数へ引数を渡すと、引数の所有権が関数の仮引数に移動します。
関数から値を返す際には、値の所有権が関数から呼び出し元に移動します。
タプルを作れば、値がタプルに移動します。

2 つ注意するべき点があります。

- 移動されるのは値そのものだけであって、それが保有するヒープ上のストレージは移動されません。ベクタや文字列の場合、値そのものは 3 ワードのヘッダ部分だけになります。

- Rust コンパイラのコード生成はこれらの移動を「見透かす」のが得意です。実際には機械語レベルで見ると、値が最終的に移動する場所に始めからオブジェクトが作られていることが多いです。

一般的な原則として、ある変数に関して、その変数から値が移動してでていき、それから絶対に新しい値が与えられていないならば、未初期化状態と考えることができます。

```
let x = vec![10, 20, 30];
if c {
    f(x);   // ... ok to move from x here
} else {
    g(x);   // ... and ok to alse move from x here
}
h(x); // bad: x is uninitialized here if either path uses it
```

```
let x = vec![10, 20, 30];
while f() {
    g(x);       // bad: x would be moved in first iteration,
                // uninitialized in second.
}
```

```
let mut x = vec![10, 20, 30];
while f() {
    g(x);       // move from x
    x = h();    // give x a fresh value
}
```

値の移動によって、移動さきがその値の所有権を持つようになるため、移動元が未初期化状態になることを見てきました。
しかし、値の所有者によっては未初期化状態にすることが難しいものもあります。

```
// build a vector of the strings "101", "102", ..., "105";
let mut v = Vec::new();
for i in 101..106 {
    v.push(i.to_string());
}

// Pull out random elements from the vector.
let third = v[2];   // error: Cannot move out of index of Vec
let fifth = v[4];   // here too
```

これができるようにするためには、ベクタの 3, 5 番目の要素が未初期化状態になったことを何らかの方法で記憶しておき、ベクタがドロップされるまでそれを管理しなければなりません。
これは明らかにシステムプログラミング用の言語として適切な動作ではありません。
実際にエラーを出します。

これを実現するには、まず `&v[2]` として借用することを考えます。他には以下の方法が考えられます。

```
// Build a vector of the strings "101", "102", ..., "105"
let mut v = Vec::new();
for i in 101..106 {
    v.push(i.to_string());
}

// 1. Pop a value off the end of the vector
let fifth = v.pop().expect("vector empty!");
assert_eq!(fifth, "105");

// 2. Move a value out of a given index in the vector, and move the last element into its spot.
let second = v.swap_remove(1);
assert_eq!(second, "102");

// 3. Swap in another value for the one we're taking out
let third = std::mem::replace(&mut v[2], "substitute".to_string());
assert_eq!(third, "103");

// Let's see what's left of out vector
assert_eq!(v, vec!["101", "104", "substitute"]);
```

これらの手法はいずれも要素をベクタから取り出しますが、ベクタの要素がかけることのないようにしています。

`Vec` のようなコレクション型は一般に、すべての要素をループで消費するためのメソッドを用意しています。

```
let v = vec!["liberte".to_string(), "egalite".to_string(), "fraternite".to_string()];
for mut s in v {
    s.push('!');
    println!("{}", s);
}
```

`for ... in v` のようにしてループにベクタを直接渡すと、ベクタは `v` から移動され、 `v` は未初期化状態になります。
`for` ループの内部構造がベクタの所有権をもち、要素に分解します。
繰り返すごとに、ループは新たな要素を変数 `s` に移動します。 `s` が文字列を所有しているので、出力する前に文字列を変更することができます。
この時点でベクタそのものは、コードから見えなくなっているので、ループの途中の、一部が空になったベクタの様子は観測できません。

コンパイラが管理できない形で、所有者から値を移動して取り出す必要があるなら、所有者の型を変更し値があるかどうかを動的に管理できるものに変更することを考慮するとよいです。

```
struct Person { name: Option<String>, birth: i32 }

let composers = Vec::new();
composers.push(Person { name: Some("Palestrina".to_string), birth: 1525 });

let first_name = composers[0].name; // Cannot move value which is index refered

// use replace
let first_name = std::mem::replace(&mut composers[0].name, None);
assert_eq!(first_name, "Palestrina".to_string());
assert_eq!(composers[0].name, None);

// same to replace
let fist_name = composers[0].name.take();
```

## コピー型

Rust は値を移動せずにコピーする型があり、そのような型を Copy 型 (Cope type) と呼びます。
代入元は初期化された状態のままで、代入前と同じ値として利用できます。
`Copy` 型を関数にわたしたりコンストラクタにわたした場合の動作も同様になります。

簡単なルールとしては、値をドロップする際に何か特別なことをしなければならない型は `Copy` 型ではないです。
`Vec` は要素を解放する必要があります。 `File` はファイルハンドルをクローズする必要があります。
`MutexGuard` は排他ロックを解放する必要があります。

ユーザー定義型が `Copy` 型にならないのは、単なるデフォルトの動作です。
構造体のすべてのフィールドが `Copy` 型であれば、構造体もコピー型にすることができます。

```
#[derive(Copy)]
struct Label { number: u32 }

#[derive(Copy)]
struct StringLabel { name: String } // error: the trait `Copy` may not be implemented for the type
```

## `Rc` と `Arc` : 所有権の共有

`Rc` と `Arc` は参照カウントのポインタです。
違いは、 `Arc` が複数スレッド間で直接共有しても安全なようにできていることだけです。
`Arc` という名前はアトミックな参照カウント (atomic reference count) からきています。
これに対して、 `Rc` 型は、高速だがスレッド安全ではないコードで参照カウントを管理します。

```
use std::rc::Rc;

// Rust can infer all these types; written out for clarify
let s: Rc<String> = Rc::new("shirataki".to_string());
let t: Rc<String> = s.clone();
let u: Rc<String> = s.clone();
```

すべての型 `T` に対して、 `Rc<T>` の値は、ヒープ上に確保された `T` とそれに付随する参照カウントをさすポインタになります。
`Rc<T>` をクローンすると、 `T` の値はコピーされず、同じものを指すポインタが作られ、参照カウントがインクリメントされます。

3 つの `Rc<String>` ポインタは、メモリ上の同じブロックを指していて、そのブロックには参照カウントと `String` が保持されています。
`Rc` 型のポインタ自体には通常の所有権ルールが適用されます。
最後に残った `Rc` がドロップされると、 `String` がドロップされます。

`String` の普通のメソッドはすべて、 `Rc<String>` に対して直接実行することができます。

```
assert_eq!(s.contains("shira"));
assert_eq!(t.find("taki"), Some(5));
println!("{} are quite chewy, almost bouncy, but lack flavor", u);
```

`Rc` ポインタに所有される値は不変となります。

```
s.push_str("  noodles");    // error: cannot borrow data in an `Rc` as mutable
```

Rust のメモリ安全生とスレッド安全性は、ある値が、共有されていると同時に可変にはならないことに依存しています。
Rust は、 `Rc` ポインタから参照されているものは共有されているものと想定するので、不変であるべきだと判断します。


