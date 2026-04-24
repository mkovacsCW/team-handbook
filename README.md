# Dark Navy Theme Update

Drop these files into your existing `engineering-handbook` repo to switch to a dark navy theme with your company logo.

## What's in here

```
theme-update/
├── mkdocs.yml                   → replaces your existing mkdocs.yml
└── docs/
    └── assets/
        └── extra.css            → new file, custom CSS for navy colors
```

## How to apply

### 1. Add your logo

Drop your company logo into `docs/assets/logo.png`. A PNG with a transparent background works best. SVG also works — if you use SVG, change the filename in `mkdocs.yml` accordingly.

**Recommended dimensions:** Any height works, but the header is about 40px tall, so the logo will be rendered around 24-30px tall. Start with something like 200×60 or 300×90.

**Two ways to add it:**

- Drag your logo file into `docs/assets/` (rename it to `logo.png`)
- Or commit it via git: `git add docs/assets/logo.png`

### 2. Replace mkdocs.yml

Copy `mkdocs.yml` from this folder over your existing one. The changes are:

- Added `logo: assets/logo.png` and `favicon: assets/logo.png` under `theme:`
- Reordered the palette so **dark mode is the default** (the first entry is dark navy)
- Added `extra_css:` section pointing to the new CSS file

### 3. Add the CSS file

Copy `docs/assets/extra.css` into your repo at the same path. This file overrides Material's default "slate" (charcoal gray) with actual dark navy blue colors.

### 4. Commit and push

```bash
git add docs/assets/logo.png docs/assets/extra.css mkdocs.yml
git commit -m "Switch to dark navy theme with company logo"
git push
```

GitHub Actions auto-deploys. Live in ~1 minute.

## Preview locally first

```bash
mkdocs serve
```

Open http://127.0.0.1:8000 to see it before pushing.

## Customization

If you want to tweak the exact shade of navy, edit `docs/assets/extra.css`. The key values are:

```css
--md-default-bg-color: hsl(220, 40%, 10%);    /* main background - darker = lower % */
--md-primary-fg-color: hsl(220, 45%, 15%);    /* header color */
--md-accent-fg-color: hsl(210, 90%, 65%);     /* links and highlights */
```

HSL format is `hsl(hue, saturation, lightness)`. For navy, keep hue around 210-225. To go darker, lower the lightness %. To go more blue-saturated, raise the saturation %.

### Want it always dark (no light mode toggle)?

Remove the second `media:` block under `palette:` in `mkdocs.yml` and remove the `toggle:` section from the first one. The sun/moon icon in the header will disappear.

### Want to change the light mode colors too?

The light mode is still Material's default — white background, indigo accents. If you want it navy-tinted in light mode too, you can add variables under `[data-md-color-scheme="default"]` in `extra.css`.
