---
title: Kubernetes 오픈소스 기여 생태계 톺아보기 01 - Enhancements Team
date: 2026-02-01 17:21:00 +0900
published: true
categories: [Cloud]
tags: [Kubernetes]
mermaid: true
media_subpath: '/assets/img/posts/20260201-kube02'
---

## Kubernetes Release Enhancements Team
Kubernetes Release Team 중 Enhancements에 대해 중점적으로 소개하려 한다.

<br />

## Kubernetes의 기여 생태계
Kubernetes 기여 생태계에 관해서는 첫 번째 게시글로 작성했다.
- [\[bianbbc87 devlog\] Kubernetes 오픈소스 기여 생태계 톺아보기](https://bianbbc87.github.io/posts/kubernetes-release-cycle-01/)

<br />

## Enhancementes Team 역할
Enhancements Team은 Kubernetes 릴리스 주기에서 새로운 기능과 개선 사항을 관리하는 역할을 담당한다.

1. Enhancement 관리 및 상태 유지
- kubernetes/enhancements 저장소 내 모든 Enhancement의 활성 상태 유지
- KEP 상태(Tracked / Implementable / Deferred / GA 등) 지속 점검
2. 커뮤니케이션 허브 역할
- Enhancement Owner ↔ SIG Leadership 간 커뮤니케이션 조율
- 일정 지연, 범위 변경, 리스크 발생 시 조정 및 중재
3. Release Highlights 취합 및 항목 분류
- 신규 Enhancement
- 장기간 대기 후 포함된 Enhancement
- GA(General Availability)로 승격된 기능
- Deprecation(기능 제거/중단 예정)
- 기존 동작 방식의 주요 변경 사항
4. 릴리스 커뮤니케이션 지원
(Communications Lead 및 CNCF Communications 팀과 협업)
- Kubernetes 공식 릴리스 블로그 포스트 초안 작성 및 리뷰

추가적으로, Enhnacements 리더는 **후속 리더십 육성 (Succession Planning)**도 수행해야 한다.
- 다음 릴리스 사이클의 Enhancements Lead 후보 식별
- Release Team 선발 프로세스에 맞춘 인재 추천
- Enhancement Shadow 선정 및 멘토링

<br />

## Enhancements Team Cycle
Enhancements Team의 사이클은 다음과 같다.

1. Start Enhancements Tracking
2. Production Readiness Review Freeze
3. Enhancements Freeze
4. Code Freeze
5. Test Freeze

<br />

### 01. Start Enhancements Tracking
트래킹 보드에 등록된 마일스톤들의 태그를 등록하거나, 미해결 이슈/PR이 없는지 확인하는 등 버전 사이클에서 관리하려는 마일스톤을 정리하는 단계이다. 

| 구분              | Release Team 트래킹 **불필요**  | Release Team 트래킹 **필요**  |
| --------------- | ------------------------- | ------------------------ |
| 변경 성격           | 동작(behaviour)에 영향이 없는 변경  | 동작(behaviour)에 영향을 주는 변경 |
| 코드 변경 유형        | 코드 정리, 리팩터링(동작 유지)        | 리팩터링 또는 기능 개선(동작 변경)     |
| 예시              | 코드 클린업, 변수명 변경, 내부 함수 재구성 | 기능 동작 변경, 에러 처리 방식 수정    |
| 버그 수정           | 의도된 기존 동작을 그대로 복구         | 동작 방식이나 결과가 달라지는 수정      |
| 성능 영향           | 외부에서 관찰 가능한 변화 없음         | 성능 특성 변화(지연, 처리량 등)      |
| 사용자 경험          | 사용자에게 체감 변화 없음            | UX에 직접적 또는 간접적 영향 발생     |
| 릴리스 노트          | 불필요                       | 필요                       |
| Release Team 관리 | 추적 대상 아님                  | 추적 및 릴리스 범위 포함 대상        |

<br />

### 02. Production Readiness Review Freeze
PRR은 Production Readiness Review Process로, 신규 기능이 프로덕션 환경에서 안정하고 확장 가능하며 운영 지원이 가능한지 보장하기 위한 절차이다.

KEP를 작성하고 SIG Leader가 검토해야 한다. 리더가 KEP를 읽고 `implementable` 단계로 넘어갈 준비가 되었는지 여부와 PRR 답변에 모두 만족하면 작성자는 PRR 승인을 요청한다.

PRR 승인 프로세스는 아래와 같다.
1. KEP 작성자가 PRR 설문지를 작성한다.
2. SIG 리드가 검토한다.
3. SIG 승인 후, PRR 승인을 요청한다.
4. PRR Reviewer팀이 PRR을 승인 해준다.

PRR의 예시는 [여기](https://github.com/kubernetes/community/blob/master/sig-architecture/production-readiness.md)를 참고

KEP 소유자는
1. KEP가 `provisinal`에서 `implementable`로 갈 준비가 되었을 때
2. KEP가 새로운 단계(`alpha` / `beta` / `stable`)로 넘어갈 때마다 새로운 PRR 승인이 필요하다.
3. KEP의 README 안에는 PRR 질문지(PRR questionnaire)가 들어있다. KEP에 큰 변경이 있는 경우, PRR도 다시 받아야 한다.


<br />

#### KEP란 ?

KEP란 Kubernetes Enhancement Proposal의 약자로, 새로운 기능이나 개선에 대해 설명하는 문서이다.
Kubernetes 공식 가이드 상 **문서를 읽는 모두가 아이디어를 읽고 동의할 수 있도록 하는 것**이 목적이다.

KEP Template은 [여기](https://github.com/kubernetes/enhancements/tree/master/keps/NNNN-kep-template)를 참고하세요.

![alt text](image.png)
![alt text](image-1.png)

KEP에 대한 YAML 파일을 `kep.yaml`이라고 부르며, 핵심 아이디어를 빠르게 확인할 수 있다. (예를 들어 현재 어떤 상태인지에 대해서)

`kep.yaml`의 예시는 [여기](https://github.com/kubernetes/enhancements/blob/master/keps/sig-architecture/0000-kep-process/kep.yaml)를 참고하세요.

![alt text](image-2.png)

### 03. Enhancements Freeze
Enhancements Freeze는 **해당 릴리스에 포함될 기능들을 확정하는 단계**이다. 이 시점 이후로는 새로운 Enhancement를 추가할 수 없으며, 예외(Exception)를 통해서만 가능하다.

#### Enhancements Freeze 요구사항
Enhancement가 현재 릴리스에 포함되기 위해서는 다음 조건을 만족해야 한다:

- KEP README가 최신 템플릿을 사용하여 k/enhancements 저장소에 병합되어 있어야 함
- KEP 상태가 `implementable`로 표시되어 있어야 함 (`latest-milestone: v1.xx`)
- KEP README에 최신 졸업 기준(graduation criteria)이 작성되어 있어야 함
- Production Readiness Review(PRR)가 완료되고 승인되어 k/enhancements에 병합되어 있어야 함

#### Enhancements Freeze Party
Enhancements Freeze 당일에는 "Enhancement Freeze Party"를 진행한다:

1. 모든 Enhancement Shadow, Emeritus Advisor, Release Team Lead 초대
2. 각 Enhancement의 최종 상태 평가
3. 요구사항을 충족하지 못한 Enhancement 제거
   - Enhancement Status를 `Removed from Milestone`로 변경
   - `/milestone clear` 명령으로 마일스톤 제거
4. 제거된 Enhancement는 이후 Exception 요청을 통해서만 복구 가능

<br />

### 04. Code Freeze
Code Freeze는 **해당 릴리스에 포함될 모든 코드 변경이 완료되어야 하는 시점**이다.

#### Code Freeze 요구사항
**주의**: 모든 Enhancement가 Code Freeze 추적 대상은 아니다. "동작(behaviour)에 영향을 주는 변경"만 추적 대상이다.

Code Freeze까지 다음 조건을 만족해야 한다:

- kubernetes/kubernetes 저장소의 모든 관련 PR이 이슈 설명에 링크되어 있어야 함
- 모든 PR이 병합 준비 상태여야 함 (`approved`, `lgtm` 레이블 적용)
- 테스트 코드도 포함되어야 함
- `stable` 단계를 목표로 하는 KEP는 코드 PR 병합 후 `implemented`로 표시되어야 함

#### Out-of-tree Enhancement
kubernetes/kubernetes 저장소 외부에서 구현되는 Enhancement는 추적 기준이 다를 수 있다:
- 관련 PR을 이슈 설명에 링크하거나
- 모든 out-of-tree 구현 작업이 완료되고 병합되었음을 확인

#### Code Freeze Party
Code Freeze 당일에도 "Code Freeze Party"를 진행한다:

1. 각 Enhancement의 코드 병합 상태 최종 점검
2. 요구사항을 충족하지 못한 Enhancement 제거
3. 관련된 모든 k/k PR에서 마일스톤 제거
4. 제거된 Enhancement는 Exception 요청 필요

<br />

### 05. Test Freeze
Test Freeze는 Code Freeze와 동시에 발생하며, 테스트 관련 코드도 모두 병합되어 있어야 한다.

<br />

## Exception Process
각 Freeze 단계에서 요구사항을 충족하지 못한 Enhancement는 마일스톤에서 제거되며, 이후 **Exception 요청**을 통해서만 다시 포함될 수 있다.

### Exception 요청 절차
1. Enhancement Owner가 Exception 요청 제출
2. Release Team Lead와 논의하여 승인/거부 결정
3. 승인된 경우:
   - Enhancement Status를 `Tracked for enhancements freeze`로 변경
   - `exceptions.yaml` 파일에 기록
4. 거부된 경우:
   - 다음 릴리스로 연기

### exceptions.yaml 파일 형식
```yaml
# Enhancements Freeze Exceptions
enhancementFreeze:
- name: "Feature Name"
  issue: 1234
  date_requested: 2025-01-15
  date_reviewed: 2025-01-18
  thread: https://groups.google.com/g/kubernetes-sig-release/c/thread-id
  pull_requests:
    - https://github.com/kubernetes/enhancements/pull/5678
  status: "approved"

# Code Freeze Exceptions
codeFreeze:
- name: "Another Feature Name"
  issue: 5678
  date_requested: 2025-02-15
  date_reviewed: 2025-02-18
  thread: https://groups.google.com/g/kubernetes-sig-release/c/thread-id
  pull_requests:
    - https://github.com/kubernetes/kubernetes/pull/123456
  status: "approved"
```

<br />

## Enhancement Tracking Board
Enhancements Tracking Board는 릴리스에 포함될 Enhancement들을 추적하는 프로젝트 보드이다.

### 주요 필드

| 필드 | 설명 |
|------|------|
| Title | k/enhancements의 이슈 제목 및 링크 |
| Type | Enhancement 유형 (Net New / Major Change / Graduating / Deprecation) |
| Stage | 목표 단계 (Alpha / Beta / Stable) |
| Status | Enhancement 상태 |
| SIG | 소유 SIG |
| PRR Status | PRR 리뷰 상태 |
| Milestone | 할당된 마일스톤 |

### Type 분류

| Type | 설명 |
|------|------|
| Net New | 완전히 새로운 기능 추가 (주로 Alpha 단계) |
| Major Change | 현재 단계에서 주요 기능 변경 |
| Graduating | Beta 또는 Stable로 승격 |
| Deprecation | 기능 제거 또는 중단 예정 추적 |

### Status 분류

| Status | 설명 |
|--------|------|
| Tracked For Enhancement Freeze | Enhancement Freeze 요구사항 충족 |
| At Risk For Enhancement Freeze | Enhancement Freeze 요구사항 미충족 |
| Tracked For Code Freeze | Code Freeze 요구사항 충족 |
| At Risk For Code Freeze | Code Freeze 요구사항 미충족 |
| Exception Required | Freeze 기한 미충족, 예외 요청 필요 |
| Deferred | 향후 릴리스로 연기 |
| Removed From Milestone | 마일스톤에서 제거됨 |

<br />

## 마무리
Kubernetes Enhancements Team은 릴리스 주기 동안 새로운 기능과 개선 사항을 체계적으로 관리하는 핵심 역할을 담당한다. PRR Freeze → Enhancements Freeze → Code Freeze로 이어지는 단계별 프로세스를 통해 각 Enhancement가 프로덕션 환경에 안전하게 배포될 수 있도록 보장한다.

특히 각 Freeze 단계마다 명확한 요구사항과 커뮤니케이션 템플릿을 제공하여, Enhancement Owner와 Release Team 간의 원활한 협업을 가능하게 한다.

