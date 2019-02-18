---
title: Benefits of InnoDB Tables 
comments: true
tags:
- intro
- innodb
- mysql
categories:
- introduction
---

# InnoDB 테이블의 장점

아래의 여러가지 이유로 인해서 *InnoDB* 테이블을 사용하길 권장한다. 

- 데이터베이스를 사용하는 도중에 소프트웨어/하드웨어적인 어떠한 이유로 인해 서버가 강제 종료 될 경우, 데이터베이스를 다시 사용하기위해 특별한 조치를 취할 필요가 없다. *InnoDB*의 자체적인 [crash recovery](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_crash_recovery) (장애 복구) 기술은, 종료 이전의 어떠한 커밋 트랜잭션에 대해서든지 모두 정상 완료됨을 보장한다. 종료 전에 커밋되지 않은 데이터들은 자연스럽게 원래 상태로 되돌린다.
- *InnoDB*는 자체적인 [buffer pool](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_buffer_pool)을 관리하며, 테이블이나 인덱스에 접근할 때 해당 데이터들을 메모리 내에 상주할 수 있도록 한다. 자주 사용되는 데이터의 경우 메모리에서 곧바로 획득할 수 있으며, 단순 데이터가 아닌 다양한 종류의 데이터및 정보들에 대해서도 캐싱을 적용하여 전체 서버의 성능을 높인다. MySQL을 가용하는 데이터베이스 서버의 경우, 실제 물리 메모리의 최대 80% 까지를 InnoDB의 buffer pool 설정하고 사용하는 것이 일반적이다.
- 관련있는 데이터의 집합을 서로다른 여러개의 테이블로 분리할 경우, [foreign key](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_foreign_key)를 사용하여 [referential integrity (참조 무결성)](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_referential_integrity)을 강제할 수 있다. 데이터를 업데이트하거나 삭제할 경우 연관되어 있는 다른 테이블의 데이터도 적절히 업데이트/삭제한다. Secondary 테이블에 데이터를 삽입 하는 경우, primary table에 해당 데이터가 참조하는 키가 존재하지 않을 경우 자동적으로 삽입이 취소된다.
- 데이터가 어떠한 이유로 인해 메모리나 디스크 내에서 내용적인 변화 (오류)가 발생할 경우, [checksum](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_checksum)을 통해 데이터에 변조 위험이 있음을 사용자에게 알린다.
- 사용자가 적절한 [primary key](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_primary_key)를 사용하여 테이블을 구성한 경우, 해당 컬럼을 참조하는 작업들의 성능이 크게 향상된다. [*WHERE*](https://dev.mysql.com/doc/refman/8.0/en/select.html) 절, [*ORDER BY*](https://dev.mysql.com/doc/refman/8.0/en/select.html), [*GROUP BY*](https://dev.mysql.com/doc/refman/8.0/en/select.html) 절 및 [join] 구문에서 Primary key로 지정된 컬럼을 참조하는 경우 처리속도가 매우 빠르다.
- Insert, update, delete는 [change buffering](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_change_buffering) 기술로 자동 최적화가 된다. *InnoDB*는 같은 테이블에 대해 동시 읽기/쓰기를 지원할 뿐만 아니라, 변경된 데이터를 캐싱하여 디스크 I/O를 최소화시킨다.
- 캐싱으로 인한 성능상ㅇ의 이점은 long running 쿼리가 수행되는 대형 테이블에 대해서만 국한되지 않는다. 한 테이블의 같은 행(row)들이 반복적으로 접근 될 경우, [Adaptive Hash Index] 기법을 적용하여 반복된 lookup 작업을 해시테이블에서 곧바로 데이터를 가져와, 데이터 탐색작업을 더욱 빠르게 수행한다.
- 테이블과 인덱스들을 압축할 수 있다.
- 인덱스의 생성과 삭제 작업이 성능과 가용성에 미치는 영향을 최소화한다.
- *InnoDB*가 직접 스페이스를 관리하는 [system tablespace]와 달리, [file-per-table]을 삭제하는 작업은 매우 빠르며, 운영체제의 다른 용도로 사용할 수 있도록 디스크 스페이스의 공간을 비워줄 수 있다.
- [DYNAMIC][dynamic-link] row 포맷을 사용할 경우, [*BLOB*][blob-link] 데이터와 긴 문자열 필드에 대한 테이블의 스토리지 레이아웃을 더욱 최적화 할 수 있다.
- [INFORMATION_SCHEMA] 쿼리를 사용하여 스토리지 엔진의 내부 동작 상황을 모니터링 할 수 있다.
- [Performance Schema] 쿼리를 사용하여 스토리지 엔진 성능에 관련된 상세 사항을 모니터링 할 수 있다.
- MySQL의 다른 스토리지 엔진과 *InnoDB* 엔진을 섞어서 사용할 수 있으며, 하나의 쿼리 문에 서로다른 엔진을 사용하는 테이블을 섞어 쓸 수도 있다. 예를 들어 [join] 작업을 사용할 때, *InnoDB* 엔진을 사용하는 테이블과 [*MEMORY*] 엔진을 사용하는 테이블을 동시에 사용할 수 있다.
- *InnoDB*는 대용량의 데이터를 사용할 때 최고의 성능을 제공하고, CPU를 효율적으로 사용하도록 설계되었다.
- *InnoDB*는 파일 사이즈의 최댓값이 2GB로 제한된 환경의 운영체제에서도 그보다 훨씬 큰 대용량의 데이터를 처리할 수 있다.

*InnoDB*와 관련된 튜닝 기술은 [Optimizing for InnoDB Tables](https://dev.mysql.com/doc/refman/8.0/en/optimizing-innodb.html)를 참조하기 바란다.

[dynamic-link]: https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_dynamic_row_format
[blob-link]: https://dev.mysql.com/doc/refman/8.0/en/blob.html
[INFORMATION_SCHEMA]: https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_information_schema
[Performance Schema]: https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_performance_schema
[join]: https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_join
[MEMORY]: https://dev.mysql.com/doc/refman/8.0/en/memory-storage-engine.html
[file-per-table]: https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_file_per_table
[system tablespace]: https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_system_tablespace
[Adaptive Hash Index]: https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_adaptive_hash_index