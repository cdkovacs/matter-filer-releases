# Download site (two-repo setup)

The download website is served by **GitHub Pages from a separate PUBLIC repo**
(`cdkovacs/matter-filer-releases`) so anonymous users can download the installer,
while this source repo stays **private**. The release workflow here mirrors the
signed `.pkg` into the public repo's Releases.

```
cdkovacs/dbox-email-filer      (private)  ── source + release.yml
        │  on release, pushes the .pkg to ↓
cdkovacs/matter-filer-releases (public)   ── GitHub Pages site + public .pkg downloads
```

`site/index.html` is the landing page. It lives in the public repo; it's staged here
for editing.

## One-time setup

1. **Create the public repo** (empty, public):
   ```sh
   gh repo create cdkovacs/matter-filer-releases --public \
     --description "Downloads and docs for Dropbox Matter Filer"
   ```

2. **Publish the site to it** (from this repo):
   ```sh
   tmp=$(mktemp -d); cp site/index.html "$tmp/"
   cd "$tmp" && git init -q && git add index.html \
     && git commit -qm "Add download site" \
     && git branch -M main \
     && git remote add origin https://github.com/cdkovacs/matter-filer-releases.git \
     && git push -u origin main
   ```

3. **Enable Pages** on the public repo: Settings → Pages → Build and deployment →
   Source = **Deploy from a branch**, Branch = **main / (root)**. The site appears at
   **https://cdkovacs.github.io/matter-filer-releases/** in a minute or two.

4. **Create a token so this repo's release workflow can publish to the public repo:**
   a fine-grained PAT with **Contents: Read and write** scoped to
   `cdkovacs/matter-filer-releases` (or a classic PAT with `repo`). Add it to THIS
   repo as a secret named `RELEASES_REPO_TOKEN`:
   ```sh
   gh secret set RELEASES_REPO_TOKEN -R cdkovacs/dbox-email-filer   # paste the PAT
   ```

That's it. On the next release, `release.yml` builds + notarizes the `.pkg`, publishes
it to this (private) repo's Releases, and also pushes it to the public repo — both the
versioned `DropboxMatterFiler-<version>.pkg` and a stable-named `DropboxMatterFiler.pkg`
that the site's **Download** button points at
(`…/releases/latest/download/DropboxMatterFiler.pkg`).

## Notes

- The public download page and the `.pkg` are world-readable. The `.pkg` payload
  contains the app's Python source, so publishing it makes the code effectively public.
  That's expected for this pattern; if the code must stay confidential, switch to a
  gated-download model instead.
- If `RELEASES_REPO_TOKEN` is not set, the mirror step is skipped and the private
  release still succeeds — so you can set the public repo up whenever.
- Editing the site: change `site/index.html` here, then re-push it to the public repo
  (step 2). Or edit it directly in the public repo.
