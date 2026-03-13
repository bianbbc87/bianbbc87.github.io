---
title: "kubernetes etcd 장애 시나리오 분석"
date: 2026-03-14 00:00:00 +0900
categories: [Infrastructure]
tags: [etcd, kubernetes, distributed-systems, raft, failure-scenario, disaster-recovery]
mermaid: true
---

## 들어가며

[이전 글](/posts/what-is-etcd)에서 etcd의 구조와 Raft 알고리즘을 살펴봤다.

etcd는 Kubernetes의 **단일 진실 공급원(single source of truth)**이다.
모든 클러스터 상태가 etcd에 저장되기 때문에, etcd에 문제가 생기면 그 영향은 단순히 "etcd가 죽었다"로 끝나지 않는다.

```
etcd 장애
    │
    ▼
kube-apiserver Write 불가
    │
    ├─→ kubectl 명령 실패
    ├─→ controller-manager 상태 조정 불가
    ├─→ scheduler 신규 Pod 배치 불가
    └─→ 기존 Running Pod는 kubelet이 자율 유지 (단기간)
```

etcd가 완전히 다운되더라도 **이미 Running 중인 Pod**는 kubelet이 로컬에서 계속 유지한다.
그러나 장애 복구, 신규 배포, 스케일링 등 **제어 기능이 전부 마비**된다.

이 글에서는 etcd에서 발생할 수 있는 장애 유형을 시나리오별로 분석하고, etcd의 메모리, 디스크 관리 방법과 백업·복구 방법까지 함께 정리한다.

<br />
<br />

## 장애 시나리오 분석

### 1. 소수 Follower 장애

**가장 흔하고, 클러스터가 알아서 처리하는 유형이다.**

쿼럼(`(N/2) + 1`) 미만의 Follower가 장애 났을 때다.

```
5대 클러스터, Follower 2대 장애

[Leader]  [Follower1]  [Follower2]  [Follower3 ✗]  [Follower4 ✗]
                                        장애              장애

쿼럼 = 3, 현재 정상 멤버 = 3 → 쿼럼 유지 → 정상 운영 계속
```

나머지 멤버들이 쿼럼을 유지하므로 **클러스터는 계속 Write 요청을 처리한다.**
kube-apiserver는 장애 난 멤버와의 연결이 끊기지만, **클라이언트 라이브러리가 자동으로 정상 멤버로 재연결**한다.

장애 멤버가 복구되면 Leader로부터 누락된 log를 자동으로 동기화한다.
단, 정상 멤버들의 **부하가 증가**하므로, 장애 멤버 수가 늘어나 쿼럼을 잃기 전에 빠르게 복구해야 한다.

<br />

---

### 2. Quorum Loss (쿼럼 손실)

**가장 심각한 etcd 장애 유형이다.**

쿼럼(`(N/2) + 1`) 이상의 노드가 동시에 장애 나면 Write가 불가능해진다.

| 클러스터 크기 | 쿼럼 | 허용 장애 | 쿼럼 손실 조건 |
|-------------|------|---------|-------------|
| 1개 | 1 | 0개 | 1대 장애 |
| 3개 | 2 | 1개 | **2대 이상 동시 장애** |
| 5개 | 3 | 2개 | **3대 이상 동시 장애** |

```
Control Plane 노드 2대 이상 동시 장애 (3대 클러스터 기준)
    │
    ▼
etcd 쿼럼 손실 → Raft Write 합의 불가
    │
    ▼
kube-apiserver: 모든 쓰기 요청 503
    │
    ▼
scheduler / controller-manager 동작 마비
    │
    ▼
클러스터 제어 기능 전체 마비
```

**증상**: `kubectl get pods` 등 모든 명령 응답 없음, ArgoCD Sync 실패

**감지 메트릭**:

```promql
etcd_server_has_leader == 0
rate(etcd_server_proposals_failed_total[5m]) > 0
```

**복구**: 과반수 멤버가 다시 정상화되면 클러스터가 자동으로 새 Leader를 선출하고 정상 복귀한다.
새 Leader는 모든 lease 타임아웃을 자동으로 연장하여 서버 측 장애로 lease가 만료되지 않도록 보장한다.
과반수 멤버가 복구 불가한 경우에는 스냅샷을 이용한 수동 복원이 필요하다.

<br />

---

### 3. Leader 장애 & 재선출

**일반적으로 자동 복구 가능한 시나리오다.**

```
etcd Leader 노드 장애
    │
    ▼
Follower들: Election Timeout 발생 (~150ms ~ 300ms)
    │
    ▼
Election Timeout 먼저 만료된 Follower → Candidate로 전환
    │
    ▼
RequestVote RPC 전송 → 쿼럼 이상 투표 획득 → 새 Leader 선출
    │
    ▼
새 Leader가 heartbeat 전송 시작 + 모든 lease 타임아웃 자동 연장
    │
    ▼
kube-apiserver: etcd 재연결 → 정상 복구
```

**Leader 선출 소요 시간**: 수백 ms ~ 수 초 (election timeout 기반 감지)

Leader 선출이 완료되기 전까지 **Write 요청은 실패가 아니라 큐에 대기**하다가 선출 후 처리된다.
단, Leader 장애 직전에 보낸 Write 중 **아직 Commit되지 않은 것은 유실될 수 있다.**

```
Client → 구 Leader: write(x=1)
    ▼
구 Leader: WAL에 기록, Follower에 AppendEntry 전송
    ▼ (이 시점에서 Leader 장애)
Follower 과반수에 복제 미완료 → 쿼럼 달성 전 장애
    ▼
새 Leader 선출 → 구 Leader의 uncommitted log 덮어씀
    ▼
write(x=1)은 유실 (Client 입장에서는 timeout)
```

이미 **Commit된 Write는 절대 유실되지 않는다.** Raft는 committed log를 영구 보장한다.

**감지 메트릭**:

```promql
# Leader 변경이 1시간에 3회 이상이면 네트워크 불안정 또는 리소스 부족 신호
rate(etcd_server_leader_changes_seen_total[1h]) > 3
```

<br />

#### Split Vote (동시에 Candidate가 되는 경우)

Raft가 Election Timeout을 **150ms~300ms 랜덤**으로 설계한 이유가 이 문제를 방지하기 위해서다.
보통은 한 Follower가 먼저 timeout되어 빠르게 리더가 선출되지만, 드물게 두 Follower가 거의 동시에 timeout되면 **Split Vote**가 발생한다.

```
Leader 다운 (3대 클러스터)

[Node2]              [Node3]
    │                    │
    └── 거의 동시에 Election Timeout 만료 ──┘
    │                    │
    ▼                    ▼
Candidate (Term:2)   Candidate (Term:2)
  자기 자신에게 투표    자기 자신에게 투표
  상대방 투표 요청 거절  상대방 투표 요청 거절

결과: 둘 다 쿼럼(2) 미달 → 아무도 Leader가 되지 못함
```

아무도 리더가 되지 못하면, 두 Candidate 모두 다시 **랜덤한 Election Timeout**을 기다린다.
다음 라운드에서 한 쪽이 먼저 timeout되면 상대방이 아직 Follower 상태여서 투표해준다.

| 상황 | 결과 |
|------|------|
| 한 Follower만 먼저 timeout | 빠르게 리더 선출 (일반적) |
| 동시에 timeout (Split Vote) | 이번 Term은 리더 없음 → 다음 Term에서 재시도 |
| Split Vote 반복 | 극히 드묾, 랜덤 timeout이 확률적으로 해소 |

> Split Vote는 **Safety(데이터 일관성)를 깨지 않는다.** 아무도 쿼럼을 달성하지 못했기 때문에 데이터 Write도 발생하지 않는다.

<br />

---

### 4. Leader Step Down (Leader 자기 자신 제거)

**멤버 제거 요청이 Leader 자신을 대상으로 할 때 발생하는 특수한 케이스다.**

일반적인 멤버 제거 요청은 log 복제 방식으로 처리된다.
그러나 Leader가 자기 자신을 제거하라는 요청을 받으면 **일반 쿼럼 계산에서 자기 자신을 제외**하고 처리한다.

```
4대 클러스터 (쿼럼=3), Leader에게 Leader 제거 요청 수신

Leader: 자신을 제외한 쿼럼(3-1=2) 이상에 Cnew 복제
    │
    ▼
나머지 3대 전부에 Cnew 복제 완료 (쿼럼 달성)
    │
    ▼
Leader: Cnew를 commit → 클라이언트에 OK 응답
    │
    ▼
Leader: Step Down (heartbeat 전송 중단)
    │
    ▼
나머지 Follower 중 election timeout → 새 Leader 선출
```

**자기 자신을 제외한 쿼럼으로 복제한 이유**: Step Down 이후에도 나머지 서버 중에서 새 Leader가 반드시 선출될 수 있음을 보장하기 위해서다.

**Step Down 전에 Write 요청을 받으면?**
Config log(Cnew)가 아직 commit되지 않은 상태에서도 자신을 제외한 쿼럼만큼 log replication을 수행한다.
이는 Step Down 이후 새 Leader가 safety를 유지하며 정상 동작할 수 있도록 하기 위한 설계다.

**멤버 삭제 제한 (Restriction)**:
현재 정상 동작하는 멤버 수(`started`)가 쿼럼보다 작아질 것으로 예상되면, Leader는 멤버 삭제 요청을 **거절**한다.

```
예: 5대 클러스터 (쿼럼=3)
  2대 삭제 후: started=3, quorum=3 → 허용
  3대 삭제 시: started=2 < quorum=3 → 거절
```

<br />

---

### 5. Network Partition (네트워크 파티션)

**etcd는 네트워크 파티션에서 Split-Brain이 발생하지 않는다.**

파티션의 동작은 **Leader가 어느 쪽에 있느냐**에 따라 두 가지로 나뉜다.

#### Case A: Leader가 다수 쪽에 있는 경우

```
[Node1 Leader] ←→ [Node2] [Node3]   |   [Node4] [Node5]
        다수 쪽 (쿼럼 달성)                  소수 쪽
```

- 다수 쪽: 쿼럼 유지 → 정상 운영 계속
- 소수 쪽: election timeout → Candidate → 투표 요청 → **쿼럼 미달로 Leader 선출 실패** → Write 불가 대기

#### Case B: Leader가 소수 쪽에 있는 경우

```
[Node1 Leader]   |   [Node2] [Node3] [Node4] [Node5]
    소수 쪽              다수 쪽 (쿼럼 달성)
```

```
소수 쪽 Leader(Node1): heartbeat 전송 → 응답 없음
    ▼
Node1: Commit 불가 (쿼럼 미달) → Write 요청 거부
    ▼
다수 쪽: election timeout → 새 Leader 선출
    ▼
Node1: 더 높은 term의 heartbeat 수신 → 자동으로 Follower 전환
```

**Split-Brain이 발생하지 않는 이유**: etcd는 **멤버 추가/제거 자체가 과반수 합의를 통해서만 가능**하다.
소수 파티션이 독립적인 클러스터로 분리되는 상황이 구조적으로 차단된다.

**파티션 복구 시**: 소수 쪽(또는 구 Leader)은 **자동으로 다수 쪽의 새 Leader를 인식하고 log를 동기화하며 Follower로 합류**한다.

<br />

---

### 6. 디스크 고갈 / I/O 지연

**조용히 발전하다 갑자기 터지는 유형이다.**

| 원인 | 설명 |
|------|------|
| Compaction 미설정 | revision 데이터가 무한 증가 |
| DB 사이즈 쿼터 초과 | etcd 기본 DB 사이즈: 2GB |
| 디스크 I/O 포화 | WAL fsync 지연 → Raft 합의 타임아웃 |
| 다른 프로세스와 디스크 경합 | 컨테이너 로그, Prometheus TSDB 등 |

DB SIZE가 기본 쿼터인 **2GB**에 도달하면 etcd는 `mvcc: database space exceeded` 에러를 반환하며 read-only 모드로 전환된다.

```
etcd DB 사이즈 2GB 초과
    ▼
etcd: "mvcc: database space exceeded" → read-only 모드
    ▼
kube-apiserver: Write 요청 전부 실패
    ▼
클러스터 제어 기능 마비
```

**감지 메트릭**:

```promql
etcd_mvcc_db_total_size_in_bytes / 2147483648 > 0.8
etcd_disk_wal_fsync_duration_seconds   # p99 > 10ms 경고
etcd_disk_backend_commit_duration_seconds  # p99 > 25ms 경고
```

대응 방법은 아래 **Compaction & Defragmentation** 섹션에서 다룬다.

<br />

---

### 7. WAL / 데이터 손상 (Corruption)

**가장 드물지만 복구가 어려운 유형이다.**

**발생 원인**:
- 노드 강제 종료 중 WAL fsync 미완료
- 스토리지 하드웨어 장애 (bad sector)
- 잘못된 etcd 버전 다운그레이드

etcd가 기동 시 다음과 같은 에러를 출력하고 시작을 거부한다:

```
wal: crc mismatch
mvcc: db file is corrupt
```

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

---

### 8. 부트스트랩 중 장애

**클러스터 초기 구성 중 발생하는 특수한 유형이다.**

etcd 클러스터 부트스트랩은 **필요한 모든 멤버가 성공적으로 기동되어야** 완료된다. 하나라도 실패하면 전체 구성이 중단된다.

```
3대 클러스터 부트스트랩:
[Node1] 기동 성공  [Node2] 기동 성공  [Node3] 기동 실패
                                             ▼
                              클러스터 부트스트랩 전체 실패
```

부트스트랩 단계에서는 Raft 자체가 아직 동작하지 않으므로 복구보다 **처음부터 재구성하는 것이 훨씬 빠르고 안전**하다.

```bash
# 모든 멤버의 데이터 디렉토리 삭제
rm -rf /var/lib/etcd/

# 새 cluster-token으로 재부트스트랩
# (기존 token 재사용 시 이전 실패 상태와 충돌 가능)
```

<br />
<br />

## etcd 운영 관리

장애를 예방하려면 etcd가 메모리와 디스크를 어떻게 사용하는지 이해해야 한다.

### Log Retention (로그 보존)

etcd는 **log entry를 메모리에 보관**한다.
모든 log를 영구적으로 메모리에 쌓으면 OOM이 발생하기 때문에, 주기적으로 **스냅샷을 생성하고 메모리를 비운다(truncate)**.

```
Write 누적
    ▼
snapshot-count 도달 (default: 100,000 entries)
    ▼
현재 메모리 상태로 snapshot 파일 생성 (파일시스템)
    ▼
메모리에서 오래된 log entry truncate
(최신 5,000개는 메모리에 유지)
```

만약 특정 Follower의 log 따라잡기 속도가 너무 느려서 Leader 메모리에 없는 log를 요구하면, Leader는 **최근 snapshot 파일을 Follower에게 전송**한다.

> `snapshot-count` 옵션으로 스냅샷 주기를 조절할 수 있다. 값을 낮추면 메모리를 더 자주 비우지만 snapshot I/O가 증가한다.

<br />

### Revision & Compaction

etcd는 하나의 key에 대한 **모든 변경사항을 파일시스템에 기록**한다. 이것을 revision이라고 한다.

```
key "x"에 대한 write 이력:

revision 3: x = "apple"
revision 7: x = "banana"
revision 12: x = "cherry"   ← 현재 최신
```

이 구조 덕분에 특정 시점의 데이터를 조회할 수 있지만, 별도 관리 없이 계속 쌓이면 디스크 공간이 고갈된다.
**Compaction**으로 오래된 revision을 삭제할 수 있으며, 삭제된 revision은 더 이상 조회할 수 없다.

#### Auto Compaction

etcd는 두 가지 Auto Compaction 모드를 제공한다.

**Revision 모드**: 5분마다 `최신 revision - retention 값` 이하를 삭제

```
auto-compaction-mode: revision
auto-compaction-retention: 1000

→ 5분마다 (최신 revision - 1000) 이전 데이터 삭제
  현재 revision이 5000이면 → 4000 이하 삭제
```

**Periodic 모드**: 지정한 시간 단위로 삭제

```
auto-compaction-mode: periodic
auto-compaction-retention: 8h

→ 8h를 10으로 나눈 1h 단위로 compaction
  1시간마다 1시간 전 revision 이하 삭제
```

<br />

### Defragmentation (단편화 제거)

Compaction으로 revision을 삭제해도 **파일시스템의 디스크 공간이 자동으로 반환되지 않는다.**
RDB에서 DELETE를 해도 디스크 공간이 확보되지 않는 것과 같은 원리다.
Defragmentation을 해야 단편화를 정리하고 디스크 공간을 실제로 확보할 수 있다.

| 항목 | Compaction | Defragmentation |
|------|-----------|----------------|
| 목적 | 오래된 revision 논리적 삭제 | 디스크 공간 실제 반환 |
| 자동화 | 가능 (auto compaction) | **없음 (수동만 가능)** |
| 처리 중 영향 | 없음 | **해당 멤버 read/write 일시 block** |
| block 시간 | - | 보통 수 ms (DB 크기에 따라 다름) |

```bash
# 1. 현재 revision 확인
CURRENT_REV=$(etcdctl endpoint status --write-out="json" \
  | jq '.[] | .Status.header.revision')

# 2. Compaction (오래된 revision 정리)
etcdctl compact $CURRENT_REV

# 3. Defrag (디스크 공간 실제 반환)
#    한 번에 하나씩 순서대로 진행 (멤버별 순차 실행 권장)
etcdctl defrag --endpoints=<etcd-endpoint>

# 4. DB 사이즈 확인
etcdctl endpoint status --write-out=table
```

**DB 사이즈 쿼터**:
- 기본값: **2GB** (`etcd_quota_backend_bytes`)
- 최대값: **8GB**
- 2GB 초과 시 read-only 모드로 전환 (Write 전부 거부)

DB 사이즈가 증가 추세라면 Compaction + Defrag 주기를 설정하거나, 쿼터를 늘려 가용성 문제가 생기지 않도록 해야 한다.

<br />
<br />

## etcd 백업과 복구

### 스냅샷 백업

etcd 복구의 기본은 **주기적인 스냅샷 백업**이다.

```bash
etcdctl snapshot save /backup/etcd-snapshot-$(date +%Y%m%d%H%M).db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
  --key=/etc/kubernetes/pki/etcd/healthcheck-client.key

# 스냅샷 상태 확인
etcdctl snapshot status /backup/etcd-snapshot.db --write-out=table
```

**`etcdctl snapshot save`와 단순 파일 복사의 차이**:
- `etcdctl snapshot save`로 만든 파일에는 **무결성 해시(hash)가 포함**된다.
- `etcdctl snapshot restore` 시 파일 변조 여부를 자동으로 검증한다.
- 단순 복사로 만든 파일로 복구할 때는 `--skip-hash-check` 옵션이 필요하다.

또한 Compaction과 Defrag를 수행한 후 백업하면, 불필요한 revision 데이터가 제거되어 **백업 파일 크기를 줄일 수 있다.**

<br />

### 스냅샷 복구 (Quorum Loss 대응)

쿼럼을 복구할 수 없을 때 스냅샷으로 클러스터를 복원한다.

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
이전 메타데이터를 그대로 사용하면, 장애 났던 서버가 다시 살아날 경우 충돌이 발생할 수 있다.
`etcdctl snapshot restore` 명령은 자동으로 새 클러스터 메타데이터를 생성하므로, 단순 파일 복사가 아닌 이 명령을 사용해야 한다.

> kubeadm 클러스터에서는 `/etc/kubernetes/manifests/` 아래 Static Pod manifest를 이동시키는 것으로 컴포넌트를 중지/재시작할 수 있다.

<br />

### 자동 백업 구성 권장

수동 백업은 누락될 수 있다. **CronJob으로 주기적 백업을 자동화**하는 것을 권장한다.

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

주요 고려사항:
- 모든 클러스터가 동시에 백업하면 네트워크 부하가 집중된다. 클러스터마다 다른 시간을 설정하는 것이 좋다.
- 백업 파일은 오브젝트 스토리지(S3 등)에 저장하여 노드 장애에도 안전하게 보존한다.
- 백업 전 Compaction + Defrag를 수행하면 백업 크기를 줄일 수 있다.

<br />
<br />

## 모니터링 지표 정리

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

## 장애 시나리오 요약

| # | 시나리오 | 자동 복구 | 서비스 영향 | 핵심 대응 |
|---|---------|---------|-----------|---------|
| 1 | 소수 Follower 장애 | 가능 (자동 재연결) | 없음 (부하 증가) | 빠른 복구로 쿼럼 여유 유지 |
| 2 | Quorum Loss | 과반수 복구 시 자동 | 클러스터 제어 전체 마비 | 노드 복구 또는 스냅샷 복원 |
| 3 | Leader 장애 | 가능 (수 초 내 재선출) | Write 큐 대기, uncommitted 유실 가능 | 모니터링으로 반복 여부 확인 |
| 4 | Leader Step Down | 가능 (자동 새 Leader 선출) | 수 초 Write 중단 | 멤버 제거 restriction 확인 |
| 5 | Network Partition | 가능 (파티션 복구 시 자동) | 소수 파티션 Write 불가 | 네트워크 복구 |
| 6 | 디스크 고갈 | 불가 | Write 거부 → 제어 마비 | Compaction + Defrag |
| 7 | WAL / 데이터 손상 | 부분 (멤버 재가입) | 개별 멤버 기동 불가 | 멤버 제거 후 재가입 |
| 8 | 부트스트랩 중 장애 | 불가 | 클러스터 구성 자체 실패 | 데이터 디렉토리 초기화 후 재구성 |

<br />
<br />

## 핵심 정리

- etcd 장애는 단독으로 끝나지 않는다. **kube-apiserver → controller-manager → scheduler** 순으로 연쇄 마비된다.
- 기존 Running Pod는 kubelet이 유지하지만, **복구·배포·스케일링이 전부 불가**해진다.
- **소수 Follower 장애와 Leader 재선출은 자동 복구**된다. 반복 발생 여부를 모니터링해 근본 원인을 파악하는 것이 중요하다.
- **Network Partition은 Split-Brain을 일으키지 않는다.** 멤버십 변경이 항상 과반수 합의를 통해서만 가능한 구조이기 때문이다.
- Leader 장애 중 Write 요청은 실패가 아니라 **큐에 대기**한다. 단, Commit 전에 장애 난 uncommitted write는 유실될 수 있다.
- **Compaction은 논리적 삭제, Defragmentation은 실제 디스크 반환**이다. Compaction만으로는 디스크 공간이 확보되지 않는다.
- **스냅샷 복구 시 새 클러스터 메타데이터를 사용해야 한다.** 이전 UUID를 유지하면 구 서버 복구 시 충돌이 발생할 수 있다.
- 가장 중요한 예방책은 **홀수 구성(3대 이상) + 자동 스냅샷 백업 + DB 사이즈 모니터링**이다.

<br />
<br />

## 참고 자료

- [etcd 공식 운영 가이드](https://etcd.io/docs/v3.5/op-guide/)
- [etcd Failure Modes 공식 문서](https://etcd.io/docs/v3.5/op-guide/failures/)
- [etcd 재해 복구](https://etcd.io/docs/v3.5/op-guide/recovery/)
- [Kubernetes etcd 백업](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)
- [kakao tech Kubernetes 운영을 위한 etcd 기본 동작 원리의 이해](https://tech.kakao.com/posts/484)
