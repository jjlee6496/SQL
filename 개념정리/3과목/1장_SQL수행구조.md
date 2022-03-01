1장 SQL 수행 구조
# 데이터베이스 아키텍처
## Oracle 아키텍처
Oracle에서의 데이터베이스: 디스크에 저장된 데이터 집합(Datafile, Redo Log File, Control File 등)  
인스턴스: SGA 공유 메모리 영역과 이를 액세스 하는 프로세스 집합
 ![image](https://user-images.githubusercontent.com/96418287/156164036-048e65d8-ea78-4766-8f64-fb9881eae0b0.png)

## 백그라운드 프로세스
|ORACLE	|SQL Server|설명|
|:--------:|:--------:|:----------------:|
|SMON(System Monitor)|Database cleanup/shirinking thread|	장애가 발생한 시스템을 재기동할 때 인스턴스 복구를 수행하고, 임시 세그먼트와 익스텐트를 모니터링 한다.|
|PMON(Process Monitor)|	OPS(Open Data Services)|이상이 생긴 프로세스가 생기면 리소스를 복구한다.|
|DBWn(Database Writers)|	Lazywriter thread)|버퍼 캐시에 있는 Dirty 버퍼를 데이터 파일에 기록한다|
|LGWR(Log Writer)| Log writer thread|로그 버퍼 엔트리를 Redo 로그 파일에 기록한다.|
|ARCn(Archiver)| N/A |꽉 찬 Redo 로그가 덮어 쓰여지기 전에 Archive 로그 디렉토리로 백업한다. |
|CKPT(Checkpoint)|Database Checkpoint thread)|이전에 Checkpoint가 일어났던 마지막 시점 이후의 데이터베이스 변경 사항을 데이터 파일에 기록하도록 트리거링하고, 기록이 완료되면 현재 어디까지 기록했는지를 컨트롤파일과 데이터파일 헤더에 저장한다. |
|RECO(Recoveror)|Distributed Transaction Coordinator(DTC) |분산 트랜잭션 과정에 발생한 문제를 해결한다.|
# 파일 구조
## 데이터 파일
   ![image](https://user-images.githubusercontent.com/96418287/156164124-0a590264-b5de-4bf3-ac0a-ce4147cab9d2.png)

1. 블록(=페이지)
대부분 DBMS에서 I/O는 블록 단위로 이루어진다. 데이터를 읽고 쓸 때의 논리적인 단위가 블록인 것이다. Oracle은 ‘블록(Block)’이라고 하고, SQL Server는 ‘페이지(Page)’라고 한다.  
Oracle은 2KB, 4KB, 8KB, 16KB, 32KB, 64KB의 다양한 블록 크기를 사용할 수 있지만, SQL Server에선 8KB 단일 크기를 사용한다. 블록 단위로 I/O 한다는 것은, 하나의 레코드에서 하나의 칼럼만을 읽으려 할 때도 레코드가 속한 블록 전체를 읽게 됨을 뜻한다.   
SQL 성능을 좌우하는 가장 중요한 성능지표는 액세스하는 블록 개수이며, 옵티마이저의 판단에 가장 큰 영향을 미치는 것도 액세스해야 할 블록 개수다.   
예를 들어, 옵티마이저가 인덱스를 이용해 테이블을 액세스할지 아니면 Full Table Scan 할지를 결정하는 데 있어 가장 중요한 판단 기준은 읽어야 할 레코드 수가 아니라 읽어야 하는 블록 개수다.
2. 익스텐트(Extent)
데이터를 읽고 쓰는 단위는 블록, 테이블 스페이스로부터 공간을 할당하는 단위는 익스텐트.  
테이블이나 인덱스에 데이터를 입력하다가 공간이 부족해지면 해당 오브젝트가 속한 테이블 스페이스를 할당받는데, 이때 정해진 익스텐트 크기의 연속된 블록을 할당받는다. 익스텐트 내의 블록은 서로 인접하지만 익스텐트끼리는 서로 인접하지 않고 떨어져 있을 수 있다. 
3. 세그먼트
세그먼트는 테이블, 인덱스, Undo처럼 저장공간을 필요로 하는 데이터베이스 오브젝트이다. 저장공간을 필요로 한다 = 한 개 이상의 익스텐트를 사용한다. 테이블을 생성할 때 내부적으로는 테이블 세그먼트가 생성된다. 인덱스를 생성할 때 내부적으로 인덱스 세그먼트가 생성된다.  
이렇듯 다른 오브젝트는 세그먼트와 1:1 대응관계를 갖지만 파티션은 1:M 관계를 갖는다. 즉, 파티션 테이블(또는 인덱스)을 만들면, 내부적으로 여러 개의 세그먼트가 만들어진다. 한 세그먼트는 자신이 속한 테이블 스페이스 내 여러 데이터 파일에 걸쳐 저장될 수 있다. 즉, 세그먼트에 할당된 익스텐트가 여러 데이터 파일에 흩어져 저장되는 것이며, 그래야 디스크 경합을 줄이고 I/O 분산효과를 얻을 수 있다.
4. 테이블 스페이스
테이블 스페이스는 세그먼트를 담는 컨테이너로서, 여러 데이터 파일로 구성된다. 데이터는 물리적으로 데이터 파일에 저장되지만, 사용자가 데이터 파일을 직접 선택하진 않는다. 사용자는 세그먼트를 위한 테이블 스페이스를 지정할 뿐, 실제 값을 저장할 데이터 파일을 선택하고 익스텐트를 할당하는 것은 DBMS의 몫이다.  
각 세그먼트는 정확히 한 테이블 스페이스에만 속한다. 하지만 한 테이블 스페이스에는 여러 세그먼트가 존재할 수 있다. 특정 세그먼트에 할당된 모든 익스텐트는 해당 세그먼트와 관련된 테이블 스페이스 내에서만 찾아진다. 한 세그먼트가 여러 데이터파일에 걸쳐 저장될 수는 있다.
 ![image](https://user-images.githubusercontent.com/96418287/156164206-86ad8f7d-31a3-43c2-8607-7c0925b1a914.png)

## 로그 파일
* Online Redo 로그
캐시에 저장된 변경사항이 아직 데이터 파일에 기록도지 않은 상태에서 인스턴스가 비정상 종료되는(정전 등)상황에 대비. Online Redo는 마지막 체크포인트 이후부터 사고 발생 직전까지 수행되었던 트랜잭션들을 Redo 로그를 이용해 재현한다.(캐시복구)  
Online Redo 로그는 최소 두개 이상의 파일로 구성되고 현재 사용 중인 파일이 꽉 차면 다음 파일로 로그 스위칭, 모든파일이 꽉 차면 라운드 로빈 방식 사용.
* Archived(=offline) Redo 로그
Online Redo로그가 재사용 되기 전에 다른 위치로 백업해 둔 파일. 디스크 손상 등 물리적인 저장 매체에 문제 생겼을경우 데이터베이스 복구를 위해 사용됨
# 메모리 구조
* System Global Area(SGA): 여러 프로세스가 동시에 액세스 할 수 있는 메모리 영역. DB버퍼캐시, Shared pool, 로그 버퍼 등이 있음
* Process Global Area(PGA): 각 Oracle 서버 프로세스는 자신만의 PGA(Process/Program/Private Global Area) 메모리 영역을 할당받고, 이를 프로세스에 종속적인 고유 데이터를 저장하는 용도로 사용한다. PGA는 다른 프로세스와 공유되지 않는 독립적인 메모리 공간으로서, 래치 메커니즘이 필요 없어 똑같은 개수의 블록을 읽더라도 SGA 버퍼 캐시에서 읽는 것보다 훨씬 빠르다.  UGA, CGA, Sort Area가 있음
## DB 버퍼 캐시
데이터 파일로부터 읽어들인 데이터 블록을 담는 캐시영역. 모든 사용자 프로세스는 서버 프로세스를 통해 DB버퍼캐시의 버퍼블록을 동시에 액세스 가능하다. 일부 Direct Path Read 매커니즘을 제외하면 모든 블록일기는 버퍼캐시를 통해 이루어진다. 즉 먼저 버퍼캐시찾아보고 없으면 디스크에서 읽는다. 디스크에서 읽을때도 먼저 버퍼캐시에 적재후 읽는다.  
데이터 변경도 버퍼캐시에 적재된 블록을 통해 이루어지며 Dirty 블록을 데이터파일에 기록하는작업은 DBWR가 함. 버퍼캐시가 꽉 차면 LRU알고리즘으로 확보
## Shared pool
### 딕셔너리 캐시 = Row cache
데이터베이스 딕셔너리는 테이블, 인덱스 같은 오브젝트는 물론 테이블 스페이스, 데이터 파일, 세그먼트, 익스텐트, 사용자, 제약에 관한 메타정보를 저장하는 곳이다. 딕셔러니 캐시는 말 그대로 딕셔너리 정보를 캐싱하는 메모리 영역.  
딕셔너리 캐시의 활동성 v$rowcache로 확인. 낮으면 shared pool size를 늘려준다
로우캐시에 관리되는 엔트리 각각에 대해 하나의 래치가 할당되어 있음
### 라이브러리 캐시
DB 버퍼, Redo log, 딕셔너리 캐시는 입출력을 빠르게 하기 위해 존재.   
라이브러리 캐시는 사용자가 던진 SQL과 실행계획을 저장해둬서 캐싱된 SQL과 실행계획의 재사용성을 높여줌.
## 로그 버퍼
서버 프로세스가 데이터 블록 버퍼에 변경을 가하기 전에 Redo 로그 버퍼에 먼저 기록해 두면 주기적으로 LGWR 프로세스가 Redo 로그파일에 기록한다.

## Redo
### Redo의 목적
1. DB Recovery = Media Recovery: Media Fail 발생 시 Archived Redo로그 사용
2. Cache Recovery = Instance Recovery
3. Fast Commit: Redo로그 믿고 우선 로그파일 기록. 블록간 동기화는 나중에 일괄수행
### Redo가 Redo 로그버퍼를 Redo로그에 기록하는 시점
1. 3초마다 DBWR 프로세스로부터 신호를 받을 때
2. 로그버퍼의 1/3이 차거나 기록된 Redo 레코드양이 1MB넘을 때
3. 사용자가 커밋 또는 롤백 명령을 날릴 때
WAL(Write Ahead Logging): Dirty 버퍼를 디스크에 기록하기 전에 해당 로그 엔트리를 먼저 로그 파일에 기록한다.  
Log Force at commit: 로그 버퍼를 주기적으로 로그파일에 기록한다는데, 늦어도 커밋 시점에는 로그파일에 기록해야한다.  
##Undo
### Undo의 목적
1. Transaction Rollback
2. Transaction Recovery
3. Read Consistency
### Undo segment block
Undo segment header: Transaction ID, Transaction Status, 커밋 SCN(커밋된 경우), Last UBA(Undo Block Access)  
* Transaction ID: [USN# + Slot# + Wrap#] undo segment number + 트랜잭션 테이블 슬롯 할당넘버
* Last UBA: 트랜잭션의 기록사항들을 가장 마지막 Undo 레코드 뒤에 계속 추가해 나가려는 일종의 포인터
