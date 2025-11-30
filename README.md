# KlustreFS Documentation Site

This repository contains the Hugo + Docsy site for the KlustreFS CSI plugin. The live site explains how to install the CSI driver, mount existing Lustre shares from Kubernetes, and understand the project’s roadmap.

## Requirements

- [Hugo Extended](https://gohugo.io/getting-started/installing/) v0.146+
- Node.js 18+
- Go 1.20+ (required by Hugo modules)

Install npm dependencies if you plan to run the SCSS/PostCSS pipeline:

```bash
npm install
```

## Local development

```bash
hugo serve -D --disableFastRender
```

Open <http://localhost:1313> to preview changes. `-D` includes draft content and `--disableFastRender` ensures layout edits are picked up immediately.

You can also use Docker:

```bash
docker-compose up --build
```

## Production build

```bash
hugo --minify
```

Artifacts land in the `public/` directory. Deploy that directory to GitHub Pages or your preferred host.

## Repository layout

- `content/` – Markdown pages (docs, blog, about, etc.)
- `assets/scss/` – KlustreFS-specific Sass overrides
- `static/` – Images (feature icons, logos)
- `layouts/` – Custom Hugo templates/partials/shortcodes
- `config` – Hugo configuration (`hugo.yaml`, module workspace)

## Issues & support

Open issues and feature requests in <https://github.com/klustrefs/website/issues>.
