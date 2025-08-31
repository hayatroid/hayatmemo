---
title: Development
---

```mermaid
graph LR
    subgraph "Client"
        Web
        CLI
        Desktop
        iOS
        Android
    end

    Web --> Server
    CLI --> Server
    Desktop --> Server
    iOS --> Server
    Android --> Server

    click Web href "./web"
    click CLI href "./cli"
    click Desktop href "./desktop"
    click iOS href "./ios"
    click Android href "./android"
    click Server href "./server"
```
