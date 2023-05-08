---
title: Advance Wars
author: Rick Gray
year: 2023
---
```mermaid
flowchart TD

1[1-3]
4A[4A-5A]
4M[4M-7M]
10F[10]
11F[11]
12[12-14]
15[15-18]
19[19-21]
19A[19-21]
19S[19-21]
A[Unlock Grit]
B[Unlock Sonja]
C[Unlock Drake]
D[Unlock Eagle]
E[Unlock Kanbei]
F[Unlock Nell]
G[Unlock Sturm]

X[Unlock all COs]
Z[Advance Campaign Clear]

1 -->|Andy Branch| 4A
1 -->|Max Branch| 4M
4A --> 6A
6A -->|HQ Cap| 8
6A -->|Rout| 7A
4M --> A
7A --> 8
A --> 8
8 --> 9
9 --> 10
9 -->|Clear in 8 days or less| 10F
10 --> 11
10F --> 11
10F -->|Clear in 10 days or less| 11F
11 --> 15
11F --> 15
11F -->|Clear in 12 days or less| 12
12 --> B
B --> 15
15 -->|Only use Andy| 19A
15 -->|Only use Sami| 19S
15 --> 19
19A --> C
19S --> 22
22 --> D
19 & C & D --> E

X --> G
G & Z --> F

style A stroke:#69a2ff,stroke-width:3px
style B stroke:#69a2ff,stroke-width:3px
style C stroke:#69a2ff,stroke-width:3px
style D stroke:#69a2ff,stroke-width:3px
style E stroke:#69a2ff,stroke-width:3px
style F stroke:#69a2ff,stroke-width:3px
style G stroke:#69a2ff,stroke-width:3px

style 7A stroke:#ff696e,stroke-width:3px
style 12 stroke:#ff696e,stroke-width:3px
style 22 stroke:#ff696e,stroke-width:3px
```