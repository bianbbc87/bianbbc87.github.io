---
title: Chirpy _config.yml CDN 비활성화 시 이미지 오류 해결하기
date: 2025-10-08 18:37:00 +0900
published: true
categories: [Github-Pages, Jekyll]
tags: [troubleshooting]
media_subpath: '/assets/img/posts/20251008-cdn'
---

## 문제 상황
지난 번 만든 Chirpy의 config 설정을 하면서 사용하는 것, 안 하는 것을 분류하였는데, 업데이트 하고 보니 갑자기 ci workflow에서 다음과 같은 에러가 발생했습니다.

   ![Build source](/image.png){: .w-75 .shadow .rounded-10 }

```bash
# ...

* At _site/posts/text-and-typography/index.html:22:

  internally linking to /posts/20190808/devtools-dark.png, which does not exist

...
```

멀쩡하게 동작하던 기본 포스터의 이미지를 갑자기 못 찾는다고 하는 상황에,
혹시 config을 수정하면서 다른 파일까지 건들였나..? 하고 commit 기록을 확인해봤지만, 정말 **`_config.yml`** 만 수정한 상황이었습니다.

## 해결 방법

우선 `_posts/2019-08-09-getting-started.md` 파일을 열어보니, 이미지 경로는 다음과 같이 되어 있었습니다.

```markdown
media_subpath: '/posts/20180809'

# ...


![Build source](pages-source-light.png){: .light .border .normal w='375' h='140' }
```

이 때, media_subpath를 기준으로 하단 md에 정의된 이미지를 찾는다고 합니다. 즉, `/posts/20180809/pages-source-light.png` 경로에 해당하는 이미지가 있어야 했습니다.

그런데 원본 레포에도, Fork한 내 레포에도 원래부터 `/posts/` 경로가 존재하지 않았습니다.
이때 저는 `_config.yml`의 `avatar:`에 정의 되어있던 경로 `/commons/avatar.jpg`도 함께 찾고 있었는데, `avatar.jpg` 파일은 없지만 `commons` 경로가 `/assets/img/commons`로 있어 모든 image에 대한 path가 `/assest/img` 에서 시작되는 절대 경로인 것으로 착각 했습니다.

그래서 `assets/img/` 경로에 `posts` 폴더를 만들어 똑같은 이미지를 똑같은 경로로 붙여 넣었습니다. (이미지는 정상적으로 호스팅 중인 사이트에서 직접 다운로드 받았습니다.)

그러나 해결되지 않았고..ㅎㅎ 결국 `media_subpath`를 제거하고 이미지 절대 경로를 다음과 같이 `root`를 기준으로, 직접 지정하는 방법으로 변경 하니 해결 되었습니다.

```markdown
   ![Build source](/assets/img/posts/20190809/pages-source-light.png){: .light .border .normal w='375' h='140' }
```

## 문제 원인

문제 자체는 절대 경로를 지정하는 방식으로 해결했지만, `“왜 갑자기 이미지 경로를 찾지 못했을까?”` 하는 의문이 남아 원인을 자세히 조사했습니다. 그 결과, **`_config.yml`의 `cdn:` 필드를 주석 처리한 것**이 원인임을 알게 되었습니다.

```yaml
# The CDN endpoint for media resources.
# Notice that once it is assigned, the CDN url
# will be added to all media resources (site avatar, posts' images, audio and video files) paths starting with '/'
#
# e.g. 'https://cdn.com'
# cdn: "https://chirpy-img.netlify.app"
```
--_config.yml--

Chirpy 테마에서 **cdn 필드는 이미지, 폰트, 스크립트 등 정적 리소스(media resource)의 기본 endpoint 를 정의하는 역할을 합니다. 즉, 이 값이 설정되어 있으면 포스트에서 사용하는 이미지 경로가 자동으로 CDN 주소를 기준으로 해석**됩니다.

저는 별도의 CDN을 사용할 계획이 없어 `cdn:` 설정을 제거했는데, 이로 인해 정적 리소스의 기본 endpoint가 프로젝트 root가 되고 이 경로에는 해당하는 이미지가 없기 때문에 직관적인 오류가 발생한 것이었습니다.

결론적으로, cdn을 해제하는 경우의 정확한 해결 방법은
1. 프로젝트 root를 기준으로 path를 올바르게 재정의 하기
2. 일치하는 이미지 파일이 모두 존재하도록 등록하기

입니다.

`media_subpath`도 지금은 잘 동작할 것이라 예상합니다.


## 회고 

이 이슈는 문제를 해결한 이후 원인 분석을 진행한 경우 였는데요, config 주석을 다시 잘 읽어보니 대놓고 media source endpoint라고 적혀 있어서.. 약간 부끄러워 졌습니다.

다음부터는 config 주석을 좀 더 꼼꼼하게 읽자..!!

이 이슈와 별개로 앞으로 이미지는 `/assets/img/posts/:Date` 경로에 저장해서 관리하려 합니다 ㅎㅎ
`_config.yml`의 `avatar:` 필드도 프로젝트 root 기준 절대 경로로 주입하면 됩니다.