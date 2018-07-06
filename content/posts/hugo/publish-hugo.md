---
date: "2018-07-02T15:08:21+09:00"
title: "Publish-Hugo"
authors: [1000jaeh]
series: [hugo]
categories:
  - posts
tags:
  - static generator
  - hugo
  - go
  - github pages
  - travis-ci
  - travis
cover:
  image: ""
draft: true
---

빌드 배포 구성
```markdown
---
title: "My First Post"
date: 2018-07-05T13:37:53+09:00
draft: false
---

Hello HUGO!!!
```


```sh
/tech-blog $ hugo

                   | EN  
+------------------+----+
  Pages            |  7  
  Paginator pages  |  0  
  Non-page files   |  0  
  Static files     |  3  
  Processed images |  0  
  Aliases          |  0  
  Sitemaps         |  1  
  Cleaned          |  0  

Total in 33 ms

1000jaeh@skcc08922  /Volumes/1000jaeh/projects/test/tech-blog   master ✚  tree
.
├── archetypes
│   └── default.md
├── config.toml
├── content
│   └── posts
│       └── my-first-post.md
├── data
├── layouts
├── public
│   ├── 404.html
│   ├── categories
│   │   ├── index.html
│   │   └── index.xml
│   ├── dist
│   │   ├── css
│   │   │   └── app.ab4b67a3ea25990fa8279f3b7ef08b61.css
│   │   └── js
│   │       └── app.3fc0f988d21662902933.js
│   ├── images
│   │   └── gohugo-default-sample-hero-image.jpg
│   ├── index.html
│   ├── index.xml
│   ├── sitemap.xml
│   └── tags
│       ├── index.html
│       └── index.xml
├── static
└── themes
    └── ananke
        ├── LICENSE.md
        ├── README.md
        ├── archetypes
        │   └── default.md
        ├── data
        │   └── webpack_assets.json
        ├── exampleSite
        │   ├── config.toml
        │   ├── content
        │   │   ├── _index.md
        │   │   ├── about
        │   │   │   └── _index.md
        │   │   ├── contact.md
        │   │   └── post
        │   │       ├── _index.md
        │   │       ├── chapter-1.md
        │   │       ├── chapter-2.md
        │   │       ├── chapter-3.md
        │   │       ├── chapter-4.md
        │   │       ├── chapter-5.md
        │   │       └── chapter-6.md
        │   └── static
        │       └── images
        │           ├── Pope-Edouard-de-Beaumont-1844.jpg
        │           ├── Victor_Hugo-Hunchback.jpg
        │           └── esmeralda.jpg
        ├── images
        │   ├── screenshot.png
        │   └── tn.png
        ├── layouts
        │   ├── 404.html
        │   ├── _default
        │   │   ├── baseof.html
        │   │   ├── list.html
        │   │   ├── single.html
        │   │   ├── taxonomy.html
        │   │   └── terms.html
        │   ├── index.html
        │   ├── page
        │   │   └── single.html
        │   ├── partials
        │   │   ├── menu-contextual.html
        │   │   ├── page-header.html
        │   │   ├── site-favicon.html
        │   │   ├── site-footer.html
        │   │   ├── site-header.html
        │   │   ├── site-navigation.html
        │   │   ├── site-scripts.html
        │   │   ├── social-follow.html
        │   │   ├── summary-with-image.html
        │   │   ├── summary.html
        │   │   ├── svg
        │   │   │   ├── facebook.svg
        │   │   │   ├── github.svg
        │   │   │   ├── instagram.svg
        │   │   │   ├── linkedin.svg
        │   │   │   ├── twitter.svg
        │   │   │   └── youtube.svg
        │   │   └── tags.html
        │   ├── post
        │   │   ├── list.html
        │   │   ├── summary-with-image.html
        │   │   └── summary.html
        │   ├── robots.txt
        │   └── shortcodes
        │       └── form-contact.html
        ├── package.json
        ├── src
        │   ├── css
        │   │   ├── _code.css
        │   │   ├── _hugo-internal-templates.css
        │   │   ├── _social-icons.css
        │   │   ├── _styles.css
        │   │   ├── _tachyons.css
        │   │   ├── main.css
        │   │   └── postcss.config.js
        │   ├── js
        │   │   └── main.js
        │   ├── package-lock.json
        │   ├── package.json
        │   ├── readme.md
        │   ├── webpack.config.js
        │   └── yarn.lock
        ├── static
        │   ├── dist
        │   │   ├── css
        │   │   │   └── app.ab4b67a3ea25990fa8279f3b7ef08b61.css
        │   │   └── js
        │   │       └── app.3fc0f988d21662902933.js
        │   └── images
        │       └── gohugo-default-sample-hero-image.jpg
        └── theme.toml

39 directories, 82 files


```

Custom domain 적용

Https 적용

## 빌드 배포 구성

### 빌드 + .sh

### git repository branch

## 배포 자동화

### travis 란

### travis 적용

```yaml
language: go

go:
  - '1.9' # This uses automatically the latest version of go

install:
  - go get github.com/spf13/hugo # This provides the latest version of Hugo to Travis CI

script:
  - hugo # This commands builds your website on travis

deploy:
  edge:
    branch: v1.8.47
  local_dir: public # Default static site output dir for Hugo
  repo: cloudzlabs/docs # This is the slug of the repo you want to deploy your site to
  target_branch: gh-pages # GitHub pages branch to deploy to (in other cases it can be gh-pages)
  provider: pages
  skip_cleanup: true
  github_token: $GITHUB_TOKEN # This is the authentication which you will setup in the next step in travis-ci dashboard
  email: withdtlabs@gmail.com
  name: "cloudzlabs"
  fqdn: tech.cloudz-labs.io
  on:
    branch: master
```

## domain 연결
```text
CNAME tech.cloudz-labs.io
```


### 설정

### https 설정.
