---
title: Chirpy Jekyll 테마로 github-pages 개발 블로그 제작하기 (25.10.08v)
date: 2025-10-08 16:52:00 +0900
published: true
categories: [Github-Pages, Jekyll]
tags: [guide]
---

## 글의 목적
jekyll Chirpy 테마로 블로그를 제작하기 위해 [JJIKIN님의 Jekyll Chirpy(v6.0.1) 테마를 활용한 Github 블로그 만들기](https://jjikin.com/posts/Jekyll-Chirpy-%ED%85%8C%EB%A7%88%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-Github-%EB%B8%94%EB%A1%9C%EA%B7%B8-%EB%A7%8C%EB%93%A4%EA%B8%B0(2023-6%EC%9B%94-%EA%B8%B0%EC%A4%80)/) 를 참고했습니다. 
정말 잘 정리 해주셨고 기본적인 세팅은 이 글을 참고하는 것이 좋습니다.

다만 버전이 업그레이드 된 것인지, 몇 가지 오류들이 발생하는데 트러블 슈팅을 공유하려 합니다.


## 배경
### 내가 Github Pages로 개발 블로그를 만드는 이유
---

원래 개발 블로그를 잘 작성하는 편이 아니었지만, 작성한다면 주로 velog에 적었는데요.
velog에서 사용하는 에디터가 불편하기도 하고, 개발을 주로 github에서 하다보니 모든 포트폴리오를 github에 모을 수 있으면 좋을 것 같아 github pages를 선택하게 되었습니다.

> github pages란 github의 repository에서 직접 웹 사이트를 만들 수 있는 문서입니다.
{: .prompt-info }


### Github Pages용 대표적 빌드 도구 오픈소스 Jekyll
---

Jekyll은 Ruby로 작성되었으며, Markdown, Liquid, HTML & CSS 및 기타 웹 자산을 사용하여 정적 웹 사이트 및 블로그를 생성합니다. Jekyll은 GitHub Pages에서 기본적으로 지원하는 빌드 도구이기도 합니다.

블로그를 몇 개 봤을 때 유료 템플릿도 있는 것 같았으나, 돈이 없기도 하고 블로그 제작에 돈을 쓰고 싶지도 않아 가장 유명하고 무료인 Jekyll을 선택했습니다.

## 블로그를 만들어보자 !


### Jekyll, Ruby 등 필수 패키지 설치
---

[jekyll-installation](https://jekyllrb.com/docs/installation/) 사이트에 들어가 OS에 맞게 설치하면 됩니다.

jekyll을 사용하는 데 필요한 Ruby, Bundle 설치 문서가 함께 포함되어 있습니다.
저는 **MacOS (M2)** 기준 homebrew로 설치 하였습니다.

설치가 잘 되었는지 버전을 확인합니다.

```bash
# jekyll version
jekyll -v

# ruby version
ruby -v

# bundle version
bundle -v

# node version
node -v
```

최종 패키지 버전은 아래와 같습니다.
- chirpy: 7.3.1v (2025.10.08 기준)
- Jekyll: 4.4.1 (2025.10.08 기준)
- Ruby: 3.4.1 (2025.10.08 기준)
- Bundler: 2.6.2


### 테마 선택하기
---

[jekyll-theme](https://github.com/topics/jekyll-theme) Topic에서 Github Pages에 대한 여러 테마(theme) 확인이 가능합니다.
저는 [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) 를 선택했습니다. 버전은 **`v7.3.1`**입니다.

### Chirpy 테마 설치하기
---


Chirpy 테마 설치 방법에는 Chirpy Stater과 Github Fork 방식이 존재한다고 합니다. Chirpy Stater의 경우 커스터 마이징이 제한적이라고 해서 저는 [JJIKIN님의 Jekyll Chirpy(v6.0.1) 테마를 활용한 Github 블로그 만들기](https://jjikin.com/posts/Jekyll-Chirpy-%ED%85%8C%EB%A7%88%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-Github-%EB%B8%94%EB%A1%9C%EA%B7%B8-%EB%A7%8C%EB%93%A4%EA%B8%B0(2023-6%EC%9B%94-%EA%B8%B0%EC%A4%80)/) 에 따라 Github Fork 방식을 사용하였습니다.

#### 1. Chirpy Repository Fork

[jekyll-theme-chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) repository를 Fork 합니다.

![Chirpy Repository Fork](/assets/img/posts/20251008-guide/image-3.png){: .w-75 .shadow .rounded-10 }

Owner에 본인 계정을 설정 해주시면 됩니다. 
저는 이미 있어서 already exists라고 뜹니다.

![Chirpy Repository Fork](/assets/img/posts/20251008-guide/image-4.png){: .w-75 .shadow .rounded-10 }
![Chirpy Repository Fork](/assets/img/posts/20251008-guide/image-5.png){: .w-75 .shadow .rounded-10 }
_여러분들은 Owner로 본인을 설정 해주시면 됩니다._

<br />

#### 2. master branch name을 main으로 변경하기

원본 레포의 기본 브랜치명이 `master`로 되어있는데요, Fork한 레포지토리에서 branch> `View all branches`를 클릭합니다.

![Chirpy Repository Fork](/assets/img/posts/20251008-guide/image-7.png){: .w-75 .shadow .rounded-10 }

Rename branch 로 브랜치명을 `main`으로 변경 해줍니다.

![Chirpy Repository Fork](/assets/img/posts/20251008-guide/image-6.png){: .w-75 .shadow .rounded-10 }

![Chirpy Repository Fork](/assets/img/posts/20251008-guide/image-8.png){: .w-75 .shadow .rounded-10 }

새로고침으로 잘 변경 되었는지 확인합니다.

![Chirpy Repository Fork](/assets/img/posts/20251008-guide/image-9.png){: .w-75 .shadow .rounded-10 }

<br />

#### 3. git clone

git clone으로 로컬 개발환경을 세팅해줍니다.

```bash
cd 프로젝트 저장 경로
git clone https://github.com/{username}/bianbbc87.github.io.git
```

### Chirpy jekyll 로컬 설치 후 확인하기
---

아래 command는 clone한 프로젝트의 root 경로에서 진행합니다. (`cd ~/{username}.github.io`)

#### 1. Gemfile 의존성 설치

bundle 명령어로 jekyll과 관련된 gem 라이브러리를 설치합니다.

```bash
bundle
```

![Chirpy Repository Fork](/assets/img/posts/20251008-guide/image-11.png){: .w-75 .shadow .rounded-10 }

<br />

#### 2. node.js 패키지 설치

npm 명령어로 필요한 node.js 패키지를 설치하고 build합니다.

```bash
npm install && npm run build
```

![Chirpy Repository Fork](/assets/img/posts/20251008-guide/image-12.png){: .w-75 .shadow .rounded-10 }

여기서 다음과 같은 폴더가 생성 됩니다.
`assets/js/dist/*`

<br />

#### 3. jekyll 로컬 실행

```bash
jekyll serve
```

![Chirpy Repository Fork](/assets/img/posts/20251008-guide/image-13.png){: .w-75 .shadow .rounded-10 }

아래 콘솔과 같이 오류 없이 localhost:4000에서 Chirpy 블로그가 로드되면 성공입니다.

![Chirpy Repository Fork](/assets/img/posts/20251008-guide/image-14.png){: .w-75 .shadow .rounded-10 }

### Github Pages actions로 배포하기
---

#### 1. Github Pages Deployment를 actions로 변경하기

로컬 테스트가 완료 되었다면, 내 Fork 레포지토리에서 `Settings`>`Code and automation/Pages`>`Build and deployment`를 확인합니다.

![Chirpy Repository Fork](/assets/img/posts/20251008-guide/image-15.png){: .w-75 .shadow .rounded-10 }

github actions를 기준으로 변경해줍니다.
그럼 Jekyll에 대한 `configure` 버튼이 보일텐데, 클릭 후 생성된 jekyll.yml을 그대로 저장 해주면 됩니다.

![Chirpy Repository Fork](/assets/img/posts/20251008-guide/image-16.png){: .w-75 .shadow .rounded-10 }

저는 미리 생성 되어 있어서 사진을 못 찍었지만 여기서 코드 수정 없이 Commit Changes로 파일을 추가해주시면 됩니다.

![Chirpy Repository Fork](/assets/img/posts/20251008-guide/image-17.png){: .w-75 .shadow .rounded-10 }

그럼 바로 actions가 동작하는데, 실패합니다.
파일 로드를 못 했다는 문제이니 지금은 무시하고 다음 단계로 넘어가면 됩니다.

![Chirpy Repository Fork](/assets/img/posts/20251008-guide/image-22.png){: .w-75 .shadow .rounded-10 }

<br />

#### 2. 변경사항 pull 받아오기

충돌을 막기 위해 원격 브랜치의 변경사항을 pull 해옵니다.

```bash
git pull origin main
```

#### 3. 기존 배포 방식 파일 삭제하기 (선택)

기존 배포 방식(Deploy from a branch)에서 사용되던 workflow 파일을 삭제합니다. 이 글을 작성하던 시점 기준으로 `.github/workflows/starter/pages-deploy.yml` 위치에 있습니다.

![Chirpy Repository Fork](/assets/img/posts/20251008-guide/image-18.png){: .w-75 .shadow .rounded-10 }

<br />

#### 4. 남은 파일의 브랜치명 변경하기

workflow에 대해 `master`로 정의된 브랜치명을 `main`으로 변경합니다.
필요한 파일만 진행합니다.

- .github/workflows/ci.yml : __HTML-Proofer 테스트__
- .github/workflows/commitlint.yml : __commit convention, 로컬에서 커밋 시 컨벤션을 잘 맞춰야 합니다.__
- .github/workflows/codeql.yml : __javascript 파일 보안 취약점 스캔__

저는 3가지 workflow 파일에서 다 변경 했습니다.

> commitlint.yml을 켜두는 경우 이 `type: message`의 형태로, type은 다음과 같이 사용합니다.
> 
> build, chore, ci, docs, feat, fix, perf, refactor, revert, style, test
{: .prompt-warning }

![Chirpy Repository Fork](/assets/img/posts/20251008-guide/image-19.png){: .w-75 .shadow .rounded-10 }

이렇게 수정한 후 push 합니다.
그럼 codeql을 제외한 `ci`, `commitlint`, `jekyll` 총 3개의 workflow가 동작합니다.

![Chirpy Repository Fork](/assets/img/posts/20251008-guide/image-20.png){: .w-75 .shadow .rounded-10 }

최종적으로 jekyll workflow 1개만 실패합니다.

### 촤종 workflow 완성하기
---

#### 1. 빌드용 git ignore 재적용

빌드 오류를 확인하면 다음과 같습니다.
`Error: Can't find stylesheet to import.`

![Chirpy Repository Fork](/assets/img/posts/20251008-guide/image-23.png){: .w-75 .shadow .rounded-10 }

이 오류는 Jekyll이 @use 'vendors/bootstrap'를 찾으려 하는데 `_sass/vendors/_bootstrap.scss` 파일이 없어서 발생하는 문제입니다.

`.gitignore` 파일에 들어가서 아래 2개를 주석으로 처리합니다.
```bash
# Misc
# _sass/vendors
# assets/js/dist
```

git ignore를 다시 적용 합니다.
```bash
git rm -r --cached .
git add .
git commit -m"fix: dist/, vendors file ignore에서 제거"
git push
```

![Chirpy Repository Fork](/assets/img/posts/20251008-guide/image-24.png){: .w-75 .shadow .rounded-10 }

다시 workflow를 확인하면 성공한 것을 볼 수 있습니다.

![Chirpy Repository Fork](/assets/img/posts/20251008-guide/image-25.png){: .w-75 .shadow .rounded-10 }

<br />

#### 2. 배포 확인하기
`http://{username}.github.io` 경로로 들어가 테스트 페이지 및 블로그 기능이 정상 동작하는지 확인합니다.
로컬 환경과 동일하게 블로그 UI가 잘 뜨면 성공한 것입니다.

![Chirpy Repository Fork](/assets/img/posts/20251008-guide/image-26.png){: .w-75 .shadow .rounded-10 }


## 회고

Github Pages로 블로그를 만들어보는건 처음인데, 일반적인 md 파일에 비해 훨씬 예쁜 것 같습니다 !

단점이라면 원래 md 파일을 작성할 때 스크린샷 후 바로 붙여넣기 하는 방식으로 진행하는데, 이건 이미지 경로가 달라서 일일히 이미지를 지정 해줘야 하네요..