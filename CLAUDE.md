# CLAUDE.md — BBL Academy Chinese Website (bblacademy.cn)

This file provides guidance to Claude Code when working with the **Chinese website** repository. For the English sister site, see `C:\Users\PhilipTeng\Dropbox\ClaudeCode\bbl-academy\CLAUDE.md`. For dual-site sync and translation workflow, see `C:\Users\PhilipTeng\Dropbox\ClaudeCode\SYNC.md`.

---

## Project Overview

BBL Academy Chinese (脑基础学习学院) is a fully translated Simplified Chinese static website for the Brain-Based Learning Academy, targeting Chinese-speaking educators in mainland China, Hong Kong, Taiwan, and Southeast Asia.

**Live URL:** https://www.bblacademy.cn
**GitHub:** https://github.com/coolriots/bblacademy-cn.git
**English sister site:** https://bblacademy.sg (see `bbl-academy/`)

The Chinese site is a **full content translation** of the English site, sharing the same HTML structure, CSS classes, JavaScript, and images. It differs in language, fonts, hosting, and deployment.

---

## Infrastructure Overview

| Component | Details |
|---|---|
| **Domain registrar** | GoDaddy (bblacademy.cn) |
| **DNS provider** | Alibaba Cloud ESA (NS mode — GoDaddy nameservers replaced) |
| **ESA Nameservers** | `dykh-tau.ns.atrustdns.com`, `arctic.ns.atrustdns.com` |
| **Hosting** | Alibaba Cloud OSS, bucket: `bblacademy-cn`, region: Hong Kong |
| **OSS Endpoint** | `bblacademy-cn.oss-cn-hongkong.aliyuncs.com` |
| **CDN** | Alibaba Cloud ESA, free plan, Global (Excluding Chinese Mainland) |
| **SSL** | ESA auto-provisioned Let's Encrypt (auto-renewed) |
| **Root redirect** | bblacademy.cn → https://www.bblacademy.cn (301, ESA rule) |
| **ICP License** | Not required (hosted in Hong Kong, not mainland China) |

### Why Hong Kong, not Mainland China?
Hosting on mainland China requires an ICP (备案) license which needs a Chinese legal entity. Hong Kong avoids this while still giving Chinese users fast access (~10–40ms latency vs mainland nodes).

### Why not Cloudflare?
Cloudflare is throttled/blocked in mainland China. Alibaba Cloud ESA + OSS HK provides reliable access for Chinese users.

---

## Alibaba Cloud Console Navigation

| Task | Path |
|---|---|
| OSS Bucket files | OSS → bblacademy-cn → Files |
| OSS Static page config | OSS → bblacademy-cn → Data Management → Static Page |
| OSS ACL | OSS → bblacademy-cn → Permission Control → Bucket ACL: Public Read |
| ESA DNS records | ESA → Websites → bblacademy.cn → DNS → Records |
| ESA Origin rules | ESA → Websites → bblacademy.cn → Rules → Origin Rules |
| ESA Redirect rules | ESA → Websites → bblacademy.cn → Rules → Redirect Rules |
| ESA SSL certificates | ESA → Websites → bblacademy.cn → SSL/TLS → Edge Certificates |
| RAM user management | RAM → Users |

---

## Key Differences from English Site

| Aspect | English (bblacademy.sg) | Chinese (bblacademy.cn) |
|---|---|---|
| Language | English | Simplified Chinese (简体中文) |
| HTML lang | `lang="en"` | `lang="zh-CN"` |
| Fonts | Google Inter (CDN) | System fonts (no CDN) |
| Font-src in CSP | `https://fonts.gstatic.com` | `'self'` only |
| Style-src in CSP | includes `fonts.googleapis.com` | no Google domains |
| Hosting | Cloudflare Pages | Alibaba Cloud OSS HK |
| CDN | Cloudflare (global) | Alibaba ESA (excl. mainland) |
| Deploy tool | `wrangler pages deploy .` | Python `oss2` SDK |
| DNS | Cloudflare | Alibaba ESA NS mode |
| SSL | Cloudflare auto | ESA Let's Encrypt auto |

---

## Font Stack

Google Fonts (Inter) is removed — it is blocked in mainland China. All pages use:

```css
font-family: 'PingFang SC', 'Hiragino Sans GB', 'Microsoft YaHei',
             'WenQuanYi Micro Hei', -apple-system, BlinkMacSystemFont, sans-serif;
```

| Font | Platform |
|---|---|
| PingFang SC | macOS, iOS |
| Hiragino Sans GB | older macOS |
| Microsoft YaHei | Windows |
| WenQuanYi Micro Hei | Linux |
| -apple-system / BlinkMacSystemFont | Safari / Chrome fallback |

This is set in `styles.css` line 38 (the `body` rule).

---

## Site Structure

| Page | File | Chinese Title |
|---|---|---|
| Home | `index.html` | 以脑基础学习法赋能教育工作者 |
| What is BBL? | `what-is-bbl.html` | 什么是脑基础学习法？ |
| Brain Share | `brain-share.html` | 脑智分享 |
| About | `about.html` | 关于我们 |
| Programmes | `programmes.html` | 培训项目 |
| Contact Us | `contact.html` | 联系我们 |
| Prof Er Meng Hwa | `prof-er-meng-hwa.html` | 余明华教授（不在导航中）|
| Preschool Course | `preschool-course.html` | 幼儿教育脑基础学习课程（不在导航中，访问码保护）|

### Standalone Protected Pages

These pages are **not linked from the navigation** and are not synced to the English site.

#### `preschool-course.html` — 幼儿教育脑基础学习课程
- **URL:** `https://www.bblacademy.cn/preschool-course.html`
- **Access control:** Client-side JavaScript key (currently `BBL2026` — change in `<script>` block at bottom of file)
- **Purpose:** Hosts curriculum materials for pre-school teacher training on BBL
- **Content:** 3 tabs — 8 audio episodes, 8 video lessons, 8 PDF handouts
- **Media file path convention:** `course-ec/audio/episode-0N.mp3`, `course-ec/video/lesson-0N.mp4`, `course-ec/pdf/handout-0N.pdf`
- **OSS upload:** Upload media files to the `course-ec/` folder in the `bblacademy-cn` bucket
- **robots:** `noindex, nofollow` — not crawled by search engines
- **To change access key:** Edit `const ACCESS_KEY = 'BBL2026';` near the bottom of the file

**Uploading media files to OSS:**
```python
# Add to your oss2 upload script; upload each file individually with correct Content-Type
type_map['.mp3'] = 'audio/mpeg'
type_map['.mp4'] = 'video/mp4'
# PDFs are already in the type_map as 'application/pdf'
# Key example: 'course-ec/audio/episode-01.mp3'
```

### File Structure
```
bbl-academy-cn/
├── index.html
├── about.html
├── what-is-bbl.html
├── programmes.html
├── contact.html
├── brain-share.html
├── prof-er-meng-hwa.html
├── preschool-course.html   ← standalone, access-key protected, CN only
├── styles.css              ← Chinese system fonts; otherwise same as English
├── script.js               ← Identical to English site
├── BBL Academy Logo.jpg
├── favicon.ico / *.png / site.webmanifest
├── images/
│   ├── about-hero-2.jpg
│   ├── about-hero.jpg
│   ├── blog-ai-learning.jpg
│   ├── blog-brain-connections.jpg
│   ├── blog-chess-strategy.jpg
│   ├── blog-motivation-learning.jpg
│   ├── blog-student-learning.jpg
│   └── blog-three-brains.jpg
└── CLAUDE.md
```

---

## Translation Conventions

### Fixed Brand Terms
| English | Chinese | Notes |
|---|---|---|
| BBL Academy | BBL Academy | Never translate — keep in English |
| Brain-Based Learning | 脑基础学习法（Brain-Based Learning）| First mention only; then 脑基础学习法 or BBL |
| CBBE | 脑基础认证教育工作者（CBBE）| First mention only |
| Brain Share | 脑智分享 | Fixed nav term |

### Navigation (use consistently across all pages)
| English | Chinese |
|---|---|
| What is BBL? | 什么是脑基础学习法？ |
| Brain Share | 脑智分享 |
| About / About Us | 关于我们 |
| Programmes | 培训项目 |
| Contact Us | 联系我们 |
| Partner With Us | 与我们合作 |
| Learn More | 了解更多 |
| Read Article | 阅读文章 |
| Send Message | 发送消息 |

### Institutions
| English | Chinese |
|---|---|
| NTU / Nanyang Technological University | 南洋理工大学（NTU）|
| NIE / National Institute of Education | 教育研究院（NIE）|
| Curtin University Singapore | 科廷大学新加坡校区 |
| Ministry of Education | 教育部 |

### People — Always Keep in English
Professor Er Meng Hwa, Dr Lillian Koh, Dr Sharon Lim, Philip Teng, Dr Mubarak Ali, Prof Liu Woon Chia, Carol Ann Tomlinson, James P. Carse

### Dates and Times
- "19 Feb 2026" → "2026年2月19日"
- "8 min read" → "8分钟阅读"

### Footer Copyright
`© 2026 BBL Academy。保留所有权利。`

---

## Security Headers

Every HTML page has this CSP (line 8):
```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; font-src 'self'; img-src 'self' data:; connect-src 'self' https://formspree.io; frame-ancestors 'none'; base-uri 'self'; form-action 'self' https://formspree.io; object-src 'none';">
```

**Key differences from English site CSP:**
- No `https://fonts.googleapis.com` in `style-src`
- No `https://fonts.gstatic.com` in `font-src`
- `font-src 'self'` only (system fonts, no external CDN)

When updating CSP, **do not** add Google Fonts domains back.

---

## Deployment

### Prerequisites
```bash
pip install oss2
```

### Uploading Files to OSS

**CRITICAL:** Content-Type headers must be set explicitly. Without them, OSS serves files as downloads instead of rendering them in the browser.

| Extension | Content-Type |
|---|---|
| `.html` | `text/html; charset=utf-8` |
| `.css` | `text/css; charset=utf-8` |
| `.js` | `application/javascript; charset=utf-8` |
| `.jpg` | `image/jpeg` |
| `.png` | `image/png` |
| `.ico` | `image/x-icon` |
| `.webmanifest` | `application/manifest+json` |

### Full Deploy Script

```python
import oss2
import os

# Credentials: use a RAM user with AliyunOSSFullAccess permission
# Delete the RAM user after upload for security
auth = oss2.Auth('ACCESS_KEY_ID', 'ACCESS_KEY_SECRET')
bucket = oss2.Bucket(auth, 'https://oss-cn-hongkong.aliyuncs.com', 'bblacademy-cn')

local_dir = r'C:\Users\PhilipTeng\Dropbox\ClaudeCode\bbl-academy-cn'

type_map = {
    '.html': 'text/html; charset=utf-8',
    '.css':  'text/css; charset=utf-8',
    '.js':   'application/javascript; charset=utf-8',
    '.jpg':  'image/jpeg',
    '.png':  'image/png',
    '.ico':  'image/x-icon',
    '.webmanifest': 'application/manifest+json',
}

for root, dirs, files in os.walk(local_dir):
    dirs[:] = [d for d in dirs if not d.startswith('.')]
    for filename in files:
        if filename.startswith('.') or filename.endswith('.py') or filename == 'CLAUDE.md':
            continue
        local_path = os.path.join(root, filename)
        oss_key = os.path.relpath(local_path, local_dir).replace('\\', '/')
        ext = os.path.splitext(filename)[1].lower()
        mime = type_map.get(ext)
        if not mime:
            continue
        headers = {'Content-Type': mime}
        bucket.put_object_from_file(oss_key, local_path, headers=headers)
        print(f'  uploaded: {oss_key}')

print('Done.')
```

### Quick Single-File Upload
```python
import oss2
auth = oss2.Auth('ACCESS_KEY_ID', 'ACCESS_KEY_SECRET')
bucket = oss2.Bucket(auth, 'https://oss-cn-hongkong.aliyuncs.com', 'bblacademy-cn')
bucket.put_object_from_file('index.html', r'C:\...\bbl-academy-cn\index.html',
    headers={'Content-Type': 'text/html; charset=utf-8'})
```

### Git Workflow (after deploying to OSS)
```bash
cd C:\Users\PhilipTeng\Dropbox\ClaudeCode\bbl-academy-cn
git add <files>
git commit -m "Your commit message"
git push origin main
```

### RAM User Security
- Create a dedicated RAM user with `AliyunOSSFullAccess` for uploads
- Delete or disable the RAM user after upload
- Never commit credentials to Git (`.gitignore` excludes `.py` files and `.env`)
- Prefer STS tokens for temporary access over permanent Access Keys

---

## ESA Configuration Reference

### DNS Records (ESA → DNS → Records)
| Type | Hostname | Value | Proxy Status |
|---|---|---|---|
| CNAME | www | bblacademy-cn.oss-cn-hongkong.aliyuncs.com | Proxied Web |
| CNAME | @ (root) | bblacademy-cn.oss-cn-hongkong.aliyuncs.com | Proxied Web |
| TXT | _dnsauth.www | (verification string) | DNS Only |

### Origin Rules (ESA → Rules → Origin Rules)
- **Apply to:** All Requests
- **Origin Host:** `bblacademy-cn.oss-cn-hongkong.aliyuncs.com`
- **DNS Override:** `bblacademy-cn.oss-cn-hongkong.aliyuncs.com`

### Redirect Rules (ESA → Rules → Redirect Rules)
- **Condition:** Hostname = `bblacademy.cn`
- **Redirect to:** `https://www.bblacademy.cn`
- **Status:** 301 (permanent)

### SSL Certificates (ESA → SSL/TLS → Edge Certificates)
- SSL/TLS toggle: **ON**
- Certificate 1: `*.bblacademy.cn` — Free Certificate (Let's Encrypt, auto-renewed)
- Certificate 2: `www.bblacademy.cn` — Free Certificate (Let's Encrypt, auto-renewed)

---

## OSS Bucket Configuration Reference

| Setting | Value |
|---|---|
| Bucket name | bblacademy-cn |
| Region | China (Hong Kong) — `oss-cn-hongkong` |
| Storage class | Standard |
| ACL | Public Read |
| Block Public Access | Disabled |
| Static website default page | index.html |
| Static website 404 page | index.html |
| Bucket policy | Allows public `GetObject` for `acs:oss:*:*:bblacademy-cn/*` |
| Custom domain | www.bblacademy.cn (status: Verified, Proxied via ESA) |

---

## Common Tasks

### Adding a New Blog Article
1. Add the article to `bbl-academy/brain-share.html` first (English source of truth)
2. Add image to `bbl-academy/images/` and copy to `bbl-academy-cn/images/`
3. Translate the article and add to `bbl-academy-cn/brain-share.html`
4. Use next sequential content ID (`post-6-content`, `post-7-content`, etc.)
5. Use descriptive article slug for `id` attribute
6. Upload `brain-share.html` and the new image to OSS
7. See SYNC.md for full translation glossary

### Adding a New Page
1. Copy an existing HTML file as template
2. Update `<title>`, `<html lang="zh-CN">`, meta tags, page content
3. Ensure all navigation links are updated in ALL HTML files
4. Do NOT add Google Fonts link — use system fonts only
5. Add page-specific styles to `styles.css` if needed
6. Upload all changed files to OSS

### Modifying Navigation
Navigation appears in header and footer of every page. When changing nav:
1. Update all 7 HTML files
2. Upload all 7 HTML files to OSS

### Updating styles.css
1. Edit `bbl-academy-cn/styles.css`
2. Upload to OSS with `Content-Type: text/css; charset=utf-8`
3. ESA CDN may cache the old file — if needed, purge cache in ESA console

### Updating script.js
`script.js` is identical to the English site. If changes are made:
1. Copy updated `script.js` from `bbl-academy/` to `bbl-academy-cn/`
2. Upload to OSS with `Content-Type: application/javascript; charset=utf-8`

---

## Brain Share Blog Page

### Existing Articles (Chinese)

| # | ID | Content ID | Category (CN) | Date | Title (CN) |
|---|---|---|---|---|---|
| 1 | `post-infinite-games` | `post-1-content` | 学习策略 | 2026年2月19日 | 有限游戏与无限游戏 |
| 2 | `post-motivation` | `post-2-content` | 动机与身心健康 | 2026年2月19日 | 教学不一定带来学习——动机才是那座桥 |
| 3 | `post-three-brains` | `post-3-content` | 神经科学与实践 | 2026年2月19日 | 你不只有一个大脑——你有三个 |
| 4 | `post-ai-bbl` | `post-4-content` | 人工智能与未来学习 | 2026年2月19日 | 人工智能时代的脑基础学习法 |
| 5 | `post-di-bbl` | `post-5-content` | 教学与学习 | 2026年2月26日 | 差异化教学与脑基础学习法 |

**Next article:** use `id="post-{slug}"` and `id="post-6-content"` / `aria-controls="post-6-content"`.

### Blog Card Template (Chinese)
```html
<article class="blog-card" id="post-{slug}">
    <div class="blog-featured-image">
        <img src="images/{image}.jpg" alt="{alt text in Chinese}" loading="lazy">
    </div>
    <div class="blog-card-header">
        <div class="blog-meta">
            <span class="blog-category">{类别}</span>
            <span class="blog-date">YYYY年M月D日</span>
            <span class="blog-read-time">N分钟阅读</span>
        </div>
        <h2 class="blog-title">{Chinese title}</h2>
        <p class="blog-excerpt">{Chinese excerpt}</p>
        <button class="blog-expand-btn" aria-expanded="false" aria-controls="post-N-content">
            <span class="expand-text">阅读文章</span>
            <span class="expand-icon">&darr;</span>
        </button>
    </div>
    <div class="blog-card-body" id="post-N-content" aria-hidden="true">
        <div class="blog-content">
            <!-- Chinese article content here -->
        </div>
    </div>
</article>
```

---

## Sync with English Site

The English site (`bbl-academy/`) is the **source of truth** for all content.

**Always sync (apply to both sites):**
- New blog articles
- New or updated programme descriptions
- Team member additions or changes
- Contact information changes
- Bug fixes in `script.js`
- CSS improvements in `styles.css`
- New images
- Security updates (CSP, etc.)

**Do NOT sync (Chinese site only):**
- Font stack (must remain Chinese system fonts)
- CSP font-src and style-src (no Google Fonts domains)
- `lang="zh-CN"` attribute
- Chinese SEO meta descriptions
- Any China-specific content, pricing, or promotions
- Alibaba Cloud / ESA configuration

**Do NOT sync (English site only):**
- Google Fonts link tag
- Cloudflare Pages configuration
- `lang="en"` attribute
- Google Analytics (if added)

See `SYNC.md` at `C:\Users\PhilipTeng\Dropbox\ClaudeCode\SYNC.md` for the full workflow and translation glossary.

---

## Important Patterns

1. **Content-Type is mandatory for OSS** — always set explicitly when uploading; omitting causes files to download instead of render
2. **No Google Fonts** — never add back `fonts.googleapis.com` or `fonts.gstatic.com`
3. **System fonts only** — the Chinese font stack is in `styles.css` body rule
4. **All pages share header/footer** — nav changes require updating all 7 HTML files
5. **Same Formspree endpoint** — both sites use `https://formspree.io/f/mykwdllq`
6. **ESA caches aggressively** — after deploying CSS/JS, purge cache via ESA console if changes don't appear
7. **GitHub + OSS are separate** — pushing to GitHub does NOT auto-deploy to OSS; you must run the upload script separately

---

## Notes

- **Working directory:** All development in `bbl-academy-cn/`, not the parent directory
- **No build process:** Direct file editing; upload changed files to OSS immediately
- **GoDaddy:** Only used as domain registrar. DNS is fully managed by Alibaba ESA. Do not add DNS records in GoDaddy — they are ignored.
- **ESA Free Plan limits:** 10 custom certificates, adequate for this site
- **OSS billing:** Near zero — standard storage + HK outbound traffic (first 5GB/month free)
- **Mainland China access:** Users in mainland China CAN access the site via HK edge nodes (~10–40ms). Content is not blocked (educational, non-political).
