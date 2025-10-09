---
title: Chirpy Config - giscus로 댓글 시스템 설정하기
date: 2025-10-09 02:46:00 +0900
published: true
categories: [Github-Pages, Jekyll]
tags: [configuation]
media_subpath: '/assets/img/posts/20251008-config'
---


#### Giscus 댓글 시스템 설정하기

[이 사이트](https://giscus.app/ko)에서 설정할 수 있습니다.

###### 1. giscus app 사이트에 접속합니다.

###### 2. 언어를 설정합니다.

![alt text](/image-12.png){: .w-75 .shadow .rounded-10 }

###### 3. repository가 public으로 등록 되었는지 확인합니다.

###### 4. 블로그 repository에 [giscus 앱](https://github.com/apps/giscus)을 설치합니다.
- 사이트 접근 후 `Configure` 버튼을 클릭해서 블로그 repository만 연결해주세요.

###### 5. repository의 `Settings` > `Generel` > `Features` 항목에서 `Discussions`를 활성화 합니다.

![alt text](/image-11.png){: .w-75 .shadow .rounded-10 }

###### 6. 다시 돌아와 저장소명(username/repository)를 새롭게 입력하고 통과를 확인합니다.

![alt text](/image-13.png){: .w-75 .shadow .rounded-10 }

###### 7. Discussion 카테고리를 General로 설정합니다.
- 페이지<->Discussions 연결과 기능은 기본 값을 사용합니다.

![alt text](/image-15.png){: .w-75 .shadow .rounded-10 }

###### 8. 테마를 선택합니다.

![alt text](/image-14.png){: .w-75 .shadow .rounded-10 }

###### 9. giscus 사용에서 보이는 script 중 다음 값만 config에 추가합니다.

```yaml
  giscus:
    repo: ${giscus에 등록한 저장소명}
    repo_id: ${data-repo-id}
    category: ${data-category}
    category_id: ${data-category-id}
    mapping: ${data-mapping} # optional, default to 'pathname'
    strict: ${data-strict} # optional, default to '0'
    input_position: ${data-input-position} # optional, default to 'bottom'
    lang: ${data-lang} # optional, default to the value of `site.lang`
    reactions_enabled: ${data-reactions-enabled} # optional, default to the value of `1`
```

###### 10. git push로 블로그를 재배포 합니다.