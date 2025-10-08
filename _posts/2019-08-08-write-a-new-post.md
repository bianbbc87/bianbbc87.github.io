---
title: 새 게시글 작성하기
author: cotes
date: 2019-08-08 14:10:00 +0800
categories: [블로깅, 튜토리얼]
tags: [글쓰기]
render_with_liquid: false
---

이 튜토리얼은 _Chirpy_ 템플릿에서 게시글을 작성하는 방법을 안내합니다. Jekyll을 사용해본 적이 있더라도 읽어볼 가치가 있습니다. 많은 기능들이 특정 변수 설정을 필요로 하기 때문입니다.

## 파일명과 경로

`YYYY-MM-DD-제목.확장자`{: .filepath} 형식으로 새 파일을 만들어 루트 디렉토리의 `_posts`{: .filepath} 폴더에 넣으세요. `확장자`{: .filepath}는 반드시 `md`{: .filepath} 또는 `markdown`{: .filepath} 중 하나여야 합니다. 파일 생성 시간을 절약하고 싶다면 [`Jekyll-Compose`](https://github.com/jekyll/jekyll-compose) 플러그인 사용을 고려해보세요.

## Front Matter

기본적으로 게시글 상단에 다음과 같은 [Front Matter](https://jekyllrb.com/docs/front-matter/)를 작성해야 합니다:

```yaml
---
title: 제목
date: YYYY-MM-DD HH:MM:SS +/-TTTT
categories: [상위카테고리, 하위카테고리]
tags: [태그]     # 태그명은 항상 소문자로 작성
---
```

> 게시글의 _layout_은 기본적으로 `post`로 설정되어 있으므로, Front Matter 블록에 _layout_ 변수를 추가할 필요가 없습니다.
{: .prompt-tip }

### 날짜의 시간대

게시글의 발행 날짜를 정확히 기록하려면 `_config.yml`{: .filepath}의 `timezone`을 설정할 뿐만 아니라 Front Matter 블록의 `date` 변수에도 게시글의 시간대를 제공해야 합니다. 형식: `+/-TTTT`, 예: `+0800`.

### 카테고리와 태그

각 게시글의 `categories`는 최대 두 개의 요소를 포함하도록 설계되었으며, `tags`의 요소 수는 0개부터 무한대까지 가능합니다. 예를 들어:

```yaml
---
categories: [동물, 곤충]
tags: [벌]
---
```

### 작성자 정보

게시글의 작성자 정보는 일반적으로 _Front Matter_에 작성할 필요가 없습니다. 기본적으로 설정 파일의 `social.name` 변수와 `social.links`의 첫 번째 항목에서 가져옵니다. 하지만 다음과 같이 재정의할 수도 있습니다:

`_data/authors.yml`에 작성자 정보를 추가하세요 (웹사이트에 이 파일이 없다면 주저하지 말고 만드세요).

```yaml
<작성자_id>:
  name: <전체 이름>
  twitter: <작성자의_트위터>
  url: <작성자의_홈페이지>
```
{: file="_data/authors.yml" }

그런 다음 `author`를 사용하여 단일 항목을 지정하거나 `authors`를 사용하여 여러 항목을 지정하세요:

```yaml
---
author: <작성자_id>                     # 단일 항목용
# 또는
authors: [<작성자1_id>, <작성자2_id>]   # 여러 항목용
---
```

그런데 `author` 키도 여러 항목을 식별할 수 있습니다.

> `_data/authors.yml`{: .filepath } 파일에서 작성자 정보를 읽는 것의 장점은 페이지에 `twitter:creator` 메타 태그가 있어 [Twitter Cards](https://developer.twitter.com/en/docs/twitter-for-websites/cards/guides/getting-started#card-and-content-attribution)를 풍부하게 하고 SEO에 좋다는 것입니다.
{: .prompt-info }

### 게시글 설명

기본적으로 게시글의 첫 번째 단어들이 홈페이지의 게시글 목록, _추가 읽기_ 섹션, RSS 피드의 XML에 표시됩니다. 게시글에 대해 자동 생성된 설명을 표시하고 싶지 않다면, _Front Matter_의 `description` 필드를 사용하여 다음과 같이 사용자 정의할 수 있습니다:

```yaml
---
description: Short summary of the post.
---
```

Additionally, the `description` text will also be displayed under the post title on the post's page.

## 목차

기본적으로 **목**차(TOC)는 게시글의 오른쪽 패널에 표시됩니다. 전역적으로 끄고 싶다면 `_config.yml`{: .filepath}로 이동하여 `toc` 변수의 값을 `false`로 설정하세요. 특정 게시글에 대해 목차를 끄고 싶다면 게시글의 [Front Matter](https://jekyllrb.com/docs/front-matter/)에 다음을 추가하세요:

```yaml
---
toc: false
---
```

## 댓글

댓글에 대한 전역 설정은 `_config.yml`{: .filepath} 파일의 `comments.provider` 옵션으로 정의됩니다. 이 변수에 댓글 시스템이 선택되면 모든 게시글에 댓글이 활성화됩니다.

특정 게시글의 댓글을 닫고 싶다면 게시글의 **Front Matter**에 다음을 추가하세요:

```yaml
---
comments: false
---
```

## 미디어

_Chirpy_에서는 이미지, 오디오, 비디오를 미디어 리소스라고 합니다.

### URL 접두사

때때로 게시글의 여러 리소스에 대해 중복된 URL 접두사를 정의해야 하는데, 이는 두 개의 매개변수를 설정하여 피할 수 있는 지루한 작업입니다.

- 미디어 파일을 호스팅하기 위해 CDN을 사용하는 경우 `_config.yml`{: .filepath }에서 `cdn`을 지정할 수 있습니다. 그러면 사이트 아바타와 게시글의 미디어 리소스 URL에 CDN 도메인 이름이 접두사로 붙습니다.

  ```yaml
  cdn: https://cdn.com
  ```
  {: file='_config.yml' .nolineno }

- 현재 게시글/페이지 범위의 리소스 경로 접두사를 지정하려면 게시글의 _front matter_에서 `media_subpath`를 설정하세요:

  ```yaml
  ---
  media_subpath: /path/to/media/
  ---
  ```
  {: .nolineno }

`site.cdn`과 `page.media_subpath` 옵션은 개별적으로 또는 조합하여 사용하여 최종 리소스 URL을 유연하게 구성할 수 있습니다: `[site.cdn/][page.media_subpath/]file.ext`

### 이미지

#### 캡션

이미지의 다음 줄에 이탤릭체를 추가하면 캡션이 되어 이미지 하단에 나타납니다:

```markdown
![img-description](/path/to/image)
_Image Caption_
```
{: .nolineno}

#### 크기

To prevent the page content layout from shifting when the image is loaded, we should set the width and height for each image.

```markdown
![Desktop View](/assets/img/sample/mockup.png){: width="700" height="400" }
```
{: .nolineno}

> For an SVG, you have to at least specify its _width_, otherwise it won't be rendered.
{: .prompt-info }

Starting from _Chirpy v5.0.0_, `height` and `width` support abbreviations (`height` → `h`, `width` → `w`). The following example has the same effect as the above:

```markdown
![Desktop View](/assets/img/sample/mockup.png){: w="700" h="400" }
```
{: .nolineno}

#### 위치

By default, the image is centered, but you can specify the position by using one of the classes `normal`, `left`, and `right`.

> Once the position is specified, the image caption should not be added.
{: .prompt-warning }

- **Normal position**

  Image will be left aligned in below sample:

  ```markdown
  ![Desktop View](/assets/img/sample/mockup.png){: .normal }
  ```
  {: .nolineno}

- **Float to the left**

  ```markdown
  ![Desktop View](/assets/img/sample/mockup.png){: .left }
  ```
  {: .nolineno}

- **Float to the right**

  ```markdown
  ![Desktop View](/assets/img/sample/mockup.png){: .right }
  ```
  {: .nolineno}

#### Dark/Light mode

You can make images follow theme preferences in dark/light mode. This requires you to prepare two images, one for dark mode and one for light mode, and then assign them a specific class (`dark` or `light`):

```markdown
![Light mode only](/path/to/light-mode.png){: .light }
<!-- ![Dark mode only](/path/to/dark-mode.png){: .dark } -->
```

#### 그림자

The screenshots of the program window can be considered to show the shadow effect:

```markdown
![Desktop View](/assets/img/sample/mockup.png){: .shadow }
```
{: .nolineno}

#### 미리보기 이미지

If you want to add an image at the top of the post, please provide an image with a resolution of `1200 x 630`. Please note that if the image aspect ratio does not meet `1.91 : 1`, the image will be scaled and cropped.

Knowing these prerequisites, you can start setting the image's attribute:

```yaml
---
image:
  path: /path/to/image
  alt: image alternative text
---
```

Note that the [`media_subpath`](#url-prefix) can also be passed to the preview image, that is, when it has been set, the attribute `path` only needs the image file name.

For simple use, you can also just use `image` to define the path.

```yml
---
image: /path/to/image
---
```

#### LQIP

For preview images:

```yaml
---
image:
  lqip: /path/to/lqip-file # or base64 URI
---
```

> You can observe LQIP in the preview image of post \"[Text and Typography](../text-and-typography/)\".

For normal images:

```markdown
![Image description](/path/to/image){: lqip="/path/to/lqip-file" }
```
{: .nolineno }

### Social Media Platforms

You can embed video/audio from social media platforms with the following syntax:

```liquid
{% include embed/{Platform}.html id='{ID}' %}
```

Where `Platform` is the lowercase of the platform name, and `ID` is the video ID.

The following table shows how to get the two parameters we need in a given video/audio URL, and you can also know the currently supported video platforms.

| Video URL                                                                                                                  | Platform   | ID                       |
| -------------------------------------------------------------------------------------------------------------------------- | ---------- | :----------------------- |
| [https://www.**youtube**.com/watch?v=**H-B46URT4mg**](https://www.youtube.com/watch?v=H-B46URT4mg)                         | `youtube`  | `H-B46URT4mg`            |
| [https://www.**twitch**.tv/videos/**1634779211**](https://www.twitch.tv/videos/1634779211)                                 | `twitch`   | `1634779211`             |
| [https://www.**bilibili**.com/video/**BV1Q44y1B7Wf**](https://www.bilibili.com/video/BV1Q44y1B7Wf)                         | `bilibili` | `BV1Q44y1B7Wf`           |
| [https://www.open.**spotify**.com/track/**3OuMIIFP5TxM8tLXMWYPGV**](https://open.spotify.com/track/3OuMIIFP5TxM8tLXMWYPGV) | `spotify`  | `3OuMIIFP5TxM8tLXMWYPGV` |

Spotify supports some additional parameters:

- `compact` - to display compact player instead (ex. `{% include embed/spotify.html id='3OuMIIFP5TxM8tLXMWYPGV' compact=1 %}`);
- `dark` - to force dark theme (ex. `{% include embed/spotify.html id='3OuMIIFP5TxM8tLXMWYPGV' dark=1 %}`).

### Video Files

If you want to embed a video file directly, use the following syntax:

```liquid
{% include embed/video.html src='{URL}' %}
```

Where `URL` is a URL to a video file e.g. `/path/to/sample/video.mp4`.

You can also specify additional attributes for the embedded video file. Here is a full list of attributes allowed.

- `poster='/path/to/poster.png'` — poster image for a video that is shown while video is downloading
- `title='Text'` — title for a video that appears below the video and looks same as for images
- `autoplay=true` — video automatically begins to play back as soon as it can
- `loop=true` — automatically seek back to the start upon reaching the end of the video
- `muted=true` — audio will be initially silenced
- `types` — specify the extensions of additional video formats separated by `|`. Ensure these files exist in the same directory as your primary video file.

Consider an example using all of the above:

```liquid
{%
  include embed/video.html
  src='/path/to/video.mp4'
  types='ogg|mov'
  poster='poster.png'
  title='Demo video'
  autoplay=true
  loop=true
  muted=true
%}
```

### Audio Files

If you want to embed an audio file directly, use the following syntax:

```liquid
{% include embed/audio.html src='{URL}' %}
```

Where `URL` is a URL to an audio file e.g. `/path/to/audio.mp3`.

You can also specify additional attributes for the embedded audio file. Here is a full list of attributes allowed.

- `title='Text'` — title for an audio that appears below the audio and looks same as for images
- `types` — specify the extensions of additional audio formats separated by `|`. Ensure these files exist in the same directory as your primary audio file.

Consider an example using all of the above:

```liquid
{%
  include embed/audio.html
  src='/path/to/audio.mp3'
  types='ogg|wav|aac'
  title='Demo audio'
%}
```

## 고정된 게시글

You can pin one or more posts to the top of the home page, and the fixed posts are sorted in reverse order according to their release date. Enable by:

```yaml
---
pin: true
---
```

## 프롬프트

There are several types of prompts: `tip`, `info`, `warning`, and `danger`. They can be generated by adding the class `prompt-{type}` to the blockquote. For example, define a prompt of type `info` as follows:

```md
> Example line for prompt.
{: .prompt-info }
```
{: .nolineno }

## 문법

### Inline Code

```md
`inline code part`
```
{: .nolineno }

### Filepath Highlight

```md
`/path/to/a/file.extend`{: .filepath}
```
{: .nolineno }

### Code Block

Markdown symbols ```` ``` ```` can easily create a code block as follows:

````md
```
This is a plaintext code snippet.
```
````

#### Specifying Language

Using ```` ```{language} ```` you will get a code block with syntax highlight:

````markdown
```yaml
key: value
```
````

> The Jekyll tag `{% highlight %}` is not compatible with this theme.
{: .prompt-danger }

#### Line Number

By default, all languages except `plaintext`, `console`, and `terminal` will display line numbers. When you want to hide the line number of a code block, add the class `nolineno` to it:

````markdown
```shell
echo 'No more line numbers!'
```
{: .nolineno }
````

#### Specifying the Filename

You may have noticed that the code language will be displayed at the top of the code block. If you want to replace it with the file name, you can add the attribute `file` to achieve this:

````markdown
```shell
# content
```
{: file="path/to/file" }
````

#### Liquid Codes

If you want to display the **Liquid** snippet, surround the liquid code with `{% raw %}` and `{% endraw %}`:

````markdown
{% raw %}
```liquid
{% if product.title contains 'Pack' %}
  This product's title contains the word Pack.
{% endif %}
```
{% endraw %}
````

Or adding `render_with_liquid: false` (Requires Jekyll 4.0 or higher) to the post's YAML block.

## 수학

We use [**MathJax**][mathjax] to generate mathematics. For website performance reasons, the mathematical feature won't be loaded by default. But it can be enabled by:

[mathjax]: https://www.mathjax.org/

```yaml
---
math: true
---
```

After enabling the mathematical feature, you can add math equations with the following syntax:

- **Block math** should be added with `$$ math $$` with **mandatory** blank lines before and after `$$`
  - **Inserting equation numbering** should be added with `$$\begin{equation} math \end{equation}$$`
  - **Referencing equation numbering** should be done with `\label{eq:label_name}` in the equation block and `\eqref{eq:label_name}` inline with text (see example below)
- **Inline math** (in lines) should be added with `$$ math $$` without any blank line before or after `$$`
- **Inline math** (in lists) should be added with `\$$ math $$`

```markdown
<!-- Block math, keep all blank lines -->

$$
LaTeX_math_expression
$$

<!-- Equation numbering, keep all blank lines  -->

$$
\begin{equation}
  LaTeX_math_expression
  \label{eq:label_name}
\end{equation}
$$

Can be referenced as \eqref{eq:label_name}.

<!-- Inline math in lines, NO blank lines -->

"Lorem ipsum dolor sit amet, $$ LaTeX_math_expression $$ consectetur adipiscing elit."

<!-- Inline math in lists, escape the first `$` -->

1. \$$ LaTeX_math_expression $$
2. \$$ LaTeX_math_expression $$
3. \$$ LaTeX_math_expression $$
```

> Starting with `v7.0.0`, configuration options for **MathJax** have been moved to file `assets/js/data/mathjax.js`{: .filepath }, and you can change the options as needed, such as adding [extensions][mathjax-exts].  
> If you are building the site via `chirpy-starter`, copy that file from the gem installation directory (check with command `bundle info --path jekyll-theme-chirpy`) to the same directory in your repository.
{: .prompt-tip }

[mathjax-exts]: https://docs.mathjax.org/en/latest/input/tex/extensions/index.html

## Mermaid

[**Mermaid**](https://github.com/mermaid-js/mermaid) is a great diagram generation tool. To enable it on your post, add the following to the YAML block:

```yaml
---
mermaid: true
---
```

Then you can use it like other markdown languages: surround the graph code with ```` ```mermaid ```` and ```` ``` ````.

## 더 알아보기

For more knowledge about Jekyll posts, visit the [Jekyll Docs: Posts](https://jekyllrb.com/docs/posts/).
