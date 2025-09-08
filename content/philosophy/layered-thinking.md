---
title: 層を意識する
---

> [!important]
>
> - とにかく関心事を切り離す。関心事が $N$ あれば、思考に $O(N^2)$ 掛かる。
> - 層をまたいでの思考を防ぐ。特に高レイヤに関する議論は低レイヤの詳細に話が行きがちであるので、これを防ぐ。

## アーキテクチャの例

アーキテクチャの話をするとき、たとえば [C4 model](https://c4model.com/diagrams) におけるどの層の話をしているか、考える。

```mermaid
graph TB
    context --> container
    container --> component
    component --> code

    click context href "https://c4model.com/diagrams/system-context"
    click container href "https://c4model.com/diagrams/container"
    click component href "https://c4model.com/diagrams/component"
    click code href "https://c4model.com/diagrams/code"
```

context について考えるときは、技術的詳細を考えないようにし、[ユースケース](https://wa3.i-3-i.info/word16097.html) に意識を集中させる。

## Web フロントエンドの例

Web フロントエンドの話をするとき、たとえば [bulletproof-react](https://github.com/alan2207/bulletproof-react/blob/master/docs/project-structure.md) におけるどの層の話をしているか、考える。

```mermaid
graph TB
    application

    subgraph features
        direction TB
        comments
        discussions
        teams
    end

    subgraph shared
        direction TB
        components
        hooks
        lib
        types
        utils
    end

    application --> features
    application --> shared
    features --> shared

    click application href "https://github.com/alan2207/bulletproof-react/tree/master/apps/react-vite/src/app"
    click comments href "https://github.com/alan2207/bulletproof-react/tree/master/apps/react-vite/src/features/comments"
    click discussions href "https://github.com/alan2207/bulletproof-react/tree/master/apps/react-vite/src/features/discussions"
    click teams href "https://github.com/alan2207/bulletproof-react/tree/master/apps/react-vite/src/features/teams"
    click components href "https://github.com/alan2207/bulletproof-react/tree/master/apps/react-vite/src/components"
    click hooks href "https://github.com/alan2207/bulletproof-react/tree/master/apps/react-vite/src/hooks"
    click lib href "https://github.com/alan2207/bulletproof-react/tree/master/apps/react-vite/src/lib"
    click types href "https://github.com/alan2207/bulletproof-react/tree/master/apps/react-vite/src/types"
    click utils href "https://github.com/alan2207/bulletproof-react/tree/master/apps/react-vite/src/utils"
```

features について考えるときは、shared の UI・UI ロジックの詳細を考えないようにし、[ビジネスロジック](https://wa3.i-3-i.info/word13666.html) に意識を集中させる。
