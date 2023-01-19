---
title: {{ replace .Name "-" " " | title }} 
date: {{ .Date }}
draft: true
---

# {{ replace .Name "-" " " | title }}

{{< figure src=/{{ title }}/{{ title }}.jpg width=240px height=240px >}}
