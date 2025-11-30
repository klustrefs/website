# Contributing to the KlustreFS docs

Thanks for helping improve the KlustreFS documentation site.

This repository hosts the Hugo + Docsy sources for <https://klustrefs.io>. If you want to
change the CSI plugin code itself, head to <https://github.com/klustrefs/klustre-csi-plugin>.

For detailed instructions (tooling, local preview, style), see the in-site docs page:

- `content/en/docs/contribution-guidelines.md` → <https://klustrefs.io/docs/contribution-guidelines/>

## Basic workflow

1. Fork <https://github.com/klustrefs/website>.
2. Create a feature branch, for example `docs/my-topic`.
3. Edit the Markdown under `content/en/...` or templates under `layouts/...`.
4. Run `npm install` once, then `npm run dev` or `hugo server` to preview locally.
5. Commit with clear messages (for example, `docs: update overview page`).
6. Open a pull request against `main`. Netlify will attach a preview URL to the PR.

All changes are reviewed via GitHub pull requests before merging.

## Reporting issues

If you spot a problem but can’t fix it immediately, open an issue in
<https://github.com/klustrefs/website/issues> with:

- The URL of the affected page.
- A short description of what’s wrong (typo, outdated content, missing section, etc.).
- Optional suggestion, log output, or screenshot.

Discussion about the CSI plugin itself happens in
<https://github.com/klustrefs/klustre-csi-plugin/discussions>.
