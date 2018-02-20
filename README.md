# Labs Tech Blog

## 개요

[![Build Status](https://travis-ci.org/withdtlabs/docs.svg?branch=master)](https://travis-ci.org/withdtlabs/docs)

### 개발 환경

| 사용 기술 | 버전    | 언어  | 역할                                                                                             | 참고                 |
|-------|-------|-----|------------------------------------------------------------------------------------------------|--------------------|
| Hugo  | v0.34 | Go  | - 정적 사이트 생성기</br> - 마크다운으로 작성된 글을 미리정의한 규칙을 바탕으로 정적인 웹페이지로 만들어주는 파싱 엔진 </br> - Jekyll, Hexo 등등 | https://gohugo.io/ |

## 환경 구성

### Git Clone https://github.com/withdtlabs/docs.git

``` bash
$ git clone https://github.com/withdtlabs/docs.git
Cloning into 'docs'...
remote: Counting objects: 214, done.
remote: Compressing objects: 100% (97/97), done.
remote: Total 214 (delta 77), reused 200 (delta 63), pack-reused 0
Receiving objects: 100% (214/214), 324.21 KiB | 103.00 KiB/s, done.
Resolving deltas: 100% (77/77), done.

$ cd docs
/docs $
```

### Hugo 설치 (Mac OS)

> Windows는 https://gohugo.io/getting-started/installing 참고

``` bash
/docs $ brew install hugo
... 설치 진행 ...

/docs $ hugo version
Hugo Static Site Generator v0.34 darwin/amd64 BuildDate: 
Local 확인 → http://localhost:1313/docs 접속
/docs $ hugo serve -D

                   | EN
+------------------+----+
  Pages            | 19
  Paginator pages  |  0
  Non-page files   |  0
  Static files     |  0
  Processed images |  0
  Aliases          |  0
  Sitemaps         |  1
  Cleaned          |  0

Total in 47 ms
Watching for changes in /Volumes/1000jaeh/Projects/test/docs/{content,data}
Serving pages from memory
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at http://localhost:1313/docs/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```

## 구조

``` text
/docs
    +- archetypes/
    |   +- default.md
    |   +- page.md
    +- content/
    |   +- posts/
    |       +- first-post.md
    |       +- second-post.md
    |       +- third-post.md
    +- data/
    |   +- authors/
    |       +- first-author.toml
    |       +- second-author.toml
    +- layouts/
    +- public/
    +- static/
    +- themes/
    |   +- labs-theme/
    +- .editorconfig
    +- .gitignore
    +- .gitmodules
    +- .travis.yml
    +- config.toml
```

| 항목              | 내용                        | 비고                         |
|-----------------|---------------------------|----------------------------|
| /docs           | 프로젝트 홈                    |                            |
| archetypes      | 컨텐츠 기본 구조 정의              | default.md 파일에서 마크다운 구조 설정 |
| content/posts   | 블로그에 올라갈 마크다운 파일 위치       |                            |
| data            | 태그, 카테고리, 저자 등 기타 항목 정의   |                            |
| layouts, static | 블로그의 템플릿 및 정적 리소스 위치      | 현재는 테마를 사용하기 때문에 사용안함      |
| public          | 블로그 빌드 타켓 폴더              | gh-pages 브랜치에 Push될 결과물    |
| themes          | 사용할 Hugo 테마 위치            |                            |
| .editorconfig   | 프로젝트 내의 코딩 컨벤션 설정 파일      |                            |
| .travis.yml     | 빌드/배포를 위한 Travis CI 설정 파일 |                            |
| config.toml     | Hugo 블로그 전체 설정 파일         |                            |

## How to posting

### New Document

``` bash
/docs $ hugo new posts/글_제목.md
/docs/content/posts/글_제목.md created
```

### 생성된 마크다운 파일 확인

#### /docs/content/posts/하위

``` text
/docs
    +- archetypes/
    |   +- default.md
    |   +- page.md
    +- content/
    |   +- posts/
    |       +- first-post.md
    |       +- second-post.md
    |       +- third-post.md
    +- data/
    |   +- authors/
    |       +- first-author.toml
    |       +- second-author.toml
    +- layouts/
    +- public/
    +- static/
    +- themes/
    |   +- labs-theme/
    +- .editorconfig
    +- .gitignore
    +- .gitmodules
    +- .travis.yml
    +- config.toml
```

``` markdown
---
date: "2018-02-07T13:18:35+09:00"
title: "Test-Post"
authors: []
categories:
  -
tags:
  -
draft: true
---


<!-- 마크다운으로 글 작성 -->
```

- date: 파일 생성 일시
- title: 글 제목. 파일명과 동일
- authors: 저자
- categories: 카테고리
- tages: 태그
- draft: 글 상태 설정. false일 경우, 블로그에 노출됨

### Posting

#### "draft" 항목을 false로 변경

``` markdown
---
date: "2018-02-07T13:18:35+09:00"
title: "글 제목"
authors: ["저자", "저자"]
categories:
  - 카테고리 1
  - 카테고리 2
tags:
  - 태그 1
  - 태그 2
draft: true
---


<!-- 글 내용 -->
```

### Git Repository에 Push

``` bash
/docs $ git add .
/docs $ git commit -m "test"
[master 8b97416] test
 1 file changed, 175 insertions(+)
 create mode 100644 content/posts/new-post.md


/docs $ git push
Counting objects: 5, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (5/5), 3.53 KiB | 3.53 MiB/s, done.
Total 5 (delta 2), reused 0 (delta 0)
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
To https://github.com/withdtlabs/docs.git
   ef2cbb9..8b97416  master -> master
```

### Travis CI에서 자동 빌드 및 gh-pages에서 배포

### [Labs 블로그](https://withdtlabs.github.io/docs)에서 확인
