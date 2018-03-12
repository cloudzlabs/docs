---
date: "2018-02-22T14:19:43+09:00"
title: "[Markdown 작성하기(2/4)] Markdown내에서 가운데 정렬 적용하기"
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
`center` shortcode를 사용하여, 작성한 Content를 중앙정렬 할 수 있습니다.

```golang
{{%/* center */%}}
_Center Aligned Text_
{{%/* /center */%}}
```

render as

{{% center %}}
_Center Aligned Text_
{{% /center %}}
