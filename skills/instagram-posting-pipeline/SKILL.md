---
name: instagram-posting-pipeline
description: Generate promotional images and post them to Instagram Business accounts via Meta Graph API. Use when: (1) user asks to create or post an Instagram course promo, educational carousel, or promotional image, (2) user wants to automate Instagram posting for any business account, (3) user says "post to Instagram", "create course promo", "schedule Instagram post", or "automate Instagram". Handles: Pillow image generation (1080x1350), Cloudinary hosting for public image URLs, Meta Graph API two-step posting (container + publish), caption and hashtag generation.
---

# Instagram Posting Pipeline

End-to-end pipeline: Brief → Image generation → Host image publicly → Post to Instagram via Meta Graph API.

## Pipeline Flow

```
Brief (course name, hook, bullets, CTA)
    → generate_course_promo.py
    → upload_cloudinary.py
    → post_to_instagram.py
    → Instagram post URL
```

## Environment Setup

Before using, set these environment variables:

```bash
# Meta Graph API (required)
export IG_ACCESS_TOKEN="your_page_access_token"
export IG_BUSINESS_ACCOUNT_ID="your_ig_business_account_id"
export IG_DEFAULT_CAPTION="Your default caption"

# Cloudinary (optional - defaults work for demo)
export CLOUDINARY_CLOUD_NAME="demo"       # default: demo
export CLOUDINARY_UPLOAD_PRESET="unsigned" # default: unsigned
export CLOUDINARY_FOLDER="myfolder"        # default: caddeskcentre

# Image output (optional)
export IG_PIPELINE_OUTPUT_DIR="./output"
```

### Getting Credentials

**Access Token:**
1. Go to https://developers.facebook.com/tools/explorer/
2. Select your Facebook App (must have `instagram_content_publish` permission)
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
| `403 Forbidden` | App not in Live mode or permission not approved |
| `Cloudinary 400` | Image too large (>10MB) or unsupported format |

## Multiple Accounts

To switch between IG accounts, update environment variables before each post:

```bash
export IG_ACCESS_TOKEN="token_for_account_a"
export IG_BUSINESS_ACCOUNT_ID="ig_id_account_a"
python scripts/post_to_instagram.py <url> <caption>
```
