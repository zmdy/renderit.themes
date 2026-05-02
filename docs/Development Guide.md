# ![Renderit.themes Logo](../assets/images/logo-renderit-themes.png)

**renderit.themes** is the official theme and addon repository for the [renderit.builder](https://github.com/seu-org/renderit.builder) engine — a static site generator built with Vanilla JS that takes HTML templates with **Magic Tags** and JSON content files to render blazing-fast static sites with native SEO and zero dependencies.

![HTML5 Badge](https://img.shields.io/badge/HTML5-E34F26.svg?style=for-the-badge&logo=HTML5&logoColor=white)
![CSS3 Badge](https://img.shields.io/badge/CSS-663399.svg?style=for-the-badge&logo=CSS&logoColor=white)
![JS Badge](https://img.shields.io/badge/JavaScript-F7DF1E.svg?style=for-the-badge&logo=JavaScript&logoColor=black)
![GPLv3 Badge](https://img.shields.io/badge/GPLv3-BD0000.svg?style=for-the-badge&logo=GPLv3&logoColor=white)

---

## Creating a Theme

A theme is a folder inside `/themes` with the following structure:

```
themes/my-theme/
├── index.html        ← main template (single-page)
│                        or multiple .html files (multi-page)
├── partials/
│   ├── header.html
│   └── footer.html
├── assets/
│   ├── style.css
│   └── main.js       ← optional
├── data.json         ← complete sample data file
└── preview.png       ← screenshot of the rendered theme
```

### Step 1 — Write the template using Magic Tags

Use Magic Tags to mark every piece of content that should come from the JSON data file:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <title>%pages.0.title% — %SITE_NAME%</title>
  <meta name="description" content="%pages.0.description%">
  <style>
    :root {
      --color-primary: %design.colors.primary%;
      --font-title: %design.fonts.primary%;
    }
  </style>
</head>
<body>

  %PARTIAL header%

  <section class="hero">
    <h1>%content.hero.title%</h1>
    <p>%content.hero.subtitle%</p>
    <a href="%content.hero.cta_url%">%content.hero.cta_label%</a>
  </section>

  %FOREACH content.features%
  <div class="feature-card">
    <h3>%title%</h3>
    <p>%description%</p>
  </div>
  %ENDFOREACH%

  %IF content.show_faq%
  <section class="faq">
    %ADDON accordion%
  </section>
  %ENDIF%

  %PARTIAL footer%

  %ADDON whatsapp%

</body>
</html>
```

### Step 2 — Create the sample `data.json`

Every Magic Tag in the template must have a corresponding key in the data file. Always include the `design` key with the full design system:

```json
{
  "meta": { "version": "1.0.0", "language": "en" },
  "site": {
    "name": "My Site",
    "url": "https://mysite.com",
    "description": "Site description for SEO",
    "contact": { "email": "", "phone": "", "whatsapp": "" }
  },
  "pages": [
    {
      "slug": "index",
      "template": "index.html",
      "title": "Home — My Site",
      "description": "Page meta description",
      "content": {
        "hero": {
          "title": "Welcome",
          "subtitle": "Your subtitle here",
          "cta_label": "Learn More",
          "cta_url": "#features"
        },
        "features": [
          { "title": "Feature One", "description": "Description here" },
          { "title": "Feature Two", "description": "Description here" }
        ],
        "show_faq": "true"
      }
    }
  ],
  "accordion": {
    "items": [
      { "pergunta": "How does it work?", "resposta": "It's very simple..." }
    ]
  },
  "whatsapp": {
    "number": "15550001111",
    "message": "Hello! I found your site and would like more information."
  },
  "design": {
    "colors": {
      "primary": "#1A1C1E",
      "secondary": "#6C7278",
      "tertiary": "#0066CC",
      "neutral": "#F7F5F2",
      "surface": "#FFFFFF"
    },
    "fonts": {
      "primary": "Inter",
      "secondary": "Space Grotesk",
      "sources": [
        "https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&family=Space+Grotesk:wght@400;500&display=swap"
      ]
    },
    "typography": {
      "headline-lg": { "fontFamily": "Inter", "fontSize": "48px", "fontWeight": 700, "lineHeight": 1.1 },
      "body-md":     { "fontFamily": "Inter", "fontSize": "16px", "fontWeight": 400, "lineHeight": 1.6 }
    },
    "spacing": { "section": "120px", "container-max": "1200px" },
    "rounded": { "md": "8px", "lg": "12px", "full": "9999px" }
  }
}
```

### Step 3 — Add a preview screenshot

Take a screenshot of the rendered theme at 1440px width and save it as `preview.png` in the theme folder. This image is displayed in the renderit.builder theme gallery.

---

## Creating an Addon

An addon is a single `.html` file inside `/addons`. It must be fully self-contained — its own markup, styles and behavior — with no dependency on the parent theme.

### Step 1 — Create the addon file

The filename defines the addon name (e.g. `my-addon.html` is called with `%ADDON my-addon%`).

```html
<!-- Addon: my-addon -->

<div class="renderit-my-addon">
  <h2>%my_addon.title%</h2>

  %FOREACH my_addon.items%
  <div class="item" data-renderit-key="%id%">
    <h3>%title%</h3>
    <p>%description%</p>
  </div>
  %ENDFOREACH%
</div>

<style>
  .renderit-my-addon {
    /* scoped styles — prefix all selectors with .renderit-my-addon */
    padding: 2rem;
  }
  .renderit-my-addon .item {
    border: 1px solid #eee;
    border-radius: 8px;
    padding: 1rem;
    margin-bottom: 1rem;
  }
</style>

<script>
(function () {
  // Always wrap in an IIFE to avoid polluting the global scope
  const container = document.querySelector('.renderit-my-addon')
  if (!container) return

  // addon behavior here
})();
</script>
```

### Step 2 — Define the expected data structure

Every Magic Tag used in the addon becomes a required key in the site's `data.json`. The addon namespace is its filename without the extension:

```json
{
  "my_addon": {
    "title": "Section Title",
    "items": [
      { "id": "1", "title": "Item One", "description": "Description here" },
      { "id": "2", "title": "Item Two", "description": "Description here" }
    ]
  }
}
```

renderit.builder automatically extracts all Magic Tags from the addon HTML and includes this structure in the generated sample JSON.

### Addon Rules

- **First line** must be an HTML comment with the addon name: `<!-- Addon: name -->`
- **All styles** must be scoped with a `.renderit-` prefixed class to avoid conflicts
- **All scripts** must be wrapped in an IIFE `(function() { ... })()`
- **External libraries** may be loaded via CDN inside the addon's `<script>` tag if needed
- **No dependencies** on parent theme CSS or JS — the addon must work standalone
- **Data namespace** uses the addon filename as prefix for all Magic Tags (`%addonname.field%`)

---

## Magic Tags Reference

| Tag | Description |
|-----|-------------|
| `%variable%` | Simple variable |
| `%object.property%` | Nested object property |
| `%array.0.field%` | Array item by index |
| `%FOREACH collection%` ... `%ENDFOREACH%` | Loop over array |
| `%INDEX%` / `%INDEX_1%` | Current loop index (0-based / 1-based) |
| `%TOTAL%` | Total items in loop |
| `%FIRST%` / `%LAST%` | `"true"` if first or last item |
| `%IF field%` ... `%ENDIF%` | Conditional block |
| `%IF field == "value"%` | Equality comparison |
| `%ELSE%` | Else branch |
| `%PARTIAL name%` | Include a partial from `/partials` |
| `%ADDON name%` | Include an addon from `/addons` |
| `%YEAR%` | Current year |
| `%SITE_NAME%` | Value of `site.name` in JSON |
| `%SITE_URL%` | Value of `site.url` in JSON |
| `%BUILD_DATE%` | Build date (ISO 8601) |
| `%%` | Literal `%` character |

For the full syntax reference, see the renderit.builder documentation.