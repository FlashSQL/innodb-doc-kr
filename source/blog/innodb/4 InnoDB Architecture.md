---
title: InnoDB Architecture
comments: true
tags:
- innodb
- mysql
- architecture
- IO
- disk
- buffer
- memory
- structure
categories:
- architecture
- IO
---

# InnoDB 구조 

아래의 그림은 *InnoDB*가 메모리 및 디스크 상에서 관리하는 데이터들의 구성을 나타낸다. 각 구조에 대한 정보를 알기 위해서는 [InnoDB In-Memory Structures][ims] 와 [InnoDB On-Disk Structures][ods] 섹션을 확인하길 바란다. 

![InnoDB Architecture](../../images/innodb-architecture.png)

[ims]: ./5%20innodb%20in-memory%20structures.html
[ods]: ./6.innodb-on-disk-structures.html