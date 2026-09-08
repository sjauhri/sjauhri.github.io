---
title: '{{ replace .File.ContentBaseName "-" " " | title }}'
date: {{ .Date }}
draft: true
tags: []
description: ""
---

{{/* Math needs no flag: write $inline$ or $$display$$ and it is rendered to
     MathML at build time, with the styles added automatically. */}}
