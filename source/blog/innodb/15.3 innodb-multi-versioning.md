---
title: Multi Versioning 
comments: true
tags:
- innodb
- mysql
- transaction
- multi versioning
categories:
- transaction
---

# InnoDB Multi-Versioning

*InnoDB*는 데이터의 멀티 버전을 지원하는 스토리지 엔진 ([multi-versioned storage engine])이다. 변경된 row들에 대한 이전 버전의 정보들을 가지고 있으며, 이를 통해서 동시성 제어와 [rollback]과 같은 트랜잭션 기능들을 제공한다. 이러한 정보들은 테이블 스페이스 내의 [rollback segment]라고 불리는 자료구조에 저장된다. 트랜잭션 진행 도중 롤백이 필요할 경우 롤백 세그먼트의 데이터 정보를 이용하여 undo 작업을 수행한다. 또한, [consistent read] 기능을 제공하기 위해 이전 버전의 데이터를 만드는 데에도 사용한다. 

*InnoDB*는 내부적으로 데이터베이스에 저장되는 모든 row 각가에 대하여 추가적인 3개의 필드를 같이 저장한다. 6 바이트 크기의 `DB_TRX_ID` 필드는 해당 로우를 추가하거나 업데이트한 가장 마지막 트랜잭션의 ID를 저장한다. 로우의 삭제 작업 또한 해당 로우의 특수 bit를 마크 하는 방식의 업데이트로 취급되므로 앞의 사례에 포함된다. 7 바이트 길이의 `DB_ROLL_PTR` 필드는 roll 포인터로 불리며, rollback segment내에서 해당 로우에 할당된 undo 로그 레코드의 위치를 가리킨다. 로우가 업데이트된 상황이라면, undo 로그 레코드는 해당 로우를 이전 버전으로 되돌리는데 필요한 정보를 포함하고 있다. 6바이트 길이의 `DB_ROW_ID` 필드는 각 로우의 ID를 갖고 있으며, 이 값은 새로운 로우가 추가될 때 마다 단순 증가 한다. 사용자가 특별한 인덱스 지정을 하지 않아 *InnoDB*가 직접 클러스터드 인덱스를 자동 생성 할 경우에 바로 이 `DB_ROW_ID`를 키로 사용한다. 그렇지 않은 경우에는 `DB_ROW_ID`가 지정되지 않는다.

Rollback segment내의 undo 로그들은 insert 로그와 update 로그로 분리된다. Insert 로그는 트랜잭션의 롤백 시에만 필요한 로그이며, 트랜잭션이 커밋하는 순간 바로 삭제 해도 되는 로그이다. Update 로그는 consistent read를 지원하기 위해서 계속 유지될 필요가 있다. Update 로그는 데이터베이스 전체에서 해당 로그가 포함된 스냅샷을 할당 받아서 이전 버전에 대한 consistent read를 요구하는 트랜잭션이 전혀 없는 경우에만 삭제가 가능하다.

다른 작업없이 consistent read를 수행하는 트랜잭션은 주기적으로 커밋을 수행해야 한다. 그렇지 않으면 위와 같은 이유로 update undo 로그를 삭제할수 없어, rollback segment가 매우 커지는 문제가 발생할 수 있다.

Rollback segment 내의 undo 로그 레코드의 크기는 일반적으로 실제 insert/update된 row의 크기보다 작다. 이러한 정보를 활용하여 가동중인 데이터베이스에서 실제로 필요한 rollback segment 크기를 계산할 수 있다.

*InnoDB*의 멀티 버전 관리 체계에서, SQL 쿼리에 의해 삭제된 row는 실제 물리적으로 즉시 삭제되지 않는다. 물리적으로 해당 레코드를 삭제하는 시점은 해당 `Delete` 쿼리를 위한 undo 로그 레코드가 삭제되는 시점이다. 이러한 삭제 작업을 [purge]라고 부르며, 실제 Delete SQL 문을 수행했던 시간과 같은 속도로 매우 빠르게 처리된다.

거의 동일한 비율로 insert와 delete를 수행하는 작은 배치 쿼리를 계속해서 수행할 경우, purge를 담당하는 쓰레드가 insert 쓰레드에 뒤쳐지게 되고, 이는 계속해서 남겨지는 "dead" 로우들 (delete 마크만 되고 삭제되지 않은 로우들) 로 인해서 테이블의 크기가 증가하고 모든 작업이 disk-bound가 되어 전체 성능 매우 떨어지는 문제를 일으킨다. 이러한 경우에는 insert에 의한 새로운 로우를 생성하는 작업을 제한하고 purge 쓰레드에 더 많은 리소스를 할당하여, disk-bound를 제거해야 한다. 적절한 purge 쓰레드 조절을 위해서는 [`innodb_max_purge_lag`][purge-option] 옵션을 사용하면 되고, 자세한 내용은 ["InnoDB Startup Options and System Variables"][innodb-options] 을 참조하길 바란다. 

## 멀티버전 관리와 Secondary 인덱스

*InnoDB*의 MVCC (MultiVersion Concurrency Control, 다중 버전 동시성 제어 기법) 는 secondary 인덱스를 메인 클러스터드 인덱스와 다르게 처리한다. 메인 클러스터드 인덱스의 경우 update 쿼리에 대한 작업을 in-place로 처리하고 해당 로우에 대해 위에서 언급한 undo 로그 레코드를 가리키는 숨겨진 컬럼을 추가하여 이전 버전의 레코드를 재생성할 수 있도록 한다. Secondary 인덱스의 경우에는 각 로우 레코드들에 대해 숨겨진 컬럼을 사용하지도 않고, in-place 업데이트를 수행하지도 않는다.

Secondary 인덱스의 컬럼이 업데이트 된 경우, 이전 버전의 레코드에는 delete 마크를 표시해두고, 새로운 레코드는 인덱스에 새롭게 insert된다. delete 마크가 표기된 이전 버전의 레코드는 향후에 purge 된다. 한 트랜잭션이 secondary index가 구성되어 있는 특정 컬럼에 대한 값을 얻어 오고자 할 때, 해당 트랜잭션 보다 이후에 시작된 트랜잭션에 의해 secondary 인덱스의 레코드가 delete 되거나 update 된 경우, 원래 자기 트랜잭션의 snapshot에 맞는 이전 버전의 레코드를 가져오기 위해서는 메인 클러스터드 인덱스를 거쳐야 한다. 클러스터드 인덱스에서는 로우 레코드들이 `DB_TRX_ID`를 가지고 있으므로 해당 아이디를 통해, 자신이 원하는 버전의 데이터를 undo 로그 레코드에서 획득하여 이전 버전의 레코드를 재생성 할 수 있다. [Covering index]에 대해서도 동일하게 동작한다. 

[Index condition pushdown][ICP] 최적화 기술을 사용하는 경우, 원래 기술은 base 테이블을 참조하지 않는 것이지만, MVCC에 의해 secondary 인덱스의 레코드가 수정 (delete/update) 되었음이 확인된 경우에는 메인 클러스터드 인덱스를 추가적으로 접근하여 원하는 버전의 데이터를 재생성하는 작업이 추가된다. 



[multi-versioned storage engine]: https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_mvcc
[rollback]: https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_rollback
[rollback segment]: https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_rollback_segment
[consistent read]: https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_consistent_read
[purge]: https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_purge
[purge-option]: https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_max_purge_lag
[innodb-options]: https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html
[Covering index]: https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_covering_index
[ICP]: https://dev.mysql.com/doc/refman/8.0/en/innodb-multi-versioning.html