---
title: "A. Stage Clear"
---

[問題はこちら](https://atcoder.jp/contests/abc422/tasks/abc422_a)

## 方針

`proconio::marker::Bytes` を用いると、文字列を `Vec<u8>` として読み取ることができる。

## 回答

問題文の条件分岐をそのままコードに落とし込むと、以下のようになる（[提出](https://atcoder.jp/contests/abc422/submissions/69209390)）。

```rs
use proconio::{input, marker::Bytes};

fn main() {
    input! {
        s: Bytes,
    }
    let i = s[0] - b'0';
    let j = s[2] - b'0';
    if j < 8 {
        println!("{}-{}", i, j + 1);
    }
    if i < 8 && j == 8 {
        println!("{}-{}", i + 1, 1);
    }
    if i == 8 && j == 8 {
        unreachable!();
    }
}
```

問題の制約下では、以下のコードと等価である（[提出](https://atcoder.jp/contests/abc422/submissions/69209402)）。

```rs
use proconio::{input, marker::Bytes};

fn main() {
    input! {
        s: Bytes,
    }
    let i = s[0] - b'0';
    let j = s[2] - b'0';
    println!("{}-{}", i + j / 8, j % 8 + 1);
}
```
