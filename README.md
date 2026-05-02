# ![Renderit.themes Logo](./assets/images/logo-renderit-themes.png)

**renderit.themes** is the official theme and addon repository for the [renderit.builder](https://github.com/seu-org/renderit.builder) engine — a static site generator built with Vanilla JS that takes HTML templates with **Magic Tags** and JSON content files to render blazing-fast static sites with native SEO and zero dependencies.

![HTML5 Badge](https://img.shields.io/badge/HTML5-E34F26.svg?style=for-the-badge&logo=HTML5&logoColor=white)
![CSS3 Badge](https://img.shields.io/badge/CSS-663399.svg?style=for-the-badge&logo=CSS&logoColor=white)
![JS Badge](https://img.shields.io/badge/JavaScript-F7DF1E.svg?style=for-the-badge&logo=JavaScript&logoColor=black)
![GPLv3 Badge](https://img.shields.io/badge/GPLv3-BD0000.svg?style=for-the-badge&logo=GPLv3&logoColor=white)

---

## What is renderit.themes?

This repository serves two purposes:

**Themes** are complete, ready-to-use HTML templates that can be loaded directly into renderit.builder. Each theme is a full website structure — with pages, partials, assets and a sample `data.json` — designed to be customized through data, not code.

**Addons** are self-contained HTML components — sliders, forms, maps, FAQs, WhatsApp buttons, and more — that can be dropped into any theme using a single Magic Tag. Each addon handles its own markup, styles and behavior without depending on the parent theme.

Both themes and addons are consumed automatically by [renderit.builder](https://github.com/seu-org/renderit.builder), which resolves, injects and renders them at build time.

---

## The Renderit Ecosystem

| Project | Role |
|---------|------|
| [renderit.editor](https://github.com/zmdy/renderit.editor) | Visual HTML editor — non-destructive, framework-agnostic |
| [renderit.builder](https://github.com/zmdy/renderit.builder) | Build engine: template + JSON → static site |
| **renderit.themes** | Themes and addons repository ← you are here |
| renderit.manager | Remote site management dashboard *(coming soon)* |

---

## Operation Modes

Themes and addons in this repository are compatible with all three renderit.builder operation modes:

**1 — Static Mode**
renderit.builder generates a `.zip` file containing all static assets needed for the site to work. Fastest and safest delivery method. Ideal for landing pages, portfolios and institutional sites with infrequent content changes.

**2 — Live Mode**
The site is partially static but hydrates specific zones with real-time data via JSON endpoints or API calls. Ideal for restaurants, stores and businesses with frequently changing content like prices, menus and stock.

**3 — Manager Mode**
Builds the site with editable attributes on every rendered element, allowing remote content editing through a backend bridge file (`bridge.php` or `bridge.asp`). Managed via the renderit.manager dashboard.

---

## Addons

Addons are plug-and-play HTML components. To use an addon in any theme, simply add its Magic Tag to the template:

```html
<section class="faq py-20">
  <div class="container">
    %ADDON accordion%
  </div>
</section>
```

renderit.builder automatically resolves the tag, injects the addon HTML, and maps its data keys to your `data.json`.

### Available Addons

| Addon | File | Description |
|-------|------|-------------|
| Slider | `slider.html` | Image carousel with autoplay, dots and controls |
| Form | `form.html` | Configurable contact form |
| Accordion / FAQ | `accordion.html` | Expandable question and answer list |
| Map | `map.html` | Interactive map via Leaflet CDN |
| WhatsApp Button | `whatsapp.html` | Floating WhatsApp button with pre-defined message |
| Gallery | `gallery.html` | Image grid with native lightbox |
| Popup | `popup.html` | Modal with time or scroll trigger |
| Testimonials | `testimonials.html` | Customer testimonials carousel |
| Pricing Table | `pricing.html` | Pricing plan cards with featured highlight |
| Countdown | `countdown.html` | Countdown timer to a configurable date |
| Tabs | `tabs.html` | Tab navigation with independent content per tab |
| Video Embed | `video.html` | Responsive YouTube/Vimeo embed |
| Newsletter | `newsletter.html` | Email capture form |
| Stats | `stats.html` | Animated counters with labels |
| Team | `team.html` | Team member card grid |
| Timeline | `timeline.html` | Vertical timeline with dates and descriptions |
| Cookie Banner | `cookie-banner.html` | GDPR consent banner |
| Social Feed | `social-feed.html` | Post grid with image, caption and link |
| Sticky CTA | `sticky-cta.html` | Fixed top or bottom call-to-action bar |
| Comparison Table | `comparison.html` | Feature comparison table |

---

## Contributing

Contributions are welcome. To add a new theme or addon:

1. Fork this repository
2. Create your theme in `themes/my-theme/` or your addon in `addons/my-addon.html`
3. Follow the structure and rules described above
4. Include a complete `data.json` sample for themes, or document the expected data structure in a comment block at the top of the addon file
5. Add a `preview.png` for themes
6. Open a pull request with a short description of the component

Please make sure your contribution works correctly when loaded into renderit.builder before submitting.

---

## License

GPL v3 — see [LICENSE](./LICENSE) for details.