# redis-tutorial

[NHN FORWARD 2021 Redis 야무지게 사용하기](https://youtu.be/92NizoBL4uA)  
[[우아한테크세미나] 191121 우아한레디스 by 강대명님](https://youtu.be/mPB2CZiAkKM)

## Redis Collections

- Strings

  - 단일 키 
  - 기본 사용법 
  - set key value
  - get key

- List

  - Lpush key value
  - Rpush key value
  - Lpop key
  - Rpop key

- Set

  - 데이터가 있는지 없는지만 체크하는 용도 
  - SADD key value (value 가 이미 key에 잇으면 추가되지 않는다)
  - SMEMBERS key (모든 value를 돌려줌)
  - SISMEMBER key value (value 가 존재하면 1, 없으면 0)

- Sorted Set (ranking / 랭킹에 따라 순서가 바뀌길 바란다면)

  - ZADD key score value
  - ZRANGE key startIndex endIndex
    - 해당 인덱스 범위 값을 모두 돌려줌
    - Zrange testkey 0 -1
      - 모든 범위를 가져옴

  - 유저 랭킹 보드로 사용할 수 있음
  - Sorted sets 의 score 는 double 타입이기 때문에 값이 정확하지 않을 수 있다 
  - 

  

- hash 

  - key 밑에 sub key 가 존재 
  - Hmset key subkey1 value1 subkey2 value2
  - Hgetall key 
    - 해당 키의 모든 subkey와 value를 가져옴 

  - Hget key subkey 
  - Hmget key subkey1 subkey2 ... subkeyN



## Collection 주의 사항

- 하나의 컬렉션에 너무 많은 아이템을 담으면 좋지 않음
  - 10000 개 이하 몇 천개 수준으로 유지하는게 좋음

- Expire 는 Collection의 item 개별로 걸리지 않고 전체 Collection 에 대해서만 걸림
  - 즉 해당 10000 개의 아이템을 가진 Collection 에 expire가 걸려 있다면 그 시간 후에 10000 개의 아이템 모두 삭제



## Redis 운영

- 메모리 관리 
- O(N) 명령어 주의

### 메모리 관리 

- Redis is In-Memory Data Store
- 물리 메모리 이상을 시용하면 문제 발생 
  - swap 이 있다면 swap 사용으로 해당 메모리 페이지 접근시마다 늦어짐
- maxmemory 를 설정하더라도 이보다 더 사용할 가능성이 큼
- RSS 값을 모니터링 해야 함 

- 큰 메모리 instance 하나 보다는 적은 메모리를 사용하는 instance 여러개가 안전함
  - 24GB -> 8 8 8 GB

### Ziplist 구조

- in-memory 특성 상 적은 개수 라면 선형 탐색을 하더라도 빠르다 
- List, hash, sorted set 등을 ziplist 로 대체해서 처리를 하는 설정이 존재 

### O(N) 관련 명령어 주의

- redis => single thread
- 동시에 한 명령어만 처리 
- 단순한 get/set 의 경우 초당 10만 TPS 이상 가능 

#### O(N) 명렁어  => 대체 명령어

1. KEYS => scan 0 // scan 17 
2. FLUSHALL, FLUSHDB
3. Delete Collections
4. Get All Collections 

## Redis Replication



