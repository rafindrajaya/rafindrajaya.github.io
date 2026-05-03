# Portfolio Repository Architecture

This is a Jekyll-based portfolio site for Rafi Indrajaya, hosted on GitHub Pages.

---

## Key Rule: Where to Make Changes

**Only ever touch these two areas for content changes:**
- `_config.yml` — for profile info, skills, links, resume, colors
- `_projects/` — for adding or editing project entries

Do NOT modify `_site/` — that is the auto-generated build output. Do NOT touch `_layouts/`, `_includes/`, or `assets/js/` unless explicitly asked to change the frontend UI.

---

## Repository Structure

```
/
├── _config.yml              ← Main site configuration (profile, skills, links, colors)
├── _projects/               ← One folder per project, each with index.md + images
│   ├── <project-folder>/
│   │   ├── index.md         ← Project content (front matter + markdown body)
│   │   └── *.png / *.jpg    ← Project images referenced in index.md
├── assets/
│   ├── images/
│   │   └── profile-image/   ← Profile photo goes here
│   ├── Rafi_Indrajaya_CV.pdf ← Current resume/CV file
│   └── js/
│       └── script.js
├── _includes/               ← Reusable HTML partials (navbar, hero, footer, etc.)
├── _layouts/                ← Jekyll page layout templates
├── _site/                   ← AUTO-GENERATED build output. Never edit directly.
├── Gemfile                  ← Ruby gem dependencies
└── README.md                ← Template usage guide
```

---

## _config.yml — What Each Field Controls

| Field | What it does |
|---|---|
| `title` | Browser tab title |
| `name` | Your name shown on the hero section |
| `headline` | Subtitle under your name |
| `description` | Bio paragraph (supports `<br>` for line breaks) |
| `profile_image` | Path to profile photo (relative to repo root) |
| `resume-url` | Path or external URL to CV/resume PDF |
| `external-links` | Social icons (linkedin, github, etc.) — only filled ones appear |
| `skills` | Skills section — list of categories, each with a list of tools |
| `formspree-key` | Contact form endpoint |
| `colors` | Theme colors (text, border, link, highlight, background, light_background) |

---

## Changing the Profile Picture

1. Place the new image in `assets/images/profile-image/`
2. Update `profile_image:` in `_config.yml` to the new filename path
   ```yaml
   profile_image: assets/images/profile-image/your-new-photo.jpg
   ```

Current profile image: `assets/images/profile-image/DSC08446-large.jpeg`

---

## Updating the Resume / CV

1. Place the new PDF in `assets/` (root of assets folder)
2. Update `resume-url:` in `_config.yml`
   ```yaml
   resume-url: /assets/Your_New_CV.pdf
   ```
   Or link to an external URL (e.g., Google Drive PDF link).

Current resume file: `assets/Rafi_Indrajaya_CV_0526.pdf`

---

## Changing the Color Theme

Edit the `colors:` block in `_config.yml`. Available preset themes are commented out in the file:
- **basic** (current): white background, blue links
- **mint**: teal/green tones
- **lemon**: yellow tones
- **pink**: rose/pink tones

---

## Updating Skills

Edit the `skills:` block in `_config.yml`. Each entry needs a `category` and a list of `tools`. Indentation must be exact.

```yaml
skills:
  - category: Category Name
    tools:
    - Tool 1
    - Tool 2
```

---

## Updating External Links

Edit `external-links:` in `_config.yml`. Only provide the URL slug (not the full URL). Icons for missing entries will not appear.

```yaml
external-links:
  linkedin: your-slug
  github: your-slug
```

---

## Embedding Content in Project Pages (Markdown Syntax)

### Images
```markdown
{% include image-gallery.html images="/image.png" height="400" %}
{% include image-gallery.html images="/img1.png, /img2.png" height="400" %}
```

### YouTube Video
```markdown
{% include youtube-video.html id="11-digit-id" autoplay="false" %}
```

### Inline image (alternative)
```markdown
![](/_projects/folder-name/image.png){:height="400px"}
```

### Tables — always leave a blank line before the table header
```markdown
| Col 1 | Col 2 |
|-------|-------|
| val   | val   |
```

---

## Light Frontend UI Modifications

If asked to make minor UI changes (colors, fonts, layout tweaks), the relevant files are:
- `_config.yml` — for theme colors
- `_includes/` — for structural HTML changes (navbar, hero, footer, etc.)
- `assets/css/` (if a custom CSS file exists) — for style overrides

Always confirm with the user before touching `_includes/` or layout files.
