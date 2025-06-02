# 式

## 6.1 式言語

Rust はいわゆる式言語 (expression language) の 1 つです。
つまり、 Lisp にまで遡る古い伝統に従い、全ての式で行います。

Rust では、 `if, match` も値をつけることができます。

`match` 式を関数やマクロの引数にすることもできます。

## 6.2 優先順位と結合性

|式のタイプ | 例 | 関連するトレイト |
| ---- | ---- | ---- |
| 配列リテラル | `[1, 2, 3]` |  |
| 繰り返し配列リテラル | `[0; 50]` |  |
| タプル | `(6, "crullers")` |  |
| ---- | ---- | ---- |
| グループ化 | `(2 + 2)` |  |
| ブロック | `{ f(); g() }` |  |
| 制御フロー式 | `if ok { f() }, if ok { 1 } else { 0 }, if let Some(x) = f() { x } else { 0 }, match x { None => 0, _ => 1 }, for v in e { f(v); }, while ok { ok = f(); }, while let Some(x) = it.next() { f(x); }, loop { next_event(); }, break, continue, return 0` | `std::iter::IntoIterator` |
| マクロ実行 | `println!("ok")` |  |
| パス | `std::f64::consts::PI` |  |
| ---- | ---- | ---- |
| 構造体リテラル | `Point { x: 0, y: 0 }` |  |
| ---- | ---- | ---- |
| タプルのフィールドアクセス | `pair.0` | `Deref, DerefMut` |
| 構造体のフィールドアクセス | `point.x` | `Deref, DerefMut` |
| メソッド呼び出し | `point.translate(50, 50)` | `Deref, DerefMut` |
| 関数呼び出し | `stdin()` | `Fn(Arg0, ...) -> T, FnMut(Arg0, ...) -> T, FnOnce(Arg0, ...) -> T` |
| インデックス | `arr[0]` | `Index, IndexMut, Deref, DerefMut` |
| エラーチェック | `create dir("tmp")?` |  |
| ---- | ---- | ---- |
| 論理 / ビット単位 NOT | `!ok` | `Not` |
| 符号反転 | `-num` | `Neg` |
| 参照解決 | `*ptr` | `Deref, DerefMut` |
| 借用 | `&val` |  |
| ---- | ---- | ---- |
| 型キャスト | `x as u32` |  |
| ---- | ---- | ---- |
| 掛け算 | `n * 2` | `Mul` |
| 割り算 | `n / 2` | `Div` |
| 余り (剰余) | `n % 2` | `Rem` |
| ---- | ---- | ---- |
| 足し算 | `n + 1` | `Add` |
| 引き算 | `n - 1` | `Sub` |
| ---- | ---- | ---- |
| 左シフト | `n << 1` | `Shl` |
| 右シフト | `n >> 1` | `Shr` |
| ---- | ---- | ---- |
| ビット単位 AND | `n & 1` | `BitAnd` |
| ---- | ---- | ---- |
| ビット単位排他 OR | `n ^ 1` | `BitXor` |
| ---- | ---- | ---- |
| ビット単位 OR | `n | 1` | `BitOr` |
| ---- | ---- | ---- |
| 小なり | `n < 1` | `std::cmp::PartialOrd` |
| 小なりイコール | `n <= 1` | `std::cmp::PartialOrd` |
| 大なり | `n > 1` | `std::cmp::PartialOrd` |
| 大なりイコール | `n >= 1` | `std::cmp::PartialOrd` |
| イコール | `n == 1` | `std::cmp::PartialEq` |
| ノットイコール | `n != 1` | `std::cmp::PartialEq` |
| ---- | ---- | ---- |
| 論理 AND | `x.ok && y.ok` |  |
| ---- | ---- | ---- |
| 論理 OR | `x.ok || backup.ok` |  |
| ---- | ---- | ---- |
| 末尾を含まない範囲 | `start..stop` |  |
| 末尾を含む範囲 | `start..=stop` |  |
| ---- | ---- | ---- |
| 代入 | `x = val` |  |
| 複合代入 | `x *= 1, x /= 1, x %= 1, x += 1, x -= 1, x <<= 1, x >>= 1, x &= 1, x ^= 1, x |= 1` | `MulAssign, DivAssign, RemAssign, AddAssign, SubAssign, ShlAssign, ShrAssign, BitAndAssign, BitXorAssign, BitOrAssign` |
| ---- | ---- | ---- |
| クロージャー | `|x, y| x + y` |  |

## 6.3 ブロックとセミコロン

ブロックは最も一般的な式です。値を生み出すので、値を必要とする任意の場所で使うことができます。
ブロック中の最後の式の値がブロックの値となります。

```Rust
let msg = {
    // let-declaration: semicolon is always required
    let dandelion_control = puffball.open();

    // expression + semicolon: method is called, return value dropped
    dandelion_control.release_all_seeds(launch_codes);

    // expression with no semicolon: method is called,
    // return value stored in `msg`
    dandelion_control.get_status()
};
```

## 6.4 宣言

ブロックには、式とセミコロン以外に、任意の数の宣言を書くことができます。

```Rust
let name: type = expr;
```

型と初期化式は省略できますが、セミコロンは必須です。

```Rust
let name;
if user.has_nickname() {
    name = user.nickname();
} else {
    name = generate_unique_name();
    user.register(&name);
}
```

変数を初期化前に使用するとエラーになります。

下のように存在している変数を、再度宣言することができます。

```Rust
for line in file.lines() {
    let line = line?;
    ...
}
```

このブロックのこの行以降では、 2 つ目の宣言がこの変数名を置き換えることになります。
このような書き方はシャドーイングと呼ばれています。

ブロック内で、アイテムの宣言 (item declaration) を行うこともできます。
ここでいうアイテムとは、 `fn, struct, use` などのプログラムやモジュールに対してグルーバルに現れるものを指します。

`fn` がブロックの内部で宣言された場合、そのスコープはそのブロックとなります。
しかし、このブロック内部で宣言された `fn` は、そのスコープにたまたま存在したローカル変数や引数にはアクセスできません。

ブロックにモジュールを丸ごと入れることもできます。

## 6.5 if と match

```Rust
if condition1 {
    block1
} else if condition2 {
    block2
} else {
    block_n
}

match code {
    0 => println!("OK"),
    1 => println!("Wires Tangled"),
    2 => println!("User Asleep"),
    _ => println!("Unrecognized Error {}", code),
}

match parames.get("name") {
    Some(name) => println!("Hello, {}!", name),
    None => println!("Greetings stranger."),
}
```

match 式には、さまざまなパターン (pattern) を使用することができます。

```Rust
match value {
    pattern => expr,
}
```

Rust は、与えられた value に対して、ここのパターンを上から順に適用していきます。
パターンがマッチすると対応する expr が評価され、 match 式は終了します。
それ以降のパターンはチェックされません。
少なくとも一つのパターンがマッチせねばならず、全てのケースをカバーしていない match 式は許されません。

if 式や match 式の全てのブロックと分岐は同じ型の値を生成せねばなりません。

## if let 式

```Rust
if let pattern = expr {
    block1
} else {
    block2
}
```

この式は、 Option や Result からデータを取り出すのに便利です。

```Rust
if let Some(cookie) = request.session_cookie {
    return restore_attempt(err);
}

if let Err(err) = show_cheesy_anti_robot_task() {
    log_robot_attempt(err);
} else {
    session.mark_as_human();
}
```

if let 式は、パターンが 1 つしかない場合の match 式の省略形のようなものです。


## 6.7 ループ

Rust には、 4 つのループ式があります。

```Rust
while condition {
    block
}

while let pattern = expr {
    block
}

loop {
    block
}

for pattern in iterable {
    block
}
```

## 6.8 ループ内の制御フロー

break 式はループ式の中で用います。
loop のボディ部では、 break に値を与えると、その値が loop 式の値となります。

continue 式を用いるとループの次の回に飛ぶことができます。

ループに生存期間でラベル付け (labeled) することができます。

```Rust
'search:
for room in apartment {
    for spot in room.hiding_spots() {
        if spot.contains(keys) {
            println!("Your keys are {} in the {}", spot, room);
            break 'search;
        }
    }
}
```

break にはラベルと値の式の両方を与えることができます。
ラベルは、 continue にも使うことができます。


## 6.9 return 式

return 式は現在の関数から脱出し、値を呼び出し元に返します。
値のない return は、 return () の省略形です。


## 6.10 関数呼び出しとメソッド呼び出し

Rust で関数やメソッドを呼び出す構文は、他の多くの言語と同じです。

```Rust
let x = gcd(1302, 462);         // function call
let room = player.location()    // method call
```

Rust は、参照とその参照の指す値を明確に区別します。しかし、 `.` 演算子はそのルールを少し緩和します。メソッド呼び出すでは、 `player` は `Player, &Player, Box<Player>, Rc<Player>` でも構いません。必要に応じて自動的に参照解決したり、参照を借用したりするからです。

型関連関数を呼ぶこともできます。

```Rust
let mut numbers = Vec::new();   // type-associated function
```

Rust の関数呼び出しやメソッド呼び出しで面白いのは、 `Vec<T>` のような通常のジェネリック型の構文が使えないことです。
このような場合は、 `::<T>` を用います。

```Rust
return Vec::<i32>::with_capacity(1000); // ok, using ::<
let ramp = (0..n).collect::<Vec<i32>>();    // ok, using ::<
```

シンボル `::<` はターボフィッシュと呼ばれています。

## 6.11 フィールドと要素

構造体のフィールドには、馴染み深い構文でアクセスできます。
タプルのアクセス構文も同様だが、タプルのフィールドには名前ではなく番号が付いています。

配列やスライスやベクタの要素にアクセスするには角括弧を用います。

```Rust
game.black_pawns    // struct field
coords.1            // tuple element
pieces[i]           // array element
```

この 3 つの式は代入分の左辺に現れることができるので、左辺値 (lvalue) と呼ばれます。

```
game.black_pawns = 0x00ff_00000000_u64;
coords.1 = 0;
pieces[2] = Some(Peice::new(Black, Knight, coords));
```

配列やベクタからスライスを取り出すのは以下のようにします。

```Rust
let second_half = &game_moves[midpoint..end];
```

長さ `end - midpoint` のスライスを借用します。元の値は配列でもスライスでもベクタでも構いません。

`..` 演算子は、どちらのオペランドも省略できます。どのオペランドが存在するかによって 4 つの異なる型が生成されます。

```Rust
..      // RangeFull
a..     // RangeFrom { start: a }
..b     // RangeTo { end: b }
a..b    // Range { start: a, end: b }
```

後ろの 2 つは半開区間 (half-open) となります。
`..=` 演算子は、閉区間 (closed) を作ります。

```Rust
..=b    // RangeToInclusive { end: b }
a..=b   // RangeInclusive { start: a, end: b }
```

## 6.12 参照演算子

単項演算子 `&, &mut` については参照を取得する演算子です。

`*` は、参照で刺される値にアクセスするために用いられます。
これが必要となるのは、参照が指す値全体を読んだり書いたりしたい場合に限られます。

```Rust
let padovan: Vec<u64> = compute_padovan_sequence(n);
for elem in&padovan {
    draw_triangle(turtle, *elem);
}
```

## 6.13 代入

`=` 演算子は、 `mut` 変数やそのフィールドや要素に代入するために用いられます。
値が `Copy` 型ではない場合、代入は値の移動を行います。
値の所有権は代入元から代入先へ移転されます。
大隠喩先に値がすでにあった場合には、その値はドロップされます。
複合代入もサポートされています。

## 6.14 型キャスト

ある値をある型から別の型に変換するには、通常は、明示的にキャストする必要があります。

いくつかの自動変換が行われます。

- `&String` の値は、 `&str` にキャストなしで自動変換されます。
- `&Vec<i32>` の値は、 `&[i32]` に自動変換されます。
- `&Box<Chessboard>` の値は `&Chessboard` に自動変換されます。

これらは、参照解決型変換 (Deref 型変換) と呼ばれます。
組み込みトレイトの `Deref` を実装している型に適用されます。
この目的は、スマートポインタ型を、参照先の値と可能な限り同じように振る舞わせることです。


## 6.14 クロージャ

軽量の関数のような値です。

```
let is_even = |x| x % 2 == 0;
```

コンパイラが引数の型と返り値の型を推論してくれますが、関数の場合と同様に明示的に書くことができます。
クロージャの呼び出しには、関数の呼び出しと同じ構文を用います。

```
let is_even = |x: u64| -> bool x % 2 == 0;  // error
let is_even = |x: u64| -> bool { x % 2 == 0; }  // ok
```

返り値の値を書いた場合、構文上の問題でボディ部をブロックにしなければなりません。
