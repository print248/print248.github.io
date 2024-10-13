---
title: "让hugo博客支持RSS全文输出"
description: bysniper的博客支持全文输出
keywords:
  - hugo
  - rss
  - rss全文输出
  - hugo博客
date: 2024-10-13T14:26:00+08:00
tags: ["rss"]
author: bysniper
---

早上醒来，朋友发来了一条消息，分享了一枚 Follow 邀请码给我，很惊喜，立马上手体验了下。相较于传统的 RSS 阅读器，Follow 不仅界面更加简洁清爽，还巧妙地融合了社交和经济生态属性，使用体验令人耳目一新。尤其是在订阅自己的博客后，发现居然有用户在 Follow 上阅读我的博客，非常开心，突然就有了继续更新博客的动力。

然而，美中不足的是，我的博客 RSS 输出并未包含全文内容。在 Follow 中阅读时，必须多点进一步，通过内置浏览器缓慢地重新加载才能查看全文，麻烦低效，一点也不优雅。

作为使用 Hugo 搭建的博客，默认支持 RSS 生成，但是文章内容却不能全文输出。在翻阅 Hugo 默认的 rss.xml 文件后，我发现了问题：

```xml
    <item>
      //其他部分省略……
      <description>{{ .Summary | transform.XMLEscape | safeHTML }}</description>
    </item>
```

其 description 字段只显示了 Summary,我想这就是我的 rss 输出不能显示全文的原因，解决方法很简单，在 根目录的 layouts 文件夹下，新建一个 rss.xml 文件，就可以覆盖系统默认的 rss.xml 了，我的 rss.xml 全文如下，（直接复制即可）

```xml
{{- $pages := where .Site.RegularPages "Type" "in" .Site.Params.mainSections -}}
{{- $limit := .Site.Config.Services.RSS.Limit -}}
{{- if ge $limit 1 -}}
{{- $pages = $pages | first $limit -}}
{{- end -}}
<rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom">
  <channel>
    <title>{{ if eq  .Title  .Site.Title }}{{ .Site.Title }}{{ else }}{{ with .Title }}{{.}} on {{ end }}{{ .Site.Title }}{{ end }}</title>
    <link>{{ .Permalink }}</link>

    <description>feedId:66124864289300480+userId:60155982460231680</description>
	<generator>Hugo -- gohugo.io</generator>{{ with .Site.LanguageCode }}
    <language>{{.}}</language>{{end}}{{ with .Site.Author.email }}
    <managingEditor>{{.}}{{ with $.Site.Author.name }} ({{.}}){{end}}</managingEditor>{{end}}{{ with .Site.Author.email }}
    <webMaster>{{.}}{{ with $.Site.Author.name }} ({{.}}){{end}}</webMaster>{{end}}{{ with .Site.Copyright }}
    <copyright>{{.}}</copyright>{{end}}{{ if not .Date.IsZero }}
    <lastBuildDate>{{ .Date.Format "Mon, 02 Jan 2006 15:04:05 -0700" | safeHTML }}</lastBuildDate>{{ end }}
    {{- with .OutputFormats.Get "RSS" -}}
      {{ printf "<atom:link href=%q rel=\"self\" type=%q />" .Permalink .MediaType | safeHTML }}
    {{- end -}}

    {{ range $pages }}
    <item>
    <title>{{ .Title }}</title>
    <link>{{ .Permalink }}</link>
    <pubDate>{{ .Date.Format "Mon, 02 Jan 2006 15:04:05 -0700" | safeHTML }}</pubDate>
    {{ with .Params.author }}<author>{{.}}</author>{{end}}
    <guid>{{ .Permalink }}</guid>
    <description>
        {{ (replace .Content "" "") | html }}
    </description>
    </item>
    {{ end }}
  </channel>
</rss>
```

不说了，我要忙着去给我的 miniflux 搬家了
