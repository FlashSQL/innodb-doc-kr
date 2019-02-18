---
title: InnoDB
comments: true
tags:
- intro
categories:
- introduction
---

## Introduction to InnoDB in MySQL 8.0

MySQL에서 사용하는 InnoDB는 높은 가용성과 성능을 보장하기 위해 사용되는 다목적 스토리지 엔진이다.
MySQL 8.0에서 기본 스토리지 엔진으로 InnoDB를 사용하며, `CREATE TABLE` 구문에
`ENGINE=` 옵션을 추가하여 다른 엔진을 사용하도록 설정할 수 있다. 

### InnoDB의 주요 장점

- 사용자의 데이터를 보호하기 위해 `commit`, `rollback` 및 `crash recovery`가 가능한 트랜잭션을 통해 ACID 모델을 지원하는 DML 오퍼레이션을 제공한다. 자세한 내용은 ["InnoDB and ACID Model]()을 통해 확인할 수 있다.
- *Row-level locking*과 오라클 스타일의 consistent read를 사용하여 다중 사용자 환경에서의 concurrency와 성능을 높인다. 자세한 내용은 ["InnoDB Locking and Transaction Model"]()을 통해 확인할 수 있다.
- `InnoDB` 테이블은 디스크에 저장되며, 프라이머리 키에 최적화된 인덱스를 제공한다. 모든 `InnoDB` 테이블은 프라이머리 키 인덱스를 가지고 있으며, 이들은 모두 `clustered index`이다. 자세한 내용은 ["Clustered and Secondary Indexes"]()를 통해 확인할 수 있다.
- 데이터의 integrity를 보장하기 위해 `Foreign key`를 지원한다. Foreign key를 사용한 insert/update/delete와 같은 작업들은 연관된 모든 테이블들에 대해서 제약사항을 위반하지 않는 지 자동 검사된다. 자세한 내용은 ["InnoDB and FOREIGN KEY Constraints"]()를 통해 확인할 수 있다.

| 기능                          | 지원 여부                |
| :---------------------------- | ------------------------ |
| B-tree 인덱스                 | o 지원                   |
| Backup/point-in-time recovery | o 지원 (서버에서 지원함) |
| Cluster 데이터베이스          | x 미지원                 |
| Clustered 인덱스              | o 지원                   |
| 데이터 압축                   | o 지원                   |
| 데이터 캐시                   | o 지원                   |
| 데이터 암호화                 | o 지원 (서버에서 지원함) |
| Foreign key                   | o 지원                   |

