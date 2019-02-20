---
title: ACID Property 
comments: true
tags:
- innodb
- mysql
- ACID
- transaction
categories:
- transaction
---

# InnoDB의 ACID 모델

[ACID] 모델은 비즈니스 데이터나 사업의 핵심 응용 프로그램을 관리하는데 있어서 중요한 데이터베이스의 안정성을 강조하고 보장하는 일련의 데이터베이스 설계 원칙이다. MySQL은 `InnoDB` 엔진과 같이 ACID 모델을 철저하게 준수하는 컴포넌트들을 사용하서, 소프트웨어 크래쉬나 하드웨어 오작동으로 인한 예외 상황에서 데이터가 손상되거나 작업 결과가 왜곡되지 않도록 방지한다. 이러한 ACID 모델은 철저히 준수하는 기능을 사용할 경우, 사용하고자 하는 응용프로그램을 위해 새로운 일관성 검사 기능 및 장애 복구 기술을 개발할 필요가 없다. 소프트웨어적인 안전장치, 극도로 신뢰할 수 있는 하드웨어 장치가 있거나 타겟 애플리케이션이 어느정도의 데이터 유실을 감수할수 있는 경우에는, MySQL의 옵션을 통해 ACID 모델 중 일부를 희생하고 응용프로그램의 성능이나 데이터 처리량을 늘릴 수도 있다.

이후의 내용은 *InnoDB* 엔진이 ACID모델을 어떻게 다루고 있는지에 대해서 설명한다.

- ***A*** : atomicity (원자성)
- ***C*** : Consistency (일관성)
- ***i*** : isolation (고립성, 독립성)
- ***D*** : durability (지속성)

## Atomicity

ACID 모델의 ***atomicity*** 는 주로 *InnoDB*의 [transaction]의 구현을 포함한다. MySQL의 관련 기능들은 아래와 같다.

- [Autocommit] 옵션
- [COMMIT] 구문
- [ROLLBACK] 구문
- `INFORMATION_SCHEMA` 테이블의 데이터베이스 운영 관련 데이터

## Consistency 

ACID모델의 ***consistency***는 주로 데이터가 손상되는것을 방지하기 위해 *InnoDB*의 내부의 동작방식을 포함한다. MySQL의 관련 기능들은 아래와 같다.

- *InnoDB*의 [doublewrite buffer]
- *InnoDB*의 [crash recovery]

## Isolation

ACID모델의 ***isolation***은 주로 *InnoDB*의 [transaction]의 구현을 포함하며, 특히 각 트랜잭션에 적용 되는 [isolation level]에 의해 정의된다. MySQL의 관련 기능들은 아래와 같다.

- [Autocommit] 옵션
- `SET ISOLATION LEVEL` 구문
- *InnoDB*의 [locking] 상세 구현. 성능 테스트 중에 `INFORMATION_SCHEMA` 테이블을 통해 확인 가능함.

## Durability

ACID모델의 ***durability***는 사용중인 하드웨어와 상호작용하는 MySQL의 소프트웨어 기능을 포함한다. 하드웨어는 사용하는 서버의 CPU, 네트워크, 저장장치의 기능과 구성에 따라서 많은 경우의 수가 존재하므로, 구체적인 가이드라인을 제시하기에 가장 복잡한 기능이다. MySQL의 관련 기능들은 아래와 같다.

- [innodb_doublewrite] 설정 옵션에 따라 조절되는 *InnoDB*의 [doublewrite buffer]
- [innodb_flush_log_at_trx_commit] 옵션
- [sync_binlog] 옵션
- [innodb_file_per_table] 옵션
- HDD, SSD 및 RAID의 저장장치 내부 쓰기 버퍼 (write buffer)
- 저장장치내의 배터리 지원 캐시 (정전으로 인한 데이터 유실 방지)
- `fsync()` 시스템 콜을 지원하는 운영체제
- UPS (Uninterruptible power supply, 무정전 전원 공급 장치)에 의해 보호되는 MySQL 서버 컴퓨터 및 데이터베이스 저장장치
- 데이터에 대한 다양한 백업 전략 (빈도, 유형, 보존 기간 등)
- 분산형 혹은 호스팅형 응용프로그램의 경우, MySQL서버를 구동하는 하드웨어가 잇는 데이터센터의 특정한 특성과 데이터 센터 간의 네트워크 연결 상태

[innodb_file_per_table]: https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_file_per_table
[sync_binlog]: https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html#sysvar_sync_binlog
[innodb_flush_log_at_trx_commit]: https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_flush_log_at_trx_commit
[ACID]: https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_acid
[transaction]: https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_transaction
[COMMIT]: https://dev.mysql.com/doc/refman/8.0/en/commit.html
[ROLLBACK]: https://dev.mysql.com/doc/refman/8.0/en/commit.html
[doublewrite buffer]: https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_doublewrite_buffer
[crash recovery]: https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_crash_recovery
[Autocommit]: https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_autocommit
[innodb_doublewrite]: https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_doublewrite
[locking]: https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_locking
[isolation level]: https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_isolation_level