# 1. 병행 제어란

DBMS의 장점 중 하나는 데이터의 공유와 병행 수행을 제엏할 수 있다는 점이다.   
일반적인 DBMS 운영 환경은 다중 사용자 환경을 지원해야 한다.    
일반적으로 여러 프로그램들은 차례로 CPU를 번갈아 가면서 사용하는 인터리빙방식으로 실행된다.   
이러한 다중 사용자 환경을 지원하는 DBMS의 경우 반드시 병행제어(동시성 제어:currency control)을 지원해야 한다.

## 2. 무제어 병행 수행에 의한 장애
### 2.1 갱신 분실
> 갱신분실은 하나의 트랜잭션이 갱신한 내용을 다른 트랜잭션이 덮어써 갱신이 무효가 되는 현상   
> ex)     
> T1이 x를 읽고, T2 역시 x의 초기값을 읽은 상태에서 T1이 x를 업데이트하고 트랜잭션을 마무리한다.   
> T2 역시 x값을 업데이트하고 값을 저장하면 T1에 의한 변화가 분실되고 T2에 의해 갱신된 내용만 저장된다.

### 2.2 모순성
> 다른 트랜잭션들이 해당 항목 값을 갱신하는 동안 한 트랜잭션이 두 개의 항목 값중 어떤 것은 갱신 전 값,
> 어떤 것은 갱신된 후의 값을 읽게 되어 데이터가 불일치 되는상황

### 2.3 연쇄 복귀
> 두 트랜잭션이 동일한 데이터 내용을 접근할 때, 한 트랜잭션이 데이터를 갱신 후 실패해 roollback을 수행하는 과정에서,
> 갱신과 rollback 연산을 실행하고 있는 사이 해당 데이터를 읽는 상황
> 이는 실패된 트랜잭션의 값을 참조하는  

## 3. 스케줄
> 다중 사용자 환셩에서 둘 이상의 트랜잭션이 동시에 접속하여 연산을 수행 할 때 문제저이 발생하지 않도록 병행제어를 해야 하며
> 이러한 병행제어는 스케쥴을 통해 관리된다.
> 트랜잭션 스케쥴이란 데이터베이스 트랜잭션을 구성하는 연산들의 실행 순서를 의미한다.

### 3.1 직렬 스케쥴(serial schedule)
> 트랜잭션의 연산을 모두 순차적으로 실행하는 유형으로 인터리빙 방식으로 처리되지 않아 cpu 활용도가 떨어진다.
> 경우의 수는 n개의 스케쥴이므로 n!이 존재하며, 수행 순서는 다를 수 있지만 항상 정확한 값을 산출한다.

### 3.2 비직렬 스케줄(nonserial schedule)
> 트랜잭션 직렬 수행 순서와 상관없이 병행 수행하는 스케줄을 의미한다. 인터리빙하게 실행되므로 CPU 활용도가 높지만 
> 정확성을 만족하지 않을 수 있다.

### 3.3 직렬 가능 스케줄(serializable schedule)
> 비직렬 스케줄이 정확한 결과를 산출하려면 직렬 스케줄고 동일한 결과를 얻어야 한다. 직렬 가능 스케줄에서는
> 동시에 수행되는 트랜잭션 간 상호 간섭이 배제되어야 하므로 이를 위해 read 연산과 write 연산의 순서가 중요하다.
> 현실적으로 직렬 가능한 스케줄을 검사하는 것은 어렵다. 따라서 실제 시스템에서는 직렬 가능성을 보장하는 병행 제어기법을 사용한다.

## 4. 병행 제어 기법
> 직렬 가능한 스케줄은 데이터의 정확성을 보장하지만 실제 시스템에서는 직렬 가능성 검사가 불가능하다.
> 따라서 규약이나 규정을 정해 놓고 모든 트랜잭션들은 해당 규약이나 규칙을 따라주도록하여 직렬 가능한스케쥴로 보장한다.
> 따라서 트랜잭션을 작성할 때 모든 트랜잭션이 규약을 따르고 있는지만 검사한다.

### 4.1 로킹 기법
> 가장 많이 사용하는 기법 여러 트랜잭션들이 동일한 데이터 항목에 대해 임의적 병행 접근을 하지 못하도록 제어하는 것.
> 공유하고자 하는 데이터 항목에 대해 트랜잭션이 lock을 걸게 되면, 작업이 종료될 때까지 독점적으로 접근할 수 있도록 보장한다.
> 따라서 다른 트랙잭션들로부터 간섭이나 방해를 받지 않는다.

1. 트랜잭션 T가 항목 x에 대해 read나 write를 수행하려면 반드리 lock(x) 연산을 수행해야 한다.
2. 트랜잭션 T가 실행한 lock(x)에 대해 해당 트랜잭션이 종료 전에 이를 반납하기 위해 unlock(x)을 수행해야한다.
3. 트랜잭션 T는 다른 트랜잭션에 걸려있는 lock이 해제되기 전까지는 x에 대해 lock(x)를 수행하지 못한다.
4. 트랜잭션 T가 x에 lock을 걸지 않았다면 unlock(x)를 수행하지 못한다.

> 한 순간에 하나의 트랜잭션만 lock을 걸수 있다는 점에서 너무 제약적일 수 도 있다. 
> 따라서 데이터 항목에 대한 접근이 판독만을 위한 목적이라면 lock을 두가지 타입으로 나누어 강도를 조절한다.

1. 공용 락(shared-lock)   
읽기 전용 락, 다른 트랜잭션도 동시에 읽기 전용 락을 걸 수 있다.
2. 전용 락(exclusive-lock)   
읽기,쓰기 락, 트랜잭션이 전용 락을 걸고있으면 어떠한 다른 트랜잭션도 공용락이나 전용락을 걸 수 없다.

### 4.2 2단계 로킹규약(2PLP: two-phase locking protocol)
직렬 가능성을 보장받을 수 있는 규약으로 가장 많이 사용   
1. 확장 단계   
트랜잭션을 새로운 lock 연산말 실행할 수 있고, unlick 연산은 실행할 수 없는 단계

2. 축소 단계   
트랜잭션은 unlock연산만 실행할 수 있고, lock 연산은 실행할 수 없는 단계