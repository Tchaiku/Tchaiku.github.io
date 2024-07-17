---
ShowReadingTime: true
ShowWordCount: true
# UseHugoToc: true
author: Me
# canonicalURL: https://canonical.url/to/page
comments: true
# cover:
#   alt: <alt text>
#   caption: <text>
#   hidden: true
#   image: <image path/url>
#   relative: false
date: {{ .Date }}
lastmod: {{ .Date }}
# description: Desc Text.
# disableHLJS: false
# disableShare: false
draft: true
# editPost:
#   Text: Suggest Changes
#   URL: https://github.com/<path_to_repo>/content
#   appendFilePath: true

# summary: "say something"
hideSummary: false
hidemeta: false
params:
  math: true
# searchHidden: true
showToc: true
tags:
# - first
title: '{{ replace .File.ContentBaseName "-" " " | title }}'
weight: 10
---