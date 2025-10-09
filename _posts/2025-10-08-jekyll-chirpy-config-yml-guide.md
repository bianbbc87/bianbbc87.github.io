---
title: Chirpy Config 톺아보기 - 언어, SEO, 소셜, Analytics, 댓글 시스템까지
date: 2025-10-08 21:45:00 +0900
published: true
categories: [Github-Pages, Jekyll]
tags: [configuation]
media_subpath: '/assets/img/posts/20251008-config'
---

## 글의 목적
지난 번 제작한 jekyll Chirpy Github 블로그에 config로 여러 가지 설정을 할 수 있는데, 이 글에서 `_config.yml`에 요소들과 어떻게 설정하면 될지에 대해 설명하려 합니다.

## Config.yml의 역할

Config.yml은 Jekyll 사이트의 전역 설정을 담당하는 핵심 파일로 사이트 정보, 테마 설정, 플러그인 구성 등 모든 기본 설정이 이 파일에서 관리됩니다.

> 디렉토리 중 `_tabs`, `_posts` 등 `_`로 시작되는 디렉토리는 모두 `_`를 빼고 정의합니다.
{: .prompt-tip }

## 기본 설정

### theme, lang

언어는 [lingoes.net](https://www.lingoes.net/en/translator/langcode.html)에서 확인할 수 있습니다.
한국어는 `kr`입니다.

```yaml
# Import the theme
theme: jekyll-theme-chirpy

# default value는 'en' (영어).
lang: kr
```

- `theme`: 사용할 Jekyll 테마 지정
- `lang`: 사이트 언어 설정 (한국어는 `ko-KR`)

### timezone

시간대는 [zones.ariyn.css](https://zones.arilyn.cc)에서 확인할 수 있습니다.
대한민국은 `Asia/Seoul` 입니다.

```yaml
# default value는 'Asia/Shanghai'
# 시간차가 있지는 않습니다.
timezone: Asia/Seoul
```

<br />

## SEO 설정

SEO 태그 (SEO meta tags)란 블로그나 웹사이트를 Google 등 검색엔진에 잘 노출되도록 돕는 HTML metadata 입니다.
[jekyll-seo-tag usage.md](https://github.com/jekyll/jekyll-seo-tag/blob/master/docs/usage.md)

### title, tagline, description
---

이 값은 사이드 바에 표시됩니다.
`description`은 표시되지 않습니다. (아마 검색 결과의 사이트 요약 부분에 표시될 것이라 예상합니다.)

```yaml
# HTML의 <title>로, 사이트의 기본 제목
title: bianbbc87 Devlog
# 사이트의 슬로건, page.title이 없는 경우 title + tagline을 합해서 제목을 만듭니다.
# 여기선 SubTitle의 역할을 합니다.
tagline: GDG Campus Korea Organizer & AWS CloudClub DGU Captain & CS Student in DGU
# HTML <meta name="description"> 에 들어가는 값, 검색 결과에 요약문으로 표시됩니다.
description: >-
  기술 블로그 겸 포트폴리오 페이지입니다. 컨테이너 등 네트워크/인프라 기술에 관심이 많습니다.
```

![alt text](/image-1.png){: .w-75 .shadow .rounded-10 }

### url
---

호스팅 url을 설정합니다.

```yaml
# https://username.github.io
# 커스텀 도메인을 연결한 경우, 커스텀 도메인을 입력합니다. (이 필드는 SEO 용도)
url: "https://bianbbc87.github.io"
```

![alt text](/image-3.png){: .w-75 .shadow .rounded-10 }

### 소셜 정보 (github, twitter, social)
---

사이드바 하단 소셜 아이콘에 연결되는 url로 사용됩니다.

```yaml
# github username을 등록합니다.
github:
  username: bianbbc87

# twitter username을 등록합니다.
twitter:
  username: twitter_username

# 작성자 이름과 이메일을 입력합니다.
social:
  name: Eunji Jung
  # 이메일은 malito로 표시됩니다.
  email: bianbbc87@gmail.com 
  # 링크를 배열로 등록할 수 있습니다.
  # 첫 번째 링크를 저작권자의 대표 링크로 사용합니다. 나머지 링크는 생략됩니다.
  links:
    - https://www.linkedin.com/in/eunji-jung-173288296
    - https://github.com/bianbbc87
```

![alt text](/image-4.png){: .w-75 .shadow .rounded-10 }

### webmaster_verifications
---

검색 엔진에 사이트를 등록하기 위한 인증 코드를 설정합니다.

- Google: Google 검색 노출
- Bing: Microsoft 검색 노출
- Alexa: 2022년 서비스 종료
- Yandex: 러시아 및 동유럽 검색 노출
- Baidu: 중국 검색 노출
- Facebook - 검색 엔진은 아니지만, Facebook과 Instargram에서 링크를 공유할 때 공식 도메인 인증이 가능하며, Insights 사용이 가능해집니다.

```yaml
webmaster_verifications:
  google: ${google_verifications_code}
  bing: ${bing_verifications_code}
  alexa: # fill in your Alexa verification code
  yandex: # fill in your Yandex verification code
  baidu: # fill in your Baidu verification code
  facebook: # fill in your Facebook verification code
```

#### 1. Google Site Verification 설정하기

###### 1. Google Search Console로 접속합니다. 
- [Google Search Console](https://search.google.com/search-console/welcome)

###### 2. `URL 접두어` 방식에 블로그 사이트 도메인을 등록합니다.

![alt text](/image-8.png){: .w-75 .shadow .rounded-10 }

###### 3. 권장 확인 방법인 `HTML 파일` 말고, **`HTML 태그`**에서 메타 태그를 복사합니다.

`<meta name="google-site-verification" content="R02x0MfSsOobEyxqa11BwJ7oTJvzfBL5jLyTiHwYUZA" />` 에서 **content 값**

![alt text](/image-9.png){: .w-75 .shadow .rounded-10 }

###### 4. `${google_verifications_code}` 자리에 붙여넣습니다.

###### 5. 이 설정을 포함한 상태에서 호스팅이 되도록 재배포 합니다. (git push)

###### 6. 다시 사이트로 돌아와 `확인` 을 클릭합니다.

###### 7. 성공 화면을 확인합니다.

![alt text](/image-6.png){: .w-75 .shadow .rounded-10 }

#### 2. Bing Site Verification 설정하기

###### 1. Help Guide에 따라 Bing WebMaster에서 인증을 진행하면 됩니다.

[Help Guide](https://www.bing.com/webmasters/help/add-and-verify-site-12184f8b?utm_source=chatgpt.com)

[Bing WebMaster](https://www.bing.com/webmasters/)

###### 2. 성공 화면을 확인합니다.

![alt text](/image-7.png){: .w-75 .shadow .rounded-10 }

#### 3. 나머지 설정하기

- Yandex: <https://webmaster.yandex.com/>
- Baidu: <https://ziyuan.baidu.com/>
- Facebook: <https://business.facebook.com/>

## 웹 분석 및 조회수 집계

### analytics
---

```yaml
analytics:
  google:
    id: ${google_analytics_id} # fill in your Google Analytics ID
  goatcounter:
    id: # fill in your GoatCounter ID
  umami:
    id: # fill in your Umami ID
    domain: # fill in your Umami domain
  matomo:
    id: # fill in your Matomo ID
    domain: # fill in your Matomo domain
  cloudflare:
    id: # fill in your Cloudflare Web Analytics token
  fathom:
    id: # fill in your Fathom Site ID
```

#### 1. Google Analytics 설정하기

저는 가장 익숙한 google analytics로 설정했습니다.

###### 1. google analytics에 접속합니다.

###### 2. 구글 계정 로그인 후 시작하기를 클릭합니다.

###### 3. 계정 세부 정보에서 이름을 입력합니다.

![alt text](/ga-1.png){: .w-75 .shadow .rounded-10 }

###### 4. 세부 정보를 설정합니다. 
세부 정보는 큰 상관이 없어 저는 대충 입력했습니다.

![alt text](/ga-3.png){: .w-75 .shadow .rounded-10 }

![alt text](/ga-4.png){: .w-75 .shadow .rounded-10 }

###### 5. 이용 약관을 동의합니다. 국가를 `한국`으로 선택해주세요.

![alt text](/ga-5.png){: .w-75 .shadow .rounded-10 }

###### 5. 웹(Web) 플랫폼을 선택합니다.

![alt text](/ga-6.png){: .w-75 .shadow .rounded-10 }

###### 6. 데이터 스트림을 설정합니다.

![alt text](/ga-7.png){: .w-75 .shadow .rounded-10 }

###### 7. 사진과 같은 화면에서, 드래그된 `id` 값만 복사합니다.

![alt text](/ga-8.png){: .w-75 .shadow .rounded-10 }

###### 8. `${google_analytics_id}`에 값을 붙여넣습니다.

###### 9. 다시 사이트로 돌아와, `설치 테스트` 버튼을 클릭합니다.

###### 10. 성공 화면을 확인합니다.

![alt text](/image-10.png){: .w-75 .shadow .rounded-10 }


#### 2. 나머지 설정하기

- goatcounter: <https://www.goatcounter.com>
- umami: <https://umami.is>
- matomo: <https://matomo.org/guide/installation-maintenance/matomo-on-premise-self-hosted>
- cloudflare: <https://developers.cloudflare.com/web-analytics/get-started>
- fathom: <https://usefathom.com>


### pageviews
---

페이지 조회수를 표시하기 위한 서비스 설정입니다. 오직 `goatcounter`만 가능합니다.

#### goatcounter로 조회수 표시 설정하기

###### 1. [goatcounter](https://www.goatcounter.com/) 사이트에 회원가입 합니다.

> Code: goatcounter 사이트에서 사용할 코드입니다.
> Site domain: 블로그 도메인 `https://{username}.github.io` 을 입력합니다.
> Email address: 로그인용 이메일 주소를 입력합니다.
> Password: 로그인용 비밀번호를 입력합니다.
> Fill in 9 here: 사람 검증 절차입니다. Fill in에 적힌 숫자 그대로 입력하면 됩니다.

![alt text](/views-1.png){: .w-75 .shadow .rounded-10 }

###### 2. 이메일을 확인합니다.

1번에서 입력한 이메일로 GoatCounter 메일이 와있습니다.
`Please go here to verify your email address:`의 링크로 접근해 이메일과 비밀번호로 로그인 합니다.

![alt text](/views-3.png){: .w-75 .shadow .rounded-10 }

###### 3. 사이트 코드를 확인합니다.

![alt text](/views-2.png){: .w-75 .shadow .rounded-10 }

방금 설정한 Account Name이 Site Code입니다.
여기서 Site Code는 `bianbbc87`입니다.

###### 4. 우측 상단 Settings 클릭 후 `Allow adding visitor counts on your website...` 체크박스를 활성화 합니다.

![alt text](/views-4.png){: .w-75 .shadow .rounded-10 }

```javascript
<script data-goatcounter="https://bianbbc87.goatcounter.com/count"
        async src="//gc.zgo.at/count.js"></script>
```

###### 3. analytics config에 goatcounter.id를 등록합니다.

```yaml
analytics:
  ...
  goatcounter:
    id: bianbbc87
  ...
```

###### 4. git push로 재배포하고 성공을 확인합니다.

## 테마 밑 화면 관련 설정

### theme_mode
---

처음에는 `light`로 설정했는데, 너무 밝아서 서는 `dark`로 변경했습니다.

```yaml
# [light|dark]
# default value는 light
theme_mode: dark
```

### cdn
---

Jekyll Chirpy에서 which does not exist 이미지 경로 오류 원인 분석 및 해결하기에서 저를 괴롭힌 cdn 필드입니다.

이미지, 문서, 오디오 등 media source에 대한 기준 경로를 설정합니다.

기본값으로 설정된 <https://chirpy-img.netlify.app>는 chirpy 테마 제작자가 제공하는 공개 CDN이며, 필요에 따라 직접 운영하는 CDN 주소로 변경하거나, 설정을 비워서 로컬 경로를 그대로 사용할 수도 있습니다.

CDN을 비워둘 경우(cdn: 을 주석 처리하거나 제거할 경우),
모든 미디어 리소스는 **프로젝트의 루트 디렉터리 경로(예: /assets/img/...)**를 기준으로 불러오게 됩니다.

```yaml
cdn: "https://chirpy-img.netlify.app"
```

### 개인 프로필 설정 (avatar, social_preview_image)
---

`avatar`는 사이드바에 표시되는 프로필 아바타 이미지 경로를 지정합니다.
`social_preview_image`는 SEO용 이미지의 기본값을 설정합니다.
- 각 포스트마다 `page.image` 를 지정하면 이 설정을 개별적으로 덮어쓸 수 있습니다.

```yaml
avatar: "/assets/img/commons/avatar.jpeg"

social_preview_image: "/assets/img/commons/avatar.jpeg"
```

### toc
---

본문 내에서 **목차 표시** 여부를 결정합니다.

```yaml
toc: true
```

`toc: true`
: 제목 아래 `Contents` 버튼으로 상세 목차 확인이 가능합니다.

| Contents 버튼  | 클릭 시 |
| :--- | ---: |
| ![alt text](/image-5.png){: .w-75 .shadow .rounded-10 } | ![alt text](/image.png){: .w-75 .shadow .rounded-10 } |



`toc: false`
: 제목 아래 `Contents` 버튼이 사라집니다.

![alt text](/image-2.png){: .w-75 .shadow .rounded-10 }

## 댓글 시스템 설정

### comments
---

게시물 하단에 표시되는 댓글 시스템 설정입니다.
- [Disqus](https://help.disqus.com/en/articles/1717111-what-s-a-shortname): Disqus 외부 호스팅을 이용한 댓글 시스템입니다.
- Utterances: Github Issues를 이용한 댓글 시스템입니다.
- Giscus: Github Discussions 기반 댓글 시스템입니다.

```yaml
comments:
  # [disqus | utterances | giscus]
  provider:
  # provider에 따라 아래 필드를 채울 수 있습니다.
  disqus:
    shortname: # fill with the Disqus shortname. › https://help.disqus.com/en/articles/1717111-what-s-a-shortname
  # utterances settings › https://utteranc.es/
  utterances:
    repo: # <gh-username>/<repo>
    issue_term: # < url | pathname | title | ...>
  # Giscus options › https://giscus.app
  giscus:
    repo: # <gh-username>/<repo>
    repo_id: 
    category: 
    category_id: 
    mapping: # optional, default to 'pathname'
    strict: # optional, default to '0'
    input_position: # optional, default to 'bottom'
    lang: # optional, default to the value of `site.lang`
    reactions_enabled: # optional, default to the value of `1`
```

#### Giscus 댓글 시스템 설정

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

## media resource

### assets
---

위에서 설정한 `cdn:` 필드에 이어서, media resource를 외부 CDN이 아닌 자신의 서버에서 직접 호스팅할지 여부를 설정합니다.

- enabled: `true`면 외부 CDN 대신 로컬에서 media resource를 불러옵니다.
- env: 특정 환경에서만 직접 호스팅하도록 제한할 수 있습니다.

로컬 환경 기준으로만 사용한다면 `true`로 설정, [공식 CDN](https://chirpy-static.netlify.app/) 혹은 커스텀 CDN을 사용할 예정이라면 `false`로 설정합니다.

```yaml
assets:
  self_host:
    # [true | false]
    # default value는 false
    enabled: true
    # [development | production]
    env: 

```

## PWA 설정

### pwa
---

이 기능을 활성화 하면 사이트를 모바일이나 데스크탑 앱처럼 설치할 수 있습니다.
또한, 오프라인에서도 **캐시된 페이지**를 열 수 있습니다.

```yaml
pwa:
  # default value는 true
  # PWA 기능 활성화 여부
  enabled: true
  cache:
    # default value는 true
    # 오프라인 캐시 기능 활성화 여부
    enabled: true 
    # 캐시를 제외활 경로를 배열로 지정
    deny_paths:
      # - "/example"
      # - "/example2"
```

## 페이지네이션 설정

### paginate
---

한 페이지에 표시할 포스트의 수를 설정합니다.

```yaml
paginate: 10
```

## 사이트 기본 배포 경로 설정

### baseurl
---

사이트의 기본 배포 경로를 설정합니다.
사이트 도메인 내에서, 설정한 경로를 기준으로 블로그가 배포됩니다.

> ex. 하위 경로(https://username.github.io/blog/)에 배포하는 경우: /blog

```yaml
baseurl: ""
```

## Markdown 설정

### kramdown
---

Kramdown은 Jekyll의 기본 Makedown Parser입니다.
여기서 이 Parser의 설정을 변경할 수 있습니다.

- `syntax_highlighter`: 코드 하이라이트 엔진 (Rouge 사용)
- `line_numbers`: 코드 블록의 줄 번호 표시 여부
- `start_line`: 줄 번호 시작값 지정

```yaml
kramdown:
  # 각주를 표시할 기호 심볼 지정
  footnote_backlink: "&#8617;&#xfe0e;"
  # `rouge`는 Ruby 기반 구문 하이라이터로, Markdown 코드블록에 자동 색상을 입힙니다.
  syntax_highlighter: rouge
  # rouge에 대한 옵션 설정 > https://github.com/jneen/rouge#full-options
  syntax_highlighter_opts: 
    css_class: highlight     # <div> 태그에 추가할 css 클래스 이름
    ## code 줄 번호 표시 여부 설정
    span:
      line_numbers: false # 인라인 코드에 대한 code 줄 번호 표시 여부
    block:
      line_numbers: true # 블록 코드에 대한 code 줄 번호 표시 여부
      start_line: 1 # 줄 번호 표시가 활성화 되어있을 때, 번호를 몇 번부터 시작할지
```

## 블로그 콘텐츠 관리

### collections
---

블로그 포스트 외의 별도의 콘텐츠 그룹을 정의하는 역할입니다.
`_tabs/` 경로의 파일대로 사이드라 콘텐츠를 생성합니다.

```yaml
collections:
  # `_tabs` 경로를 컬렉션으로 정의
  tabs:
    output: true # 렌더링 여부
    # `_tabs/*.md` 파일의 출력 순서를 결정하는 변수명
    sort_by: order
```

### defaults
---

모든 포스트나 페이지 등에 공통적으로 적용한 기본 Front Matter를 지정합니다.

- `scope`: 어떤 파일에 적용할지 대상 범위
- `values`: 적용할 실제 설정 값들

```yaml
defaults:
  - scope:
      path: "" # 모든 파일의
      type: posts # post에 대해
    # 적용할 기본 설정
    values:
      layout: post # _layout/post.html 템플릿 사용
      comments: true # 댓글 활성화
      toc: true # 글의 목차 표시
      permalink: /posts/:title/ # 포스트 URL 패턴 지정
  - scope:
      path: _drafts # 임시 저장 파일의
      # 모든 파일에 대해
    values:
      comments: false # 댓글 비활성화
  - scope:
      path: "" # 모든 파일의
      type: tabs # tab에 대해
    values:
      layout: page # _layout/page.html 템플릿 사용
      permalink: /:title/ # 페이지 URL 패턴 지정
```

### sass
---

sass 스타일 파일을 빌드할 때 출력 형식을 지정합니다.

기본값인 ```compressed```는 불필요한 공백을 제거하여 최소화된 css를 생성합니다.

```yaml
sass:
  style: compressed
```

### compress_html
---

Jekyll 빌드 시 HTML 결과물 압축 설정입니다.

```yaml
compress_html:
  clippings: all # HTML 내 들여쓰기, 여백 등을 삭제
  comments: all # HTML 주석 제거
  endings: all # `<div>\n</div>`와 같은 줄바꿈을 압축
  profile: false # 개발용 로그 출력을 끔
  blanklines: false # 완전히 비어있는 줄 제거
  # 특정 환경에서의 압축을 비활성화
  ignore:
    envs: [development]
```

### exclude
---

Jekyll 빌드 시 사이트에 포함하지 않을 파일 및 디렉터리를 지정합니다.

```yaml
exclude:
  - "*.gem"
  - "*.gemspec"
  - docs
  - tools
  - README.md
  - LICENSE
  - purgecss.js
  - "*.config.js"
  - "package*.json"
```

### jekyll-archives
---

`jekyll-archives`는 블로그 사이트에서 tag나 category 등 자동으로 글 아카이브 페이지를 생성해주는 jekyll plugin 입니다.

```yaml
jekyll-archives:
  enabled: [categories, tags]   # 어떤 종류의 아카이브를 생성할지 지정
  # 아카이브 페이지를 렌더링할 때 사용할 레이아웃 파일 지정
  layouts:
    category: category # category는 `_layout/catefory.html`
    tag: tag # tag는 `_layout/tag.html`
  # 생성될 아카이브 페이지의 URL 규칙 정의
  permalinks:
    tag: /tags/:name/
    category: /categories/:name/
```

## 최종 결과물

제 최종 config는 아래와 같습니다.

```yaml
# The Site Configuration

# Import the theme
theme: jekyll-theme-chirpy

# The language of the webpage › http://www.lingoes.net/en/translator/langcode.htm
# If it has the same name as one of the files in folder `_data/locales`, the layout language will also be changed,
# otherwise, the layout language will use the default value of 'en'.
lang: kr

# Change to your timezone › https://zones.arilyn.cc
timezone: Asia/Seoul

# jekyll-seo-tag settings › https://github.com/jekyll/jekyll-seo-tag/blob/master/docs/usage.md
# ↓ --------------------------

title: bianbbc87 Devlog # the main title

tagline: GDG Campus Korea Organizer & AWS CloudClub DGU Captain & CS Student in DGU # it will display as the subtitle

description: >- # used by seo meta and the atom feed
  기술 블로그 겸 포트폴리오 페이지입니다. 컨테이너 등 네트워크/인프라 기술에 관심이 많습니다.

# Fill in the protocol & hostname for your site.
# E.g. 'https://username.github.io', note that it does not end with a '/'.
url: "https://bianbbc87.github.io"

github:
  username: bianbbc87 # change to your GitHub username

twitter:
  username: bianbbc87 # change to your Twitter username

social:
  # Change to your full name.
  # It will be displayed as the default author of the posts and the copyright owner in the Footer
  name: Eunji Jung
  email: bianbbc87@gmail.com # change to your email address
  links:
    # The first element serves as the copyright owner's link
    - https://www.linkedin.com/in/eunji-jung-173288296 # change to your Twitter homepage
    - https://github.com/bianbbc87 # change to your GitHub homepage
    # Uncomment below to add more social links
    # - https://www.facebook.com/username
    # - https://www.linkedin.com/in/username

# Site Verification Settings
webmaster_verifications:
  google: R02x0MfSsOobEyxqa11BwJ7oTJvzfBL5jLyTiHwYUZA
  bing: CA2C49EBA8929E49A29FD1D9323B2EEC
  alexa: # fill in your Alexa verification code
  yandex: # fill in your Yandex verification code
  baidu: # fill in your Baidu verification code
  facebook: # fill in your Facebook verification code

# ↑ --------------------------
# The end of `jekyll-seo-tag` settings

# Web Analytics Settings
analytics:
  google:
    id: G-N9YHP67YSF # fill in your Google Analytics ID
  goatcounter:
    id: # fill in your GoatCounter ID
  umami:
    id: # fill in your Umami ID
    domain: # fill in your Umami domain
  matomo:
    id: # fill in your Matomo ID
    domain: # fill in your Matomo domain
  cloudflare:
    id: # fill in your Cloudflare Web Analytics token
  fathom:
    id: # fill in your Fathom Site ID

# Page views settings
pageviews:
  provider: goatcounter # now only supports 'goatcounter'

# Prefer color scheme setting.
#
# Note: Keep empty will follow the system prefer color by default,
# and there will be a toggle to switch the theme between dark and light
# on the bottom left of the sidebar.
#
# Available options:
#
#     light — Use the light color scheme
#     dark — Use the dark color scheme
#
theme_mode: dark # [light | dark]

# The CDN endpoint for media resources.
# Notice that once it is assigned, the CDN url
# will be added to all media resources (site avatar, posts' images, audio and video files) paths starting with '/'
#
# e.g. 'https://cdn.com'
# cdn: "https://chirpy-img.netlify.app"

# the avatar on sidebar, support local or CORS resources
avatar: "/assets/img/commons/avatar.jpeg"

# The URL of the site-wide social preview image used in SEO `og:image` meta tag.
# It can be overridden by a customized `page.image` in front matter.
social_preview_image: # string, local or CORS resources

# boolean type, the global switch for TOC in posts.
toc: true

comments:
  # Global switch for the post-comment system. Keeping it empty means disabled.
  provider: giscus # [disqus | utterances | giscus]
  # The provider options are as follows:
  disqus:
    shortname: # fill with the Disqus shortname. › https://help.disqus.com/en/articles/1717111-what-s-a-shortname
  # utterances settings › https://utteranc.es/
  utterances:
    repo: # <gh-username>/<repo>
    issue_term: # < url | pathname | title | ...>
  # Giscus options › https://giscus.app
  giscus:
    repo: bianbbc87/bianbbc87.github.io # <gh-username>/<repo>
    repo_id: R_kgDOP-VlDg
    category: General
    category_id: DIC_kwDOP-VlDs4CwYib
    mapping: pathname # optional, default to 'pathname'
    strict: 0 # optional, default to '0'
    input_position: bottom # optional, default to 'bottom'
    lang: ko # optional, default to the value of `site.lang`
    reactions_enabled: 1 # optional, default to the value of `1`

# Self-hosted static assets, optional › https://github.com/cotes2020/chirpy-static-assets
assets:
  self_host:
    enabled: true # boolean, keep empty means false
    # specify the Jekyll environment, empty means both
    # only works if `assets.self_host.enabled` is 'true'
    env: # [development | production]

pwa:
  enabled: true # The option for PWA feature (installable)
  cache:
    enabled: true # The option for PWA offline cache
    # Paths defined here will be excluded from the PWA cache.
    # Usually its value is the `baseurl` of another website that
    # shares the same domain name as the current website.
    deny_paths:
      # - "/example"  # URLs match `<SITE_URL>/example/*` will not be cached by the PWA

paginate: 10

# The base URL of your site
baseurl: ""

# ------------ The following options are not recommended to be modified ------------------

kramdown:
  footnote_backlink: "&#8617;&#xfe0e;"
  syntax_highlighter: rouge
  syntax_highlighter_opts: # Rouge Options › https://github.com/jneen/rouge#full-options
    css_class: highlight
    # default_lang: console
    span:
      line_numbers: false
    block:
      line_numbers: true
      start_line: 1

collections:
  tabs:
    output: true
    sort_by: order

defaults:
  - scope:
      path: "" # An empty string here means all files in the project
      type: posts
    values:
      layout: post
      comments: true # Enable comments in posts.
      toc: true # Display TOC column in posts.
      # DO NOT modify the following parameter unless you are confident enough
      # to update the code of all other post links in this project.
      permalink: /posts/:title/
  - scope:
      path: _drafts
    values:
      comments: false
  - scope:
      path: ""
      type: tabs # see `site.collections`
    values:
      layout: page
      permalink: /:title/

sass:
  style: compressed

compress_html:
  clippings: all
  comments: all
  endings: all
  profile: false
  blanklines: false
  ignore:
    envs: [development]

exclude:
  - "*.gem"
  - "*.gemspec"
  - docs
  - tools
  - README.md
  - LICENSE
  - purgecss.js
  - "*.config.js"
  - "package*.json"

jekyll-archives:
  enabled: [categories, tags]
  layouts:
    category: category
    tag: tag
  permalinks:
    tag: /tags/:name/
    category: /categories/:name/
```

## 회고

앞서 소개한 설정 중 analytics, giscus, site verification 키는 유출될 수 있으니 환경 변수로 관리하는 게 좋아보입니다.

다음번에는 이 값을 환경 변수로 관리하는 방법을 블로그로 적어보겠습니다 !