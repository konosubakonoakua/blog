---
number: 55
title: "[ci][github][action][workflow] github action intro"
state: open
url: https://github.com/konosubakonoakua/blog/issues/55
author: konosubakonoakua
createdAt: 2024-05-28T03:21:33Z
updatedAt: 2024-05-28T03:21:38Z
labels: []
---

# 55 [ci][github][action][workflow] github action intro

## how to add manual trigger
need `workflow_dispatch`.
```yaml
name: update lockfile
on:
  # Scheduled update (each day)
  schedule: [{ cron: "30 01 * * *" }]
  workflow_dispatch:
```
