---
title: グリッド上の探索
---

$H \times W$ のグリッドにおいて、$(x, y)$ の上下左右マスのうちグリッドをはみ出さないマスを探索する際の、効率的な実装を考える。

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

`.wrapping_add_signed(-1)` は `.wrapping_add(!0)` と等価である。
さらにリリースビルドでは `+ !0` とも等価である。
これより 1. は以下のように書き換えられる。

リリースビルドでないと使えないが、タイプ数を極限まで減らせる。

```rs
for (dx, dy) in [(1, 0), (0, 1), (!0, 0), (0, !0)] {
    let (nx, ny) = (x + dx, y + dy);
    if nx < h && ny < w {
        todo!();
    }
}
```

## 3. 型変換する方法

つらい。型変換 → 非負であるかチェック → 型変換 の手順を踏まざるを得ず、冗長。

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
