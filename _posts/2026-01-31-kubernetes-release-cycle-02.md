---
title: Kubernetes 오픈소스 기여 생태계 톺아보기 - Enhancements Team
date: 2025-01-31 15:03:00 +0900
published: true
categories: [Cloud]
tags: [Kubernetes]
media_subpath: '/assets/img/posts/20251229-kube01'
---

## Kubernetes란 ?
[k8s](https://kubernetes.io/ko/)라고도 알려진 kubernetes는 컨테이너화된 애플리케이션을 자동으로 배포, 스케일링 및 관리해주는 오픈소스 시스템

## Kubernetes의 기여 생태계


## KEP (Kubernetes Enhancement Proposal)

KEP란 새로운 기능이나 개선에 대해 설명하는 문서이다.
Kubernetes 공식 가이드 상 **문서를 읽는 모두가 아이디어를 읽고 동의할 수 있도록 하는 것**이 목적이다.

KEP Template은 [여기](https://github.com/kubernetes/enhancements/tree/master/keps/NNNN-kep-template)를 참고하세요.

![alt text](image.png)
![alt text](image-1.png)

KEP에 대한 YAML 파일을 `kep.yaml`이라고 부르며, 핵심 아이디어를 빠르게 확인할 수 있다. (예를 들어 현재 어떤 상태인지에 대해서)

`kep.yaml`의 예시는 [여기](https://github.com/kubernetes/enhancements/blob/master/keps/sig-architecture/0000-kep-process/kep.yaml)를 참고하세요.

![alt text](image-2.png)

PRR은 Production Readiness Review Process로, 신규 기능이 프로덕션 환경에서 안정하고 확장 가능하며 운영 지원이 가능한지 보장하기 위한 절차이다.
1. KEP 작성자가 PRR 설문지를 작성한다.
2. SIG 리드가 검토한다.
3. SIG 승인 후, PRR 승인을 요청한다.
4. PRR Reviewer팀이 PRR을 승인 해준다.

PRR의 예시는 [여기](https://github.com/kubernetes/community/blob/master/sig-architecture/production-readiness.md)를 참고하세요.


## 

```mermaid
graph LR
    START([Start of Release Cycle])
    START --> SET
    START --> MONITOR
    EF --> FBOD
    EF --> CMT
    EIP --> CF
    CF --> DPD
    TF --> FDRN

    END((Release Day))
    TEST --> END
    EAARB --> END
    DCF --> END
    FBRD --> END
    RNC --> END

    subgraph Enhancements
        SET[Start Enhancements Tracking] --> PRRF[Production Readiness Review Freeze]
        PRRF --> EF[Enhancements Freeze]
        EF --> CF[Code Freeze]
        CF --> TF[Test Freeze]
    end

    subgraph Docs
        DPD[Docs Placeholder Deadline] --> DRD[Docs Ready For Review Deadline]
        DRD --> DCF[Docs Complete Deadline]
        CMT[Start Collecting Major Themes] --> FDRN[Start Final Draft of Release Notes]
        FDRN --> RNC[Release Note Complete]
    end

    subgraph Comms
        FBOD[Feature Blog Opt-in Deadline] --> RBRD[Release Blog Ready For Review Deadline]
        RBRD --> FBRD[Feature Blogs Ready For Review Deadline]
    end

    subgraph Release Signal
        TA[Track All Issues and PRs] --> EIP[Escalate Issues and PRs]
        MONITOR[Monitor E2E Tests and All Jobs In SIG Release Dashboards]
        MONITOR --> OPEN[Open Issues For Failing Or Flaking Jobs]
        CF --> ESCALATE[Escalate Any New Failures]
        TF --> EAARB[Ensure Attention on Any Release-Blocking and Major Bug Fix PRs]
        OPEN --> ESCALATE
        EAARB --> TEST[Ensure Tests have Stabilized]
        ESCALATE --> TEST
    end

    classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
    classDef plain fill:#ddd,stroke:#fff,stroke-width:4px,color:#000;
    classDef k8s fill:#326ce5,stroke:#fff,stroke-width:4px,color:#fff;
    class START,END plain;
    class SET,PRRF,EF k8s;
    class DPD,DRD,DCF k8s;
    class FBOD,RBRD,FBRD k8s;
    class CMT,FDRN,RNC k8s;
    class MONITOR,OPEN,CF,ESCALATE,EAARB,TEST,TA,EIP,TF k8s;
```

## 