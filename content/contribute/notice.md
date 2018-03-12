---
date: "2018-02-23T15:21:51+09:00"
title: "[Markdown 작성하기(3/4)] Notice 적용하기"
authors: ["1000jaeh"]
series: ["markdown"]
categories:
  - contributes
tags:
  - markdown
cover:
  image: ../images/markdown.png
  caption: ""
draft: true
---
공지사항을 나타내기 위한 `notice` shortcode는 4가지 타입이 있으며, 형태는 다음과 같습니다.

### Note

``` go
{{%/* notice note */%}}
A notice disclaimer
{{%/* /notice */%}}
```

renders as

{{% notice note %}}
A notice disclaimer
{{% /notice %}}

### Info

``` go
{{%/* notice info */%}}
An information disclaimer
{{%/* /notice */%}}
```

renders as

{{% notice info %}}
An information disclaimer
{{% /notice %}}

### Tip

``` go
{{%/* notice tip */%}}
A tip disclaimer
{{%/* /notice */%}}
```

renders as

{{% notice tip %}}
A tip disclaimer
{{% /notice %}}

### Warning

``` go
{{%/* notice warning */%}}
An warning disclaimer
{{%/* /notice */%}}
```

renders as

{{% notice warning %}}
A warning disclaimer
{{% /notice %}}
