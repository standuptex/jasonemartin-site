# jasonemartin.info

Personal website for Jason Martin, served by GitHub Pages at
[https://jasonemartin.info](https://jasonemartin.info). A single static page:
no frameworks, no build step, no analytics, no cookies.

## Preview locally

```sh
python3 -m http.server 8000
```

Then open <http://localhost:8000/>. Any static file server works; nothing needs
to be installed or compiled.

## File map

```
index.html      The entire page: content, metadata, structured data
styles.css      All styling (design tokens at the top)
404.html        Shown by GitHub Pages for unknown URLs
CNAME           Custom domain (exactly: jasonemartin.info)
robots.txt      Allows all crawlers, points to the sitemap
sitemap.xml     Single-URL sitemap
assets/         Images and icons
assets/source/  Source photo the delivery images are generated from
                (not referenced by the page, but public once deployed)
```

## Updating the text

All visible copy lives in `index.html`. Edit the text inside the `<section>`
elements (hero, About, Currently, Education, Elsewhere) and the footer.
Style notes for consistency: short paragraphs, no em or en dashes.

Each January, bump the year in the footer of `index.html` and `404.html`
(it is intentionally static text, which keeps the site JavaScript-free).

If the page title, description, or social captions change, update the matching
`<meta>` tags and the JSON-LD block in the `<head>` as well.

## Replacing the profile photo

Ideal source: a portrait-orientation photo, at least 1200x1500 pixels. Put the
original in `assets/source/`, then regenerate the delivery files (requires
ImageMagick, cwebp, and avifenc, all available via Homebrew):

```sh
SRC="assets/source/your-new-photo.jpg"
magick "$SRC" -resize 480x600^ -gravity center -extent 480x600 +repage /tmp/profile-master.png
magick /tmp/profile-master.png -strip -sampling-factor 4:2:0 -interlace JPEG -quality 78 assets/profile-photo.jpg
cwebp -q 80 -m 6 -metadata none /tmp/profile-master.png -o assets/profile-photo.webp
avifenc -q 60 --speed 4 /tmp/profile-master.png assets/profile-photo.avif
```

This fills a 4:5 frame from the center of any larger source. Keep the
`width="480" height="600"` attributes in `index.html` in sync with the true
pixel size. Never upscale a source smaller than 480x600.

## Updating links

The LinkedIn, GitHub, and email links appear in two places in `index.html`:
the hero buttons and the Elsewhere section. The same URLs also appear in the
JSON-LD block (`sameAs`). Update all occurrences together.

## Checks

With the local server running:

```sh
npx -y html-validate index.html 404.html
npx -y lighthouse http://localhost:8000/ --chrome-flags="--headless=new" --view
```

After deploying, validate structured data at <https://validator.schema.org/>
and social previews with a share debugger.

## Publishing through GitHub Pages

1. Create a public repository (for example `jasonemartin-site`).
2. Push this folder's contents to the `main` branch.
3. In the repository: Settings, then Pages, then set Source to
   "Deploy from a branch", branch `main`, folder `/ (root)`.
4. Wait for the first deploy. Because the repository contains a `CNAME` file,
   the `https://<user>.github.io/...` URL will immediately redirect to
   `jasonemartin.info`, so the site is only fully verifiable after the DNS
   cutover described below.
5. In Settings, then Pages, set the custom domain to `jasonemartin.info`
   (this matches the `CNAME` file already in the repository).

## Custom domain and HTTPS

At the DNS provider, point the apex domain at GitHub Pages with the A/AAAA
records from GitHub's current documentation, and add a `www` CNAME record
pointing at `<user>.github.io`. Follow the exact values in
[GitHub's custom-domain docs](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site);
do not use IP addresses from old tutorials.

After DNS propagates, open Settings, then Pages, and enable "Enforce HTTPS".
Certificate provisioning can take from a few minutes to a few hours after the
DNS change. Verify by loading `https://jasonemartin.info` and checking the
certificate. Also verify the domain under Settings, then Pages, then "Verified
domains" (or organization/user settings, "Pages verified domains") to prevent
domain takeover.

Do not change or remove MX, TXT, SPF, DKIM, or DMARC records: email for
`support@jasonemartin.info` depends on them.

## Rolling back

Every deploy is just a git commit on `main`:

```sh
git log --oneline          # find the last good commit
git revert <bad-commit>    # undo a specific commit, or:
git reset --hard <good>    # move back entirely (then push with --force-with-lease)
```

GitHub Pages redeploys automatically from whatever `main` contains.

## Privacy

The site intentionally contains no analytics, tracking, cookies, or third-party
requests of any kind. Keep it that way: everything the page loads comes from
this repository.
