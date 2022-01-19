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

- Async Replication
  - Replication Lag 발생 가능

- Replicaof >= 5.0.0 or slaveof 명령으로 설정 가능 
  - Replicaof hostname port

- DBMS 로 보면 statement replication 가 유사 

- Replication 설정 과정

  - Secondary 에 replicaof or slaveof 명령을 전달 

  - Secondary 는 primary  에 sync 명령 전달

  - primanry 는 현재 메모리 상태를 저장하기 위해 Fork

  - Fork 한 프로세서는 현재 메모리 정보를 disk 에 dump 

  - 해당 정보를 secondary 에 전달 

  - fork 이후 데이터를 secondary 에 계속 전달 

    

### 주의점

- Replication 과정에서 fork 발생하므로 메모리 부족이 발생할 수 있다 
- Redis-cli --rdb 명령은 현재 상태 메모리 스냅샷을 가져오므로 같은 문제를 발생시킴
- AWS나 클라우드의 Redis 는 좀 다르게 구현되어서 좀 더 해당 부분이 안정적



## redis.conf 권장 설정 Tip

- maxclient 설정 50000
- RDB / AOF 설정 off
- 특정 commands disable
  - keys
  - AWS elasticCache 는 이미 하고 있음

- 전체 장애의 90% 이상이 keys 와 save 설정을 사용해서 발생

- 적절한 ziplist 설정

  

## Redis 데이터 분산

- 데이터 특성에 따라 선택 방법이 달라진다 

### 데이터 분산 방법

- application

  -  consistent hashing
    - twemproxy 를 사용하는 방법으로 쉽게 사용 가능

  - sharding 

- redis cluster



## sharding

- 데이터를 어떻게 나눌 것인가 
- 데이터를 어떻게 찾을 것인가 

### Range 

- 특정 Range 를 정의하고 해당 Range 에 속하면 거기에 저장

### Moduler

-  N % K 로 서버 데이터 결정 
- Range 보다 데이터를 균등하게 분배할 가능성이 높다 
- 서버 한대가 추가될 때마다 재분배 양이 많아진다 

### Indexed 

- 해당 key 가 어디에 저장되어야 할 관리 서버가 따로 존재 



### Redis Cluster

- 특정 Redis 서버는 slot range  를 가지고 있고 데이터 migration 은 이 slot 단위의 데이터를 다른 서버로 전달하게 된다  (migrateCommand  이용)
- 장점 
  - 자체적인 Primary Secondary Failover
  - slot 단위의 데이터 관리 

- 단점 
  - 메모리 사용량이 더 많음
  - migration 자체는 관리자 시점을 결정해야 함
  - Library 구현이 필요함

## Redis Failover

- Coordinator 기반  Failover
- VIP / DNS 기반 Failover 

