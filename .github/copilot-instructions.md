# Copilot Instructions

This is a Jekyll blog deployed to GitHub Pages at www.prioritized.net.

## Development

```sh
bundle exec jekyll serve
```

## Blog Post Conventions

Posts live in `_posts/` and follow Jekyll's `YYYY-MM-DD-title-slug.md` naming.

**Frontmatter** uses `layout: post` and includes `title` and `summary`. The
`summary` field is displayed on the index page; if omitted, Jekyll's auto
excerpt is used instead.

**External posts** (published on other sites like thoughtbot) use
`external_url` and `external_site` instead of a post body. The index page links
directly to the external URL.

## Layouts and Includes

- `default` — base HTML shell; all other layouts wrap this
- `post` — single blog post
- `page` — static pages (contact, talks, podcasts)
- `youtube.html` include — embed YouTube videos with `{% include youtube.html id="VIDEO_ID" %}`

## Styles

Sass lives in `_sass/` and is compiled through `css/main.scss`. Partials are
named by the component they style (e.g., `_post.scss`, `_footer.scss`). Base
styles (typography, forms, buttons, etc.) live in `_sass/base/`.

Sass variables for colors, spacing, and breakpoints are defined in
`_sass/base/_variables.scss` and `_sass/base/_grid-settings.scss`. Use plain
CSS features — no Bourbon, Neat, or other mixin libraries.

## Permalink Structure

Posts use `/blog/:title` (configured in `_config.yml`), producing URLs like
`/blog/testing-ruby-gems-with-github-actions`.
