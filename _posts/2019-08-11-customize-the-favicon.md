---
title: 파비콘 커스터마이징
author: cotes
date: 2019-08-11 00:34:00 +0800
categories: [블로깅, 튜토리얼]
tags: [파비콘]
---

[**Chirpy**](https://github.com/cotes2020/jekyll-theme-chirpy/)의 [파비콘](https://www.favicon-generator.org/about/)은 `assets/img/favicons/`{: .filepath} 디렉토리에 위치합니다. 자신만의 파비콘으로 교체하고 싶을 수 있습니다. 다음 섹션에서는 기본 파비콘을 생성하고 교체하는 방법을 안내합니다.

## 파비콘 생성

512x512 이상 크기의 정사각형 이미지(PNG, JPG 또는 SVG)를 준비한 다음, 온라인 도구 [**Real Favicon Generator**](https://realfavicongenerator.net/)로 이동하여 <kbd>Select your Favicon image</kbd> 버튼을 클릭해 이미지 파일을 업로드하세요.

다음 단계에서 웹페이지는 모든 사용 시나리오를 보여줍니다. 기본 옵션을 유지하고 페이지 하단으로 스크롤하여 <kbd>Generate your Favicons and HTML code</kbd> 버튼을 클릭해 파비콘을 생성하세요.

## 다운로드 및 교체

생성된 패키지를 다운로드하고 압축을 해제한 후, 추출된 파일에서 다음 두 파일을 삭제하세요:

- `browserconfig.xml`{: .filepath}
- `site.webmanifest`{: .filepath}

그런 다음 나머지 이미지 파일들(`.PNG`{: .filepath}과 `.ICO`{: .filepath})을 복사하여 Jekyll 사이트의 `assets/img/favicons/`{: .filepath} 디렉토리에 있는 원본 파일들을 덮어쓰세요. Jekyll 사이트에 이 디렉토리가 아직 없다면 새로 만드세요.

다음 표는 파비콘 파일 변경 사항을 이해하는 데 도움이 됩니다:

| 파일             | 온라인 도구에서                  | Chirpy에서 |
|---------------------|:---------------------------------:|:-----------:|
| `*.PNG`             | ✓                                 | ✗           |
| `*.ICO`             | ✓                                 | ✗           |

<!-- markdownlint-disable-next-line -->
>  ✓는 유지, ✗는 삭제를 의미합니다.
{: .prompt-info }

다음에 사이트를 빌드할 때 파비콘이 커스터마이징된 버전으로 교체됩니다.
