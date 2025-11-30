---
title: Contribution Guidelines
description: How to propose doc changes for the Klustre website.
weight: 100
---

{{% pageinfo %}}
These instructions target the [`klustrefs/website`](https://github.com/klustrefs/website) repository. Follow them whenever you update content, navigation, or styling.
{{% /pageinfo %}}

## Toolchain

- Static site generator: [Hugo](https://gohugo.io/) **extended** edition, v0.146.0 or newer.
- Theme: [Docsy](https://github.com/google/docsy).
- Package manager: npm (used for Docsy assets).
- Hosting: Netlify deploy previews triggered from pull requests.

## Contribution workflow

1. Fork `https://github.com/klustrefs/website`.
2. Create a feature branch (`docs/my-topic`).
3. Make your edits (Markdown lives under `content/en/...`).
4. Run `npm install` once, then `npm run dev` or `hugo server` to preview locally.
5. Commit with clear messages (`docs: add getting started guide`).
6. Open a pull request against `main`. If the work is in progress, prefix the title with `WIP`.
7. Ensure the Netlify preview (`deploy/netlify — Deploy preview ready!`) renders correctly before requesting review.

All PRs need review by a maintainer before merging. We follow the standard GitHub review process.

## Editing tips

- Prefer short paragraphs and actionable steps. Use ordered lists for sequences and fenced code blocks with language hints (` ```bash `).
- Keep front matter tidy: `title`, `description`, and `weight` control sidebar order.
- When adding a new page, update any index pages or navigation lists that should reference it (for example, the Introduction landing page).
- Avoid using `draft: true` in front matter; draft pages don’t deploy to previews. Instead keep work-in-progress content on a branch until it’s review-ready.

## Updating a page from the browser

Use the **Edit this page** link:

1. Click the link in the top right corner of any doc page.
2. GitHub opens the corresponding file in your fork (you may need to create/update your fork first).
3. Make the edit, describe the change, and open a pull request. Review the Netlify preview before merging.

## Local preview

```bash
git clone git@github.com:<your-username>/website.git
cd website
npm install
hugo server --bind 0.0.0.0 --buildDrafts=false
```

Visit `http://localhost:1313` to preview. Hugo watches for changes and automatically reloads the browser.

## Filing issues

If you notice an error but can’t fix it immediately, open an issue in [`klustrefs/website`](https://github.com/klustrefs/website/issues). Include:

- URL of the affected page.
- Description of the problem (typo, outdated instructions, missing section, etc.).
- Optional suggestion or screenshot.

## Style reminders

- Use present tense (“Run `kubectl get pods`”) and active voice.
- Prefer metric units and standard Kubernetes terminology.
- Provide prerequisites and cleanup steps for any workflow.
- When referencing code or file paths, wrap them in backticks (`kubectl`, `/etc/hosts`).

Need help? Ask in the [GitHub discussions](https://github.com/klustrefs/klustre-csi-plugin/discussions) or on the Klustre Slack workspace.
