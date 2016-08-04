# 8월 1주
### 트랜잭션
  * 트랜잭션 특성
    1. Atomicity
      - 트랜잭션 하나가 쪼개질 수 없다.
    2. Consistency
      - Locking
    3. Isolation
      - 트랜잭션과 트랜잭션은 서로 분리됨
    4. Durability
      - 한 번 끝난 트랜잭션은 영원히 적용

  * 트랜잭션 종류
    1. 자동 커밋 트랜잭션
      - SQL Server default
    2. 명시적Explicit 트랜잭션
      - BEGIN/COMMIT 직접 써주기
    3. 암시적Implicit 트랜직션
      - Oracle default
    4. 일괄 처리 범위의 트랜잭션
      - Multiple Active Result Sets에만 해당

  * [트랜잭션 관리 방법](http://d2.naver.com/helloworld/407507)
    1. 구조
      - 질의 처리기(Query Processor)
      - 저장 시스템(Storage System)
      - 페이지 버퍼(page buffer)

    2. 복구 종류
      - UNDO
      - REDO

    3. 로그
      - 로깅 방법
      - 로그 데이터 분류
      - 로깅 시 규칙
      - 로깅 속도
      - 로그 복구

    4. 장애 대응
      - 트랜잭션 롤백
      - 재시작 복구 과정
      - 백업 및 복구

    5. 커밋
      - 과정
      - 오류 발생

---


#### 트랜잭션 관리 방법

 | State  | Transition 
---------|----------|-------------
Logical  | X        | DML/DDL 자체
Physical | 로깅 전/후 | XOR 결과 기록

  2. 복구 종류
    * UNDO
      - 트랜잭션 비정상 종료 시 이제껏 변경한 페이지들을 복구
      - 트랜잭션 미완 시 수정한 페이지들이 디스크에 Write 가능
      - 수정은 버퍼 교체 알고리즘 사용
      - 전체 메모리 관점에서 보면 랜덤하게 교체
      - 정책
        - STEAL / 수정된 페이지를 언제든지 디스크에 쓰기
        - NON-STEAL / 수정된 페이지들을 최소한 트랜잭션 종료시까지 버퍼에 유지
    * REDO
      - Durability를 위해 수행
      - 정책
        - FORCE / 수정했던 모든 페이지를 트랜잭션 커밋 시점에 디스크에 반영
        - NON-FORCE / 수정했던 페이지를 트랜잭션 커밋 시점에 디스크에 반영하지 않음

  3. 로그
    - 로깅 방법 / appending, 레코드별 식별자(Log Sequence Number:Adress) / 선형 증가함
    - 로그 데이터 분류 / 물리, 논리
    - 로깅 시 규칙
      - Write Ahead Logging / DML이 DB에 써지기 전에 UNDO 정보가 로그에 써져야 함
      - 트랜잭션이 종료 처리되기 전에 REDO 정보가 로그에 써져야 함
      - 로그 버퍼
        - 로그 기록용
        - 버퍼에서 디스크로 쓰이는 경우
          1. 트랜잭션 커밋
          2. WAL
          3. 로그 버퍼가 꽉 참
          4. DBMS가 내부적으로 필요로 하는 경우
        - 커밋 전까지의 레코드에 대해 flush
      - 로깅 속도
        - 왜 느린가
          - 로그 파일에 기록 시 write(OS 버퍼에 쓰기), fsync(디스크에 쓰기) 실행
          - fsync()는 매우 느리며, WAL을 위해 DBMS가 fsync() 종료까지 기다려야 됨
          - 로그 버퍼에서 다수의 페이지를 동시에 디스크로 씀 / 버퍼 끝(쓰기)에서 헤더(갱신) 방향으로 작업
        - 개선 방안
          - Group commit / 각각의 커밋을 모아서 한꺼번에 처리. response time 희생, throughput 향상
          - Durability 희생 / 커밋 수행 전 로그를 100% 미만으로 기록.
          - Asynchronous commit / 커밋 작업 시작 전 로그 쓰기 완료를 기다리지 않음. ROLLBACK만 가능.
      - 로그 복구
        - 사용자의 요청, 오류 / 시스템의 롤백
        - SW, HW 장애 / DB 재시작 복구
    4. 장애 대응
      - 트랜잭션 롤백
        - UNDO 복구 / 로그 역방향 탐색.
        - REDO 전용 로그(Compensation Log Record) 작성 / 로그 레코드 위치를 UNDO 전으로 가리키도록 함.
        - 완료 / 해당 트랜잭션의 시작 로그까지 도달.
      - 재시작 복구 과정
        1. 로그 분석 / 마지막 체크포인트부터 최근 로그까지 탐색, 어디서부터 시스템 복구, 어느 트랜잭션 복구 확인.
        2. REDO 복구(Repeating history) / 복구 시작 시점 → 장애 발생 직전까지 REDO가 필요한 모든 트랜잭션(로그)을 REDO 복구.
        3. UNDO 복구(Global undo) / 최신 시점 → 복구 시작 시점 UNDO 복구 필요한 모든 트랜잭션(로그)을 UNDO 복구
      - 백업 및 복구
        -
    5. 커밋
      - 과정
        - 커서(질의 결과 집합) 정리
        - Holdable Cursor
      - 오류 발생
        - 트랜잭션 커밋이 완료되지 않으면 수행되지 않은 것과 같다.
        
