---
date: "2018-02-23T15:21:51+09:00"
title: "[Markdown 작성하기(3/4)] Notice 적용하기"
authors: ["1000jaeh"]
series: ["markdown"]
categories:
  - posts
tags:
  - markdown
cover:
  image: /images/default/markdown.png
  caption: ""
draft: false
---
The notice shortcode shows 4 types of disclaimers to help you structure your page.

### Note

```
{{%/* notice note */%}}
A notice disclaimer
{{%/* /notice */%}}
```

renders as

{{% notice note %}}
A notice disclaimer
{{% /notice %}}

### Info

```
{{%/* notice info */%}}
An information disclaimer
{{%/* /notice */%}}
```

renders as

{{% notice info %}}
An information disclaimer
{{% /notice %}}

### Tip

```
{{%/* notice tip */%}}
A tip disclaimer
{{%/* /notice */%}}
```

renders as

{{% notice tip %}}
A tip disclaimer
{{% /notice %}}

### Warning

```
{{%/* notice warning */%}}
An warning disclaimer
{{%/* /notice */%}}
```

renders as

{{% notice warning %}}
A warning disclaimer
{{% /notice %}}
