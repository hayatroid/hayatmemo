---
title: グリッド上の探索
---

Rust でグリッド上の探索をするとき、座標を `usize` で持ちたいが、差分の $-1$ を `usize` で持つことができず、そのまま足し算できないといった問題が起こる。

これを解決するための方法をいくつか示す。

## 1. `usize::wrapping_add_signed` を使う方法

デバッグビルドでもリリースビルドでも使える。わかりやすい。

```rs
for (dx, dy) in [(1, 0), (0, 1), (-1, 0), (0, -1)] {
    let (nx, ny) = (x.wrapping_add_signed(dx), y.wrapping_add_signed(dy));
    if nx < h && ny < w {
        todo!();
    }
}
```

## 2. `!0` を足す方法

リリースビルドでないと使えないが、タイプ数を極限まで減らせる。

```rs
for (dx, dy) in [(1, 0), (0, 1), (!0, 0), (0, !0)] {
    let (nx, ny) = (x + dx, y + dy);
    if nx < h && ny < w {
        todo!();
    }
}
```

> [!tip]
>
> 一般に、オーバーフローを無視すると `-n == !n + 1` が成り立つ（[$2$ の補数](https://ja.wikipedia.org/wiki/2%E3%81%AE%E8%A3%9C%E6%95%B0)）。
> これより、オーバーフローを無視するリリースビルドにおいて、`x - 1` は `x + !0` と等価であるし、`x - 2` は `x + !1` と等価である。

## 3. 型変換する方法

なにも工夫せずに書くとこうなる。冗長。

```rs
for (dx, dy) in [(1, 0), (0, 1), (-1, 0), (0, -1)] {
    let (nx, ny) = (x as isize + dx, y as isize + dy);
    if nx < 0 || ny < 0 {
        continue;
    }
    let (nx, ny) = (x as usize, y as usize);
    if nx < h && ny < w {
        todo!();
    }
}
```
