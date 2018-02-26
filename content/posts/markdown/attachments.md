---
date: "2018-02-24T09:33:07+09:00"
title: "[Markdown 작성하기(4/5)] Attachments 추가하기"
authors: ["1000jaeh"]
series: ["markdown"]
categories:
  - posts
tags:
  - Markdown
draft: false
---
The Attachments shortcode displays a list of files attached to a page.

{{% attachments /%}}

## Usage

The shortcurt lists files found in a **specific folder**.
Currently, it support two implementations for pages

1. If your page is a markdown file, attachements must be place in a **folder** named like your page and ending with **.files**.

    > * content
    >   * _index.md
    >   * page.files
    >      * attachment.pdf
    >   * page.md

2. If your page is a **folder**, attachements must be place in a nested **'files'** folder.

    > * content
    >   * _index.md
    >   * page
    >      * index.md
    >      * files
    >          * attachment.pdf

Be aware that if you use a multilingual website, you will need to have as many folders as languages.

That's all !

### Parameters

| Parameter | Default | Description |
|:--|:--|:--|
| title | "Attachments" | List's title  |
| pattern | ".*" | A regular expressions, used to filter the attachments by file name. <br/><br/>The **pattern** parameter value must be [regular expressions](https://en.wikipedia.org/wiki/Regular_expression).

For example:

* To match a file suffix of 'jpg', use **.*jpg** (not *.jpg).
* To match file names ending in 'jpg' or 'png', use **.*(jpg|png)**

### Examples

#### List of attachments ending in pdf or mp4


    {{%/*attachments title="Related files" pattern=".*(pdf|mp4)"/*/%}}

renders as

{{%attachments title="Related files" pattern=".*(pdf|mp4)"/%}}
