---
authors:
  - rafael
title: Blog of Blog
description: blog of setup
date:
  created: 2024-01-24
  updated: 2024-01-24
categories:
  - Setup
comments:
  true
---
# Setup blog

!!! example

    Added a demo to test features supported by `mkdocs-material`.
    If we want to add a link to another page, do not use the local `flepath`.

    - [demo](/blog/2024/01/31/demodemodemo/)
## Markdown elements reference
!!! info

    Here's the official tutorial:

    - [reference](https://squidfunk.github.io/mkdocs-material/reference/)

## Comments system
Create a new file in `overrides/partials/comments.html`.
Replace the selected line with your own `js` code.
```html hl_lines="3 3" title="overrides/partials/comments.html"
{% if page.meta.comments %}
  <h2 id="__comments">{{ lang.t("meta.comments") }}</h2>
  <!-- Insert generated snippet here -->

  <!-- Synchronize Giscus theme with palette -->
  <script>
    var giscus = document.querySelector("script[src*=giscus]")

    // Set palette on initial load
    var palette = __md_get("__palette")
    if (palette && typeof palette.color === "object") {
      var theme = palette.color.scheme === "slate"
        ? "transparent_dark"
        : "light"

      // Instruct Giscus to set theme
      giscus.setAttribute("data-theme", theme) 
    }

    // Register event handlers after documented loaded
    document.addEventListener("DOMContentLoaded", function() {
      var ref = document.querySelector("[data-md-component=palette]")
      ref.addEventListener("change", function() {
        var palette = __md_get("__palette")
        if (palette && typeof palette.color === "object") {
          var theme = palette.color.scheme === "slate"
            ? "transparent_dark"
            : "light"

          // Instruct Giscus to change theme
          var frame = document.querySelector(".giscus-frame")
          frame.contentWindow.postMessage(
            { giscus: { setConfig: { theme } } },
            "https://giscus.app"
          )
        }
      })
    })
  </script>
{% endif %}
```

## Bugs
- [ ] date time in blog preview page not showing.
