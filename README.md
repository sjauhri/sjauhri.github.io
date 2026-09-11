# sjauhri.github.io

Source for [snehal.jauhri](https://sjauhri.github.io). Forked from [Jon Barron's template](https://github.com/jonbarron/jonbarron_website).

The portfolio is plain HTML and CSS with no build step. The blog under `/blog/`
is Hugo. A GitHub Actions workflow builds both and publishes them together on a
push to `main` — see [Publishing](#publishing) for how to push without updating
the live site.

## Layout

```
index.html        # the portfolio: header, news, research interests, publications
stylesheet.css    # typography + a couple of helpers
images/           # headshot (snehal.jpg) + one GIF thumbnail per paper
data/             # CV PDF (and bibtex if added later)
blog/             # Hugo site published at /blog/
  content/posts/  # one Markdown file per post
  assets/diagrams/# SVGs inlined by the figure-svg shortcode
  static/         # favicons and other files served as-is
  hugo.yaml       # title, tagline, menu, header brand
.github/workflows/deploy.yml   # builds and publishes both
```

## Preview locally

**Portfolio** — from this directory:

```bash
python3 -m http.server 8765
```

Open <http://127.0.0.1:8765/>. Edits to any file are reflected on refresh.

**Blog** — needs [Hugo extended](https://gohugo.io/installation/) (0.146 or newer,
which the PaperMod theme requires):

```bash
hugo server --source blog --buildDrafts
```

Open <http://localhost:1313/blog/>. This live-reloads on save and shows posts
marked `draft: true`.

To check the whole site exactly as the workflow builds it:

```bash
rm -rf _site && mkdir _site
for item in *; do case "$item" in blog|_site) continue;; esac; cp -r "$item" _site/; done
hugo --source blog --minify --baseURL "http://localhost:8765/blog/" --destination "$PWD/_site/blog"
(cd _site && python3 -m http.server 8765)
```

## Editing the portfolio

Everything lives in `index.html`. Three editable sections, marked by `<!-- ============ ... ============ -->` banners:

### Add a news item

Open the `<!-- ============ NEWS ============ -->` block and copy an existing `<tr>`. Most recent goes at the top:

```html
<tr>
  <td style="padding:4px 16px;vertical-align:top;white-space:nowrap;color:#888;width:100px;">Jul 2026</td>
  <td style="padding:4px 16px;vertical-align:top;">Short sentence describing the news, with <a href="...">a link</a>.</td>
</tr>
```

### Add a publication

Drop a thumbnail (GIF, JPG, or PNG, ~160px square or wider) into `images/<slug>.gif`, then copy an existing publication `<tr>` inside the `<!-- ============ PUBLICATIONS ============ -->` block. Most recent first:

```html
<tr>
  <td style="padding:16px;width:20%;vertical-align:middle">
    <img src="images/<slug>.gif" width="160" loading="lazy" alt="<paper> thumbnail">
  </td>
  <td style="padding:8px;width:80%;vertical-align:middle">
    <a href="<project page url>">
      <span class="papertitle">Paper Title</span>
    </a>
    <br>
    <strong>Snehal Jauhri</strong>, Coauthor One, Coauthor Two
    <br>
    <em>Venue Acronym</em>, 2026
    <br>
    <a href="<project page>">project page</a> /
    <a href="<arxiv>">arXiv</a> /
    <a href="<github>">code</a>
  </td>
</tr>
```

For an award, append after the venue/year line:

```html
&nbsp; <font color="#FF8080"><strong>(Award name)</strong></font>
```

To highlight a row in pale yellow (like the IROS 2022 paper), add `bgcolor="#ffffd0"` to the `<tr>`.

### Update headshot, CV, or bio

- Headshot: replace `12

GitHub Pages serves whatever `.github/workflows/deploy.yml` publishes. To publish edits:

```bash
git add -A
git commit -m "..."
git push
```

The site rebuilds within ~1 minute at <https://sjauhri.github.io>.

## Writing a blog post

Create a Markdown file under `blog/content/posts/`, or run
`hugo new content posts/my-post.md --source blog` to start from the archetype.
Frontmatter needs `title`, `date`, `tags` and `description`; set `draft: true`
while working, and remove it (or set `false`) to publish.

- **Math** needs no flag. `$inline$` and `$$display$$` are rendered to MathML at
  build time, and the styles are added automatically on pages that contain math.
- **Code fences** get syntax highlighting, line numbers and a copy button.
- **Images** go in `blog/static/` and are referenced as `/blog/name.png`. Files
  already at the repo root are reachable as `/images/name.gif` — no need to copy
  them into `blog/`.
- **Diagrams** live in `blog/assets/diagrams/` as SVG and are placed with
  `{{< figure-svg src="name.svg" caption="..." >}}`. The shortcode inlines them
  so they follow the site's dark/light toggle; an SVG loaded through `<img>`
  would only see the reader's OS setting. Style them with the `dg-*` classes
  defined in `blog/layouts/partials/figures.html`.
- **Captions** are written `<figcaption><p>…</p></figcaption>` — PaperMod styles
  a bare `figcaption` as a bold title and the inner `<p>` as the caption.

## Publishing

A push to `main` publishes by default. Pushes to any other branch never publish,
so a working branch is always safe.

To land commits on `main` **without** updating the live site, there are three
controls. The workflow's summary on each run says which applied and why.

### 1. Skip one push

Put `[skip deploy]` anywhere in the commit message:

```bash
git commit -m "Draft second post [skip deploy]"
```

Commits land, the site stays as it is. `[deploy skip]` works too, and both are
case-insensitive. GitHub's own `[skip ci]` also works but is blunter — it skips
every workflow, including any tests added later.

### 2. Hold every deploy

For a stretch of work spanning several commits, create the pause file:

```bash
touch .deploy-paused
git add .deploy-paused && git commit -m "Pause deploys while drafting"
```

Every push then lands without publishing. Resume with:

```bash
git rm .deploy-paused && git commit -m "Resume deploys"
```

That push publishes, carrying everything held back with it. The file is a
dotfile, so it is never copied into the built site.

### 3. Publish on demand

Actions → **Deploy to GitHub Pages** → *Run workflow*. Tick **force** to publish
even while paused or when the last commit said to skip — useful for shipping a
held-back batch without an extra commit.

To preview what a deploy would produce, build locally first (see
[Preview locally](#preview-locally)).

## Notes

- All GIFs always animate. They're lazy-loaded (`loading="lazy"`) so off-screen ones don't fetch until you scroll near. If page weight gets uncomfortable later, convert the heavier GIFs to MP4 with `ffmpeg -i in.gif -movflags faststart -pix_fmt yuv420p out.mp4` and use `<video autoplay loop muted playsinline>` instead.
- No analytics, no third-party scripts. Adding any (e.g. plausible.io, GA) means inserting a `<script>` in `<head>`.
- Custom domain: put a `CNAME` file at the repo root containing the domain (e.g. `snehaljauhri.com`), set DNS A/CNAME records per [GitHub's docs](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site).
