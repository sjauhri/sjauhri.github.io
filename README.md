# snehaljauhri.github.io

Source for [snehal.jauhri](https://snehaljauhri.github.io). Forked from [Jon Barron's template](https://github.com/jonbarron/jonbarron_website).

Plain HTML and CSS — no build step, no Jekyll, no Node. Edit a file, push to `main`, GitHub Pages serves it.

## Layout

```
index.html        # the entire site: header, news, research interests, publications
stylesheet.css    # typography + a couple of helpers
images/           # headshot (snehal.jpg) + one GIF thumbnail per paper
data/             # CV PDF (and bibtex if added later)
```

## Preview locally

From this directory:

```bash
python3 -m http.server 8765
```

Open <http://127.0.0.1:8765/>. Edits to any file are reflected on refresh.

## Editing

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

GitHub Pages is configured to serve `main` of the `sjauhri/snehaljauhri.github.io` repo. To publish edits:

```bash
git add -A
git commit -m "..."
git push
```

The site rebuilds within ~1 minute at <https://snehaljauhri.github.io>.

## Notes

- All GIFs always animate. They're lazy-loaded (`loading="lazy"`) so off-screen ones don't fetch until you scroll near. If page weight gets uncomfortable later, convert the heavier GIFs to MP4 with `ffmpeg -i in.gif -movflags faststart -pix_fmt yuv420p out.mp4` and use `<video autoplay loop muted playsinline>` instead.
- No analytics, no third-party scripts. Adding any (e.g. plausible.io, GA) means inserting a `<script>` in `<head>`.
- Custom domain: put a `CNAME` file at the repo root containing the domain (e.g. `snehaljauhri.com`), set DNS A/CNAME records per [GitHub's docs](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site).
