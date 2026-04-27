---
title: "型で守るということ"
description: "型を意図的に使い分けることで、ある種のミスをコンパイル時に弾けることがある。その例をいくつか示す。"
---

型を意図的に使い分けることで、ある種のミスをコンパイル時に弾けることがある。

その例をいくつか示す。

## `usize` と `u64` を使い分ける例

ナップザック DP では重さ `w` と価値 `v` を扱う。
どちらも `usize` で持つと、`dp[i - w] + v` と書くべき場所に `dp[i - v] + w` と書いてもコンパイルは通ってしまう。
しかも、添字も値も範囲内に収まることがあり、バグに気づきにくい。

そこで、重さを `usize` で、価値を `u64` で持つ。

```rs
input! {
    n: usize,
    cap: usize,
    wv: [(usize, u64); n],
}
let mut dp = vec![0; cap + 1];
for (w, v) in wv {
    for i in (w..=cap).rev() {
        dp[i] = dp[i].max(dp[i - w] + v);
        // dp[i - v]  // cannot subtract `u64` from `usize`
    }
}
```

Rust に暗黙の型変換がないので、両者は混ざらない。

> [!example]
>
> 上記のコードを用いて、次の問題を解くことができる。
>
> - [Educational DP Contest - D - Knapsack 1](https://atcoder.jp/contests/dp/tasks/dp_d)
>   - [提出](https://atcoder.jp/contests/dp/submissions/75314100)

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

内積 $v \cdot w$ は `Vector` のメソッドとして実装する。

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
