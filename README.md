# dil4rd.me

Personal site — [Hugo](https://gohugo.io/) + the
[Congo](https://github.com/jpanther/congo) theme, deployed to GitHub Pages.

## Requirements

Docker only. The Hugo toolchain runs in a container, so there is nothing to
install on the host and the version is pinned in one place: the image tag in
`compose.yaml`. CI uses that same file, so local and production can't drift.

## Everyday commands

```bash
docker compose up                 # dev server, drafts included, http://localhost:1313
docker compose down               # stop it

docker compose run --rm hugo new content posts/my-post.md
docker compose run --rm hugo --gc --minify        # production build into public/
docker compose run --rm hugo mod get -u github.com/jpanther/congo/v2   # upgrade theme
```

## Previewing without opening a browser

`docker compose up -d server` first, then:

```bash
docker compose run --rm shot                    # -> .preview/home.png
docker compose run --rm shot --screenshot=/out/404.png http://localhost:1313/404.html

# dark mode
APPEARANCE=dark AUTO_APPEARANCE=false docker compose up -d server
docker compose run --rm shot --screenshot=/out/home-dark.png http://localhost:1313/
```

Headless Chrome, so fonts come from the container and won't match your
desktop exactly — it's for checking layout, not typography.

Hugo itself cannot do this. It has no layout or raster engine, so nothing
short of a browser turns HTML + CSS into pixels. Hugo *can* generate images
from primitives (`images.Text`, `images.Filter`, `.Resize`) which is how
you'd build Open Graph share cards — but that composites text onto a source
image, it does not render a page.

## Layout

| Path                  | What it is                                        |
| --------------------- | ------------------------------------------------- |
| `config/_default/`    | Site + theme config, split the way Congo expects   |
| `content/`            | Markdown. Just `_index.md` (the placeholder home) for now |
| `static/CNAME`        | Custom domain — Hugo copies it into `public/`      |
| `.github/workflows/`  | Build + deploy on push to `main`                   |
| `go.mod` / `go.sum`   | Pins the Congo version (it's a Hugo Module)        |

## Upgrading Hugo

Change the image tag in `compose.yaml`. That's the whole procedure — CI reads
the same file.

## Current state

Placeholder only: the home page says "under construction". There are no posts
or About page yet, the nav is empty (`_merge = "none"` in `menus.en.toml`
blocks Congo's own Blog/Categories/Tags entries), search is off and taxonomy
pages are disabled. Comments in those files mark what to restore.

## Deployment notes

- Pages source must be set to **GitHub Actions** in repo settings.
- The domain lives in `static/CNAME`, not the Pages UI — Actions deploys
  overwrite whatever the UI sets.
- Cloudflare DNS records stay **DNS-only (grey cloud)**. GitHub provisions
  its TLS cert over an HTTP-01 challenge that the Cloudflare proxy breaks,
  on first issue and on every ~90-day renewal.
