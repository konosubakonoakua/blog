---
number: 120
title: "[sw][linux] awsome oneliners"
state: open
url: https://github.com/konosubakonoakua/blog/issues/120
author: konosubakonoakua
createdAt: 2024-10-28T03:27:27Z
updatedAt: 2024-10-28T03:27:32Z
labels: []
---

# 120 [sw][linux] awsome oneliners

## git
- make clean and clean files on all sub-folders (depth 1)
  ```bash
  find . -maxdepth 1 -type d -not -name "." -exec sh -c '[ -d "{}/.git" ] && echo "Processing directory: {}"; (cd {} && make distclean && git clean -fd)' \;
  ```
