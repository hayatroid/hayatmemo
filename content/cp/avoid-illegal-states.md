---
title: "型で守るということ"
description: "型を意図的に使い分けることで、ある種のミスをコンパイル時に弾けることがある。その例をいくつか示す。"
---

型を意図的に使い分けることで、ある種のミスをコンパイル時に弾けることがある。

その例をいくつか示す。

## `usize` と `u64` を使い分ける例

グラフの問題では、頂点番号や辺の重みなど、次元の異なる値が入力に現れる。
これらをすべて `usize` で受け取ると、頂点番号と辺の重みを型で区別できない。

たとえば、本来 `dist[u]` と書く場所で `dist[w]` と書いてもコンパイルが通ってしまう。

そこで、頂点番号を `usize` で受け取り、辺の重みを `u64` で受け取る。
このとき、`u64` を添字として使えないので、誤った添字アクセスをコンパイルエラーにできる。

```rs
input! {
    n: usize,
    m: usize,
    uvw: [(usize, usize, u64); m],
}
let mut dist = vec![0; n];
for (u, v, w) in uvw {
    dist[w] += 1; // the type `[{integer}]` cannot be indexed by `u64`
}
```

Rust では、暗黙の型変換がないので、両者は混ざらない。

実務でも、`UserId` と `PostId` のような ID 系の取り違え防止に同じ手が使える。

## `Point` と `Vector` を使い分ける例

$2$ 次元幾何では、点とベクタを同じ `(f64, f64)` として持てる。
しかし、点とベクタでは意味を持つ演算が異なる。

たとえば、点 $P, Q$、ベクタ $v, w$、スカラ $c$ について、次の演算は自然に定義できる。

- $P + v$ は点になる
- $P - Q$ はベクタになる
- $v + w$ はベクタになる
- $vc$ はベクタになる
- $v \cdot w$ はスカラになる

一方で、$P + Q$ のような演算には自然な意味を与えにくい。
そこで、`Point` と `Vector` を別の型として定義し、意味を持つ演算だけを実装する。

```rs
#[derive(Clone, Copy)]
struct Point(f64, f64);

#[derive(Clone, Copy)]
struct Vector(f64, f64);
```

$P + v$ は、点をベクタだけ平行移動する演算である。
これは `Add<Vector> for Point` として実装し、戻り値を `Point` にする。

```rs
impl Add<Vector> for Point {
    type Output = Point;
    fn add(self, rhs: Vector) -> Point {
        Point(self.0 + rhs.0, self.1 + rhs.1)
    }
}
```

$P - Q$ は、点から点を引いてベクタを得る演算である。
これは `Sub<Point> for Point` として実装し、戻り値を `Vector` にする。

```rs
impl Sub<Point> for Point {
    type Output = Vector;
    fn sub(self, rhs: Point) -> Vector {
        Vector(self.0 - rhs.0, self.1 - rhs.1)
    }
}
```

同様に、$v + w$ は `Add<Vector> for Vector` として実装し、$vc$ は `Mul<f64> for Vector` として実装する。

```rs
impl Add<Vector> for Vector {
    type Output = Vector;
    fn add(self, rhs: Vector) -> Vector {
        Vector(self.0 + rhs.0, self.1 + rhs.1)
    }
}

impl Mul<f64> for Vector {
    type Output = Vector;
    fn mul(self, rhs: f64) -> Vector {
        Vector(self.0 * rhs, self.1 * rhs)
    }
}
```

内積 $v \cdot w$ は `impl Vector` のメソッドとして実装する。

```rs
impl Vector {
    fn dot(self, rhs: Vector) -> f64 {
        self.0 * rhs.0 + self.1 * rhs.1
    }
}
```

これで、たとえば点 $P$ から直線 $AB$ への射影を、意味を持つ演算だけで書ける。

```rs
let proj = a + (b - a) * ((p - a).dot(b - a) / (b - a).dot(b - a));
```

一方で、`Point + Point` のような実装していない演算は、書こうとしてもコンパイルが通らない。

> [!example]
>
> 上記のコードを用いて、次の問題を解くことができる。
>
> - [Aizu Online Judge - CGL_1_A - Projection](https://onlinejudge.u-aizu.ac.jp/courses/library/4/CGL/1/CGL_1_A)
>   - [提出](https://onlinejudge.u-aizu.ac.jp/solutions/problem/CGL_1_A/review/11442971/hayatroid/Rust)
