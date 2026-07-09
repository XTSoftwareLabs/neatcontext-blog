# NeatContext Blog

This repository hosts the official blog for [NeatContext](https://www.neatcontext.com), a product focused on making context easier to capture, organize, and use across knowledge work.

The blog is configured with Jekyll and GitHub Pages. Posts are written in markdown and published as web pages.

## Write a Post

Create a markdown file in `_posts`:

```text
_posts/YYYY-MM-DD-your-post-title.md
```

Use this front matter:

```yaml
---
layout: post
title: "Your Post Title"
date: 2026-07-09
categories: updates
---
```

Then write the body in markdown.

## Publish

1. Commit and push changes to the `main` branch.
2. In GitHub, open **Settings > Pages**.
3. Set **Source** to **GitHub Actions**.
4. The included workflow builds and deploys the site.

The site URL is expected to be:

```text
https://xtsoftwarelabs.github.io/neatcontext-blog/
```

## Local Preview

Install Ruby and Bundler, then run:

```bash
bundle install
bundle exec jekyll serve
```

Open `http://127.0.0.1:4000/neatcontext-blog/`.
