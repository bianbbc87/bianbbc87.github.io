---
title: "kubernetes etcd 장애 시나리오 분석"
date: 2026-03-14 00:00:00 +0900
categories: [Infrastructure]
tags: [etcd, kubernetes, distributed-systems, raft, failure-scenario, disaster-recovery]
mermaid: true
---

# 들어가며

[이전 글](/posts/what-is-etcd)에서 etcd의 구조와 Raft 알고리즘을 살펴봤다.
요약하면 다음과 같다.

```mermaid
graph LR
    B["⭐ ETCD B<br/>(Leader)"]:::leader
    A["ETCD A<br/>(Follower)"]:::follower
    C["ETCD C<br/>(Follower)"]:::follower
    B -->|heartbeat| A
    B -->|heartbeat| C
    A -->|AppendEntry ACK| B
    C -->|AppendEntry ACK| B
    classDef leader fill:#dcfce7,stroke:#16a34a,color:#14532d,font-weight:bold
    classDef follower fill:#dbeafe,stroke:#3b82f6,color:#1e3a5f
```

| 구분 | 데이터 | 설명 |
|---|---|---|
| 영속 (디스크) | currentTerm | 현재 선거 회차 |
| 영속 (디스크) | votedFor | 이번 회차에서 내가 찍은 후보 |
| 영속 (디스크) | log[] | 클러스터에 적용된 모든 명령 기록 (멤버 설정 포함) |
| 휘발성 (모든 노드) | commitIndex | 과반수가 복제 완료해서 확정된 마지막 로그 번
호 |
| 휘발성 (모든 노드) | lastApplied | 실제 상태 머신(DB)에 반영된 마지막 로그 번
호 |
| 휘발성 (리더만) | nextIndex[] | 각 follower에게 다음에 보낼 로그 번호 |
| 휘발성 (리더만) | matchIndex[] | 각 follower가 여기까지 받았다고 확인된 로그 번
호 |

<br />
<br />

# ETCD의 장애
etcd는 Kubernetes의 **단일 진실 공급원(single source of truth)**이다.
모든 클러스터 상태가 etcd에 저장되기 때문에, etcd에 문제가 생기면 전체 클러스터에 영향을 준다.

```mermaid
graph TB
    subgraph normal["✅ 정상 상태"]
        direction LR
        e_ok["etcd 정상"]:::success
        api_ok["kube-apiserver<br/>정상 운영"]:::success
        ctl_ok["controller-manager"]:::success
        sched_ok["scheduler"]:::success
        kctl_ok["kubectl"]:::success
    end
    subgraph broken["⚡ etcd 장애 발생 후"]
        direction LR
        e_fail["🔴 etcd 장애"]:::failure
        api_fail["kube-apiserver<br/>Write 불가"]:::warning
        ctl_fail["❌ controller-manager<br/>상태 조정 불가"]:::impact
        sched_fail["❌ scheduler<br/>신규 Pod 배치 불가"]:::impact
        kctl_fail["❌ kubectl<br/>명령 실패"]:::impact
        kubelet["🟡 kubelet<br/>기존 Pod 자율 유지<br/>(단기간)"]:::note
        e_fail --> api_fail
        api_fail --> ctl_fail
        api_fail --> sched_fail
        api_fail --> kctl_fail
        api_fail --> kubelet
    end
    normal --> broken
    classDef success fill:#dcfce7,stroke:#22c55e,color:#14532d
    classDef failure fill:#fee2e2,stroke:#ef4444,color:#991b1b,font-weight:bold
    classDef warning fill:#fef3c7,stroke:#f59e0b,color:#92400e,font-weight:bold
    classDef impact fill:#fecaca,stroke:#dc2626,color:#7f1d1d
    classDef note fill:#f0fdf4,stroke:#22c55e,color:#14532d
```

이 글에서는 etcd에서 발생할 수 있는 장애 유형을 시나리오별로 분석하고, etcd의 메모리, 디스크 관리 방법과 백업, 복구 방법까지 함께 정리한다.

<br />
<br />

# 장애 시나리오 분석

etcd 장애는 발생 원인에 따라 크게 **세 계층**으로 나눌 수 있다.

**1. Raft 합의 계층** — 노드 상태나 네트워크 문제로 합의 자체가 흔들리는 경우

**2. 스토리지 계층** — 디스크 문제로 etcd 데이터 저장 자체가 불가해지는 경우

**3. 운영/멤버십 계층** — 클러스터를 구성하거나 노드를 추가/제거하는 과정에서 발생하는 경우


<br />
<br />

## Raft 합의 계층 장애

Raft 합의 계층의 장애는 달성 가능한 Quorom (정족수)가 전체 노드 중 몇 %냐에 따라 다르다.

### 1. ETCD node의 비정상 장애
#### 1.1. 소수 Follower 장애 (서비스 영향 없음)

**가장 흔하지만, 클러스터가 알아서 처리하는 유형이다.**

정족수(`(N/2) + 1`) 미만의 Follower가 장애 났을 때다.

```mermaid
graph TB
    B["⭐ ETCD B<br/>(Leader)"]:::leader
    A["ETCD A<br/>(Follower)"]:::follower
    C["ETCD C<br/>(Follower)"]:::follower
    D["❌ ETCD D<br/>(장애)"]:::failure
    E["❌ ETCD E<br/>(장애)"]:::failure
    B -->|"② AppendEntry"| A
    B -->|"③ AppendEntry"| C
    B -. "응답 없음" .- D
    B -. "응답 없음" .- E
    A -->|"① ACK"| B
    C -->|ACK| B
    classDef leader fill:#dcfce7,stroke:#16a34a,color:#14532d,font-weight:bold
    classDef follower fill:#dbeafe,stroke:#3b82f6,color:#1e3a5f
    classDef failure fill:#fee2e2,stroke:#ef4444,color:#991b1b
```

나머지 멤버들이 쿼럼을 유지하므로 **클러스터는 계속 Write 요청을 처리한다.**
kube-apiserver는 장애 난 멤버와의 연결이 끊기지만, **클라이언트 라이브러리가 자동으로 정상 멤버로 재연결**한다.

<br />

### 1.2. 다수 Follower 장애 (Quorom Loss)

정족수(`(N/2) + 1`) 이상의 노드가 동시에 장애 나면 Write가 불가능해진다.

```mermaid
graph TB
    Client["🚫 Client<br/>Write 불가"]:::blocked
    B["ETCD B<br/>(Follower/Candidate 반복)"]:::unstable
    A["ETCD A<br/>(Follower/Candidate 반복)"]:::unstable
    C["❌ ETCD C<br/>(장애)"]:::failure
    D["❌ ETCD D<br/>(장애)"]:::failure
    E["❌ ETCD E<br/>(장애)"]:::failure
    Client -. "❌" .-> B
    B --- A
    B -.- C
    B -.- D
    B -.- E
    classDef blocked fill:#fee2e2,stroke:#ef4444,color:#991b1b,font-weight:bold
    classDef unstable fill:#fef3c7,stroke:#f59e0b,color:#92400e
    classDef failure fill:#fecaca,stroke:#dc2626,color:#7f1d1d
```

노드 개수에 따른 정속수와 허용 장애 노드 수는 다음과 같다.
위 케이스는 5 nodes에 3대 이상 동시 장애에 해당한다.

| 노드 개수 | 정족수 | 허용 장애 | 쿼럼 손실 조건 |
|-------------|------|---------|-------------|
| 1개 | 1 | 0개 | 1대 장애 |
| 3개 | 2 | 1개 | **2대 이상 동시 장애** |
| 5개 | 3 | 2개 | **3대 이상 동시 장애** |

<br />
<br />

## Network 계층 장애

네트워크 스위치 장애, 방화벽 정책 변경, 클라우드 AZ 간 회선 단절 등이 원인으로, ETCD node 간 네트워크 경로가 끊어져 서로 통신할 수 없게 될 수 있다.

### 1. 단절된 Network Partition 장애

예룰 들어 단절된 두 Network Partition이 있다고 가정한다.

```mermaid
graph LR
    subgraph partA ["🟢 파티션 A (쿼럼 달성)"]
        N1["Node1<br/>(Leader)"]:::leader
        N2[Node2]:::normal
        N3[Node3]:::normal
    end
    subgraph partB ["🔴 파티션 B (고립)"]
        N4[Node4]:::isolated
        N5[Node5]:::isolated
    end
    N1 <--> N2
    N2 <--> N3
    N4 <--> N5
    N3 -. "❌ 단절" .- N4
    classDef leader fill:#dcfce7,stroke:#16a34a,color:#14532d,font-weight:bold
    classDef normal fill:#dbeafe,stroke:#3b82f6,color:#1e3a5f
    classDef isolated fill:#fee2e2,stroke:#ef4444,color:#991b1b
```

양쪽 파티션이 독립적으로 Write를 처리하여 데이터 일관성이 파괴되는 **Split-Brain**가 발생할 수 있다.
그러나 **etcd는 네트워크 파티션에서 Split-Brain이 발생하지 않는다.** ETCD 멤버 추가/제거 자체가 과반수 합의를 통해서만 가능하기 때문이다.
즉, 소수 파티션이 독립적인 클러스터로 분리되는 상황이 구조적으로 차단된다.

파티션의 동작은 **Leader가 어느 쪽에 있느냐**에 따라 두 가지로 나뉜다.

#### 1.1. 안전한 파티션 장애 - 정족수 유지

```
# Leader가 다수 쪽에 있는 경우
[Node1 Leader] ←→ [Node2] [Node3]   |   [Node4] [Node5]
        다수 쪽 (쿼럼 달성)                  소수 쪽
```

- 다수 쪽: 쿼럼 유지 → 정상 운영 계속
- 소수 쪽: election timeout → Candidate → 투표 요청 → **쿼럼 미달로 Leader 선출 실패** → Write 불가 대기


```
# Leader가 소수 쪽에 있는 경우
[Node1 Leader]   |   [Node2] [Node3] [Node4] [Node5]
    소수 쪽              다수 쪽 (쿼럼 달성)
```

```mermaid
graph TB
    subgraph cb1["① 파티션 발생 — Node1(Leader) 소수 쪽에 고립"]
        direction LR
        cb_n1a["🔴 Node1<br/>(Leader, 소수쪽)"]:::minority
        cb_n2a["Node2"]:::follower
        cb_n3a["Node3"]:::follower
        cb_n4a["Node4"]:::follower
        cb_n5a["Node5"]:::follower
        cb_n1a -. "❌ 단절" .- cb_n2a
        cb_n2a --- cb_n3a
        cb_n3a --- cb_n4a
    end
    subgraph cb2["② Node1: Commit 불가 / 다수 쪽: 새 Leader 선출"]
        direction LR
        cb_n1b["🔴 Node1<br/>Write 요청 거부<br/>(쿼럼 미달)"]:::blocked
        cb_n2b["⭐ Node2<br/>(새 Leader)"]:::leader
        cb_n3b["Node3"]:::follower
        cb_n2b -->|heartbeat| cb_n3b
    end
    subgraph cb3["③ 파티션 복구 — Node1 자동 Follower 전환"]
        direction LR
        cb_n2c["⭐ Node2<br/>(Leader)"]:::leader
        cb_n1c["✅ Node1<br/>(Follower 전환)"]:::success
        cb_n3c["Node3"]:::follower
        cb_n2c -->|heartbeat| cb_n1c
        cb_n2c -->|heartbeat| cb_n3c
    end
    cb1 --> cb2 --> cb3
    classDef minority fill:#fef3c7,stroke:#f59e0b,color:#92400e,font-weight:bold
    classDef follower fill:#dbeafe,stroke:#3b82f6,color:#1e3a5f
    classDef leader fill:#dcfce7,stroke:#16a34a,color:#14532d,font-weight:bold
    classDef blocked fill:#fee2e2,stroke:#ef4444,color:#991b1b,font-weight:bold
    classDef success fill:#dcfce7,stroke:#22c55e,color:#14532d
```

<br />

#### 1.2. 안전하지 않은 파티션 장애 - 정족수 미달

그러나 파티션 내에서 리더를 정하지 못하면 장애가 발생할 수 있다.
예를 들어 2:2로 Partition이 나뉘게 되는 경우, Leader 선출 실패로 장애가 발생할 수 있다.

<br />
<br />

## 스토리지 계층 장애

ETCD는 DB SIZE가 기본 쿼터인 **2GB**에 도달하면 etcd는 `mvcc: database space exceeded` 에러를 반환하며 read-only 모드로 전환된다.

```mermaid
graph TB
    subgraph db1["① etcd DB 2GB 한계 도달"]
        direction LR
        db_etcd1["⚠️ etcd<br/>DB: ~2GB (한계 도달)"]:::trigger
        db_note1["Compaction 미설정으로<br/>revision 데이터 누적"]:::note
    end
    subgraph db2["② etcd read-only 전환"]
        direction LR
        db_etcd2["🔴 etcd<br/>read-only 모드<br/>'mvcc: database space exceeded'"]:::failure
        db_api2["kube-apiserver<br/>Write 요청 전부 실패"]:::failure
        db_etcd2 --> db_api2
    end
    subgraph db3["③ 클러스터 제어 마비"]
        direction LR
        db_ctl["❌ controller-manager<br/>조정 불가"]:::impact
        db_sch["❌ scheduler<br/>배치 불가"]:::impact
        db_kctl["❌ kubectl<br/>명령 실패"]:::impact
    end
    db1 --> db2 --> db3
    classDef trigger fill:#fef3c7,stroke:#f59e0b,color:#92400e,font-weight:bold
    classDef note fill:#dbeafe,stroke:#3b82f6,color:#1e3a5f
    classDef failure fill:#fee2e2,stroke:#ef4444,color:#991b1b,font-weight:bold
    classDef impact fill:#fecaca,stroke:#dc2626,color:#7f1d1d
```

### 1. I/O 지연 장애
#### 1.1. 리더 재선출

WAL fsync가 지연되면 Leader가 heartbeat를 제때 보내지 못하게 된다.
그 결과로 **불필요한 리더 재선출이 발생**한다.

```mermaid
graph TB
    subgraph io1["① Leader 디스크 I/O 포화 — fdatasync 지연"]
        direction LR
        io_l1["⚠️ Leader<br/>fdatasync 지연 중"]:::io
        io_d1["💾 디스크 I/O 포화"]:::disk
        io_fa1["Follower A<br/>정상"]:::follower
        io_fb1["Follower B<br/>정상"]:::follower
        io_l1 --- io_d1
    end
    subgraph io2["② heartbeat 누락 → Follower election timeout"]
        direction LR
        io_l2["⚠️ Leader<br/>heartbeat 지연"]:::warning
        io_fa2["Follower A<br/>Election Timeout ⏰"]:::timeout
        io_fb2["Follower B<br/>Election Timeout ⏰"]:::timeout
    end
    subgraph io3["③ 리더 재선출 — 구 Leader Follower로 전환"]
        direction LR
        io_fa3["⭐ Follower A<br/>(새 Leader)"]:::leader
        io_l3["구 Leader<br/>(Follower 전환)"]:::demoted
        io_fb3["Follower B"]:::follower
        io_fa3 -->|heartbeat| io_l3
        io_fa3 -->|heartbeat| io_fb3
    end
    subgraph io4["④ 모든 lease revoke"]
        direction LR
        io_result["🔴 모든 lease 일괄 revoke<br/>분산 락 해제<br/>서비스 등록 전부 해제"]:::critical
    end
    io1 --> io2 --> io3 --> io4
    classDef io fill:#fef3c7,stroke:#f59e0b,color:#92400e
    classDef disk fill:#f3e8ff,stroke:#a855f7,color:#581c87
    classDef follower fill:#dbeafe,stroke:#3b82f6,color:#1e3a5f
    classDef warning fill:#ffe4e6,stroke:#f43f5e,color:#881337,font-weight:bold
    classDef timeout fill:#fef3c7,stroke:#f59e0b,color:#92400e
    classDef leader fill:#dcfce7,stroke:#16a34a,color:#14532d,font-weight:bold
    classDef demoted fill:#f3e8ff,stroke:#a855f7,color:#581c87
    classDef critical fill:#fecaca,stroke:#dc2626,color:#7f1d1d,font-weight:bold
```

그래서 ETCD는 **control plane과 동일한 디스크를 사용하지 않는 것**이 권장된다.
- 속도가 빠른 SSD 사용
- External ETCD Cluster 구성

#### 추가 사례
- [Issue #13648](https://github.com/etcd-io/etcd/issues/13648)에서는 Pod 약 50개, 초당 7개 key 변경이라는 **매우 낮은 부하**의 클러스터에서도 I/O 경합으로 인해 `Range request took too long`, WAL 타임아웃, 리더 재선출이 연쇄적으로 발생한 사례가 보고됐다.

- [Issue #15247](https://github.com/etcd-io/etcd/issues/15247)에서는 Leader가 `fdatasync`에 묶이면서 heartbeat를 놓쳐 리더십을 잃었고, 이 과정에서 **클러스터의 모든 lease가 일괄 revoke**됐다. 분산 락, 서비스 등록 등 lease 기반 기능을 사용하는 애플리케이션이 동시에 전부 영향을 받는 상황이다.

<br />

#### 1.2. 장애 복구 중 발생하는 추가 부하

장애 멤버가 복구되면 Follower는 Leader로부터 누락된 log를 동기화한다.
etcd는 snapshot-count(기본 100,000 entries) 이상이 쌓이면 메모리의 log를 파일로 truncate하는데, 만일 truncate된 구간이 있으면, Leader는 **log 대신 snapshot 파일 전체를 복구 멤버에게 전송**한다.

그런데 많은 log를 복구하는 과정에서 Leader의 I/O 지연이 발생하여 1.1과 같은 문제가 발생할 수 있다.

```mermaid
graph TB
    subgraph rb1["① Follower B 복구 — log 동기화 요청"]
        direction LR
        rb_l1["⭐ Leader"]:::leader
        rb_fa1["Follower A"]:::follower
        rb_fb1["🔧 Follower B<br/>(복구, 재연결)"]:::recovery
        rb_fb1 -->|"log 동기화 요청"| rb_l1
    end
    subgraph rb2["② log 구간이 truncate됨 → snapshot 전체 전송"]
        direction LR
        rb_l2["⭐ Leader<br/>📦 snapshot 전송 중<br/>(수십~수백 MB)"]:::heavy
        rb_fb2["Follower B<br/>snapshot 수신 중"]:::recovery
        rb_fa2["Follower A"]:::follower
        rb_l2 -->|"대용량 snapshot"| rb_fb2
        rb_l2 -. "heartbeat 지연" .- rb_fa2
    end
    subgraph rb3["③ Leader I/O 부하 → heartbeat 누락 → 리더 재선출"]
        direction LR
        rb_l3["⚠️ Leader<br/>heartbeat 지연"]:::warning
        rb_fa3["Follower A<br/>Election Timeout ⏰"]:::timeout
        rb_fa3new["⭐ Follower A<br/>(새 Leader)"]:::leader
        rb_fa3 --> rb_fa3new
    end
    subgraph rb4["④ 클러스터 Write 일시 중단"]
        direction LR
        rb_result["🔴 Write 중단<br/>(새 리더 선출 중)"]:::critical
    end
    rb1 --> rb2 --> rb3 --> rb4
    classDef leader fill:#dcfce7,stroke:#16a34a,color:#14532d,font-weight:bold
    classDef follower fill:#dbeafe,stroke:#3b82f6,color:#1e3a5f
    classDef recovery fill:#e0f2fe,stroke:#0284c7,color:#0c4a6e
    classDef heavy fill:#ffe4e6,stroke:#f43f5e,color:#881337,font-weight:bold
    classDef warning fill:#fef3c7,stroke:#f59e0b,color:#92400e
    classDef timeout fill:#fef3c7,stroke:#f59e0b,color:#92400e
    classDef critical fill:#fecaca,stroke:#dc2626,color:#7f1d1d,font-weight:bold
```

이는 etcd 저장 log가 너무 많은 것이 문제인 것으로, 주기적으로 etcd DB 크기를 확인하고, 가능하면 Compaction + Defrag로 이전 revision을 지우는 것이 좋다.

#### 추가 사례
- Issue [#13913](https://github.com/etcd-io/etcd/issues/13913)에서는 DB 크기가 35MB 이상인 클러스터에서 Follower 복구 중 대형 snapshot 전송으로 인해 **Leader가 heartbeat를 놓쳐 의도치 않은 리더 재선출이 반복**된 사례가 보고됐다.
- Issue [#2662](https://github.com/etcd-io/etcd/issues/2662)에서는 Follower 복구 중 Leader의 메모리가 **최대 40배**까지 치솟은 케이스도 있다.
- [Issue #14501](https://github.com/etcd-io/etcd/issues/14501)에서는 단일 Follower의 fsync 지연이 **클러스터 전체 Write latency를 끌어올린** 사례가 보고됐다. 심한 경우 "slow follower livelock"이 발생한다. (Follower가 snapshot을 받는 속도보다 쓰기가 더 빠르게 쌓여 영원히 따라잡지 못하는 상태)
- [실제 운영 포스트모텀](https://medium.com/@truonghongcuong68/when-etcd-collapses-how-a-single-node-replacement-took-down-my-kubernetes-control-plane-f913eec825cd) 사례 중 

1. etcd 멤버 1대 교체 → 신규 멤버 기동 → snapshot 동기화 시작
2. 대형 snapshot 전송으로 Leader 네트워크 포화
3. Leader → 나머지 Follower heartbeat 누락 → 리더 재선출
4. 구 멤버는 제거됐고 신규 멤버는 아직 동기화 중 → **쿼럼 손실**
5. control plane 전체 다운

이 발생한 케이스도 있다.


### 2. WAL / 데이터 손상 장애 (Corruption)

**가장 드물지만 복구가 어려운 유형이다.**

#### 2.1. 발생 원인
- 노드 강제 종료 중 WAL fsync 미완료
- 스토리지 하드웨어 장애 (bad sector)
- 잘못된 etcd 버전 다운그레이드

etcd가 기동 시 다음과 같은 에러를 출력하고 시작을 거부한다.

```
wal: crc mismatch
mvcc: db file is corrupt
```

#### 2.2. 복구 방법
3대 클러스터 기준, 1대가 손상되면 나머지 2대로 쿼럼을 유지하면서 손상된 멤버를 교체한다.

```bash
# 1. 손상된 멤버 제거
etcdctl member remove <member-id>

# 2. 해당 노드의 etcd 데이터 디렉토리 삭제
rm -rf /var/lib/etcd/

# 3. 기존 클러스터에 새 멤버로 재가입
etcdctl member add <new-member-name> --peer-urls=<peer-url>

# 4. etcd 재시작 (INITIAL_CLUSTER_STATE=existing 으로 설정)
```

<br />

## Leaner 도입으로 해겨된 장애

etcd v3.4부터 **Learner** 상태가 도입됐다.

Learner는 **투표권 없이 클러스터에 참여**하는 비투표 멤버다.
쿼럼 크기에 영향을 주지 않으면서 Leader의 log를 복제받아 따라잡을 수 있다.

```mermaid
graph TB
    subgraph lr1["① Learner로 참여 — 쿼럼 크기 변화 없음 (안전)"]
        direction LR
        lr_l1["⭐ Leader"]:::leader
        lr_fa1["Follower A"]:::follower
        lr_new1["🎓 new-node<br/>(Learner, 투표권 없음)"]:::learner
        lr_note1["쿼럼 = Leader + A = 2<br/>안전하게 유지"]:::note
    end
    subgraph lr2["② log 복제 — 데이터 따라잡기"]
        direction LR
        lr_l2["⭐ Leader"]:::leader
        lr_fa2["Follower A"]:::follower
        lr_new2["🎓 new-node<br/>(Learner, 동기화 중)"]:::learner
        lr_l2 -->|"log 복제"| lr_new2
    end
    subgraph lr3["③ member promote — 투표 멤버로 승격"]
        direction LR
        lr_l3["⭐ Leader"]:::leader
        lr_fa3["Follower A"]:::follower
        lr_new3["✅ new-node<br/>(Follower, 투표권 있음)"]:::success
        lr_l3 -->|heartbeat| lr_new3
        lr_note3["쿼럼 크기 증가 시점<br/>(동기화 완료 후 — 안전)"]:::note
    end
    lr1 --> lr2 --> lr3
    classDef leader fill:#dcfce7,stroke:#16a34a,color:#14532d,font-weight:bold
    classDef follower fill:#dbeafe,stroke:#3b82f6,color:#1e3a5f
    classDef learner fill:#e0f2fe,stroke:#0284c7,color:#0c4a6e,font-weight:bold
    classDef success fill:#dcfce7,stroke:#22c55e,color:#14532d,font-weight:bold
    classDef note fill:#f0fdf4,stroke:#22c55e,color:#14532d
```

이전까지는 다음과 같은 장애가 발생할 수 있었다.

### 1. I/O 지연 장애 CASE
#### 1.1. 신규 멤버가 Leader를 과부하시키는 경우

신규 멤버는 데이터가 전혀 없는 상태로 시작한다. 그래서 Follower 복구보다 **따라잡아야 할 양이 더 많다**.
Leader로부터 모든 log를 받아야 하는데, 장애 기간이 길수록 받아야 할 양이 많아지고, 이미 truncate된 구간은 snapshot 전체를 전송받아야 한다. 이로 인해 I/O 지연 장애가 발생할 가능성이 높아진다.

<br />

### 2. 네트워크 장애

이전에는 신규 멤버가 추가되면 **클러스터 크기와 정족수가 바뀌었다.**
예를 들어, 다음과 같은 상황이 발생할 수 있었다.

#### 2.1. 신규 멤버 추가 후 네트워크 파티션 장애 발생
**Case 1: 3대 클러스터에 1대 추가 → 총 4대 (쿼럼=3)**
**Case 2: 파티션 장애 발생 상태에서의 멤버가 추가 (쿼럼=3)**
| 파티션 구성 | 결과 |
|-----------|------|
| 3+1 (Leader가 3쪽) | 쿼럼 3 유지 → 정상 운영 |
| 2+2 | **양쪽 모두 쿼럼(3) 미달 → 리더 선출 불가** |

기존 3대일 때는 2+1 파티션이어도 2쪽에서 쿼럼을 유지할 수 있었지만, 4대 클러스터에서 2+2로 갈리면 양쪽 모두 쿼럼이 없어 **클러스터 전체가 마비**된다.

<br />

### 2.3. 잘못된 peer URL로 멤버 추가

`etcdctl member add` 명령은 peer URL의 유효성을 검증하지 않고 즉시 적용된다.
잘못된 URL이 추가되면 신규 etcd 프로세스 자체가 기동되지 못하고, 이미 쿼럼이 바뀐 상태가 된다.

```mermaid
graph TB
    subgraph wu1["① member add 실행 — URL 검증 없이 즉시 쿼럼 크기 증가"]
        direction LR
        wu_l1["⭐ Leader"]:::leader
        wu_fa1["Follower A"]:::follower
        wu_new1["➕ new-node<br/>--peer-urls=wrong-ip:2380"]:::newmember
        wu_note1["쿼럼 크기 즉시 2→3"]:::warning
    end
    subgraph wu2["② new-node 기동 실패 — 쿼럼은 이미 3"]
        direction LR
        wu_l2["⭐ Leader"]:::leader
        wu_fa2["Follower A"]:::follower
        wu_new2["❌ new-node<br/>(기동 실패)"]:::failure
        wu_note2["정상 멤버=2, 쿼럼=3<br/>→ 쿼럼 미달"]:::critical_note
    end
    subgraph wu3["③ Write 불가 — member remove도 불가 (쿼럼 없음)"]
        direction LR
        wu_stuck["🔴 쿼럼 미달로 Write 불가<br/>member remove도 쿼럼 필요<br/>→ 되돌릴 수 없음"]:::critical
    end
    subgraph wu4["④ 강제 복구 필요"]
        direction LR
        wu_force["etcd --force-new-cluster<br/>강제 재구성"]:::action
    end
    wu1 --> wu2 --> wu3 --> wu4
    classDef leader fill:#dcfce7,stroke:#16a34a,color:#14532d,font-weight:bold
    classDef follower fill:#dbeafe,stroke:#3b82f6,color:#1e3a5f
    classDef newmember fill:#e0f2fe,stroke:#0284c7,color:#0c4a6e
    classDef warning fill:#fef3c7,stroke:#f59e0b,color:#92400e,font-weight:bold
    classDef failure fill:#fee2e2,stroke:#ef4444,color:#991b1b
    classDef critical_note fill:#fef3c7,stroke:#f59e0b,color:#92400e
    classDef critical fill:#fecaca,stroke:#dc2626,color:#7f1d1d,font-weight:bold
    classDef action fill:#f3e8ff,stroke:#a855f7,color:#581c87
```

잘못된 멤버 추가 하나가 클러스터 전체를 불능 상태로 만들 수 있었다.

<br />
<br />

## ETCD node의 장애 복구 

### 1. 자동 장애 복구
과반수 멤버가 다시 정상화되면 클러스터가 자동으로 새 Leader를 선출하고 정상 복귀한다. 

<br />

#### 2.1. Leader 장애 & 재선출

**Leader node에 장애가 발생하는 경우이다.**

```mermaid
graph TB
    subgraph step1["① 정상 운영"]
        direction LR
        B1["⭐ ETCD B<br/>(Leader)"]:::leader
        C1a["ETCD C<br/>(Follower)"]:::follower
        C1b["ETCD D<br/>(Follower)"]:::follower
        B1 -->|heartbeat| C1a
        B1 -->|heartbeat| C1b
    end

    subgraph step2["② Leader 장애"]
        direction LR
        B2["❌ ETCD B<br/>(장애)"]:::failure
        C2a["ETCD C<br/>Election Timeout ⏰"]:::timeout
        C2b["ETCD D<br/>heartbeat 대기 중..."]:::timeout
    end

    subgraph step3["③ 선거 시작"]
        direction LR
        C3a["🗳️ ETCD C<br/>(Candidate)"]:::candidate
        B3["ETCD B"]:::normal
        C3b["ETCD D<br/>Election Timeout ⏰"]:::timeout
        C3a -->|"RequestVote RPC"| B3
        C3a --> C3b
    end

    subgraph step4["④ 새 Leader 선출"]
        direction LR
        C4a["⭐ ETCD C<br/>(새 Leader)"]:::leader
        B4["ETCD B<br/>(Follower)"]:::follower
        C4b["ETCD D<br/>(Follower)"]:::follower
        C4a -->|heartbeat| B4
        C4a -->|heartbeat| C4b
    end

    step1 --> step2 --> step3 --> step4

    classDef leader fill:#dcfce7,stroke:#16a34a,color:#14532d,font-weight:bold
    classDef follower fill:#dbeafe,stroke:#3b82f6,color:#1e3a5f
    classDef failure fill:#fee2e2,stroke:#ef4444,color:#991b1b,font-weight:bold
    classDef timeout fill:#fef3c7,stroke:#f59e0b,color:#92400e
    classDef candidate fill:#fef3c7,stroke:#f59e0b,color:#92400e,font-weight:bold
    classDef normal fill:#dbeafe,stroke:#3b82f6,color:#1e3a5f
```

**Leader 선출 소요 시간**은 수백 ms ~ 수 초 정도로, 실제로는 거의 실시간이라고 한다.
(election timeout 기반 감지)

- Leader 선출이 완료되기 전까지 [Write 요청은 실패가 아니라 큐에 대기](https://etcd.io/docs/v3.5/op-guide/failures/#leader-failure) 하다가 선출 후 처리된다.
- 이미 [Commit된 Write는 절대 유실되지 않는다.]((https://etcd.io/docs/v3.5/op-guide/failures/#leader-failure)) **단, Leader 장애 직전에 보낸 Write 중 아직 Commit되지 않은 것은 유실될 수 있다.**


```mermaid
graph TB
    subgraph uw1["① Client → Leader write 요청 / AppendEntry 전송 중"]
        direction LR
        uw_c1["Client"]:::client
        uw_l1["⭐ Leader<br/>WAL 기록 완료"]:::leader
        uw_fa1["Follower A<br/>(복제 중...)"]:::pending
        uw_fb1["Follower B<br/>(복제 중...)"]:::pending
        uw_c1 -->|"write(x=1)"| uw_l1
        uw_l1 -. "AppendEntry" .-> uw_fa1
        uw_l1 -. "AppendEntry" .-> uw_fb1
    end
    subgraph uw2["② ⚡ 쿼럼 달성 전 Leader 장애"]
        direction LR
        uw_l2["❌ Leader (장애)"]:::failure
        uw_fa2["Follower A<br/>(복제 미완료)"]:::pending
        uw_fb2["Follower B<br/>(복제 미완료)"]:::pending
    end
    subgraph uw3["③ 새 Leader 선출 — uncommitted log 덮어씀"]
        direction LR
        uw_fa3["⭐ Follower A<br/>(새 Leader)"]:::leader
        uw_fb3["Follower B"]:::follower
        uw_l3["구 Leader<br/>(Follower 전환)"]:::demoted
        uw_fa3 -->|heartbeat| uw_fb3
        uw_fa3 -->|heartbeat| uw_l3
    end
    subgraph uw4["④ 결과: write(x=1) 유실"]
        direction LR
        uw_c4["Client<br/>timeout"]:::client
        uw_lost["❌ uncommitted log 유실<br/>쿼럼 달성 전 장애로<br/>새 Leader가 덮어씌움"]:::lost
    end
    uw1 --> uw2 --> uw3 --> uw4
    classDef client fill:#f3e8ff,stroke:#a855f7,color:#581c87
    classDef leader fill:#dcfce7,stroke:#16a34a,color:#14532d,font-weight:bold
    classDef follower fill:#dbeafe,stroke:#3b82f6,color:#1e3a5f
    classDef pending fill:#fef3c7,stroke:#f59e0b,color:#92400e
    classDef failure fill:#fee2e2,stroke:#ef4444,color:#991b1b,font-weight:bold
    classDef demoted fill:#f3e8ff,stroke:#a855f7,color:#581c87
    classDef lost fill:#fecaca,stroke:#dc2626,color:#7f1d1d,font-weight:bold
```

<br />

#### 2.2. Follower 장애 & 복구

**Follower node에 장애가 발생하는 경우이다.**

쿼럼 이하의 Follower 장애라면 [클러스터는 계속 정상 운영된다.](https://etcd.io/docs/v3.5/op-guide/failures/#follower-failure) Leader는 장애 난 Follower에게 `AppendEntry`를 계속 시도하고, 나머지 멤버로 쿼럼을 채워 Write를 처리한다.

```mermaid
graph TB
    subgraph fw1["① Follower B 장애 — 쿼럼은 유지됨"]
        direction LR
        fw_l1["⭐ Leader"]:::leader
        fw_fa1["Follower A"]:::follower
        fw_fb1["❌ Follower B<br/>(장애)"]:::failure
        fw_l1 -->|AppendEntry| fw_fa1
        fw_l1 -. "응답 없음" .- fw_fb1
        fw_note1["쿼럼 = Leader + A = 유지<br/>클러스터 정상 운영 지속"]:::note
    end
    subgraph fw2["② Follower B 복구 — log 동기화 요청"]
        direction LR
        fw_l2["⭐ Leader"]:::leader
        fw_fa2["Follower A"]:::follower
        fw_fb2["🔧 Follower B<br/>(복구, 재연결)"]:::recovery
        fw_l2 -->|AppendEntry| fw_fa2
        fw_fb2 -->|"log 동기화 요청"| fw_l2
    end
    subgraph fw3["③ Leader → 누락 log 전송 (nextIndex[] 기준)"]
        direction LR
        fw_l3["⭐ Leader"]:::leader
        fw_fa3["Follower A"]:::follower
        fw_fb3["🔧 Follower B<br/>(log 수신 중)"]:::recovery
        fw_l3 -->|AppendEntry| fw_fa3
        fw_l3 -->|"누락 log 전송"| fw_fb3
    end
    subgraph fw4["④ 동기화 완료 — 정상 멤버 복귀"]
        direction LR
        fw_l4["⭐ Leader"]:::leader
        fw_fa4["Follower A"]:::follower
        fw_fb4["✅ Follower B<br/>(복귀)"]:::success
        fw_l4 -->|AppendEntry| fw_fa4
        fw_l4 -->|AppendEntry| fw_fb4
    end
    fw1 --> fw2 --> fw3 --> fw4
    classDef leader fill:#dcfce7,stroke:#16a34a,color:#14532d,font-weight:bold
    classDef follower fill:#dbeafe,stroke:#3b82f6,color:#1e3a5f
    classDef failure fill:#fee2e2,stroke:#ef4444,color:#991b1b,font-weight:bold
    classDef recovery fill:#e0f2fe,stroke:#0284c7,color:#0c4a6e
    classDef success fill:#dcfce7,stroke:#22c55e,color:#14532d,font-weight:bold
    classDef note fill:#f0fdf4,stroke:#22c55e,color:#14532d
```

<br />

#### 추가 개념: Split Vote (동시에 Candidate가 되는 경우)

만일 Leader node에 장애가 발생해서 여러 Follower 중 하나가 Leader가 되야하는 상황이라 가정하자. 만일 둘 다 Candidate가 되는 경우에는 어떻게 해야할까 ?

ETCD는 이 문제를 방지하기 위해 Raft 알고리즘의 Election Timeout을 **150ms~300ms 랜덤**으로 설계하였다.
그래서 보통 한 Follower가 먼저 timeout되어 빠르게 리더가 선출된다.
하지만 드물게 **두 Follower가 거의 동시에 timeout되면 Split Vote가 발생**한다.

```mermaid
graph TB
    subgraph sv1["① Leader 다운"]
        direction LR
        N1["❌ Node1<br/>(Leader 장애)"]:::failure
        N2a["Node2<br/>(Follower)"]:::follower
        N3a["Node3<br/>(Follower)"]:::follower
    end

    subgraph sv2["② 동시에 Election Timeout — Split Vote 발생"]
        direction LR
        N2b["🗳️ Node2<br/>(Candidate, Term:2)<br/>자기 자신에게 투표"]:::candidate
        N3b["🗳️ Node3<br/>(Candidate, Term:2)<br/>자기 자신에게 투표"]:::candidate
        N2b -. "투표 요청 거절<br/>(이미 투표함)" .- N3b
    end

    subgraph sv3["③ 결과: 쿼럼 미달 — 아무도 Leader 없음"]
        direction LR
        N2c["Node2<br/>쿼럼(2) 미달"]:::blocked
        N3c["Node3<br/>쿼럼(2) 미달"]:::blocked
        retry["⏰ 다음 Term에서<br/>랜덤 timeout 재시도"]:::retry
    end

    sv1 --> sv2 --> sv3

    classDef failure fill:#fee2e2,stroke:#ef4444,color:#991b1b,font-weight:bold
    classDef follower fill:#dbeafe,stroke:#3b82f6,color:#1e3a5f
    classDef candidate fill:#fef3c7,stroke:#f59e0b,color:#92400e,font-weight:bold
    classDef blocked fill:#f3e8ff,stroke:#a855f7,color:#581c87
    classDef retry fill:#e0f2fe,stroke:#0284c7,color:#0c4a6e
```

아무도 리더가 되지 못하면, 두 Candidate 모두 다시 **랜덤한 Election Timeout**을 기다린다.
다음 라운드에서 한 쪽이 먼저 timeout되면 상대방이 아직 Follower 상태여서 투표해준다.

| 상황 | 결과 |
|------|------|
| 한 Follower만 먼저 timeout | 빠르게 리더 선출 (일반적) |
| 동시에 timeout (Split Vote) | 이번 Term은 리더 없음 → 다음 Term에서 재시도 |
| Split Vote 반복 | 극히 드묾, 랜덤 timeout이 확률적으로 해소 |

> Split Vote는 **Safety(데이터 일관성)를 깨지 않는다.** 아무도 쿼럼을 달성하지 못했기 때문에 **데이터 Write도 발생하지 않는다.**

<br />
<br />

### 2. 수동 장애 복구
`Quorom Loss`로 **과반수 멤버가 복구 불가한 경우에는 스냅샷을 이용한 수동 복원이 필요하다.** 

#### 2.1. 스냅샷을 이용한 복원

etcd 복구를 위해 **주기적인 스냅샷 백업**이 필요하다.

```bash
# 스냅샷 백업
etcdctl snapshot save /backup/etcd-snapshot-$(date +%Y%m%d%H%M).db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
  --key=/etc/kubernetes/pki/etcd/healthcheck-client.key

# 스냅샷 상태 확인
etcdctl snapshot status /backup/etcd-snapshot.db --write-out=table
```

**`etcdctl snapshot save`과 단순 파일 복사의 차이**
- `etcdctl snapshot save`로 만든 파일에는 **무결성 해시(hash)가 포함**된다.
- `etcdctl snapshot restore` 시 파일 변조 여부를 자동으로 검증한다.
- 단순 복사로 만든 파일로 복구할 때는 `--skip-hash-check` 옵션이 필요하다.

`Compaction`과 `Defrag`를 수행한 후 백업하면, 불필요한 revision 데이터가 제거되어 **백업 파일 크기를 줄일 수 있다.**

<br />

```bash
# 1. kube-apiserver 중지
#    /etc/kubernetes/manifests/kube-apiserver.yaml 임시 이동

# 2. 기존 etcd 데이터 백업
mv /var/lib/etcd /var/lib/etcd.bak

# 3. 스냅샷으로 데이터 복원
etcdctl snapshot restore /backup/etcd-snapshot.db \
  --name=<member-name> \
  --initial-cluster=<member-name>=https://<ip>:2380 \
  --initial-advertise-peer-urls=https://<ip>:2380 \
  --data-dir=/var/lib/etcd

# 4. etcd 재시작

# 5. kube-apiserver 복구
#    /etc/kubernetes/manifests/kube-apiserver.yaml 복원
```

**복구 시 새로운 클러스터 메타데이터 사용이 중요하다.**

etcd 메타데이터에는 클러스터 UUID와 멤버 UUID가 저장된다.
이전 메타데이터를 그대로 사용하면, **장애 났던 서버가 다시 살아날 경우 충돌이 발생**할 수 있다.
`etcdctl snapshot restore` 명령은 자동으로 새 클러스터 메타데이터를 생성하므로, 단순 파일 복사가 아닌 이 명령을 사용해야 한다.

> kubeadm 클러스터에서는 `/etc/kubernetes/manifests/` 아래 Static Pod manifest를 이동시키는 것으로 컴포넌트를 중지/재시작할 수 있다.

<br />

### 추가 개념: 자동 백업 구성 방법

수동 백업은 누락될 수 있기 때문에 **CronJob으로 주기적 백업을 자동화**하는 것을 권장한다고 한다.

```yaml
# etcd-backup-cronjob.yaml (예시)
apiVersion: batch/v1
kind: CronJob
metadata:
  name: etcd-backup
  namespace: kube-system
spec:
  schedule: "0 2 * * *"   # 매일 새벽 2시
  jobTemplate:
    spec:
      template:
        spec:
          hostNetwork: true
          containers:
          - name: etcd-backup
            image: bitnami/etcd:latest
            command:
            - /bin/sh
            - -c
            - |
              etcdctl snapshot save /backup/etcd-$(date +%Y%m%d).db
              # 오래된 백업 삭제 (5일치 유지)
              find /backup -name "etcd-*.db" -mtime +5 -delete
            env:
            - name: ETCDCTL_API
              value: "3"
            - name: ETCDCTL_ENDPOINTS
              value: "https://127.0.0.1:2379"
            # cacert, cert, key 마운트 필요
          restartPolicy: OnFailure
```

주요 고려사항
- 모든 클러스터가 동시에 백업하면 네트워크 부하가 집중된다. 클러스터마다 다른 시간을 설정하는 것이 좋다.
- 백업 파일은 오브젝트 스토리지(S3 등)에 저장하여 노드 장애에도 안전하게 보존한다.
- 백업 전 `Compaction` + `Defrag`를 수행하면 백업 크기를 줄일 수 있다. (오래된 revision 제거)

<br />
<br />


## 장애 시나리오 요약

| # | 시나리오 | 자동 복구 | 서비스 영향 | 핵심 대응 |
|---|---------|---------|-----------|---------|
| 1 | 소수 Follower 장애 | 가능 (자동 재연결) | 없음 (부하 증가) | 빠른 복구로 쿼럼 여유 유지 |
| 2 | Quorum Loss | 과반수 복구 시 자동 | 클러스터 제어 전체 마비 | 노드 복구 또는 스냅샷 복원 |
| 3 | Leader 장애 | 가능 (수 초 내 재선출) | Write 큐 대기, uncommitted 유실 가능 | 모니터링으로 반복 여부 확인 |
| 5 | Network Partition | 가능 (파티션 복구 시 자동) | 소수 파티션 Write 불가 | 네트워크 복구 |
| 6 | 디스크 고갈 | 불가 | Write 거부 → 제어 마비 | Compaction + Defrag |
| 7 | WAL / 데이터 손상 | 부분 (멤버 재가입) | 개별 멤버 기동 불가 | 멤버 제거 후 재가입 |

<br />
<br />

## 함께 사용하면 좋을 모니터링 지표

| 메트릭 | 설명 | 임계값 |
|--------|------|--------|
| `etcd_server_has_leader` | 리더 존재 여부 | `== 0` 이면 Alert |
| `etcd_server_leader_changes_seen_total` | 리더 변경 횟수 | 1시간에 3회 이상이면 이상 징후 |
| `etcd_server_proposals_failed_total` | 합의 실패 횟수 | 증가 추세이면 Alert |
| `etcd_disk_wal_fsync_duration_seconds` | WAL fsync 지연 | p99 > 10ms 이상이면 경고 |
| `etcd_disk_backend_commit_duration_seconds` | BoltDB commit 지연 | p99 > 25ms 이상이면 경고 |
| `etcd_mvcc_db_total_size_in_bytes` | DB 사이즈 | 쿼터의 80% 초과 시 경고 |
| `etcd_network_peer_round_trip_time_seconds` | 피어 간 RTT | p99 > 100ms 이상이면 경고 |

I/O 지연 지표(`wal_fsync`, `backend_commit`)는 단순 장애 감지를 넘어 **하드웨어 성능 저하나 디스크 경합의 조기 신호**로 활용할 수 있다.

<br />
<br />

## 참고 자료

- [etcd 공식 운영 가이드](https://etcd.io/docs/v3.5/op-guide/)
- [etcd Failure Modes 공식 문서](https://etcd.io/docs/v3.5/op-guide/failures/)
- [etcd 재해 복구](https://etcd.io/docs/v3.5/op-guide/recovery/)
- [etcd Learner 설계 문서 — 멤버십 재구성 문제 및 Learner 도입 배경](https://etcd.io/docs/v3.3/learning/learner/)
- [Kubernetes etcd 백업](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)
- [kakao tech Kubernetes 운영을 위한 etcd 기본 동작 원리의 이해](https://tech.kakao.com/posts/484)
- [GitHub #13913 — Large snapshots cause missed leader heartbeats](https://github.com/etcd-io/etcd/issues/13913)
- [GitHub #2662 — 40x memory overhead on leader when recovering dead follower](https://github.com/etcd-io/etcd/issues/2662)
- [GitHub #14501 — Slow follower degrading overall write latency](https://github.com/etcd-io/etcd/issues/14501)
- [GitHub #13648 — High I/O load causes range errors and leader re-elections](https://github.com/etcd-io/etcd/issues/13648)
- [GitHub #15247 — All leases revoked when leader stuck in fdatasync](https://github.com/etcd-io/etcd/issues/15247)
- [GitHub #18052 — Broken TCP connection causes ~17-minute reconnect delay](https://github.com/etcd-io/etcd/issues/18052)
- [Postmortem — When etcd Collapses: Single Node Replacement Took Down Kubernetes Control Plane](https://medium.com/@truonghongcuong68/when-etcd-collapses-how-a-single-node-replacement-took-down-my-kubernetes-control-plane-f913eec825cd)
