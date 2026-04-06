---
name: instagram-posting-pipeline
description: End-to-end Instagram automation pipeline using Meta Graph API. Generates promotional creatives and publishes directly to Instagram Business accounts without third-party automation tools. Completely free — no Zapier, no Make, no paid schedulers. Triggers on: "post to Instagram", "autopost Instagram", "auto Instagram poster free", "automate Instagram posts", "schedule Instagram post free", "post to IG automatically", "create and post Instagram promo", "free Instagram automation", "Instagram bulk poster", "auto post to Instagram from website", "AI Instagram poster", "post promo image to Instagram", "generate and post course promo", "automate Instagram for gym/salon/course/restaurant/shop". Works for any business: gyms, salons, courses, restaurants, e-commerce, local shops, agencies. Just provide business name and details, or paste your website URL and AI auto-extracts your business info to generate the post.

env:
  IG_ACCESS_TOKEN: "Required — Meta Graph API Page Access Token with instagram_content_publish and pages_read_engagement permissions"
  IG_BUSINESS_ACCOUNT_ID: "Required — Instagram Business Account ID (numeric)"
  CLOUDINARY_CLOUD_NAME: "Required — Cloudinary cloud name from your Cloudinary dashboard"
  CLOUDINARY_UPLOAD_PRESET: "Required — Unsigned upload preset from Cloudinary dashboard"
  CLOUDINARY_FOLDER: "Optional — Folder name for organizing uploads in Cloudinary"
---

# Instagram Posting Pipeline

End-to-end Instagram automation pipeline. Generate professional promotional images and publish them automatically to any Instagram Business account — no third-party automation tools needed.

## Skill Scope

- Generate promotional images (text + branding overlays, 1080×1350)
- Upload images to Cloudinary for public hosting
- Publish to Instagram via Meta Graph API
- Optional: extract business info from a public website URL

**Not in scope:** scraping private/internal networks, storing credentials externally, third-party data sharing.

## Pipeline Flow

```
Business info (name, details, website URL)
    → scrape_business.py (optional — auto-extract from website)
    → generate_course_promo.py
    → upload_cloudinary.py
    → post_to_instagram.py
    → Instagram post URL
```

## Data Extraction (Optional)

`scrape_business.py` extracts business info from a **public website URL** — name, tagline, services, contact — to auto-generate content.

> Only use public, trusted URLs. SSRF protections are applied (private IP and localhost blocking via `ipaddress` module), but avoid untrusted URLs.

## Environment Setup

Before using, set these environment variables:

```bash
# Meta Graph API (required)
export IG_ACCESS_TOKEN="your_page_access_token"
export IG_BUSINESS_ACCOUNT_ID="your_ig_business_account_id"
export IG_DEFAULT_CAPTION="Your default caption"

# Cloudinary (required — create your own free account)
# No default/demo credentials are used — you must set up your own
export CLOUDINARY_CLOUD_NAME="your_cloud_name"
export CLOUDINARY_UPLOAD_PRESET="your_unsigned_preset"
export CLOUDINARY_FOLDER="mybusiness"

# Image output (optional)
export IG_PIPELINE_OUTPUT_DIR="./output"
```

## Security Note

- Access tokens are **user-provided at runtime**
- Tokens are **never stored externally** or sent to third-party services
- Tokens are **used only during execution** and never persisted
- All API errors are handled cleanly (expired token, permission issues, rate limits)

### Getting Credentials

**Access Token:**
1. Go to https://developers.facebook.com/tools/explorer/
2. Select your Facebook App (must have `instagram_content_publish` and `pages_read_engagement` permissions)
3. Generate token for your Page
4. Grant `pages_read_engagement`, `instagram_content_publish`

**IG Business Account ID:**
- Found in Meta Business Suite → Instagram settings → Account ID

## Step-by-Step Usage

### Step 1: Generate Image

```python
from generate_course_promo import generate_course_promo

path = generate_course_promo(
    course_name="Diploma in Artificial Intelligence",
    institution="CADDESK Centre",
    duration="90 Weeks",
    bullets=[
        "Machine Learning & Data Science",
        "Building Intelligent Systems",
        "Algorithm Development",
        "Real-World AI Projects",
    ],
    hook_lines=["Your Future in AI", "Starts Here."],
    cta_text="Ready to shape the future with AI?",
    handle="@caddeskcentre",
    output_filename="course_promo.png"
)
```

CLI:
```bash
python scripts/generate_course_promo.py
```

### Step 2: Upload to Cloudinary

```bash
python scripts/upload_cloudinary.py <image_path> [folder]
```

Returns a public URL like `https://res.cloudinary.com/demo/image/upload/xyz.png`

### Step 3: Post to Instagram

```bash
python scripts/post_to_instagram.py <image_url> <caption>
```

Or programmatically:

```python
from post_to_instagram import post_to_instagram

success, post_id, ig_url = post_to_instagram(
    image_url="https://res.cloudinary.com/demo/image/upload/xyz.png",
    caption="Your caption with #hashtags"
)
```

## Caption Structure

```
[HOOK - bold claim or question, 1-2 lines]
[VALUE - what they'll learn/achieve]
[CTA - link in bio / DM to enroll / visit website]
[8-15 hashtags - mix of broad + niche]
```

## Troubleshooting

| Error | Fix |
|-------|-----|
| `401 Invalid OAuth` | Token expired — regenerate at Graph API Explorer |
| `IG token format error` | Use Page Access Token, not IG-only token |
| `image_url required` | Image not publicly accessible — upload to Cloudinary first |
| `403 Forbidden` | App not in Live mode or `instagram_content_publish` permission not approved |
| `Cloudinary 400` | Image too large (>10MB), unsupported format, or credentials not configured |
| `IG account not found` | Account not set as Business/Creator mode in Meta |
| Scrape returns None | Website uses JS rendering — provide details manually |

## Multiple Accounts

To switch between IG accounts, update environment variables before each post:

```bash
export IG_ACCESS_TOKEN="token_for_account_a"
export IG_BUSINESS_ACCOUNT_ID="ig_id_account_a"
python scripts/post_to_instagram.py <url> <caption>
```
