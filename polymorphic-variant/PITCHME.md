## 多相バリアント入門
### 2018/05/18 勝島達也

---

### 話すこと

- 多相バリアントの基本的な使い方


### 話さないこと

- 多相バリアントの理論

+++

### **注意事項**

- 基本的に[『プログラミング in OCaml』五十嵐淳 著](http://www.fos.kuis.kyoto-u.ac.jp/~igarashi/OCaml/) の14章の内容の**一部**を取り扱ったものになります
  - 細かい制約の説明は省いてます

- サンプルコードはOCamlで書かれてます。  
  - 読み方が分からなければ、随時止めて質問してもらって大丈夫です。

---

### とりあえずバリアントの話

### バリアントとは

異なる種類の値を同じ型の値として混ぜて扱うためのデータ構造

+++

### とりあえずバリアントの話

### 具体例

```ocaml
type color = Red | Blue | Yellow
```

```ocaml
type 'a option = None | Some of 'a
```

+++

### とりあえずバリアントの話

### Scalaで言うと

```scala
sealed trait Color
case object Red extends Color
case object Blue extends Color
case object Yellow extends Color
```

```scala
sealed trait Option[+A]
case object None extends Option[Nothing]
case class Some[+A](x: A) extends Option[A]
```

---
### 多相バリアントとは

一つのコンストラクタを複数の型のために使われることを許す仕組み

---

### お気持ち

- 同じコンストラクタが別の型に使えたらいいのに

- 一旦定義した型にあとからコンストラクタが追加できたらいいのに

  
後ほど詳しく話します。 |

---

### 使い方

#### 記法

```ocaml
`Red
```

`(バッククオート) + (大文字始まりのコンストラクタ名)`

で表記する。

+++

### 使い方

#### 使用例１

事前に型宣言・定義をせずに使える

**コード**
```ocaml
`Red
```
**型**
```ocaml
[> `Red ]
```

+++

### 使い方

#### 使用例２

引数付きのコンストラクタ

**コード**
```ocaml
`Some 42
```
**型**
```ocaml
[> `Some of int ]
```

+++

### 使い方

#### 使用例３

ひとつの型の値に、異なるコンストラクタが含まれる場合

**コード**
```ocaml
let f b =
  if b then `Blue else `Red
```
**型**
```ocaml
val f : bool -> [> `Blue | `Red ]
```

---

### 疑問

Q: `事前に型宣言・定義をせずに使える`、 こんなことを許して網羅性チェックとか大丈夫？

---

### 多相バリアントのコンストラクタを使用する例
#### 関数定義

定義
```ocaml
let color_to_signal color =
  match color with
  | `Red -> "stop"
  | `Blue -> "go"
```

型
```ocaml
val color_to_com : [< `Blue | `Red ] -> string
```

+++

#### 多相バリアントのコンストラクタを使用する例
#### 関数の使用例

定義
```ocaml
let color_to_signal color =
  match color with
  | `Red -> "stop"
  | `Blue -> "go"
```

`` `Red`` に適用
```ocaml
color_to_com `Red
```

`"stop"` が返る

+++
#### 多相バリアントのコンストラクタを使用する例
#### 関数の使用例

コード
```ocaml
let color_to_signal color =
  match color with
  | `Red -> "stop"
  | `Blue -> "go"
```

`` `Gold`` に適用
```ocaml
color_to_com `Gold
```

結果
```ocaml
Characters 13-18:
  color_to_com `Gold
               ^^^^^
Error: This expression has type [> `Gold ]
       but an expression was expected of type [< `Blue | `Red ]
       The second variant type does not allow tag(s) `Gold
```

型エラー（not 実行時エラー）

コンストラクタを**使用する側**が制約を掛けてくれる |

---
### 型の説明

``[> `Red ]`` とか ``[< `Blue | `Red ]`` とは何なのか


+++
### 型の説明

#### 記法

``[> `Red ]``、 ``[< `Blue | `Red ]``

読み方としては `[> ]` （or `[< ]`）にコンストラクタが囲まれている、となります

+++
### 型の説明

#### 結局なに？

``[> `Red ]``、 ``[< `Blue | `Red ]``

動く範囲が制限された型変数（`'a` の変種(?)）.

例えば、``[> `Red ]`` は `` `Red``を必ず含むような任意の型（本当はもう少し制約がありますが省略）、  
逆に、 ``[< `Blue | `Red ]`` は、`` `Blue``か`` `Red`` しか含まれないような任意の型を表す


+++
### 型の説明

#### もう少し噛み砕いて

もう一つ、`[ ]` という型の記法を紹介します。

ex) ``[ `Red ]``, ``[ `Blue | `Red ]``

これで表されるものは具体型（not 型変数）です。


+++
### 型の説明

#### もう少し噛み砕いて

``[> `Red ]`` は `` `Red``を必ず含むような任意の型。  

つまり、``[ `Red ]``, ``[ `Blue | `Red ]``, ``[ `Blue | `Red | `Black ]``   
などは全て``[> `Red ]`` 型の具体型

逆に、 ``[ `Yellow ]``, ``[ `Blue | `Black ]`` などはこれに該当しない

+++
### 型の説明

#### もう少し噛み砕いて

``[< `Blue | `Red ]`` は、`` `Blue``か`` `Red`` しか含まれないような任意の型。

つまり、``[ `Blue | `Red ]``, ``[ `Red ]``, ``[ `Blue ]``   
は全て``[< `Blue | `Red ]`` 型の具体型。

逆に、 ``[ `Yellow ]``, ``[ `Blue | `Red | `Black ]`` などはこれに該当しない

---

### [再掲] 多相バリアントのコンストラクタを使用する例
#### 関数定義

定義
```ocaml
let color_to_signal color =
  match color with
  | `Red -> "stop"
  | `Blue -> "go"
```

型
```ocaml
val color_to_com : [< `Blue | `Red ] -> string
```

網羅性チェックも大丈夫！

---

### [再掲] お気持ち

- 同じコンストラクタが別の型に使えたらいいのに

- 一旦定義した型にあとからコンストラクタが追加できたらいいのに

---
### 信号機の例
#### 通常のバリアントの場合

プログラム内で、基本は２色信号を扱うけど、一部で３色信号も使われる場合。  

どう書きましょう？

```ocaml
type signal2 = Red | Blue
```

```ocaml
type signal3 = Red | Blue | Yellow
```

+++
### 信号機の例
#### 通常のバリアントの場合
#### 案１: それぞれに処理を用意する

```ocaml
type signal2 = Red | Blue

let signal2_to_str color =
  match color with
  | Red -> "go"
  | Blue -> "stop"
```

```ocaml
type signal3 = Red | Blue | Yellow

let signal3_to_str color =
  match color with
  | Red -> "go"
  | Blue -> "stop"
  | Yellow -> "should stop"
```

### ボイラープレート(´･_･`)

+++
### 信号機の例
#### 通常のバリアントの場合
#### 案2: ３色信号として統一する

```ocaml
type signal3 = Red | Blue | Yellow

(* ２色信号が期待される場合の関数 *)
let signal2_to_str color =
  match color with
  | Red -> "go"
  | Blue -> "stop"
  | Yellow -> ???
```
@[8](unusedなマッチケース(´･_･`))

---
### 信号機の例
#### 多相バリアントの場合

型宣言
```ocaml
type signal2 = [ `Red | `Blue ] 
```

２色信号用の関数
```ocaml
let signal2_to_str color =
  match color with
  | `Red -> "go"
  | `Blue -> "stop"
```

関数の型
```ocaml
val signal2_to_str : [< `Blue | `Red ] -> string
```

+++
### 信号機の例
#### 多相バリアントの場合

型宣言
```ocaml
type signal3 = [ signal2 | `Yellow ] 
```

３色信号用の関数
```ocaml
let signal3_to_str color =
  match color with
  | #signal2 as x -> signal2_to_str x
  | `Yellow -> "should stop"
```
@[3](signal2用の関数に処理を投げれる)

関数の型
```ocaml
val signal3_to_str : [< `Blue | `Red | `Yellow ] -> string
```

---

### 嬉しみ

- 同じコンストラクタが別の型に使える

- 一旦定義した型にあとからコンストラクタが追加できる


---
### 他にも面白い例

**head**

定義
```ocaml
let head l =
  match l with
  | `Cons (h, `Nil) -> h
  | `Cons (h, `Cons (_, _)) -> h
```

型
```ocaml
val head : [< `Cons of 'a * [< `Cons of 'b * 'c | `Nil ] ] -> 'a
```

+++
### 面白い例

**head**

定義
```ocaml
let head l =
  match l with
  | `Cons (h, `Nil) -> h
  | `Cons (h, `Cons (_, _)) -> h
```

使用例
```ocaml
head (`Cons (5, `Nil))
```

結果
```ocaml
5
```

+++
### 面白い例

**head**

定義
```ocaml
let head l =
  match l with
  | `Cons (h, `Nil) -> h
  | `Cons (h, `Cons (_, _)) -> h
```

使用例
```ocaml
head `Nil
```

結果
```ocaml
Characters 5-9:
  head `Nil
       ^^^^
Error: This expression has type [> `Nil ]
       but an expression was expected of type
         [< `Cons of 'a * [< `Cons of 'b * 'c | `Nil ] ]
       The second variant type does not allow tag(s) `Nil
```
型エラー（not 実行時エラー）


---
 ### 一方でデメリットも・・

 - プログラムの型推論・型 の表記が，かなり複雑になる
 - 型推論が気を効かせすぎる（時間の都合で具体例を出せませんでしたが・・）
 
---

### まとめ

- 一つのコンストラクタを複数の型のために使える
- 通常のバリアントでは書けないような型制約が書ける
- デメリットも多いので、特別な理由が無ければ普通のバリアントを使おう
- 他にも面白い例（再帰的なデータ構造の拡張例とか）が出てくるので、[『プログラミング in OCaml』](http://www.fos.kuis.kyoto-u.ac.jp/~igarashi/OCaml/)を読もう！ね！
