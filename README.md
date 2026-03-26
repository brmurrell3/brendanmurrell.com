# brendanmurrell.com

Personal site built with Hugo + PaperMod, deployed via GitHub Pages.

## Setup

### 1. Install Hugo

```bash
# macOS
brew install hugo

# or download from https://gohugo.io/installation/
```

### 2. Add the PaperMod theme

```bash
cd brendanmurrell.com
git init
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

### 3. Update config

Edit `hugo.yaml`:
- Replace `YOURUSERNAME` in the GitHub and LinkedIn URLs with your actual usernames
- Update the `baseURL` if using a different domain
- Update `static/CNAME` if using a different domain

### 4. Preview locally

```bash
hugo server -D
# Site available at http://localhost:1313
```

### 5. Push to GitHub

```bash
git add .
git commit -m "Initial site scaffold"
git remote add origin git@github.com:YOURUSERNAME/brendanmurrell.com.git
git push -u origin main
```

### 6. Enable GitHub Pages

1. Go to repo Settings > Pages
2. Under "Build and deployment", set Source to **GitHub Actions**
3. The workflow will run automatically on push

### 7. Configure DNS

At your domain registrar, add these records:

```
Type  Name  Value
A     @     185.199.108.153
A     @     185.199.109.153
A     @     185.199.110.153
A     @     185.199.111.153
CNAME www   YOURUSERNAME.github.io
```

Then in repo Settings > Pages, add your custom domain and enable "Enforce HTTPS" once DNS propagates (can take up to 24h).

## Writing research posts

```bash
hugo new research/your-post-slug.md
```

Edit the generated file in `content/research/`. Set `draft: false` when ready to publish.
