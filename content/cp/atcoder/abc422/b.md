---
title: "B. Looped Rope"
---

[問題はこちら](https://atcoder.jp/contests/abc422/tasks/abc422_b)

## 方針

[[./cp/grid-search | グリッド上の探索]] を行う。

## 回答

14–20 行目のようにして、上下左右で隣り合うマスのうち黒く塗られているものを数え上げることができる（[提出](https://atcoder.jp/contests/abc422/submissions/69246692)）。

```rs
use proconio::{input, marker::Chars};

fn main() {
    input! {
        h: usize,
        w: usize,
        s: [Chars; h],
    }
    for i in 0..h {
        for j in 0..w {
            if s[i][j] != '#' {
                continue;
            }
            let mut cnt = 0;
            for (di, dj) in [(1, 0), (0, 1), (!0, 0), (0, !0)] {
                let (ni, nj) = (i + di, j + dj);
                if ni < h && nj < w && s[ni][nj] == '#' {
                    cnt += 1;
                }
            }
            if cnt != 2 && cnt != 4 {
                println!("No");
                return;
            }
        }
    }
    println!("Yes");
}
```
