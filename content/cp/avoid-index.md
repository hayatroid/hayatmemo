---
title: "添字を書かないということ"
description: "Rust のベクタやイテレータに実装されたメソッドを用いることで、添字を隠蔽し、ミスを防げることがある。その例をいくつか示す。"
---

Rust のベクタやイテレータに実装されたメソッドを用いることで、添字を隠蔽し、ミスを防げることがある。

その例をいくつか示す。

## 等比数列かどうか判定する例

等比数列かどうかは、列の連続 $3$ 項に対する判定を行えばよい。

これは次のように実装できるが、列の長さが $3$ に満たない場合でも判定できているか、境界までくまなく判定できているかが怖い。

```rs
let ok = if a.len() < 3 {
    true
} else {
    (0..=a.len() - 3).all(|i| a[i] * a[i + 2] == a[i + 1] * a[i + 1])
};
```

ベクタやスライスに対する `windows` を用いると、境界を意識せずに済む。

```rs
let ok = a.windows(3).all(|p| p[0] * p[2] == p[1] * p[1]);
```

また、itertools の `tuple_windows` を用いると、添字を完全に隠蔽できる。

```rs
// use itertools::Itertools;
let ok = a.iter().tuple_windows().all(|(x, y, z)| x * z == y * y);
```

> [!example]
>
> 上記のコードを用いて、次の問題を解くことができる。
>
> - [AtCoder Beginner Contest 390 - B - Geometric Sequence](https://atcoder.jp/contests/abc390/tasks/abc390_b)
>   - [提出 1](https://atcoder.jp/contests/abc390/submissions/69794711)（添字を書く）
>   - [提出 2](https://atcoder.jp/contests/abc390/submissions/69794713)（添字を書かない・itertools なし）
>   - [提出 3](https://atcoder.jp/contests/abc390/submissions/69794729)（添字を書かない・itertools あり）

## 畳み込みをする例

畳み込みでは、列を $2$ 冪のチャンクに区切り、隣接チャンク・同一インデックスの値同士に演算を施す場面がある。

これは次のように実装できるが、外側のループでは `i` のオフセットが、内側のループでは `i + j` のオフセットがあることを忘れてしまうのが怖い。

```rs
for i in (0..n).step_by(len * 2) {
    let mut w = M::new(1);
    for j in 0..len {
        let (u, v) = (a[i + j], a[i + j + len]);
        (a[i + j], a[i + j + len]) = (u + v * w, u - v * w);
        w *= w_len;
    }
}
```

ベクタやスライスに対する `chunks_mut` や `split_at_mut` を用いると、オフセットを意識せずに済む。
また、長さ $\mathrm{len} \times 2$ のチャンク内で操作を行うことで、関心のあるチャンク以外へのアクセスを防げる。

```rs
for a in a.chunks_mut(len * 2) {
    let mut w = M::new(1);
    let (u, v) = a.split_at_mut(len);
    for (u, v) in u.iter_mut().zip(v) {
        (*u, *v) = (*u + *v * w, *u - *v * w);
        w *= w_len;
    }
}
```

また、itertools の `tuples` を用いると、長さ $\mathrm{len}$ のチャンクを先頭から 2 つずつ取り出す操作が直感的に書ける。

```rs
// use itertools::Itertools;
for (u, v) in a.chunks_mut(len).tuples() {
    let mut w = M::new(1);
    for (u, v) in u.iter_mut().zip(v) {
        (*u, *v) = (*u + *v * w, *u - *v * w);
        w *= w_len;
    }
}
```

> [!example]
>
> 上記のコードを用いて、次の問題を解くことができる。
>
> - [AtCoder Library Practice Contest - F - Convolution](https://atcoder.jp/contests/practice2/tasks/practice2_f)
>   - [提出 1](https://atcoder.jp/contests/practice2/submissions/69794883)（添字を書く）
>   - [提出 2](https://atcoder.jp/contests/practice2/submissions/69794893)（添字を書かない・itertools なし）
>   - [提出 3](https://atcoder.jp/contests/practice2/submissions/69794905)（添字を書かない・itertools あり）
