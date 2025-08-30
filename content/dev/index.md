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
```
