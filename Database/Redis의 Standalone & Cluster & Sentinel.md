# 📌 Redis의 Standalone & **Cluster & Sentinel**

# 1. Standalone

![0_ImoeLTgBK-e2NRAn](https://github.com/princenim/TIL/assets/59499600/5a412522-c655-42ee-ad84-e0fd47987c37)

레디스 서버 1대로 운영되며 이를 master 노드라고 한다. 만약 서버 다운시 `AOF(Append-Only File)`와 `스냅샷 파일`을 이용해 영속성을 보장한다.

AOF는 Redis의 모든 쓰기 명령을 로그 파일에 기록해 데이터의 영속성을 보장한다. 따라서 서버가 다운되더라도 다시 명령어를 실행해서 데이터를 복구할 수 있다. 스냅샷은 일정 주기마다 메모리의 데이터를 디스크에 저장해 데이터 복구에 사용한다.

하지만 이는 마스터 노드가 다운될 경우 개발자가 다시 서버를 시작해야하는 과정이 필요하다. 이러한 과정을 방지하기 위해서 운영환경에서는 고가용성 구성을 사용해 마스터 노드 다운 시 자동으로 복구를 수행할 수 있다. Redis의 고가용성(HA,High Availability)을 보장하는 방법은 `cluster`와 `sentinel` 2가지이다.

# 2. **Sentinel**
## 2.1  master- slave 구조

`master 노드`는 데이터의 쓰기, 읽기에 대한 모든 요청을 처리한다. 따라서 쓰기 작업이 일어나면 하나 이상의 slave 노드에 복제(replica)한다.

`slave 노드`는 maste 노드의 복사본을 가지고 있어서 데이터 손실 시에 데이터를 보호하는 역할을 한다. 복제하는 역할 뿐만 아니라 읽기작업을 분산시킴으로써 성능을 향상시킬 수 있다.

![sentinel](https://github.com/princenim/TIL/assets/59499600/f7ebc5d5-2096-4e25-8487-d60160807d60)

**그렇다면 `sentinel 노드`는 무엇일까?**

sentinel은 master 노드의 상태를 모니터링하고 장애 발생 시 자동으로 slave 노드를 새로운 마스터 서버로 승격시킨다(= 자동 failover). 따라서 sentinel은 master와 slave가 제대로 동작하고 있는지 감지하고 있어야한다.

![se](https://github.com/princenim/TIL/assets/59499600/72ece3d4-f22b-449d-b11e-4a5f7695bdcc)

또한 sentinel 노드는 1개도 실행하지만 상태 체크를 위해 3개 이상의 홀수개 구성을 권장한다. 홀수개여야 하는 이유는 다수 결정 원칙을 적용하기 위함이다.

## 2.2 sync 방식

기본적으로 master의 데이터를 slave에 복사하는 방법은 2가지가 존재한다.

### Fullsync(전체동기화)

1. `Disk-to-Disk`

![full_sync_disk_disk](https://github.com/princenim/TIL/assets/59499600/717fb13f-6a81-46c3-a906-8538800728ec)

master는 메모리에 있는 전체 데이터를 파일로 저장 후 디스크에 전송한다. slave는 데이터를 받아서 디스크로 파일로 저장한 다음 메모리에 적재한다. (버전 6.x까지 기본 동작)

2.  `Memory-to-Disk`

![full_sync_mem_disk](https://github.com/princenim/TIL/assets/59499600/d8b40dc9-96ca-4e98-ba85-7dbd816fbc70)

master는 메모리에 있는 전체 데이터를 소켓으로 복제본에 바로 전송한다. 복제본은 데이터를 받아 디스크에 파일로 저장한 다음 메모리에 적재한다. (버전 7.0부터 기본 동작)

3.  `Memory-to-Memory`

![full_sync_mem_mem](https://github.com/princenim/TIL/assets/59499600/f9163488-fbe2-4f2d-a932-7845b62aa4da)

master는 메모리에 있는 전체 데이터를 소켓으로 복제본에 바로 전송한다. 복제본은 데이터를 받아 바로 메모리에 적재한다.

### Partialsync(부분 동기화)

부분 동기화는 slave노드가 master 노드로부터 일부 데이터를 다시 동기화하는 과정을 의미한다. 네트워크가 끊어져서 다시 연결될 때 누락된 일부분만들 동기화한다.

master 노드와 slave노드는 각 서버의 replication id와 offset을 가지고 있다. replication id는 master 노드와 slave 노드가 가지고 있는 고유한 식별자를 말하고, offset은 데이터의 위치를 나타낸다. 즉 master에서 slave 전송된 데이터의 위치를 말한다. AOF 파일 내 위치를 말한다.

따라서 네트워크가 끊어지면 master노드는 replication id와 offset을 포함한 데이터를 backlog-buffer에 저장한다. 그리고 다시 slave노드와 연결되면 master 노드는 replication id와 offset을 가지고 다시 부분 동기화를 시작한다.

## 2.3 **Sentinel**의 장단점

sentinel이 자동으로 master,slave 상태를 지속적으로 모니터링하고 장애를 감지하면 자동으로 failover하므로 서비스 중단없이 운영을 할 수 있다.  redis 서버 수, failover 정책 등을 사용자가 지정하여 유연하게 고가용성 솔루션을 구성할 수 있다.

설정 및 관리가 복잡하고 초기 설정이 복잡하다. sentinel과 노드간의 안정적인 네트워크 통신을 고려해야한다.

# 3. Redis **Cluster**

데이터를 여러 노드에 데이터를 분산 저장해 확장성을 제공한다. (DB의 샤딩과 비슷한 개념) 따라서 일부 노드가 죽거나 통신이 되지 않을때에도 작업을 계속할 수 있다. 단 과반수의 master 노드가 죽으면 클러스터도 중단된다.

## 3.1 동작 과정

Redis Cluster는 Hash slot를 사용하여 데이터를 분산하여 저장한다.

![CLUSTER](https://github.com/princenim/TIL/assets/59499600/5f147abd-a879-4063-84ed-39415e330cd2)

이때 Hash slot은 총 16,3874개 이며 만약에 Cluster를 노드3개로 구성한다면 Master Node1(0-5500), Master Node2(5501-11000), Master Node3(11001-16383)로 hash slot이 부여된다.

따라서 ‘user : 1000’ 이라는 키를 저장할때 이를 해싱하여 해시 슬롯을 결정하고, 그에 해당하는 노드에 데이터를 저장한다.

## 3.2 Auto Failover

![cluster2](https://github.com/princenim/TIL/assets/59499600/5f75b734-25c2-4ccc-aaf8-f966c4a04d48)

그리고 cluster도 master 노드에 대해서 slave(replica) 노드를 둘 수 있다. redis sentinel과 달리 클러스터 자체 내에서 장애 복구 매커니즘을 포함한다. 클러스터의 노드들이 서로 상태를 모니터링한다. Master노드가 죽을경우 master의 slave노드는 Gossip protocol을 통해 master의 죽음을 파악한 뒤 스스로 master로 승격하여 master을 대신한다. 그리고 죽은 master가 살아나서 동작하는 경우 다시 slave로 강등한다.

`Gossip protocol`은 모든 노드가 동일한 역할을 하고 중앙 관리자가 필요없다. 각 노드는 주기적으로 다른 노드에게 자신의 상태 정보를 전송한다. 이 정보에는 노드의 상태(정상, 실패),슬롯 정보, 복제 정보가 포함된다. 그리고 노드가 특정 기간동안 다른 노드로부터 응답을 받지 못하면 해당 노드를 잠재적 장애 상태로 간주하고 여러 노드가 해당 노드에 대해 장애를 감지하면 실패 상태로 확정한다.