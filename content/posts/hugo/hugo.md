---
date: "2018-06-21T10:01:45+09:00"
title: "Hugo를 활용한 Blog 구축기"
authors: [1000jaeh]
series: [hugo]
categories:
  - posts
tags:
  - static generator
  - hugo
  - golang
  - go
cover:
  image: "../images/hugo.png"
draft: true
---

현재 보고계신 블로그는 **Hugo + Github Pages**로 운영되고 있습니다.
이번 글에서는 Hugo를 활용한 Static Site 구축에 대해서 얘기해볼까 합니다.

## CMS에서 Static Generator까지

많은 사람들이 블로그를 구축한다고 했을 때, 가장 많이 떠올리는 것은 다음과 같이 2가지 형태일 것입니다.

- Naver Blog, Tistroy, Blogger 등등
- Wordpress, Drupal, Joomla

첫번째 형태인 Naver나 Tistroy의 포털 사이트에서 제공하는 가입형 블로그 서비스는, 단순하게 가입만 해도 사용할 수 있기 때문에, 도메인이나 호스팅 등등의 것들을 신경쓰지 않고 바로 글을 작성할 수 있죠. 그러나, 운영자가 블로그를 커스터마이징할 수 있는 영역이 매우 적습니다.

그래서 처음에 기술 블로그를 운영하려고 했을 때 선택한 형태는, CMS를 사용하는 것이었습니다. CMS는 'Contents Management System'의 약자로 Blog를 직접 설치해서 사용한다고 이해를 하면 쉬울 것 같습니다. 저희가 생각했던 초기 컨셉에 맞게 블로그를 구성하고 싶었고, 다양한 기능들을 블로그에 적용하고 싶었습니다. 최종적으로 저희는 **Wordpress**를 선택했었습니다.

### CMS 대장 Wordpress

Wordpress는 PHP로 이루어진 [CMS의 최강자](https://stackshare.io/self-hosted-blogging-cms)입니다. 수천가지의 테마와 플러그인을 유/무료 제공하고 있으며, 커뮤니티도 활발하게 이루어져 있습니다. Linkedin, ebay, mozilla등의 기업들도 Wordpress를 이용하여 블로그를 운영하고 있습니다.

Wordpress는 [오픈소스 버전](https://wordpress.org/)과 [상용 버전](https://ko.wordpress.com/)이 존재합니다. 오픈소스는 직접 Wordpress를 설치하여 구성해야합니다. 서버, 도메인, 호스팅까지 모든 것들을 사용자가 직접 설정하고 운영해야합니다. 반대로 상용버전은 최소한의 기능만 제공되는 블로그를 가입하면 무료로 사용할 수 있도록 제공하고, 지불되는 금액에 따라 기능을 추가로 제공하는 형식을 띄고 있습니다. 하지만, 블로그를 직접 호스팅하지 않아도 된다는 장점이 있죠. 그래서 저희는 상용 버전의 Wordpress를 사용했습니다. 별도의 서버 및 호스팅에 대한 부분을 Wordpress에 맡기고, 저희는 블로그만 잘 구성하고 글만 관리하면 될 줄 알았습니다...

### Too Much

저희가 Wordpress를 사용하면서 느낀 점은,
첫번째. 너무 많다는 것입니다. 블로그를 기획하면서 가장 깊게 고민했던 사항은 얼마나 글을 쉽게 쓸 수 있는 환경이 제공되는지에 대한 여부였습니다. 

두번째. 

스태틱제너레이터다

이런것들에는 지킬과 기타등등등이 있따.

휴고를 선ㅌ택하게 된 이유는?

이런 것들이 있어서 그렇다.

Wordpress에서의 글을 작성하는 방식은 일반적인 웹 에디터(ex. tinymce)를 이용하여 작성하거나, 별도의 글쓰기 App을 설치하여 Wordpress와 연동해서 작성하는 방식이 있었습니다. 글쓰기 App은 당연히 추가로 구입을 해야하기 때문에, 

웹 에디터의 경우, 
플러그인을 통해서 다양한 기능을 확장

{{% center %}}
***아무리 이 플러그인 저 플러그인을 설치해도 저희들에겐 글쓰기가 너무 불편하고 별로 였습니다.***
{{% /center %}}

## 그래서 찾은 해답: Static Generator

저희는 기술 블로그를 운영함에 있어서 가장 중요하게 고려되어야 할 점들에 대해서 다시 정리했습니다.

1. Blog를 운영하기 쉬울 것: 최대한 Backend, DB, Server 등등의 우리가 직접 관리해야할 포인트가 적으면 적을수록 좋다!
2. 글을 쓰고 꾸미는 것이 쉬울 것: 작성한 글을 별도로 꾸미지 않아도 어느정도 가다듬어 나타날 수 있으면 좋다!
3. 커스터마이징이 쉬울 것: 우리가 원하는 대로, UI를 쉽게 바꿀 수 있으면 좋다!


결국, 도달한 결론은 ***Static Generator(정적 웹사이트 생성기)***로 블로그를 구성하는 방법이었습니다.

 지정된 Template을 바탕으로 HTML 만들어주는 것입니다.

### Hugo

Hugo는 Go언어로 이루어진 Static Generator입니다. 실제 휴고를 좀 살펴보면 이렇게 되어 있다.

## Blog 구축

자, 지금부터는 Hugo를 사용하여 Blog를 구성해보겠습니다.

{{% notice info %}}
본 내용은 Mac을 기준으로 작성하였습니다.
{{% /notice %}}

### Hugo 설치

Homebrew로 hugo를 설치합니다. 타 OS는 [Install Hugo](https://gohugo.io/getting-started/installing/)를 참고하여 설치하시기 바랍니다.

```sh
$ brew install hugo
... 설치 진행 ...

$ hugo version
Hugo Static Site Generator v0.34 darwin/amd64 BuildDate:
```

### Static Blog 생성하기

`hugo new site [PATH]`로 Static Site를 생성합니다.
```sh
$ hugo new site tech-blog
Congratulations! Your new Hugo site is created in /tech-blog.

Just a few more steps and you're ready to go:

1. Download a theme into the same-named folder.
   Choose a theme from https://themes.gohugo.io/, or
   create your own with the "hugo new theme <THEMENAME>" command.
2. Perhaps you want to add some content. You can add single files
   with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
3. Start the built-in live server via "hugo server".

Visit https://gohugo.io/ for quickstart guide and full documentation.
```

Hugo로 생성된 Static Site의 구조는 다음과 같습니다.

```sh
/tech-blog $ tree
.
├── archetypes
│   └── default.md
├── config.toml
├── content
├── data
├── layouts
├── static
└── themes
```

각 항목에 대한 상세역할은 다음과 같습니다.

| 항목 | 내용 | 비고 |
| --- | --- | --- |
| archetypes | 컨텐츠 기본 구조 정의 | default.md 파일에서 마크다운 구조 설정
| config.toml | Hugo 블로그 전체 설정 파일 |
| content/posts | 블로그에 올라갈 마크다운 파일 위치 |
| data | 태그, 카테고리, 저자 등 기타 항목 정의 |
| layouts, static | 블로그의 템플릿 및 정적 리소스 위치 | 현재는 테마를 사용하기 때문에 사용안함
| themes | 사용할 Hugo 테마 위치 |
| public | 블로그 빌드 타켓 폴더 | gh-pages 브랜치에 Push될 결과물

## Theme 적용하기

갓 생성된 Static Site는 디렉토리로만 구성되어 있습니다. 따라서, Theme를 적용하여 화면이 보일 수 있도록 하겠습니다. Hugo도 Wordpress와 마찬가지로 다양한 Theme이 제공되고 있으며, 대부분 Open Source로 자유롭게 사용하실 수 있습니다. Theme에 대한 상세내용 [Hugo Themes](https://themes.gohugo.io/)에서 확인할 수 있습니다.

```sh
$ cd tech-blog
/tech-blog $ git init;
Initialized empty Git repository in /Volumes/1000jaeh/test/workspace/tech-blog/.git/
/tech-blog $ git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke;
Cloning into '/Volumes/1000jaeh/test/workspace/tech-blog/themes/ananke'...
remote: Counting objects: 1104, done.
remote: Compressing objects: 100% (17/17), done.
remote: Total 1104 (delta 9), reused 9 (delta 4), pack-reused 1083
Receiving objects: 100% (1104/1104), 2.51 MiB | 1.85 MiB/s, done.
Resolving deltas: 100% (584/584), done.
```

Git으로 Pull 받은 Ananke Theme을 사용하기 위해서 `config.toml` 파일에 `theme`의 속성값을 `ananke`로 적용합니다.

```toml
baseURL = "http://example.org/"
languageCode = "en-us"
title = "My New Hugo Site"
theme = "ananke"
```

{{% notice info %}}
TOML이란 "Tom's Obvious, Minimal Language."의 약자로 Github의 창업자인 Tom Preston-Werner가 만든 새로운 설정 파일 형식이다.
{{% /notice %}}

### Posting

`hugo new [PATH]`를 사용하여 새로 포스팅할 글을 `/content/posts`내에 Markdown형식으로 생성합니다.

```sh
/tech-blog $ hugo new posts/my-first-post.md
/Volumes/1000jaeh/test/workspace/tech-blog/content/posts/my-first-post.md created
```

생성된 Markdown은 다음과 같이 구성되어 있습니다.

```markdown
---
title: "My First Post"
date: 2018-06-26T13:35:54+09:00
draft: true
---

Hello HUGO!!!
```

상단의 `---` 영역에 작성된 부분은 생성된 Post에 대한 Metadata를 담고 있는 **Front Matter**부분이며, 하단 부분부터 실제 글이 작성되는 부분입니다. Front Matter는 `hugo new`로 새로운 Post가 생성될 때 자동으로 작성되며, Metadata에 대한 Template는 `/archetypes/default.md`에서 확인하고 수정할 수 있습니다.

### Run Hugo

구성된 Static Site를 `hugo serve [FLAGS]`로 실행시킵니다. Hugo의 Default Port는 1313입니다. `-D` 옵션은 `draft`상태의 글을 Build시킬지 확인하는 옵션입니다.

```sh
/tech-blog $ hugo serve -D

                   | EN  
+------------------+----+
  Pages            | 10  
  Paginator pages  |  0  
  Non-page files   |  0  
  Static files     |  3  
  Processed images |  0  
  Aliases          |  1  
  Sitemaps         |  1  
  Cleaned          |  0  

Total in 49 ms
Watching for changes in /Volumes/1000jaeh/test/workspace/tech-blog/{content,data,layouts,static,themes}
Watching for config changes in /Volumes/1000jaeh/test/workspace/tech-blog/config.toml
Serving pages from memory
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```

http://localhost:1313에 접속하여, 정상적으로 Static Site가 구성되었는지, 작성한 글이 제대로 Posting 되었는지 확인합니다.

{{% center %}}
![tech-blog](../images/hugo-site.png)
{{% /center %}}

이번 글에서는 Static Generator인 Hugo로 간단하게 Blog를 구성해 봤습니다. 다음 글에서는 Travis-CI를 이용하여, Github에 Blog가 자동으로 Hosting될 수 있는 환경을 구축하고, 작성한 글이 제대로 Publishing되는지까지 확인해보겠습니다.
